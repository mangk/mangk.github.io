---
title: docker swarm集群的使用
comments: true
banner_img_height: 30
date: 2022-09-30 12:48:35
categories:
- Docker
tags:
---
## 序
咱们书接[上回](/2022/09/27/使用alpine构建docker镜像/)，我搞定了golang程序的编译，搞定了docker镜像的打包，搞定了流水线。现在项目部署与热更新成了我要面对的问题。
受制于公司提供的部署环境（只提供给我一台主机，主机上有docker），我暂时没有k8s集群可用，但又希望能完成基于流水线的自动部署、热更新和不停机更新。所以返现了下面要说的`docker swarm`

## 什么是 swarm mode
Swarm 是使用 SwarmKit 构建的 Docker 引擎内置（原生）的集群管理和编排工具。具体内容可以参见：[基本概念](https://yeasy.gitbook.io/docker_practice/swarm_mode/overview)，这里不再详细解释。

## 集群创建
``` shell
docker swarm init
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
使用`docker swarm init`命令，创建一个集群，本机的docker就会变成一个单节点的集群。