---
title: "NGINX配置http强制跳转https"                         
author: "雨吁"  
description : "NGINX配置http强制跳转https"    
date: 2022-07-01        
lastmod: 2022-07-01             

tags : [                                    
"Linux",
"NGINX"
]
categories : [                              
"Linux",
"NGINX"
]
keywords : [                                
"Linux",
"NGINX"
]
---

Nginx配置http强制跳转https，分两种情况，一种是标准80/443端口跳转，而另一种是非80/443端口的http强制跳转https。

下面对这两种情况进行具体配置：

## 1.标准端口跳转配置

​	具体方法是采用rewrite方式

```shell
server{
         listen 80;
         server_name xxxxxx.com;   
         #http强制跳转https
         rewrite ^(.*)$  https://$host$1 permanent;
}
#$host可以换成${server_name}
```

​	完整的配置文件：

```shell
#以下属性中，以ssl开头的属性表示与证书配置有关。
server {
    listen 443 ssl;
    #配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    server_name xxxxxx.com; #需要将xxxxxx.com替换成证书绑定的域名。
    root html;
    index index.html index.htm;
    ssl_certificate cert/xxxxxx.com.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key cert/xxxxxx.com.key; #需要将cert-file-name.key替换成已上传的证书私钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # 防止爬虫抓取
    if ($http_user_agent ~* "360Spider|JikeSpider|Spider|spider|bot|Bot|2345Explorer|curl|wget|webZIP|qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot|NSPlayer|bingbot")
    {
        return 403;
    }
    error_page   403 404 500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html/error-page;
    }
}

server {
    listen 80;
    server_name xxxxxx.com; #这里修改成自己的域名
 
   #核心代码
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}

```

## 2.非标准端口跳转配置

​	nginx增加以下配置

```shell
proxy_set_header Host $host:$server_port;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

​	完整配置文件

```shell
upstream web {
    server 127.0.0.1:8080;
}
server {
    access_log  logs/access.log;
    error_log   logs/error.log;
    index       index.html;
    root        /usr/local/nginx/html;
    server_name xxx.com;
    listen 8000 ssl;
    ssl on;
    ssl_certificate      ssl.cer;
    ssl_certificate_key  ssl.key; 
    ## Only allow these request methods
    if ($request_method !~ ^(GET|HEAD|POST)$ ) {
        return 444;
    }
    location / {
        proxy_pass  http://web
        proxy_ignore_headers   Expires Cache-Control;
 
        proxy_set_header        Host            $host:$server_port;
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
    } 
    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```
