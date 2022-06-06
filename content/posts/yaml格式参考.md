---
title: "docker-compose yaml文件格式参考"                         
author: "雨吁"  
description : "docker-compose yaml文件格式参考"    
date: 2022-05-28        
lastmod: 2022-05-28             

tags : [                                    
"docker-compose",
"yaml",
"mysql"
]
categories : [                         
"docker-compose",
"yaml",
"mysql"
]
keywords : [                                
"docker-compose",
"yaml",
"mysql"
]
---



````yaml
version: '3'
services:
  mysql:
    image: mysql
    container_name: mysql
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ****** # root用户的密码
      MYSQL_USER: zabbix # 创建新用户
      MYSQL_PASSWORD: zabbix # 新用户的密码
    ports:
      - 3306:3306
    volumes:
      - /opt/mysql/data:/var/lib/mysql
      - /opt/mysql/conf:/etc/mysql/conf.d
      - /opt/mysql/logs:/logs
````
