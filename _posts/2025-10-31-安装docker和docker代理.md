---
title: 安装docker和docker代理
description: 
date: 2025-10-31 11:26:17 +0800
categories: []
tags: [docker]
pin: false
math: false
mermaid: false
---
## docker代理工具
开源地址：[https://github.com/cmliu/CF-Workers-docker.io](https://github.com/cmliu/CF-Workers-docker.io)
原：`docker pull stilleshan/frpc:latest`
改：`docker pull docker.fxxk.dedyn.io/stilleshan/frpc:latest`

另外的docker镜像地址

地址①:[https://dockerpull.com/](https://dockerpull.com/)
地址②:[https://dockerproxy.cn/](https://dockerproxy.cn/)  


## 安装docker：
访问 [https://get.docker.com/](https://get.docker.com/) 保存成 `install-docker.sh`

安装1（使用阿里云源）推荐
`bash install-docker.sh --mirror Aliyun`
安装2（使用亚马逊源）
`bash install-docker.sh --mirror AzureChinaCloud`

## 查看docker版本和信息
`docker version`  

`docker info`