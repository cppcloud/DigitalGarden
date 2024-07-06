---
{"dg-publish":true,"permalink":"/tools/docker-proxy/","tags":["tools"]}
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