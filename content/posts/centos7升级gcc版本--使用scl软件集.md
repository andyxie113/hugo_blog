---
title: "CentOS7升级gcc版本"                         
author: "雨吁"  
description : "CentOS7升级gcc版本--使用scl软件集"    
date: 2022-06-06        
lastmod: 2022-06-06             

tags : [                                    
"centos7",
"gcc",
"scl"
]
categories : [                         
"centos7",
"gcc",
"scl"
]
keywords : [                                
"centos7",
"gcc",
"scl"
]
---

​	Centos7 gcc版本默认4.8.3，Red Hat 为了软件的稳定和版本支持，yum 上版本也是4.8.3，所以无法使用yum进行软件更新，所以使用scl。

​	scl软件集(Software Collections),是为了给 RHEL/CentOS 用户提供一种以方便、安全地安装和使用应用程序和运行时环境的多个（而且可能是更新的）版本的方式，同时避免把系统搞乱。

使用scl升级gcc步骤：

1.安装scl源：

```shell
yum install centos-release-scl scl-utils-build
```
2.列出scl有哪些源可以用
```shell
yum list all --enablerepo='centos-sclo-rh'
```
3.安装5.3版本的gcc、gcc-c++、gdb
```shell
yum install devtoolset-4-gcc.x86_64 devtoolset-4-gcc-c++.x86_64 devtoolset-4-gcc-gdb-plugin.x86_64 
```
4.查看从 SCL 中安装的包的列表：
```shell
scl --list 或 scl -l
```
5.切换版本
```shell
scl enable devtoolset-4 bash
```
6.使用exit 退出当前scl版本的bash环境
```shell
scl --list 或scl -l
scl --help 或 scl -h
scl enable <scl-package-name> <command>  #使用scl来执行command命令
scl enable  devtoolset-4 bash  #使用scl创建一个scl包的bash会话环境
exit  #退出当前scl bash环境，恢复成系统bash环境
```
