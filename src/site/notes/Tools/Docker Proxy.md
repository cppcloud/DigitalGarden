---
{"dg-publish":true,"permalink":"/Tools/Docker Proxy/","tags":["tools"],"dgPassFrontmatter":true,"noteIcon":"","created":"2024-07-07T01:56:59.928+08:00","updated":"2024-07-07T02:03:50.774+08:00"}
---


## 配置docker代理

- 编辑daemon.json文件

```bash
vim /etc/docker/daemon.json
```

- 配置registry-mirrors

```bash
{
  "registry-mirrors": [
    "https://docker.z23.cc"
  ]
}
```

- 重启docker,不同平台命令可能不一样

```bash

systemctl restart docker

```