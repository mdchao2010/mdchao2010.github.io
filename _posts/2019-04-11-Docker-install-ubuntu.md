---
layout: post
title:  "ubuntu 16 安装 Docker"
date:   2019-04-11
categories: docker
tags: docker ubuntu 16
---

* content
{:toc}



## 前言

Docker 要求 Ubuntu 系统的内核版本高于 3.10,通过下面的命令查看内核版本：
```bash
uname -r
```


### 卸载老版本

Docker老版本（例如1.13），叫做docker-engine。Docker进入17.x版本后，名称发生了变化，叫做docker-ce或者docker-ee。
因此，如果有安装老版本的Docker，必须先删除老版本的Docker。

执行以下命令即可：
```bash
 sudo apt-get remove docker docker-engine docker.io
```
需要注意的是，执行该命令只会卸载Docker本身，而不会删除Docker内容，例如镜像、容器、卷以及网络。这些文件保
存在`/var/lib/docker` 目录中，需要手动删除。

### 安装步骤

由于docker安装需要使用https，所以需要使 apt 支持 https 的拉取方式。

1.安装https。
```bash
sudo apt-get update # 先更新一下软件源库信息
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
2.添加Docker软件包源,设置apt仓库地址

鉴于国内网络问题，强烈建议使用国内地址,添加 阿里云 的apt仓库（使用国内源）
```bash
$ curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

$ sudo add-apt-repository \
     "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
```
**TIPS**： Docker有两种构建方式，Stable（稳定）构建一般是一个季度发布一次；Edge(边缘)构建一般是一个月发布一次。


### 安装Docker

1.执行以下命令更新apt的包索引
```bash
$ sudo apt-get update

$ sudo apt-get install docker-ce # 安装最新版的docker
```
2.安装你想要的Docker版本(CE/EE)

```bash
$ apt-cache policy docker-ce # 查看可供安装的所有docker版本
$ sudo apt-get install docker-ce=18.03.0~ce-0~ubuntu # 安装指定版本的docker
```
3.验证安装是否正确
```bash
 docker info
```

### docker 配置http_proxy

docker deamon不会读取系统代理，需要手动配置
根据经验需要修改/etc/default/docker这个配置文件，文件中也确实提供了配置代理的注释和参考，此配置方法适
用于低于Ubuntu16版本系统，Ubuntu 16配置方法如下：

1.创建目录`/etc/systemd/system/docker.service.d` 
```bash
sudo mkdir docker.service.d
```
2.在该目录下创建文件`http-proxy.conf`，在文件中添加配置：
```bash
[Service]
Environment="HTTP_PROXY=http://[proxy-addr]:[proxy-port]/"
Environment="HTTPS_PROXY=http://[proxy-addr]:[proxy-port]/"
```
或者执行
```bash
sh -c "cat << EOF > /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.199.62:808" 
Environment="HTTP_PROXY=http://192.168.199.62:808" 
EOF"
```

3.刷新配置 
```
sudo systemctl daemon-reload
```
4.重启docker服务 
```bash
sudo systemctl restart docker
```

### 采坑指南

按照docker官方安装教程在执行以下命令的时候：`sudo apt-get install docker-ce`会出现如下的报错：
```bash
The following packages have unmet dependencies:
docker-ce : Depends: libltdl7 (>= 2.4.6) but it is not going to be installed
Recommends: aufs-tools but it is not going to be installed
Recommends: cgroupfs-mount but it is not installable or
cgroup-lite but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```
此问题源于libltdl7版本过低，ubuntu16.04默认无更高版本。

解决方案：

搜索到libltdl7 2.4.6的包，下载：
```
wget http://launchpadlibrarian.net/236916213/libltdl7_2.4.6-0.1_amd64.deb
```
要先安装`libltdl7`，否则安装`docker-ce`会报错
```
sudo dpkg -i libltdl7_2.4.6-0.1_amd64.deb 
```
然后再安装 `docker-ce-17.03.2.ce`，就能正常安装
```bash
sudo apt-get install docker-ce
```
