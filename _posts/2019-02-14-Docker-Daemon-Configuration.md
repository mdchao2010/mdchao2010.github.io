---
layout:     post
title:      "Docker Daemon Configuration"
subtitle:   "A better way to config Docker Daemon"
date:       2019-02-15
header-img: "img/post-bg-2015.jpg"
categories: Docker
catalog: true
tags:
    - Docker
---

## Installation

### Check Prerequisites

Docker对操作系统的版本有一定要求。目前，已经支持大多数Linux发行版本，macOS以及Windows，甚至支持一些公有云平台，例如：AWS，Azure等。
具体支持的平台，以及相应的要求，参考官网[Install Docker Engine](https://docs.docker.com/engine/installation/)。

### Install from Docker’s Repositories

CentOS官方提供docker RPM安装包，不过这个安装包是经过定制化的。与Docker官方提供的RPM包有些差异：  
* CentOS提供的RPM包的名字是`docker`，而Docker官方提供的RPM包的名字是`docker-engine`
* 提供docker-storage-setup，能够自动为Docker创建direct LVM的Device mapper

因此，已经在CentOS上安装了旧的Docker，需要先将旧的Docker卸载：`yum -y remove docker docker-selinux`，然后才能安装Docker官方提供的最新RPM包。

### Install from Docker Binaries

从官方[Release Notes](https://github.com/docker/docker/releases)下载平台对应的二进制文件压缩包，格式为`tar.gz`或者`zip`。将压缩包解压，并将解压后的二进制文件复制到/usr/bin/目录下。然后就可以后台启动Docker Daemon: `sudo dockerd &`。

一般情况下，不建议通过二进制文件安装。因为，这种方式安装的Docker不会被systemd管理，不方便Docker运行的管理和Debug。但是，这种方式非常适合Docker升级，不用卸载老的Docker，只需替换二进制文件，还是可以按以前的方式执行Docker。

## Configuration

由于一直使用的操作系统是CentOS 7.1，因此下面介绍的Docker Configuration都是基于该平台的。Docker同时支持`Command Options`和`--config-file`两种配置Docker Daemon的方式。之前在Docker v1.10.3中，使用`Command Options`的方式，后来升级到Docker v1.13.0之后，开始使用`--config-file`的方式。选择使用`--config-file`的主要原因：  
* JSON格式的配置文件，简单、清晰和集中；不像`Command Options`配置分散在多个文件和变量中
* docker.service更加简单，不用`EnvironmentFile`导入环境变量，`ExecStart`后面也不用跟各种参数
* 支持通过systemd动态加载配置，不用重启Docker（Docker v1.12.0开始引入）
* 这也是Docker官方建议的配置方式。

`Command Options`和`--config-file`两种配置方式可以一起使用，不过需要注意的是，它们之间不能有冲突。这两种配置中不能存在相同的option，无论它们的值是否相同，否则Docker Daemon将无法启动。

### Useful Options

```text
{ 
 "debug": true,       #当设置为true时，它将守护程序更改为调试模式。可以看到很多的启动信息。默认false
 "live-restore": true,    #允许在守护程序停机期间保持容器处于活动状态。
 "api-cors-header":"", 
 "bip": "",
 "bridge":"",     #标志设置docker0为默认桥接网络。它是在您安装Docker时自动创建的。如果未使用默认值，则必须手动创建和配置网桥或仅将其设置为“none”：--bridge=none
 "cgroup-parent":"",
 "cluster-store":"",       #它使用新地址重新加载发现存储。
 "cluster-store-opts":{},  #它使用新选项重新加载发现存储。
 "cluster-advertise":"",   #它修改重新加载后通告的地址
 "default-gateway":"",
 "default-gateway-v6":"",
 "default-runtime":"runc",    #如果在创建容器时未指定，则更新要使用的运行时。它默认为“default”，这是官方docker软件包附带的运行时。
 "default-ulimits":{},
 "disable-legacy-registry":false,
 "registry-mirrors":["xxxx"],  #镜像加速的地址，增加后在 docker info中可查看。可以使用Daocloud加速器（需注册，使用免费）或者其他云厂商提供的免费的加速服务
 "insecure-registries": [],   #配置docker的私库地址
 "dns": ["192.168.1.1"],   #设定容器DNS的地址，在容器的 /etc/resolv.conf文件中可查看。dns参数值为Google DNS nameserver：8.8.8.8和8.8.4.4,阿里DNS：["223.5.5.5", "223.6.6.6"]
 "dns-opts": [],       #容器 /etc/resolv.conf 文件，其他设置
 "dns-search": [],      #设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的 主机时，DNS不仅搜索host，还会搜                             
 "shutdown-timeout":int,     #它使用新的超时来替换守护程序的现有配置超时，以关闭所有容器。
 "labels":["nodeName=node-121"], #docker主机的标签，很实用的功能,例如定义：–label nodeName=host-121                                  索host.example.com 。 注意：如果不设置， Docker 会默认用主机上的 /etc/resolv.conf 来配置容器。
 "max-concurrent-downloads":3,  #它更新每次拉动的最大并发下载量。
 "max-concurrent-uploads":5,   #它更新每次推送的最大并发上传次数。
 "authorization-plugins":[],   #它指定要使用的授权插件。
  
 "exec-opts": [],
 "fixed-cidr":"",
 "fixed-cidr-v6":"",
 "hosts": [],          #指定Docker守护程序将侦听客户端连接的位置。,如果未指定，则默认为/var/run/docker.sock。
 "pidfile":"/var/run/docker.pid",    #是存储守护程序的进程ID的路径。在此处指定pid文件的路径。
 "exec-root":"",  #存储容器状态的路径。默认值是/var/run/docker。在这里指定运行守护进程的路径。
 "graph":"/var/lib/docker",   #已废弃，使用data-root代替,这个主要看docker的版本:Deprecated In Release: v17.05.0
 "data-root":"",  #是存储持久数据（如图像，卷和群集状态）的路径。默认值是/var/lib/docker。为避免与其他守护进程发生冲突，请为每个守护进程单独设置此参数。
 "group": "",     #Unix套接字的属组,仅指/var/run/docker.sock
 "iptables": false,    #阻止Docker守护进程添加iptables规则。如果多个守护进程管理iptables规则，它们可能会覆盖另一个守护进程设置的规则。请注意，禁用此选项需要您手动添加iptables规则以公开容器端口。如果你阻止Docker添加iptables规则，Docker也不会添加IP伪装规则，即使你设置 --ip-masq为true。如果没有IP伪装规则，当使用默认网桥以外的网络时，Docker容器将无法连接到外部主机或Internet。
 "icc": false,
 "ip":"0.0.0.0",
 "ipv6": false,
 "ip-forward": false,    #默认true, 启用 net.ipv4.ip_forward ,进入容器后使用 sysctl -a | grepnet.ipv4.ip_forward 查看
 "ip-masq":false,

 "log-driver": "json-file", #Docker用来接收来自容器内部stdout/stderr的日志的模块，默认的log driver是JSON File logging driver
 "log-level":"",
 "log-opts": {
     "max-size": "100m",
     "max-files":"5"
 },
 "mtu": 0,
 "oom-score-adjust":-500,
 "raw-logs": false,
 "runtimes": 
 {
    "runc": 
    {
        "path": "runc"
    },
    "custom": 
    {
        "path":"/usr/local/bin/my-runc-replacement",
        "runtimeArgs": [ "--debug" ]
    }
 },
 "selinux-enabled": false, #默认 false，启用selinux支持
 
 "storage-driver":"overlay2", #Docker推荐使用overlay2作为Storage driver, 该参数和linux内核版本也有关
 "storage-opts": [],
 "swarm-default-advertise-addr":"",
 "tls": true,      #默认 false, 启动TLS认证开关
 "tlscacert": "",    #默认 ~/.docker/ca.pem，通过CA认证过的的certificate文件路径
 "tlscert": "",     #默认 ~/.docker/cert.pem ，TLS的certificate文件路径
 "tlskey": "",      #默认~/.docker/key.pem，TLS的key文件路径
 "tlsverify": true,   #默认false，使用TLS并做后台进程与客户端通讯的验证
 "userland-proxy":false,
 "userns-remap":""
 }
```

### Configuration for Docker v1.10.3

service文件中需要指定配置文件来导入环境变量，而且环境变量需要加到`ExecStart`才能生效。

```
# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target
Wants=docker-storage-setup.service

[Service]
LimitMEMLOCK=1288490188800
LimitSTACK=infinity
LimitNPROC=infinity
LimitNOFILE=1310720
LimitCORE=infinity
Type=notify
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
ExecStart=/usr/bin/docker daemon $OPTIONS \
          $DOCKER_CERT \
          $DEFAULT_ULIMIT \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY
MountFlags=slave
TimeoutStartSec=1min
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

多个配置文件

/etc/sysconfig/docker
```
# cat /etc/sysconfig/docker
# /etc/sysconfig/docker

# Modify these options if you want to change the way the docker daemon runs
OPTIONS='--disable-legacy-registry --log-opt max-size=2m --log-opt max-file=5 --log-level=warn'
DEFAULT_ULIMIT='--default-ulimit nofile=131072 --default-ulimit memlock=131941395333120 --default-ulimit core=-1 --default-ulimit nproc=-1 --default-ulimit stack=-1'
INSECURE_REGISTRY='--insecure-registry https://registry.access.redhat.com'

DOCKER_CERT_PATH=/etc/docker

# If you want to add your own registry to be used for docker search and docker
# pull use the ADD_REGISTRY option to list a set of registries, each prepended
# with --add-registry flag. The first registry added will be the first registry
# searched.
#ADD_REGISTRY='--add-registry registry.access.redhat.com'

# If you want to block registries from being used, uncomment the BLOCK_REGISTRY
# option and give it a set of registries, each prepended with --block-registry
# flag. For example adding docker.io will stop users from downloading images
# from docker.io
# BLOCK_REGISTRY='--block-registry'

# If you have a registry secured with https but do not have proper certs
# distributed, you can tell docker to not look for full authorization by
# adding the registry to the INSECURE_REGISTRY line and uncommenting it.
# INSECURE_REGISTRY='--insecure-registry'

# On an SELinux system, if you remove the --selinux-enabled option, you
# also need to turn on the docker_transition_unconfined boolean.
# setsebool -P docker_transition_unconfined 1

# Location used for temporary files, such as those created by
# docker load and build operations. Default is /var/lib/docker/tmp
# Can be overriden by setting the following environment variable.
# DOCKER_TMPDIR=/var/tmp

# Controls the /etc/cron.daily/docker-logrotate cron job status.
# To disable, uncomment the line below.
# LOGROTATE=false
```

/etc/sysconfig/docker-storage
```
# cat /etc/sysconfig/docker-storage
DOCKER_STORAGE_OPTIONS=--storage-driver devicemapper --storage-opt dm.fs=xfs --storage-opt dm.thinpooldev=/dev/mapper/docker--vg-docker--pool --storage-opt dm.use_deferred_removal=true --storage-opt dm.use_deferred_deletion=true
```

/etc/sysconfig/docker-network
```
# cat /etc/sysconfig/docker-network 
# /etc/sysconfig/docker-network
DOCKER_NETWORK_OPTIONS=
```

### Configuration for Docker v1.13.0

service文件采用默认的即可，不需要额外配置。

```
# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target firewalld.service

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
```

`daemon.json`配置文件集中配置，结构清晰。

```
# cat /etc/docker/daemon.json
{
    "disable-legacy-registry": true,
    "max-concurrent-downloads": 10,
    "max-concurrent-uploads": 20,
    "log-level": "warn",
    "log-opts": {
        "max-size": "2m",
        "max-file": "5"
    },
    "storage-driver": "devicemapper",
    "storage-opts": [
        "dm.fs=xfs",
        "dm.thinpooldev=/dev/mapper/docker--vg-docker--pool",
        "dm.use_deferred_removal=true",
        "dm.use_deferred_deletion=true"
    ],
    "insecure-registries": ["https://registry.access.redhat.com"],
    "live-restore": true
}
```
### make daemon.json file work

```text
a.修改完成后reload配置文件
sudo systemctl daemon-reload

b.重启docker服务
sudo systemctl restartdocker.service

c.查看状态
sudo systemctl status docker -l

d.查看服务
sudo docker info
```

## Reference

* [Install Docker Engine](https://docs.docker.com/engine/installation/)
* [Configure and run Docker on various distributions](https://docs.docker.com/engine/admin/)
* [Docker Daemon Options](https://docs.docker.com/engine/reference/commandline/dockerd/)