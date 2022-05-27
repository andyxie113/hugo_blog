---
title: "CentOS7安装NSQ"                         
author: "雨吁"  
description : "CentOS7安装NSQ"    
date: 2022-05-26        
lastmod: 2022-05-26             

tags : [                                    
"centos",
"go",
"nsq"
]
categories : [                         
"centos",
"go",
"nsq"
]
keywords : [                                
"centos",
"go",
"nsq"
]
---

#### 安装go环境

##### 获取包

```bash
#下载包
wget https://golang.google.cn/dl/go1.18.2.linux-amd64.tar.gz
#解压go包到/usr/local
tar -zxf go1.18.2.linux-amd64.tar.gz -C /usr/local
```

##### 配置go环境变量

```bash
vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
#环境变量生效
source /etc/profile        #和下面的命令等价
 . /etc/profile             #和上面的命令等价
#检查验证
go env
go version
```

#### nsq安装

##### 获取安装包

```bash
下载地址 https://nsq.io/deployment/installing.html
wget https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.2.0.linux-amd64.go1.12.9.tar.gz
tar -zxvf nsq-1.2.0.linux-amd64.go1.12.9.tar.gz -C /usr/local/
cd /usr/local
mv nsq-1.2.0.linux-amd64.go1.12.9/ nsq-1.2
```

##### 配置nsqlookupd服务

```bash
vi /usr/lib/systemd/system/nsqlookupd.service
[Unit]
Description=nsqlookup daemon Service
After=network.target remote-fs.target nss-lookup.target
[Service]
#Type=
PrivateTmp=yes
ExecStart=/usr/local/nsq-1.2/bin/nsqlookupd
Restart=on-abort
[Install]
WantedBy=multi-user.target
```

##### 配置nsqd服务

```bash
vi /usr/lib/systemd/system/nsqd.service 
[Unit]
Description=nsqd daemon Service
After=network.target remote-fs.target nss-lookup.target
[Service]
#Type=
PrivateTmp=yes
ExecStart=/usr/local/nsq-1.2/bin/nsqd --lookupd-tcp-address=0.0.0.0:4160 --tcp-address=0.0.0.0:4150 --http-address=0.0.0.0:4151 --broadcast-address=127.0.0.1
#修改broadcast-address
Restart=on-abort 
[Install]
WantedBy=multi-user.target
```

##### 配置nsqadmin服务

```bash
vi /usr/lib/systemd/system/nsqadmin.service
[Unit]
Description=nsqadmin daemon Service
After=network.target remote-fs.target nss-lookup.target
[Service]
#Type=
PrivateTmp=yes
ExecStart=/usr/local/nsq-1.2/bin/nsqadmin --lookupd-http-address=127.0.0.1:4161
Restart=on-abort 
[Install]
WantedBy=multi-user.target
```

##### 启动nsq

```bash
systemctl start nsqd  
systemctl start nsqadmin  
systemctl start nsqlookupd
```



