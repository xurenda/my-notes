---
date: 2023-11-07 11:35
modified: 2024-01-03 10:54
---

# 拉取 github 最新代码
如报错：
```bash
fatal: Need to specify how to reconcile divergent branches.
```
运行：
```bash
git config pull.rebase false
```

# 构建 docker 镜像
项目根目录下执行 ：
```bash
docker build -t my/fastgpt --build-arg name=app .
```

# docker -compose 快速启动
复制 `files/deploy/my/docker-compose.yml` 到新目录，并修改 `远程配置请求地址` 环境变量。

在新目录下执行：
```bash
docker-compose pull
docker-compose up -d
```

项目访问路径：`http://localhost:3000/`