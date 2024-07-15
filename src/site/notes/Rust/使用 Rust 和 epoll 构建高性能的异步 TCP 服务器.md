---
{"dg-publish":true,"tags":["rust"],"permalink":"/Rust/使用 Rust 和 epoll 构建高性能的异步 TCP 服务器/","dgPassFrontmatter":true}
---

### 源码链接

[rust-epoll-example/src/main.rs at main · zupzup/rust-epoll-example · GitHub](https://github.com/zupzup/rust-epoll-example/blob/main/src/main.rs)

### 依赖项

```toml
[dependencies]
libc = "0.2"
```

首先引入 Rust 的标准库中的几个关键模块，将支持网络通信、I/O 操作和底层操作系统交互：

```rust
use std::collections::HashMap;
use std::io;
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::os::unix::io::{AsRawFd, RawFd};
```

#### 简化系统调用处理的宏

简化错误处理，我们定义了一个名为 `syscall!` 的宏，该宏自动处理错误并返回相应的结果：
这个宏没有太多内容，它只是在 `unsafe` 块中调用给定的 libc 系统调用，如果它返回错误 (-1)，则该错误将包装在 Result 中，否则返回值已返回。

```rust
#[allow(unused_macros)]
macro_rules! syscall {
    ($fn: ident ( $($arg: expr),* $(,)* ) ) => {{
        let res = unsafe { libc::$fn($($arg, )*) };
        if res == -1 {
            Err(std::io::Error::last_os_error())
        } else {
            Ok(res)
        }
    }};
}

```

## 创建 TcpListener 并设置为非阻塞模式

```rust
fn main() -> io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8000")?;
    listener.set_nonblocking(true)?;
    let listener_fd = listener.as_raw_fd();
    ...
    Ok(())
}
```

### epoll 事件

```rust
fn main() -> io::Result<()> {
    ...
    let epoll_fd = epoll_create().expect("can create epoll queue");
    ...
}
fn epoll_create() -> io::Result<RawFd> {
    let fd = syscall!(epoll_create1(0))?;
    if let Ok(flags) = syscall!(fcntl(fd, libc::F_GETFD)) {
        let _ = syscall!(fcntl(fd, libc::F_SETFD, flags | libc::FD_CLOEXEC));
    }
    Ok(fd)
}
```

`add_interest` 函数获取 `epoll` 队列的文件描述符、我们想要添加兴趣的任何内容的文件描述符以及定义哪种类型的 `epoll_event` 我们希望收到通知的事件。

```rust
const READ_FLAGS: i32 = libc::EPOLLONESHOT | libc::EPOLLIN;
const WRITE_FLAGS: i32 = libc::EPOLLONESHOT | libc::EPOLLOUT;
fn main() -> io::Result<()> {
    ...
    let mut key = 100;
    add_interest(epoll_fd, listener_fd, listener_read_event(key))?;
    ...
}
fn add_interest(epoll_fd: RawFd, fd: RawFd, mut event: libc::epoll_event) -> io::Result<()> {
    syscall!(epoll_ctl(epoll_fd, libc::EPOLL_CTL_ADD, fd, &mut event))?;
    Ok(())
}
fn listener_read_event(key: u64) -> libc::epoll_event {
    libc::epoll_event {
        events: READ_FLAGS as u32,
        u64: key,
    }
}
```

### 事件循环

```rust
fn main() -> io::Result<()> {
    ...
    let mut events: Vec<libc::epoll_event> = Vec::with_capacity(1024);
    ...
    loop {
        events.clear();
        let res = match syscall!(epoll_wait(
            epoll_fd,
            events.as_mut_ptr() as *mut libc::epoll_event,
            1024,
            1000 as libc::c_int,
        )) {
            Ok(v) => v,
            Err(e) => panic!("error during epoll wait: {}", e),
        };
        // safe  as long as the kernel does nothing wrong - copied from mio
        unsafe { events.set_len(res as usize) };
        ...
    }
}
```

一旦调用， `epoll_wait` 将阻塞，直到发生以下三种情况之一：

+ 我们注册感兴趣的文件描述符有一个事件
+ 信号处理程序中断调用
+ 超时时间已到

我们可以将超时设置为 `-1` ，这将使 `epoll_wait` 无限阻塞，或者直到其他两件事发生为止。


#### 请求上下文结构体

服务器中的每个连接都通过一个名为 `RequestContext` 的结构体来管理。这个结构体包含 TCP 流、内容长度和数据缓冲区：

```rust
#[derive(Debug)]
pub struct RequestContext {
    pub stream: TcpStream,
    pub content_length: usize,
    pub buf: Vec<u8>,
}
impl RequestContext {
    fn new(stream: TcpStream) -> Self {
        Self {
            stream,
            buf: Vec::new(),
            content_length: 0,
        }
    }
}
fn main() -> io::Result<()> {
    ...
    let mut request_contexts: HashMap<u64, RequestContext> = HashMap::new();
    ...
        println!("requests in flight: {}", request_contexts.len());
        for ev in &events {
            match ev.u64 {
                100 => {
                    match listener.accept() {
                        Ok((stream, addr)) => {
                            stream.set_nonblocking(true)?;
                            println!("new client: {}", addr);
                            key += 1;
                            add_interest(epoll_fd, stream.as_raw_fd(), listener_read_event(key))?;
                            request_contexts.insert(key, RequestContext::new(stream));
                        }
                        Err(e) => eprintln!("couldn't accept: {}", e),
                    };
                    modify_interest(epoll_fd, listener_fd, listener_read_event(100))?;
                }
                ...
}
fn modify_interest(epoll_fd: RawFd, fd: RawFd, mut event: libc::epoll_event) -> io::Result<()> {
    syscall!(epoll_ctl(epoll_fd, libc::EPOLL_CTL_MOD, fd, &mut event))?;
    Ok(())
}
```

## 完整代码

```rust
use std::collections::HashMap;
use std::io;
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::os::unix::io::{AsRawFd, RawFd};

#[allow(unused_macros)]
macro_rules! syscall {
    ($fn: ident ( $($arg: expr),* $(,)* ) ) => {{
        let res = unsafe { libc::$fn($($arg, )*) };
        if res == -1 {
            Err(std::io::Error::last_os_error())
        } else {
            Ok(res)
        }
    }};
}

#[derive(Debug)]
pub struct RequestContext {
    pub stream: TcpStream,
    pub content_length: usize,
    pub buf: Vec<u8>,
}

impl RequestContext {
    fn new(stream: TcpStream) -> Self {
        Self {
            stream,
            buf: Vec::new(),
            content_length: 0,
        }
    }

    fn read_cb(&mut self, key: u64, epoll_fd: RawFd) -> io::Result<()> {
        let mut buf = [0u8; 4096];
        match self.stream.read(&mut buf) {
            Ok(_) => {
                if let Ok(data) = std::str::from_utf8(&buf) {
                    self.parse_and_set_content_length(data);
                }
            }
            Err(e) if e.kind() == io::ErrorKind::WouldBlock => {}
            Err(e) => {
                return Err(e);
            }
        };
        self.buf.extend_from_slice(&buf);
        if self.buf.len() >= self.content_length {
            println!("got all data: {} bytes", self.buf.len());
            modify_interest(epoll_fd, self.stream.as_raw_fd(), listener_write_event(key))?;
        } else {
            modify_interest(epoll_fd, self.stream.as_raw_fd(), listener_read_event(key))?;
        }
        Ok(())
    }

    fn parse_and_set_content_length(&mut self, data: &str) {
        if data.contains("HTTP") {
            if let Some(content_length) = data
                .lines()
                .find(|l| l.to_lowercase().starts_with("content-length: "))
            {
                if let Some(len) = content_length
                    .to_lowercase()
                    .strip_prefix("content-length: ")
                {
                    self.content_length = len.parse::<usize>().expect("content-length is valid");
                    println!("set content length: {} bytes", self.content_length);
                }
            }
        }
    }

    fn write_cb(&mut self, key: u64, epoll_fd: RawFd) -> io::Result<()> {
        match self.stream.write(HTTP_RESP) {
            Ok(_) => println!("answered from request {}", key),
            Err(e) => eprintln!("could not answer to request {}, {}", key, e),
        };
        self.stream.shutdown(std::net::Shutdown::Both)?;
        let fd = self.stream.as_raw_fd();
        remove_interest(epoll_fd, fd)?;
        close(fd);
        Ok(())
    }
}

const READ_FLAGS: i32 = libc::EPOLLONESHOT | libc::EPOLLIN;
const WRITE_FLAGS: i32 = libc::EPOLLONESHOT | libc::EPOLLOUT;

const HTTP_RESP: &[u8] = br#"HTTP/1.1 200 OK
content-type: text/html
content-length: 5

Hello"#;

fn main() -> io::Result<()> {
    let mut request_contexts: HashMap<u64, RequestContext> = HashMap::new();
    let mut events: Vec<libc::epoll_event> = Vec::with_capacity(1024);
    let mut key = 100;

    let listener = TcpListener::bind("127.0.0.1:8000")?;
    listener.set_nonblocking(true)?;
    let listener_fd = listener.as_raw_fd();

    let epoll_fd = epoll_create().expect("can create epoll queue");
    add_interest(epoll_fd, listener_fd, listener_read_event(key))?;

    loop {
        println!("requests in flight: {}", request_contexts.len());
        events.clear();
        let res = match syscall!(epoll_wait(
            epoll_fd,
            events.as_mut_ptr() as *mut libc::epoll_event,
            1024,
            1000 as libc::c_int,
        )) {
            Ok(v) => v,
            Err(e) => panic!("error during epoll wait: {}", e),
        };

        // safe  as long as the kernel does nothing wrong - copied from mio
        unsafe { events.set_len(res as usize) };

        for ev in &events {
            match ev.u64 {
                100 => {
                    match listener.accept() {
                        Ok((stream, addr)) => {
                            stream.set_nonblocking(true)?;
                            println!("new client: {}", addr);
                            key += 1;
                            add_interest(epoll_fd, stream.as_raw_fd(), listener_read_event(key))?;
                            request_contexts.insert(key, RequestContext::new(stream));
                        }
                        Err(e) => eprintln!("couldn't accept: {}", e),
                    };
                    modify_interest(epoll_fd, listener_fd, listener_read_event(100))?;
                }
                key => {
                    let mut to_delete = None;
                    if let Some(context) = request_contexts.get_mut(&key) {
                        let events: u32 = ev.events;
                        match events {
                            v if v as i32 & libc::EPOLLIN == libc::EPOLLIN => {
                                context.read_cb(key, epoll_fd)?;
                            }
                            v if v as i32 & libc::EPOLLOUT == libc::EPOLLOUT => {
                                context.write_cb(key, epoll_fd)?;
                                to_delete = Some(key);
                            }
                            v => println!("unexpected events: {}", v),
                        };
                    }
                    if let Some(key) = to_delete {
                        request_contexts.remove(&key);
                    }
                }
            }
        }
    }
}

fn epoll_create() -> io::Result<RawFd> {
    let fd = syscall!(epoll_create1(0))?;
    if let Ok(flags) = syscall!(fcntl(fd, libc::F_GETFD)) {
        let _ = syscall!(fcntl(fd, libc::F_SETFD, flags | libc::FD_CLOEXEC));
    }

    Ok(fd)
}

fn listener_read_event(key: u64) -> libc::epoll_event {
    libc::epoll_event {
        events: READ_FLAGS as u32,
        u64: key,
    }
}

fn listener_write_event(key: u64) -> libc::epoll_event {
    libc::epoll_event {
        events: WRITE_FLAGS as u32,
        u64: key,
    }
}

fn close(fd: RawFd) {
    let _ = syscall!(close(fd));
}

fn add_interest(epoll_fd: RawFd, fd: RawFd, mut event: libc::epoll_event) -> io::Result<()> {
    syscall!(epoll_ctl(epoll_fd, libc::EPOLL_CTL_ADD, fd, &mut event))?;
    Ok(())
}

fn modify_interest(epoll_fd: RawFd, fd: RawFd, mut event: libc::epoll_event) -> io::Result<()> {
    syscall!(epoll_ctl(epoll_fd, libc::EPOLL_CTL_MOD, fd, &mut event))?;
    Ok(())
}

fn remove_interest(epoll_fd: RawFd, fd: RawFd) -> io::Result<()> {
    syscall!(epoll_ctl(
        epoll_fd,
        libc::EPOLL_CTL_DEL,
        fd,
        std::ptr::null_mut()
    ))?;
    Ok(())
}
```