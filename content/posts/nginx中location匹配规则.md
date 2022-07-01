---
title: "NGINX中location匹配规则"                         
author: "雨吁"  
description : "NGINX中location匹配规则"    
date: 2022-07-01        
lastmod: 2022-07-01             

tags : [                                    
"Linux",
"NGINX",
"location"
]
categories : [                              
"Linux",
"NGINX",
"location"
]
keywords : [                                
"Linux",
"NGINX",
"location"
]
---

## 语法规则

```shell
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```

​	语法规则很简单，一个`location`关键字，后面跟着可选的修饰符，后面是要匹配的字符，花括号中是要执行的操作	

### 修饰符

- `=` 表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中
- `~` 表示该规则是使用正则定义的，区分大小写
- `~*` 表示该规则是使用正则定义的，不区分大小写
- `^~` 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找

## 匹配顺序

- 首先精确匹配 `=`
- 其次前缀匹配 `^~`
- 其次是按文件中顺序的正则匹配
- 然后匹配不带任何修饰的前缀匹配。
- 最后是交给 `/` 通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

*注意：前缀匹配，如果有包含关系时，按最大匹配原则进行匹配。比如在前缀匹配：`location /dir01` 与 `location /dir01/dir02`，如有请求 `http://localhost/dir01/dir02/file` 将最终匹配到 `location /dir01/dir02`*

## 匹配过程

```shell
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /user/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

请求`/`精准匹配A，不再往下查找。

请求`/index.html`匹配B。首先查找匹配的前缀字符，找到最长匹配是配置B，接着又按照顺序查找匹配的正则。结果没有找到，因此使用先前标记的最长匹配，即配置B。

请求`/user/index.html`匹配C。首先找到最长匹配C，由于后面没有匹配的正则，所以使用最长匹配C。
请求`/user/1.jpg`匹配E。首先进行前缀字符的查找，找到最长匹配项C，继续进行正则查找，找到匹配项E。因此使用E。

请求`/images/1.jpg`匹配D。首先进行前缀字符的查找，找到最长匹配D。但是，特殊的是它使用了`^~`修饰符，不再进行接下来的正则的匹配查找，因此使用D。这里，如果没有前面的修饰符，其实最终的匹配是E。大家可以想一想为什么。

请求`/documents/about.html`匹配B。因为B表示任何以`/`开头的URL都匹配。在上面的配置中，只有B能满足，所以匹配B。

## `location @name`的用法

​	@用来定义一个命名location。主要用于内部重定向，不能用来处理正常的请求。其用法如下：

```shell
location / {
    try_files $uri $uri/ @custom
}
location @custom {
    # ...do something
}
```

​	上例中，当尝试访问url找不到对应的文件就重定向到我们自定义的命名location（此处为custom）。

**此时，命名location中不能再嵌套其它的命名location**

## URL尾部的`/`需不需要

关于URL尾部的`/`有三点也需要说明一下。第一点与location配置有关，其他两点无关。

1. location中的字符有没有`/`都没有影响。也就是说`/user/`和`/user`是一样的。
2. 如果URL结构是`https://domain.com/`的形式，尾部有没有`/`都不会造成重定向。因为浏览器在发起请求的时候，默认加上了`/`。虽然很多浏览器在地址栏里也不会显示`/`。这一点，可以访问[baidu](https://link.segmentfault.com/?enc=ggDBDDpQc0guXp6wBaPEMw%3D%3D.5qC6L96U%2BoIEwAQL9AO7fFt1Cbzsss0gzx4U3%2B3WbL4%3D)验证一下。
3. 如果URL的结构是`https://domain.com/some-dir/`。尾部如果缺少`/`将导致重定向。因为根据约定，URL尾部的`/`表示目录，没有`/`表示文件。所以访问`/some-dir/`时，服务器会自动去该目录下找对应的默认文件。如果访问`/some-dir`的话，服务器会先去找`some-dir`文件，找不到的话会将`some-dir`当成目录，重定向到`/some-dir/`，去该目录下找默认文件。可以去测试一下你的网站是不是这样的。

详细参考链接：

[location 匹配规则 · OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/nginx_local_pcre.html)

