---
title: "CentOS7安装MinIO单机版"                         
author: "雨吁"  
description : "CentOS7安装MinIO单机版"    
date: 2022-05-29        
lastmod: 2022-05-29             

tags : [                                    
"centos7",
"minio",
"安装"
]
categories : [                         
"centos7",
"minio",
"安装"
]
keywords : [                                
"centos7",
"minio",
"安装"
]
---

#获取minio
https://dl.min.io/server/minio/release/linux-amd64/minio

#创建目录
- 数据存储目录
- 配置文件目录
- 启动脚本及二进制文件目录

```shell
mkdir -p /opt/minio
mkdir -p /opt/minio/{etc,data,run}

```
将 minio 二进制文件上传到 /opt/minio/run 目录
默认的配置目录是 ${HOME}/.minio，可以使用--config-dir命令行选项重写之。MinIO server在首次启动时会生成一个新的config.json，里面带有自动生成的访问凭据

#启动设置
minio的 参数 最后面的就是数据目录位置
注意 密码一旦设置了，就不可以修改了，如果修改，导致目录不可以访问。
```shell
vim /opt/minio/run/run.sh
#!/bin/bash
export MINIO_ACCESS_KEY=minio
export MINIO_SECRET_KEY=*****************

/opt/minio/run/minio server --config-dir /opt/minio/etc \
--console-address ":9999" \
/opt/minio/data

```
#服务设置
- WorkingDirectory：二进制文件目录
- ExecStart：指定集群启动脚本

/usr/lib/systemd/system 所有服务都放在其下
```shell
vim /usr/lib/systemd/system/minio.service
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/

[Service]
WorkingDirectory=/opt/minio/run/
ExecStart=/opt/minio/run/run.sh

Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```
#启停服务
```shell
systemctl enable minio.service   
systemctl daemon-reload
systemctl start minio
systemctl status minio.service
systemctl stop minio

```
#权限修改
- service文件
- 二进制文件
- 集群启动脚本

#启动
```shell
systemctl daemon-reload
systemctl enable minio && systemctl start minio
```
