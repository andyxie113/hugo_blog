---
title: "CentOS7编译安装Redis5.0"                         
author: "雨吁"  
description : "CentOS7编译安装Redis5.0"    
date: 2022-05-27        
lastmod: 2022-05-27             
tags : [                                    
"centos7",
"redis",
"编译"
]
categories : [                              
"centos7",
"redis",
"编译"
]
keywords : [                                
"centos7",
"redis",
"编译"
]
---

​	**Centos7 Redis5安装及配置，自启动配置**

## 1.解压、编译、安装reids

​	[官网地址](https://redis.io/)

```bash
#获取安装包
cd /opt/soft
wget https://download.redis.io/releases/redis-5.0.14.tar.gz
#解压
tar xf redis-5.0.14.tar.gz
#安装依赖
yum -y install gcc automake autoconf libtool make
#编译
cd redis-5.0.14/
make
#编译安装
make install PREFIX=/opt/redis5
```

## 2.Redis配置文件修改

​	创建redis的data目录，存放redis配置文件

```bash
mkdir -p /opt/data/redis
#复制redis.conf配置文件
cp redis.conf /opt/redis5/bin/
cd /opt/redis5/bin/

#配置文件设置
vim redis.conf
daemonize yes	--开启守护进程模式
bind 127.0.0.1	--注释bind 127.0.0.1这行，否则只能本地连接redis
protected-mode no	--关闭保护模式，开启远程连接
pidfile /opt/data/redis/redis.pid
logfile "redis.log"
dbfilename dump.rdb
dir /opt/data/redis
requirepass foobared	--在这行下面增加一行“requirepass password”来开启Redis密码认证
```

## 3.Redis启动

```bash
/opt/redis5/bin/redis-server /opt/redis5/bin/redis.conf
```

## 4.Centos7设置Redis开机启动

```bash
#创建服务文件
vi /etc/systemd/system/redis.service

[Unit]
#Description:描述服务
Description=Redis
#After:描述服务类别 
After=network.target
#服务运行参数的设置 
[Service]
#Type=forking是后台运行的形式 
Type=forking
#ExecStart为服务的具体运行命令，路径必须是绝对路径 
ExecStart=/opt/redis5/bin/redis-server /opt/redis5/bin/redis.conf
#ExecReload为重启命令 ，路径必须是绝对路径 
ExecReload=/opt/redis5/bin/redis-server -s reload
#ExecStop为停止命令 ，路径必须是绝对路径 
ExecStop=/opt/redis5/bin/redis-server -s stop
#PrivateTmp=True表示给服务分配独立的临时空间 
PrivateTmp=true
#运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3
[Install]
WantedBy=multi-user.target

#重载系统服务
systemctl daemon-reload

#命令
systemctl start redis.service #启动redis服务 
systemctl enable redis.service #设置开机自启动 
systemctl disable redis.service #停止开机自启动 
systemctl status redis.service #查看服务当前状态 
systemctl restart redis.service　 #重新启动服务 
systemctl list-units --type=service #查看所有已启动的服务 
```

