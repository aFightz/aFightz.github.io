---
title: 08 | 发布到阿里云
date: 2019-07-09 11:35:40
tags: [Docker]
categories :
- 学习笔记
- Docker
- Docker核心技术
---

**1、登录**
```
sudo docker login --username=[阿里账号] registry.cn-hangzhou.aliyuncs.com
```

**2、命名**
```
sudo docker tag [IMAGE_ID] [REPOSITORY]:[TAG]
```

**3、发布**
```
sudo docker push [REPOSITORY]:[TAG]
```
















