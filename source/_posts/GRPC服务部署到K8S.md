---
title: GRPC服务部署到K8S
date: 2023-11-13 15:11:03
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

1、先购买[集群](https://console.cloud.tencent.com/tke2/cluster?rid=1)，后创建命名空间 test_app_namespace


2、后创建镜像仓库 my_repo


3、本地登录腾讯云容器镜像服务 Docker Registry
```bash
docker login ccr.ccs.tencentyun.com --username=
```


4、配置好Dockerfile，打包镜像
```bash
docker build -t test_app_namespace/gin-service:1.0 -f ./Dockerfile .
```

5、docker本地试跑
```bash
docker run -d --name gin-service -p 8080:8080 test_app_namespace/gin-service:1.0
```
或
```bash
docker run -d --name gin-service --net host test_app_namespace/gin-service:1.0
```

6、向 Registry 中推送镜像

当前 imageId = test_app_namespace/gin-service:1.0
```bash
docker tag [imageId] ccr.ccs.tencentyun.com/test_app_namespace/my_repo:[tag]

docker push ccr.ccs.tencentyun.com/test_app_namespace/my_repo:[tag]
```