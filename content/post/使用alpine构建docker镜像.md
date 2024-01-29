---
title: 使用alpine构建Docker镜像
comments: true
date: 2022-09-27 11:21:51
banner_img_height: 30
categories:
- Docker
tags:
- Golang
---

## 序
最近的工作中，我使用Golang开发了一个web项目，并准备通过流水线的方式做持续集成与发布。在这是用过程中遇到了一些技术与问题包括：[通过make构建golang程序](#通过Makefile打包Golang程序)、[dockerfile不能访问父级目录](#Dockerfile不能访问父级目录)、[使用Alpine作为基础来制作我的容器镜像](#使用Alpine作为基础来制作我的容器镜像)

# 通过Makefile打包Golang程序
## 什么是Makefile
>代码变成可执行文件，叫做编译（compile）；先编译这个，还是先编译那个（即编译的安排），叫做构建（build）。Make是最常用的构建工具，诞生于1977年，主要用于C语言的项目。但是实际上，任何只要某个文件有变化，就要重新构建的项目，都可以用Make构建。
>
>关于make命令这里不做深入介绍，引用链接提供介绍清晰明了的直通车（让我们感谢大神🙏）
> [阮一峰的网络日志-Make命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)
## Golang项目使用Makefile
我对makefile文件的理解是它有点像时一个命令的集合，通过一个个target<目标>可以按照顺序快速的执行多条命令而不用再手动一条条输入。if else与变量的加入让这个流程变得更加丰富和多变。省去了命令输入的同时也让代码运行的流程更加清楚。
### 示例代码
``` makefile
.PHONY: build clean run

# golang 打包可执行文件的运行环境 (可以在运行make命令时传入：make OS=windows)
OS=linux
BUILD_DOCKER_IMAGE=0

all: build

run:
    go run ./cmd/main.go

build:
    # 下面的GOOS=$(OS)使用了上面定义的OS变量
    @GO111MODULE=on GOPROXY=https://goproxy.cn,direct CGO_ENABLED=0 go mod tidy && GOOS=$(OS) go build -o app main.go
# if判断中同样可以使用变量
ifeq ($(BUILD_DOCKER_IMAGE),1)
    @cd deploy && docker build -t .
endif

clean:
    # 命令前加@会让命令执行但不在控制台输出，这里没有@命令执行时会输出到终端
    rm app
ifeq ($(BUILD_DOCKER_IMAGE),1)
    @docker rmi recommend_photo
endif
```

# Dockerfile不能访问父级目录
项目汇中我使用Dockerfile作为构架docker镜像的基础。通过`docker build -t image_name .`可以将内容打包成本地的docker镜像。但是鉴于我现有的项目目录结构使得`COPY`无法找到对应的文件，在查阅`docker build`命令后，发现可以通过`-f`选项来执行相关目录。
``` shell
我的目录结构
.
├── README.md
├── cmd
│   └── main.go
├── conf
│   ├── ...
├── deploy
│   └── Dockerfile
├── http
│   ├── ...
├── model
│   ├── ...
├── makefile
├── go.mod
├── go.sum
├── config.yaml
├── sample.config.yaml
```
## 解决Dockerbuild目录可见性问题（使用 -f 选项）
docker与dockerfile的关系这里不再阐述。这只单独介绍docker build`-f`选项的使用
``` shell
➜  project git:(dev) docker build -h   
Flag shorthand -h has been deprecated, please use --help

Usage:  docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
  ...
```
平时使用docker build一般是在根目录存在Dockerfile，构建的时候Dockerfile会在当前目录寻找需要的文件，但是在我的目录结构中，Dockerfile被放到了deploy目录下，导致我在根目录执行docker build会找不到file，但是cd到deploy目录里又会导致Dockerfile找不到根目录的文件。所以这里`-f`选项就可以使用了：
``` shell
  docker build -t name:tag  -f deploy/Dockerfile ./deploy
```
`-f`选项指定了Dockerfile所在的目录，最后的`./deploy`是docker构建使用的根目录

## 向Dockerfile中传递编译参数
通过`-build-arg {ARG_NAME}={value}`可以传递参数，那么在dockerfile中，通过`arg` 定义的变量可以接收到传递到参数`ARG ARG_NAME=hello`

# 使用Alpine作为基础来制作我的容器镜像
## Docker打包出的镜像超过1G让我不能接受
在项目刚写完的时候，我使用golang官网的docker镜像作为基础来构建我自己的镜像。这样做可以不用担心环境问题，打包golang构建docker一气呵成固然方便，但当我看到我的镜像有1.2G大的时候猛打了一个寒颤～这是在超出我的所料。
作为追求精简的程序员，这种情况不允许发生在我的项目里，于是便有了下面的内容
## Alpine镜像是什么
> Alpine 操作系统是一个面向安全的轻型 Linux 发行版。它不同于通常 Linux 发行版，Alpine 采用了 musl libc(注意这里，是个坑) 和 busybox 以减小系统的体积和运行时资源消耗，但功能上比 busybox 又完善的多，因此得到开源社区越来越多的青睐。在保持瘦身的同时，Alpine 还提供了自己的包管理工具 apk，可以通过 https://pkgs.alpinelinux.org/packages 网站上查询包信息，也可以直接通过 apk 命令直接查询和安装各种软件。
## 将golang编译与docker打包拆分
为了尽可能的减小docker镜像的体积，我采用了只将golang可执行文件与config文件打包进镜像的方案（大家可以制作自己的打包服务器或者直接使用各种CI/CD流水线来帮助构建镜像）。在这之后，我的小而美的docker镜像中只存在两个必要文件`app(可执行文件) & config.yaml`。而我的镜像也变成了精简的49MB
## 制作镜像时遇到的问题
可是当我使用镜像构建容器的时候，却总是提示
``` shell
➜  project git:(dev) docker run -it --rm 351b4d544b4e
exec /app/app: no such file or directory
```
开始很没头绪，因为使用了阿里云的流水线服务所以我还特意发了工单去询问（这显得我很白痴）。后来我通过`docker save imageID > filename.tar`的方式将镜像下载到本地，然后解压去查找对应的文件，发现`/app/app`可执行文件就好好的躺在它该在的目录。这时候我才把目光转向了我的基础镜像。于是发现并解决了如下问题(部分内容引用自[使用Alpine作为基础镜像时可能会遇到的常见问题的解决方法](https://mozillazg.com/2020/03/use-alpine-image-common-issues.rst.html))：

## 镜像中存在可执行文件但是报错no such file or directory
由于golang构建与docker基于alpine打包分开
我的二进制文件是使用动态链接方式编译了一个使用了GLIBC库的程序生成的，但是alpne镜像中没有GLIBC库而是用的MUSL LIBC库，这样就会导致该二进制文件无法被执行。
解决方案一般有两种：
> - 改为静态编译
> - 如果要使用动态链接函数编译的话，不要依赖GLIBC（比如编译Go程序的时候指定CGO_ENABLED=0 ）或者在alpine中编译一个依赖MUSL LIBC的版本

追求精简的我选择了第一种[静态编译](https://juejin.cn/post/7053450610386468894)
同时也实测简单的指定`CGO_ENABLED=0`并不完全解决问题
## alpine时区问题
有些使用 alpine 作为基础镜像的 go 程序镜像可能会出现类似下面这样的错误:
``` shell
panic: open /usr/local/go/lib/time/zoneinfo.zip: no such file or directory

Init mysql error: unknown time zone Asia/Shanghai
```
常见原因：alpine 基础镜像中没有包含时区信息文件，当代码中有调用类似下面这样的通过名称获取时区信息的时候，就会出现上面的错误。
所以需要我们在构建镜像是安装自己需要的时区文件,之后就不会存在时区问题了
``` shell
FROM alpine:latest

WORKDIR /app

ENV TZ Asia/Shanghai
RUN apk update && apk add tzdata
RUN cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone
...
```

以上，解决了我这次项目中的问题。

本来还想写一下关于golang静态编译的问题，但发现我还没有真正搞明白，所以先贴个链接吧，以后再写：https://juejin.cn/post/7053450610386468894