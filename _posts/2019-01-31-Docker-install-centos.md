---
layout: post
title:  "Centos7 安装 Docker"
date:   2019-01-31
categories: docker
tags: docker centos7
---

* content
{:toc}



## 前言

在使用centos7，并使用yum安装docker的时候，往往不希望安装最新版本的docker，而是希望安装与自己熟悉或者当前业务环境需要的版本，例如目前Kubernetes支持的最新docker版本为v17.03，所以就产生了安装指定版本docker的需求。


### 卸载老版本

Docker老版本（例如1.13），叫做docker-engine。Docker进入17.x版本后，名称发生了变化，叫做docker-ce或者docker-ee。因此，如果有安装老版本的Docker，必须先删除老版本的Docker。

执行以下命令即可：
```bash
sudo yum remove docker docker-common container-selinux docker-selinux docker-engine
```
需要注意的是，执行该命令只会卸载Docker本身，而不会删除Docker内容，例如镜像、容器、卷以及网络。这些文件保存在`/var/lib/docker` 目录中，需要手动删除。

### 安装步骤

1.安装yum-utils ，这样我们就能使用yum-config-manager 设置Yum源。
```bash
yum install -y yum-utils
```
2.添加Docker软件包源
```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
3.【可选】执行以下命令，启用“edge”仓库。edge仓库其实也包含在了docker.repo文件中了，但默认是禁用的，我们可使用以下命令启用edge仓库。
```bash
sudo yum-config-manager --enable docker-ce-edge
```
如果想要再次禁用edge仓库，可执行以下命令。
```bash
sudo yum-config-manager --disable docker-ce-edge
```
**TIPS**： Docker有两种构建方式，Stable（稳定）构建一般是一个季度发布一次；Edge(边缘)构建一般是一个月发布一次。


### 安装Docker

1.执行以下命令更新Yum的包索引
```bash
sudo yum makecache fast
```
2.安装你想要的Docker版本(CE/EE)

Docker版本 |	命令
---|---
Docker CE|	sudo yum install docker-ce
Docker EE|	sudo yum install docker-ee
3.在生产环境中，我们可能需要指定想要安装的版本。可使用以下命令列出当前可用的Docker版本。
```bash
yum list docker-ce.x86_64  --showduplicates |sort -r
```
这样，我们可使用以下命令安装指定版本的Docker。

Docker版本 |	命令
---|---
Docker CE|	sudo yum install docker-ce-
Docker EE|	sudo yum install docker-ee-

4.启动Docker
```bash
systemctl start docker & systemctl enable docker
```
5.验证安装是否正确
```bash
 docker info
```


### 采坑指南

在执行以下命令的时候：`yum install docker-ce-17.03.0.ce -y`会出现如下的报错：
```bash
--> Finished Dependency Resolution
Error: Package: docker-ce-17.03.0.ce-1.el7.centos.x86_64 (docker-ce-stable)
           Requires: docker-ce-selinux >= 17.03.0.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.0.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.1.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.2.ce-1.el7.centos
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```
在出现这个问题之后，需要执行以下命令：

要先安装`docker-ce-selinux-17.03.2.ce`，否则安装`docker-ce`会报错
```
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm 
```
然后再安装 `docker-ce-17.03.2.ce`，就能正常安装
```bash
yum install docker-ce-17.03.2.ce-1.el7.centos
```
