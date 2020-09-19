---
layout: post
title:  "编译Docker源码（19.03.12）"
date:   2020-09-18
categories: docker/moby
tags: docker centos7
---

* content
{:toc}



## 前言

很久前Docker就已经更名为moby了，就moby的的编译与之前docker的编译相同，因为无论是docker还是moby，
都将自己的编译放在了容器中，编译所依赖的包也在容器中完成安装。

本文在centos7系统下，对moby的编译配置文件进行了修改，从而使moby快速完成编译


### 系统环境与软件版本

```properties
Linux ks-0 3.10.0-1062.18.1.el7.x86_64 #1 SMP Tue Mar 17 23:49:17 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
go version go1.13.4 linux/amd64
Docker version 19.03.8, build afacb8b
```
### 编译原理

```properties
 Debian:jessie
+-------------+                              golang 1.7.5-alpine
|             |                          +------------------------+ 
|  Moby       | ----- docker run ----->  | docker manpage compile |
|  Compile    |                          +------------------------+             centos:7
|  Container  |                                                         +--------------------+
|             | -------------- docker run ----------------------------> |  build centos rpm  |                           
+-------------+                                                         +--------------------+
```
Moby先创建一个Debian的容器，编译Moby，然后该容器根据用户输入的参数决定使用哪些容器编译哪些安装包，在编译目标安装包之前会使用golang容器编译manpage。总之moby的所有编译工作都是在容器中进行的。

官方提供编译步骤依次为：`make build`和`make binary`。先看懂`Makefile`会帮助理解docker基本结构。

`make build`其实就是docker build，于是要看Dockerfile文件。其制作一个叫docker-dev的镜像，镜像中会生成源码编译的环境。

`make binary`其实就是docker run docker-dev，即运行docker-dev一个容器，并在容器中的bundles文件夹下生成dockers所需的二进制文件。
### 安装步骤


1.下载docker源码。
```bash
git clone -b v19.03.12 https://github.com/moby/moby.git  /gospace/src/github.com/docker/docker 
```
NOTE: moby 的源代码, 必须 git clone 到  ${GOPATH}/src/github.com/docker/docker 目录, 并且目录名叫做 docker, 而不是 moby

2.修改Dockerfile

由于使用局域网环境添加代理`http_proxy`和`https_proxy`
```properties
ENV http_proxy 172.0.0.1:808
ENV http_proxys 172.0.0.1:808
```

Dockerfile的基础镜像是debian，鉴于国内网络问题，强烈建议使用国内源地址,（这里使用清华源）
```bash
RUN sed -ri "s/(httpredir|deb).debian.org/${APT_MIRROR:-deb.debian.org}/g" /etc/apt/sources.list \
 && sed -ri "s/(security).debian.org/${APT_MIRROR:-security.debian.org}/g" /etc/apt/sources.list
```
修改为
```properties
RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free"  | tee /etc/apt/sources.list
```
此外文件中还有很多需要在github上下载源码的命令，这里在码云上找到对应的代码仓库，替换成了国内码云地址
```properties
https://github.com/XXX/XXX.git
```
修改为：
```properties
https://gitee.com/XXX/XXX.git
```


### 修改 `hack/dockerfile/install`下 ` *.installer`脚本

此外文件中还有很多需要在github上下载源码的命令，这里也替换成了国内码云地址
```properties
https://github.com/XXX/XXX.git
```
修改为：
```properties
https://gitee.com/XXX/XXX.git
```
#### 修改`dockercli.installer`脚本文件

```properties
url=https://download.docker.com/linux/static
```
修改为
```properties
url=https://mirrors.aliyun.com/docker-ce/linux/static
```

### 修改`download-frozen-image-v2.sh`文件

镜像下载慢问题：由于sh脚本中定义的镜像仓库都是国外的仓库，下载镜像比较慢，甚至终端，可以把镜像仓库替换为国内的镜像仓库，download-frozen-image-v2.sh文件中的registryBase
```
registryBase='https://registry-1.docker.io'
authBase='https://auth.docker.io'
authService='registry.docker.io'

替换为：
registryBase='https://5162s31v.mirror.aliyuncs.com'
authBase='https://auth.docker.io'
authService='registry.docker.io'
```
### 编译 docker container 

```properties
# cd /$GOPATH/src/github.com/docker/docker
# make build

如果成功的话
可以看到
# docker images
docker-dev             HEAD                47558c9ebb21        18 hours ago        2.06GB
```
如果需要配置代理，在`build: bundles` 命令中添加代理，即`docker build`后添加`--build-arg http_proxy=http://172.0.0.1:808 --build-arg https_proxy=http://172.0.0.1:808`
上面是在Dockerfile中配置了代理，但是执行`make build`时，代理并未生效。
### 在容器中编译 moby

```properties
# cd /$GOPATH/src/github.com/docker/docker
# make binary
```

编译成功的话, 在`/$GOPATH/src/github.com/docker/docker/bundles`目录下可以看到
```properties
 binary-daemon
    ├── containerd
    ├── containerd.md5
    ├── containerd.sha256
    ├── containerd-shim
    ├── containerd-shim.md5
    ├── containerd-shim.sha256
    ├── ctr
    ├── ctr.md5
    ├── ctr.sha256
    ├── dockerd -> dockerd-dev
    ├── dockerd-dev
    ├── dockerd-dev.md5
    ├── dockerd-dev.sha256
    ├── dockerd-rootless.sh
    ├── dockerd-rootless.sh.md5
    ├── dockerd-rootless.sh.sha256
    ├── docker-init
    ├── docker-init.md5
    ├── docker-init.sha256
    ├── docker-proxy
    ├── docker-proxy.md5
    ├── docker-proxy.sha256
    ├── rootlesskit
    ├── rootlesskit-docker-proxy
    ├── rootlesskit-docker-proxy.md5
    ├── rootlesskit-docker-proxy.sha256
    ├── rootlesskit.md5
    ├── rootlesskit.sha256
    ├── runc
    ├── runc.md5
    ├── runc.sha256
    ├── vpnkit
    ├── vpnkit.md5
    └── vpnkit.sha256

1 directory, 34 files

```