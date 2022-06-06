---
title: "MinIO Client配置与使用"                         
author: "雨吁"  
description : "MinIO Client配置与使用"    
date: 2022-05-29        
lastmod: 2022-05-29             

tags : [                                    
"minio",
"mc",
"minIO client"
]
categories : [                         
"minio",
"mc",
"minIO client"
]
keywords : [                                
"minio",
"mc",
"minIO client"
]
---

​	MinIO客户端mc命令行工具提供了一种现代的替代UNIX命令， 如 ls、cat、cp、mirror和diff支持文件系统和兼容Amazon S3 的云存储服务（AWS Signature v2和v4）。
基本语法格式：

```shell
mc [FLAGS] COMMAND [ARGUMENTS...] [COMMAND FLAGS | -h]
```
### 快速安装
#### Linux环境：
​	添加一个 temporary 扩展 用于运行 mc 实用程序的 PATH。或者，执行 mc 通过导航到父文件夹和 运行 ./mc --help
```shell
curl https://dl.minio.org.cn/client/mc/release/linux-amd64/mc \
  --create-dirs \
  -o $HOME/minio-binaries/mc

chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/

mc --help
```
### 初始化配置
​	使用 mc alias set 添加 Amazon S3 兼容服务的命令 到 mc configuration.
egs:

```shell
MinIO Server：
mc alias set myminio https://minioserver.example.net ACCESS_KEY SECRET KEY

AWS S3:
mc alias set myS3 https://s3.amazon.com/endpoint ACCESS_KEY SECRET KEY

Google Cloud Storage:
mc alias set myGCS https://storage.googleapis.com/endpoint ACCESS_KEY SECRET KEY

```
​	mc使用json格式的配置文件，默认路径：~/.mc/config.json,可以使用--config-dir指定配置文件
### 测试连接
```shell
mc admin info myminio
```
如果成功，该命令将返回有关 S3 服务的信息。 如果不成功，请检查以下各项:
- 指定的 ACCESSKEY 和 SECRETKEY是否有S3服务的权限
- 主机已连接到 S3 服务 URL (i.e. using ping or traceroute)

### 命令参考

| Command  | Description  |
| :------------ | :------------ |
| mc alias | 在S3 兼容服务上运行的命令，为该服务指定别名  |
|mc cp|将数据从一个或多个源复制到目标 S3 兼容服务|
|mc find | 查询S3兼容的对象主机,在文件系统上搜索文件|
|mc mv|将数据从一个或多个源移动到目标 S3 兼容服务|
|mc mb|在指定路径对于S3兼容服务上的目标,mc mb 创建一个新的存储桶.对于文件系统上的目标，mc mb有等效于mkdir -p 的功能|
|mc rb|删除目标 S3 兼容服务上的存储桶及其所有内容|
|mc rm|删除目标 S3 兼容服务上的对象|
官方参考文档：
[MinIO Client (mc)](http://docs.minio.org.cn/minio/baremetal/reference/minio-cli/minio-mc.html "MinIO Client (mc)")

