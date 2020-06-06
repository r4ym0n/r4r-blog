---
title: Nginx 折腾笔记（prefix和basePath）
tags:
  - Nginx
  - OPS
  - proxy
url: 1048.html
id: 1048
categories:
  - OP之路
date: 2019-06-07 23:45:22
---

为每一个服务直接注册一个子域名，虽然干净利落，但是肯定没有直接改Nginx的配置来的快。想实现的功能呢？就是直接通过不同的路由路径直接进行不同端口的反向代理，但是经过尝试之后，发现，并没这么简单

前面的尝试
-----

在前面进行的尝试的诶之代码如下，看起来很简单，但是实际上问题是很大的，在这里的尝试时候会发现，在进行第一级路由的时候，的确是可以进行正确的代理和显示，

    location /wiki/ {
        proxy_pass http://127.0.0.1:xxxx/
        proxy_redirect  off;
        proxy_set_header  Host  $host;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    }

问题发生在第二次的点击的时候，由于我们的应用默认的是使用的 `/` 作为**basePath**，所以在进行第二次点击的时候，就会出现404的问题，因为根本没有被正确的代理嘛。

怎么解决呢？
------

所以，发现这个问题的确是绕不开的了。所以，需要修改我们的代理到的应用的支持，在应用的配置文件中理应有`bashPath` 或者 `prefix` 的配置，保持配置和nginx的配置的内容保持一致应该就可以了。

所以，综上，无法通过简单的修改nginx的配置的方法来实现对一个应用的反代，必须保持应用的`prefix`和配置一才OK。

* * *

所以目前最简单的方法，还是进行子域名的配置，使用server_name 来进行不同业务的分析的依据才可以。