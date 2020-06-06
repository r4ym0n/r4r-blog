---
title: Nginx 折腾笔记 （路由配置以及认证）
tags:
  - Linux
  - Nginx
  - OPS
url: 476.html
id: 476
categories:
  - OP之路
date: 2019-02-07 00:00:00
---

前面的话
----

> 2008年，第一款安卓手机诞生，HTC G1。智能手机的十年，像是人类的一个世纪
> 
> 在十年前的时候，互联网哪里有普及呢？
> 
> 十年之后，世界又将怎样？
> 
> 每种技能的经济价值是很快会下降的

又是拖延症爆炸的时间，这篇博文开了都自己墨迹了这么久，竟然几天之后才开始继续写，没救了。

这篇主要就是写，Nignx 的路由匹配，和一些基本的认证操作。

Nginx 的认证配置
-----------

这里使用 Nginx 的基础的身份认证功能，来实现对页面访问的授权访问。采用 `htpasswd` 来实现了鉴权。这个是 apache 的资自带的工具，这里也可以使用 `OpenSSL`来生成密码文件。

    sudo sh -c "echo -n 'admin:' >> /etc/nginx/.htpasswd"
    sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd" 

* * *

在 Nginx 的conf 文件中，在指定的路由路径的配置下面使用如下的配置语句：

    auth_basic "Authorized users only";     # 验证提示框
    auth_basic_user_file /home/.htpasswd;   # 鉴权密码文件

在最终的 Nginx 配置里的实例配置如下：

    location ^~/asd/ {
        auth_basic "Is You?";
        auth_basic_user_file /etc/nginx/.htpasswd_k;    
    
        proxy_set_header Host $host;
        proxy_pass http://localhost:5601/;
    }

这样在对目的路由进行请求的时候，会被要求用户鉴权。

至此，配置完成

* * *

2019-06-02 21:33:28 星期日 更新 为了提高安全性，应避免文件被直接访问，使用以下配置：

    location ~/\.ht {
        deny all;
    }

Nginx 路由配置
----------

`location` 在 Nginx 里面是相当常见的，用于对路由路径的匹配。

通配符在这里是一个重点，

    location [=|~|~*|^~] /uri/ { … }
    location modifier uri {...}

### 匹配类型

匹配类型

描述

=

完全匹配 斜杠都不能多

~

大小写敏感 pattern 是正则表达式

(None)

匹配相似的部分pattern 部分相似即匹配

~*

大小写不敏感 pattern 是正则表达式

^~

和 None 类似，但是一旦匹配，即停止匹配

@

只能被内部访问 同 internal？

### 匹配示例

    location ^~ /us/ {
        auth_basic "Is You?";
        auth_basic_user_file /etc/nginx/.htpasswd;
        alias /usr/share/nginx/html/us/;
    }

这里使用 `^~` 修饰符指定了是部分匹配的，但是使用了 `/xxx/` 的结构基本上也是实现了完全匹配。

    location ~ \.php$ {
        proxy_pass   http://127.0.0.1;
    }

大小写匹配的正则表达式，匹配以 `.php` 结尾的项目，转发到 fastcgi。`\.` 是转义的 `.`

    location ~* .(gif|jpg|jpeg)$ {
    }