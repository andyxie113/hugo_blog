---
title: "CentOS7安装nsq"                         
author: "雨吁"  
description : "CentOS7安装nsq"    
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

### go环境安装

```bash
#下载包
wget https://golang.google.cn/dl/go1.18.2.linux-amd64.tar.gz
#解压go包到/usr/local
tar -zxf go1.18.2.linux-amd64.tar.gz -C /usr/local
#配置环境变量
vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

source /etc/profile        #和下面的命令等价
 . /etc/profile             #和上面的命令等价

#检查
go env
go version
```

### nsq安装

```bash
下载地址 https://nsq.io/deployment/installing.html
wget https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.2.0.linux-amd64.go1.12.9.tar.gz
tar -zxvf nsq-1.2.0.linux-amd64.go1.12.9.tar.gz -C /usr/local/
cd /usr/local
mv nsq-1.2.0.linux-amd64.go1.12.9/ nsq-1.2

##配置开机启动
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

#启动 
systemctl start nsqd  
systemctl start nsqadmin  
systemctl start nsqlookupd
```



