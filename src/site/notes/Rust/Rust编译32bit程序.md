---
{"dg-publish":true,"tags":["rust"],"permalink":"/Rust/Rust编译32bit程序/","dgPassFrontmatter":true}
---

### 查看安装的编译工具链

```bash
rustup show
```

### 查看所有的编译工具链

```bash
rustup target list
```

会得到类似这样的列表

```bash
C:\Users\duang\Desktop>rustup target list
aarch64-apple-darwin
aarch64-apple-ios
aarch64-apple-ios-sim
aarch64-linux-android
aarch64-pc-windows-gnullvm
aarch64-pc-windows-msvc
aarch64-unknown-fuchsia
aarch64-unknown-linux-gnu
aarch64-unknown-linux-musl
aarch64-unknown-linux-ohos
aarch64-unknown-none
aarch64-unknown-none-softfloat
aarch64-unknown-uefi
arm-linux-androideabi
```

### 安装 msvc 32 bit 的编译工具链

记得要加 stable 才可以安装

```bash
rustup install stable-i686-pc-windows-msvc
```

切换 32 bit 的工具链

```bash
rustup default stable-i686-pc-windows-msvc
```