---
title: "zabbix_agent安装"                         
author: "雨吁"  
description : "zabbix_agent安装"    
date: 2022-06-06        
lastmod: 2022-06-06             

tags : [                                    
"centos7",
"zabbix_agent",
"安装"
]
categories : [                         
"centos7",
"zabbix_agent",
"安装"
]
keywords : [                                
"centos7",
"zabbix_agent",
"安装"
]
---

1.把zabbix.tar.gz包上传/opt目录下解压,上传agent包至zabbix目录下并修改配置

```shell
cd /opt
rz zabbix.tar.gz
tar xvf zabbix.tar.gz && rm zabbix.tar.gz
cd zabbix/
rz zabbix_agent-5.0.15-linux-3.0-amd64-static.tar.gz
tar zxvf zabbix_agent-5.0.15-linux-3.0-amd64-static.tar.gz && rm zabbix_agent-5.0.15-linux-3.0-amd64-static.tar.gz
rm -rf conf/
```
2.创建Zabbix用户及日志目录
```shell
chattr -i /etc/{passwd,group,gshadow,shadow}
useradd -s /sbin/nologin -u 550 -c "zabbix agentd" zabbix 
chattr +i /etc/{passwd,group,gshadow,shadow}
mkdir -p  /opt/zabbix
chown -R zabbix.zabbix /opt/zabbix
```

3.配置zabbix_agentd.conf，以下是配置完成后的内容：
```shell
egrep -v “(#|$)” /opt/zabbix/etc/zabbix_agentd.conf
PidFile=/opt/zabbix/log/zabbix_agentd.pid 
LogFile=/opt/zabbix/log/zabbix_agentd.log 
EnableRemoteCommands=1
Server=0.0.0.0
ListenPort=10050  
Hostname=localhost 
Include=/opt/zabbix/etc/zabbix_agentd.conf.d/ 
```
4.创建systemctl系统zabbix_agent 单元文件
```shell
[Unit]
Description=Zabbix Agent
After=syslog.target
After=network.target

[Service]
Environment="CONFFILE=/opt/zabbix/etc/zabbix_agentd.conf"
Type=forking
Restart=on-failure
PIDFile=/opt/zabbix/log/zabbix_agentd.pid
KillMode=control-group
ExecStart=/opt/zabbix/sbin/zabbix_agentd -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s

[Install]
WantedBy=multi-user.target

```
5.启动 Zabbix Agentd 客户端服务
```shell
systemctl start zabbix-agent.service
```