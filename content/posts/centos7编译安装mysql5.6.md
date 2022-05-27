---
title: "centos7编译安装mysql5.6"                         
author: "雨吁"  
description : "centos7编译安装mysql5.6"    
date: 2022-05-27        
lastmod: 2022-05-27             

tags : [                                    
"centos7",
"mysql",
"编译"
]
categories : [                              
"centos7",
"mysql",
"编译"
]
keywords : [                                
"centos7",
"mysql",
"编译"
]
---


## 1.软件准备

```bash
cd /opt/soft
wget https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.51.tar.gz
tar xf mysql-5.6.51.tar.gz
```

## 2.创建用户

```bash
groupadd -g 550 mysql
useradd -g mysql -u 550 mysql
```

## 3.安装依赖

```bash
yum install cmake autoconf wget gcc-c++ gcc ncurses-devel openssl openssl-devel -y
cd mysql-5.6.51/
cmake \
-DCMAKE_INSTALL_PREFIX=/opt/mysql \
-DMYSQL_DATADIR=/opt/data/mysql \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1  \
-DMYSQL_UNIX_ADDR=/var/run/mysql/mysqld.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DENABLE_DOWNLOADS=1 \
-DEXTRA_CHARSETS=all  \
-DDEFAULT_CHARSET=utf8  \
-DDEFAULT_COLLATION=utf8_general_ci
```

执行完cmake之后、执行make和make install 

###### 参数详解：

```bash
-DCMAKE_INSTALL_PREFIX=/opt/mysql \   #安装路径
-DMYSQL_DATADIR=/opt/data/mysql \     #数据文件存放地
-DSYSCONFDIR=/etc \         #配置文件my.cnf存放地 
-DWITH_MYISAM_STORAGE_ENGINE=1 \    #支持MyIASM引擎 
-DWITH_INNOBASE_STORAGE_ENGINE=1 \  #支持InnoDB引擎   
-DWITH_MEMORY_STORAGE_ENGINE=1  \   #支持Memory引擎   
-DMYSQL_UNIX_ADDR=/var/run/mysql/mysqld.sock \  #连接数据库socket路径  
-DMYSQL_TCP_PORT=3306 \     #数据库端口号 
-DENABLED_LOCAL_INFILE=1 \  #允许从本地导入数据   
-DWITH_PARTITION_STORAGE_ENGINE=1 \ #安装支持数据库分区 
-DEXTRA_CHARSETS=all  \     #安装所有的字符集   
-DDEFAULT_CHARSET=utf8  \   #默认字符 
-DDEFAULT_COLLATION=utf8_general_ci  
```

## 4.数据库初始化

```bash
#对目录授权

```

