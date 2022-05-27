---
title: "CentOS7安装NSQ"                         
author: "雨吁"  
description : "go环境安装，nsq安装"    
date: 2022-05-26        
lastmod: 2022-05-26             

tags : [                                    
"centos7",
"go",
"nsq"
]
categories : [                         
"centos7",
"go",
"nsq"
]
keywords : [                                
"centos7",
"go",
"nsq"
]
---

#### nsq简介

 	实时的分布式消息处理平台，其设计的目的是用来大规模地处理每天数以十亿计级别的消息。它具有分布式和去中心化拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征

#### 依赖

​	nsq使用go语言编写，需要安装go的环境。本文选用最新版go语言

#### go环境安装

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

[下载地址]: https://nsq.io/deployment/installing.html

```bash
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

#### nsq特性

1. 支持无 SPOF 的分布式拓扑
2. 水平扩展(没有中间件，无缝地添加更多的节点到集群)
3. 低延迟消息传递 (性能)
4. 结合负载均衡和多播消息路由风格
5. 擅长面向流媒体(高通量)和工作(低吞吐量)工作负载
6. 主要是内存中(除了高水位线消息透明地保存在磁盘上)
7. 运行时发现消费者找到生产者服务(nsqlookupd)
8. 传输层安全性 (TLS)
9. 数据格式不可知
10. 一些依赖项(容易部署)和健全的，有界，默认配置
11. 任何语言都有简单 TCP 协议支持客户端库
12. HTTP 接口统计、管理行为和生产者(不需要客户端库发布)
13. 为实时检测集成了 statsd
14. 健壮的集群管理界面 (nsqadmin)

#### NSQ组件

##### NSQD

​	nsqd 是一个守护进程，负责接收，排队，投递消息给客户端。也就是说这个服务是干活的。它可以独立运行，不过通常它是由 nsqlookupd 实例所在集群配置的。
nsqd特性如下：

1. 对订阅了同一个topic，同一个channel的消费者使用负载均衡策略（不是轮询）
2. 只要channel存在，即使没有该channel的消费者，也会将生产者的message缓存到队列中（注意消息的过期处理）
3. 保证队列中的message至少会被消费一次，即使nsqd退出，也会将队列中的消息暂存磁盘上(结束进程等意外情况除外)
4. 限定内存占用，能够配置nsqd中每个channel队列在内存中缓存的message数量，一旦超出，message将被缓存到磁盘中
5. topic，channel一旦建立，将会一直存在，要及时在管理台或者用代码清除无效的topic和channel，避免资源的浪费

##### NSQLOOKUPD

​	nsqlookupd 是守护进程负责管理拓扑信息。客户端通过查询 nsqlookupd 来发现指定话题（topic）的生产者，并且 nsqd 节点广播话题（topic）和通道（channel）信息。也就是说nsqlookupd是管理者。
nsqlookupd 特性：

1. 唯一性，在一个Nsq服务中只有一个nsqlookupd服务。当然也可以在集群中部署多个nsqlookupd，但它们之间是没有关联的.
2. 去中心化，即使nsqlookupd崩溃，也会不影响正在运行的nsqd服务
3. 充当nsqd和naqadmin信息交互的中间件
4.  提供一个http查询服务，给客户端定时更新nsqd的地址目录

##### NSQADMIN

​	nsqadmin是web管理界面，用来汇集集群的实时统计，并执行不同的管理任务。也就是说nsqadmin就是我们在浏览器中看到的那个。
nsqadmin特性如下：

1. 提供一个对topic和channel统一管理的操作界面以及各种实时监控数据的展示，界面设计的很简洁，操作也很简单
2. 展示所有message的数量
3. 能够在后台创建topic和channel
4. nsqadmin的所有功能都必须依赖于nsqlookupd，nsqadmin只是向nsqlookupd传递用户作并展示来自nsqlokupd的数据
