---
title: "CentOS7快速安装docker"                         
author: "雨吁"  
description : "CentOS7快速安装docker"    
date: 2022-05-28        
lastmod: 2022-05-28             

tags : [                                    
"centos7",
"docker",
"安装"
]
categories : [                         
"centos7",
"docker",
"安装"
]
keywords : [                                
"centos7",
"docker",
"安装"
]
---

# 卸载旧版本

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
## 使用yum安装
```shell
$ sudo yum install -y yum-utils

添加yum软件源
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo
```
###安装
```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```
##使用脚本自动安装

```shell
# $ curl -fsSL test.docker.com -o get-docker.sh
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
# $ sudo sh get-docker.sh --mirror AzureChinaCloud
```
##启动Docker

```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```
##建立 docker 用户组
	默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。
建立 docker 组：

```shell
$ sudo groupadd docker
```
将当前用户加入 docker 组：
```shell
$ sudo usermod -aG docker $USER
```