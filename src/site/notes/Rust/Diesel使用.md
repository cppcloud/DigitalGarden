---
{"dg-publish":true,"tags":["rust"],"permalink":"/Rust/Diesel使用/","dgPassFrontmatter":true}
---

## 安装 Diesel

[Getting Started with Diesel](https://diesel.rs/guides/getting-started)

[Diesel install on Windows with support for SQLite, MySQL and Postgresql | diesel\_windows\_install\_doc](https://duredhelfinceleb.github.io/diesel_windows_install_doc/)

参考上面 2 个链接，建议使用 vcpkg+scoop 来安装依赖，注意事项：使用配置 mysql 的时候，需要库目录和 mysql 版本号

## 简化安装流程

注意：vcpkg 安装的 mysql 的库存在问题，故而选择下载官方的，下载下方链接的 zip 包，重命名并解压到合适的位置。

[MySQL :: Download MySQL Community Server (Archived Versions)](https://downloads.mysql.com/archives/community/)

**添加环境变量**

根据时间不同，下载的 mysql 的版本不同，目前是 8.4.0 版本

![Pasted image 20240724122309.png](/img/user/Rust/assert/Pasted%20image%2020240724122309.png)

```bash
MYSQLCLIENT_LIB_DIR="D:\Dev\mysql\lib"  # 替换为实际的路径
MYSQLCLIENT_VERSION="8.4.0"  # 替换为您的实际 mysql 版本号
```

![Pasted image 20240724122403.png](/img/user/Rust/assert/Pasted%20image%2020240724122403.png)

### **重新安装 diesel**

分别安装 mysql, sqlite, postgre

```bash
cargo install diesel_cli --no-default-features --features sqlite-bundled
cargo install diesel_cli --no-default-features --features mysql
cargo install diesel_cli --no-default-features --features postgres
```
# Diesel 使用 mysql

#### 安装 cargo generate

```shell
cargo install cargo-generate --locked
```
#### 根据模板创建项目

```shell
cargo generate --git https://github.com/cppcloud/rust-project-templete.git
```

#### 添加依赖

```toml
[dependencies]

diesel = { version = "2.2.0", features = ["mysql"] }

dotenvy = "0.15"
```

#### 创建并生成migrations

```bash
mkdir migrations
diesel migration generate create_products
```

上面的命令会在 migration 目录下面创建两个名为 `up.sql` 和 `down.sql` 的文件。

添加 sql 语句，在 `up.sql` 中添加如下的语句

```sql
CREATE TABLE products (

  id INT PRIMARY KEY,

  name VARCHAR(255) NOT NULL,

  cost DOUBLE NOT NULL,

  active BOOLEAN NOT NULL DEFAULT 0 

);
```

在终端中运行以下命令来运行 `migration` 来创建 `products` 表

```bash
diesel migration run
```

在 `src` 文件夹中为模型添加一个名为 `models.rs` 的文件，其中包含 `products` 模型。
