---
title: "CentOS7编译安装MySQL5.6"                         
author: "雨吁"  
description : "CentOS7编译安装MySQL5.6"    
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
​	执行完cmake之后、执行make和make install 

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
​	安装目录授权
```bash
chown -R mysql.mysql /opt/mysql/
chown -R mysql.mysql /opt/data/mysql/
```

​	修改my.cnf文件

```
[mysqld]
skip-grant-tables
explicit_defaults_for_timestamp=true
skip-name-resolve
port = 3306
socket=/opt/mysql/mysqld.sock
basedir=/opt/mysql
datadir=/opt/data/mysql
max_connections = 2000
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
log-error=/opt/mysql/mysql.log
pid-file=/opt/mysql/mysql.pid
!includedir /etc/my.cnf.d
```

​	开始初始化

```bash
./scripts/mysql_install_db --user=mysql --datadir=/opt/data/mysql --no-defaults
```

​	启动mysql

```bash
#拷贝启动文件
cp ./support-files/mysql.server /etc/init.d/mysql
chmod +x /etc/init.d/mysql
/etc/init.d/mysql start
```

​	设置mysql环境变量

```bash
vim /etc/profile
export PATH=$PATH:/usr/local/mysql/bin
source /etc/profile
```

​	修改数据库密码

```sql
mysql
use mysql;
update user set password=password("newpasswd") where user='root' and host='localhost';
flush privileges;
```

​	开启远程连接访问

```sql
#允许任意主机以用户root和密码连接到mysql服务器
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'passwd' WITH GRANT OPTION;
#特定ip的主机以特定用户和密码连接到mysql 服务器
GRANT ALL PRIVILEGES ON *.* TO 'user'@'x.x.x.x' IDENTIFIED BY 'userpwd' WITH GRANT OPTION; 
#刷新MySQL的系统权限相关表
flush PRIVILEGES;
```

