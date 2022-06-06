---
title: "CentOS7源码编译安装Zabbix5.0"                         
author: "雨吁"  
description : "CentOS7源码编译安装zabbix5.0"    
date: 2022-05-26        
lastmod: 2022-05-26             

tags : [                                    
"centos7",
"zabbix",
"安装"
]
categories : [                         
"centos7",
"zabbix",
"安装"
]
keywords : [                                
"centos7",
"zabbix",
"安装"
]
---

​	LNMP就是 Linux+nginx + mysql + PHP，把Apache替换为Nginx

## 一、安装php7.2（zabbix5.0以上版本最低要求7.2）
```shell
[root@ec-zabbix ~]# cd /opt/
[root@ec-zabbix opt]# groupadd www
[root@ec-zabbix opt]# useradd -g www www
```
## 二、安装扩展包并更新系统内核和安装php需要的拓展(包含Nginx依赖）
```shell
yum install epel-release -y
yum update -y
yum -y install wget vim pcre pcre-devel openssl openssl-devel libicu-devel gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel ncurses ncurses-devel curl curl-devel krb5-devel libidn libidn-devel openldap openldap-devel nss_ldap jemalloc-devel cmake boost-devel bison automake libevent libevent-devel gd gd-devel libtool* libmcrypt libmcrypt-devel mcrypt mhash libxslt libxslt-devel readline readline-devel gmp gmp-devel libcurl libcurl-devel openjpeg-devel
```
## 三、开始配置php7.2参数
```shell
$ cp -frp /usr/lib64/libldap* /usr/lib/
$ ./configure --prefix=/opt/php \
--with-config-file-path=/opt/php/etc \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-jpeg-dir=/usr/local/jpeg/ \
--with-png-dir=/usr/local/png/ \
--with-freetype-dir=/usr/local/freetype \
--enable-mysqlnd-compression-support \
--with-iconv-dir \
--with-zlib \
--enable-xml \
--disable-rpath \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--with-curl \
--with-gd=/usr/local/gd \
--enable-mbregex \
--enable-mbstring \
--enable-intl \
--enable-ftp \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-soap \
--with-gettext \
--disable-fileinfo \
--enable-opcache \
--with-pear \
--enable-maintainer-zts \
--with-ldap=shared \
--without-gdbm
```
## 四、开始编译安装
```shell
make -j 4 && make install
```
#### 完成安装后配置php.ini文件：
```shell
$ cp php.ini-development /opt/php/etc/php.ini
$ cp /opt/php/etc/php-fpm.conf.default /opt/php/etc/php-fpm.conf
$ cp /opt/php/etc/php-fpm.d/www.conf.default /opt/php/etc/php-fpm.d/www.conf
ln -s /opt/php/bin/php /usr/bin/php7
php7 -v
```
#### 修改 php.ini 相关参数：
```shell
$ vim /opt/php/etc/php.ini
expose_php = Off
short_open_tag = ON
max_execution_time = 300
max_input_time = 300
memory_limit = 128M
post_max_size = 32M
date.timezone = Asia/Shanghai
;mbstring.func_overload=2 注释
```
#### 设置 OPcache 缓存：
```shell
[opcache]
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1
```
#### 设置php安全函数（可选，如果程序需要这些函数，可以取消禁止）:
```shell
$ vim /opt/php/etc/php.ini
默认值：
disable_functions =
修改为：
disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname
或通配：
disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru
```
#### 配置www.conf

##### 取消以下注释并修改优化其参数：
```shell
listen = /opt/php/php-cgi.sock
listen.owner = www
listen.group = www
listen.mode = 0660
listen.allowed_clients = 127.0.0.1
pm = dynamic
listen.backlog = -1
pm.max_children = 180
pm.start_servers = 50
pm.min_spare_servers = 50
pm.max_spare_servers = 180
request_terminate_timeout = 120
request_slowlog_timeout = 50
```
#### 配置php-fpm.conf
##### 取下以下注释并填写完整路径：
```shell
pid = /opt/php/php-fpm.pid
```
#### 创建system系统单元文件php-fpm启动脚本：
```shell
$ vim /usr/lib/systemd/system/php-fpm.service
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=simple
PIDFile=/opt/php/php-fpm.pid
ExecStart=/opt/php/sbin/php-fpm --nodaemonize --fpm-config /opt/php/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```
##### 启动php-fpm服务并加入开机自启动：
```shell
$ systemctl enable php-fpm.service
$ systemctl restart php-fpm.service
```
## 五、安装mysql8.0*
​	由于 MySQL Server 是 C/C++ 开发的，所以编译安装需要先安装 cmake 和 gcc 等编译工具，构造编译环境
#### 环境准备
```shell
##安装 cmake：
wget https://cmake.org/files/v3.21/cmake-3.21.0-linux-x86_64.tar.gz
tar xvf cmake-3.21.0-linux-x86_64.tar.gz
mv cmake-3.21.0-linux-x86_64 /usr/local/cmake
echo -e "\nexport PATH=/usr/local/cmake/bin:\$PATH" >>/etc/profile
source /etc/profile

##安装gcc,并安装编译安装时需要用的工具包：
yum -y install bzip2 gcc gcc-c++
wget http://ftp.gnu.org/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.gz
tar zxvf gcc-11.2.0.tar.gz
cd gcc-11.2.0

## 下载mpfr、gmp、mpc、isl等依赖包：
./contrib/download_prerequisites

## 开始编译安装 gcc：
mkdir build && cd build
../configure --prefix=/usr/ --enable-checking=release \
--enable-languages=c,c++ --disable-multilib
make -j4 && make install
## 注意了，上面那个 prefix 必须用 usr/，以便覆盖掉旧版的 gcc ，以免编译程序找不到新版 gcc

## 检查验证 gcc 版本
gcc -v
```
```shell
##安装 ncurses-devel、bison、openssl-devel：
yum -y install ncurses-devel bison openssl-devel
```
#### 下载MYSQL源代码
##### 打开 http://mysql.com/downloads/下载页面，找到 MySQL Community(GPL)Downloads
​	还是找到 MySQL Communtiy Server，选择 "**Source Code**" 下载源码，再选择 **Generic Linux (Architecture Independent)**, 下载选择 "**Compressed TAR Archive, Includes Boost Headers**"。选择 "**Source Code**" 下载源码，再选择 **Generic Linux (Architecture Independent)**, 下载选择 "**Compressed TAR Archive, Includes Boost Headers**"，或者右键 "复制链接地址"到 Linux 上，下载

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-boost-8.0.26.tar.gz
```
#### 开始编译mysql8.0*
```shell
vim /etc/my.conf
[client]
port=3306
socket=/tmp/mysql.sock
default-character-set=utf8
#user=root
#password=123
[mysqld]
server-id=1
#skip-grant-tables
port=3306
user=mysql
max_connections=200
socket=/tmp/mysql.sock
basedir=/opt/mysql
datadir=/opt/mysql/data
pid-file=/opt/data/mysql.pid
init-connect='SET NAMES utf8'
character-set-server=utf8
default-storage-engine=INNODB
log_error=/opt/mysql/log/mysql-error.log
slow_query_log_file=/opt/mysql/log/mysql-slow.log
[mysqldump]
quick
max_allowed_packet=16M

cmake . \
-DWITH_BOOST=./boost/ \
-DCMAKE_INSTALL_PREFIX=/opt/mysql \
-DMYSQL_DATADIR=/opt/mysql/data \
-DSYSCONFDIR=/etc \
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1 \
-DWITHOUT_ARCHIVE_STORAGE_ENGINE=1 \
-DFORCE_INSOURCE_BUILD=1

## SYSCONFDIR=/ # 修改读取 my.cnf 的路径
## WITHOUT_FEDERATED_STORAGE_ENGINE=1 \ # 摘掉 FEDERATED 存储引擎
## WITHOUT_ARCHIVE_STORAGE_ENGINE=1 \ # 摘掉 ARCHIVE 存储引擎
```
#### 安装mysql8.0*：
```shell
make -j4 && make install
```
#### mysql基本设置：
```shell
##配置环境变量:
echo -e "export PATH=/opt/mysql/bin:\$PATH" >>/etc/profile
source etc/profile

##建组建用户
groupadd mysql
useradd -M -g mysql -s /sbin/nologin -d /opt/mysql mysql

##创建目录结构
mkdir -p /opt/mysql/{log,tmp}
chown mysql:mysql /opt/mysql/ -R

```

#### 初始化数据库
```shell
cd /opt/mysql/bin/
./mysqld --initialize-insecure --user=mysql --basedir=/opt/mysql --datadir=/opt/mysql/data
```
#### 配置mysql服务并启动
```shell
##从源代码中拷贝msyql.server到系统目录下：
cp /opt/sourece/msyql/support-files/mysql.server /etc/init.d/mysqld

vim /etc/init.d/mysqld
basedir=/opt/mysql
datadir=/opt/mysql/data
chmod +x /etc/init.d/mysqld
/etc/init.d/mysqld start
```

## 源码编译安装nginx:
```shell
##创建nginx用户和目录：
groupadd -r nginx
useradd -r -g nginx -s /bin/false -M nginx
id nginx

##开始编译：
./configure \
--user=nginx \
--group=nginx \
--prefix=/opt/nginx \
--sbin-path=/opt/nginx/sbin/nginx \
--conf-path=/opt/nginx/conf/nginx.conf \
--pid-path=/opt/nginx/nginx.pid \
--lock-path=/opt/nginx/nginx.lock \
--error-log-path=/opt/nginx/logs/error.log \
--http-log-path=/opt/nginx/logs/access.log \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-pcre \
--with-file-aio \
--with-http_realip_module \
--http-client-body-temp-path=/opt/nginx/client/ \
--http-proxy-temp-path=/opt/nginx/proxy/ \
--http-fastcgi-temp-path=/opt/nginx/fcgi/ \
--http-uwsgi-temp-path=/opt/nginx/uwsgi \
--http-scgi-temp-path=/opt/nginx/scgi

ln -s /opt/nginx/sbin/nginx /usr/sbin/

##开始安装：
make && make install
```
#### 设置nginx开机启动
```shell
vi /lib/systemd/system/nginx.service
[Unit]
 
Description=nginx service
 
After=network.target
 
   
 
[Service]
 
Type=forking
 
ExecStart=/opt/nginx/sbin/nginx
 
ExecReload=/opt/nginx/sbin/nginx -s reload
 
ExecStop=/opt/nginx/sbin/nginx -s quit
 
PrivateTmp=true
    
[Install]
WantedBy=multi-user.target
```
```shell
# systemctl enable nginx  // 加入开机启动
# systemctl disable nginx // 禁止开启启动
```
```shell
##以服务的方式启动：
pkill nginx   // 杀死nginx进程
systemctl start nginx

##启动/停止/重启/查看：
systemctl start nginx　        	启动服务
systemctl stop nginx　         	停止服务
systemctl restart nginx　      	重启服务
systemctl status nginx   		 查看服务当前状态
systemctl list-units --type=service      查看所有已启动的服务
```
## 编译安装zabbix：
#### 新建zabbix用户：
```shell
groupadd --system zabbix
useradd --system -g zabbix -d /opt/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix
mkdir -mu=rwx,g=rwx,o= -p /opt/zabbix
chown zabbix:zabbix /opt/zabbix
```
#### 新建zabbix数据库和用户：
```sql
msyql -uroot -p
create database zabbix character set utf8 collate utf8_bin;
create user 'zabbix'@'%' identified by 'zabbix';
grant all privileges on *.* to 'zabbix'@'%' with grant option;

##修改密码授权规则
ALTER USER 'zabbix'@'%' IDENTIFIED BY 'zabbix' PASSWORD EXPIRE NEVER;
ALTER USER 'zabbix'@'%' IDENTIFIED WITH mysql_native_password BY 'zabbix';

##刷新权限
FLUSH PRIVILEGES;
```
#### 安装依赖：
```shell
yum -y install net-snmp net-snmp-devel libssh2-devel libevent-devel pcre-devel libcurl-devel java-devel openldap-devel --nogpgcheck

##安装fping（由于zabbix使用fping替代了ping作为icmp的工具，所以还要安装 fping）
wget http://fping.org/dist/fping-5.0.tar.gz
tar zxvf fping-5.0.tar.gz && cd fping-5.0 && ./configure --prefix=/usr/local/fping
make && make install
ln -s  /usr/local/fping/sbin/fping /usr/local/sbin/ 
chown root:zabbix /usr/local/sbin/fping
chmod 4710 /usr/local/sbin/fping

##在Zabbbix上安装unixODBC
yum -y install unixODBC unixODBC-devel
##在Zabbix上安装对应数据库的unixODBC驱动,对于MySQL，安装unixODBC驱动
yum -y install mariadb-connector-odbc
```
#### 开始编译zabbix：
```shell
./configure \
--prefix=/opt/zabbix \
--enable-server \
--enable-agent \
--with-mysql=/usr/bin/mysql_config \
--with-net-snmp \
--enable-ipv6 \
--with-libcurl \
--with-ssh2 \
--with-unixodbc \
--enable-java \
--with-libxml2
```
#### 安装zabbix
```shell
make && make install
```
#### 导入zabbix初始数据：
```shell
cd /opt/zabbix/database/mysql
mysql> use zabbix;
mysql> set sql_log_bin=0;
mysql> source ./schema.sql;
mysql> source ./images.sql;
mysql> source ./data.sql;
mysql> set sql_log_bin=1;
```
#### Zabbix_server.conf 配置:
```shell
# 创建 ZabbixServer 日志目录
mkdir -p /opt/zabbix/logs

# 修改 ZabbixServer 配置文件
grep -Ev "#|^$" /opt/zabbix/etc/zabbix_server.conf
LogFile=/opt/zabbix/logs/zabbix_server.log
LogFileSize=0
DebugLevel=3
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
DBSocket=/tmp/mysql.sock
DBPort=3306

#系统报警：Zabbix poller processes more than 75% busy
#解决办法：修改此值StartPollers
StartPollers=50
StartPreprocessors=30
StartPollersUnreachable=5
StartTrappers=300

#系统报警Zabbix icmp pinger processes more than 75% busy
#解决办法：修改StartPingers此值
StartPingers=50
StartDiscoverers=50
StartHTTPPollers=50

#系统告警：More than 75% used in the configuration cache， 可用的配置缓存超过75%
#解决办法： 更改默认缓存配置的大小，默认CacheSize只有8M,可根据自己需求改大即可
CacheSize=2G
StartDBSyncers=32
HistoryCacheSize=512M
HistoryIndexCacheSize=512M
TrendCacheSize=512M

#系统告警：Zabbix value cache working in low memory mode
#定位到ValueCacheSize关键字位置，然后调高ValueCacheSize大小，根据自己环境调整
ValueCacheSize=1G
Timeout=10
AlertScriptsPath=/opt/data/zabbix/alertscripts
ExternalScripts=/opt/data/zabbix/externalscripts
FpingLocation=/usr/sbin/fping
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
```
#### Zabbix_agentd.conf 配置
```shell
vim /opt/zabbix/etc/zabbix_agentd.conf
## Passive checks related #被动检查相关配置
Server=172.18.8.8     # 指向当前zabbix server

## Option: ListenPort
ListenPort=10050  # 监听端口
### Option: StartAgents 
StartAgents=5  # 被动状态时默认启动的实例数(进程数)，为0不监听任何端口
### Option: Hostname 
Hostname=172.18.8.9  # 区分大小写且在zabbix server唯一的值
```
#### Zabbix Web前端设置
```shell
# 在安装目录将 ui 拷贝到指定的 web root：
mkdir -p /opt/nginx/html

# 从 5.0 开始，放在 ui 文件夹下
cp -a /opt/source/zabbix/ui/* /opt/nginx/ngx_web
cd /opt/nginx/html/conf
cp zabbix.conf.php.example zabbix.conf.php

# zabbix的运行权限
chown -R zabbix.zabbix /opt/nginx/html

# 修改配置文件
# 注意修改后，访问 ZabbixServer 的安装文件 setup.php 的时候将跳过环境检查，将直接进入监控页面
cat /opt/nginx/html/conf/zabbix.conf.php
<?php
// Zabbix GUI configuration file.
global $DB;
$DB['TYPE']     = 'MYSQL';
# $DB['SERVER']   = 'localhost';  # 此项配置为 localhost 在安装 ZabbixServer 的时候会报错
$DB['SERVER']	  = '127.0.0.1';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = 'zabbix';
// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';
$ZBX_SERVER      = 'localhost';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = 'Zabbix server';
$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
```
#### 启动zabbix：
```shell
 /opt/zabbix/sbin/zabbix_server -c /opt/zabbix/etc/zabbix_server.conf
 netstat -antup | grep zabbix
```
![](http://doc.thexw.cn/server/index.php?s=/api/attachment/visitFile/sign/d058f02ad84858bb5fcaa489be486fa8)
解决：
```shell
mkdir -p /var/lib/mysql　　#建立所需目录
ln -s /tmp/mysql.sock /var/lib/mysql/    #链接指向文件所在目录
```
#### Zabbix设置开机自启和自启动
```shell
# 在zabbix源码包中找到启动脚本，复制到/etc/init.d/目录下
ll /opt/source/zabbix/misc/init.d/tru64/*
-rw-r--r--. 1 sekorm sekorm 1519 Aug 23 19:50 zabbix_agentd
-rw-r--r--. 1 sekorm sekorm 1521 Aug 23 19:50 zabbix_server
cp /opt/source/zabbix/misc/init.d/tru64/* /etc/init.d/

# 修改启动脚本的配置文件
在第二行插入：# chkconfig: 2345 10 90
vim /etc/init.d/zabbix_server 
22         BASEDIR=/opt/zabbix
vim /etc/init.d/zabbix_agentd
22         BASEDIR=/opt/zabbix

# 启动zbbix_server、zabbix_agentd
/etc/init.d/zabbix_server start
Starting zabbix_server (via systemctl):                    [  确定  ]
/etc/init.d/zabbix_agentd start
Starting zabbix_agentd (via systemctl):                    [  确定  ]

# 自启动zbbix_server、zabbix_agentd
chkconfig zabbix_server on
chkconfig zabbix_agentd on
```