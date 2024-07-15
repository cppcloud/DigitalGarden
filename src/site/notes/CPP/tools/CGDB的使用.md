---
{"dg-publish":true,"tags":["tools"],"permalink":"/CPP/tools/CGDB的使用/","dgPassFrontmatter":true}
---

### 什么是 cgdb

cgdb 是 GNU 调试器 (GDB) 的轻量级curses（基于终端）接口。除了标准的 gdb 控制台之外，cgdb 还提供了一个分屏视图，可以在执行时显示源代码。

![Pasted image 20240715164350.png](/img/user/CPP/tools/assert/Pasted%20image%2020240715164350.png)


## 源码构建

```bash
git clone git://github.com/cgdb/cgdb.git 
cd cgdb 
./autogen.sh
```

## 安装

cgdb 依赖于 libreadline 和 ncurses 开发库。

```bash
./configure --prefix=/usr/local 
make 
sudo make install
```
