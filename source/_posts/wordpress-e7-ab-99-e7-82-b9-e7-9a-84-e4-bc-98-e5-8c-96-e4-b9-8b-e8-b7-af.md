---
title: WordPress 站点的优化之路
tags:
  - Docker
  - Nginx
  - OPS
  - redis
  - wordpress
url: 686.html
id: 686
categories:
  - OP之路
date: 2019-04-04 00:26:50
---

前
-

上一篇已经很OK的部署了一个 wordpress 的stack跑在Swarm 的集群上面，现在还挺稳定的，比较开心。 不过为了追求更快更好的体验，和对页面体验的提升，这里就要进行大优化了。 因为描述一下现状，现在的中间的隧道节点是很可怜的 1M 带宽的机器，怎么把这个 1M 的带宽用好就是关键了。

这里先给个数据：站点的加载内容为 205k ![file](/wp-content/uploads/2019/04/image-1554225388137.png) 1M 带宽其理论上下行总和速度 ： 125k/s 理论传输时间：205/125=1.64s 实际上传输时间: 2.6s +- 0.5

> Not Special ，It's unique

Gzip 压缩
-------

GZIP 压缩，用于压缩HTTP的请求文件的大小，得到其可以在有限带宽下面的传输能力。配置方法如下，直接在Nginx的虚拟主机的 块里面进行配置，来启用Nginx 的Gzip功能。

    gzip on;
    gzip_disable &quot;msie6&quot;; # IE6 不使用GZIP
    
    gzip_vary on;
    gzip_proxied any;   
    gzip_comp_level 6;  # 压缩等级
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_min_length 256;    # 256K以下文件不压缩
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

在进行配置之后直接 reload Nginx 的配置即可生效：查看服务的响应头内容如下： ![file](/wp-content/uploads/2019/04/image-1554272652260.png) 可以得到，已经开启 Gzip 的压缩功能。这里的压缩级别为 6，实际上这里是消耗CPU的运算，如果太高，有可能反而导致变慢的情况。

参考链接：

> *   \[加速nginx: 开启gzip和缓存\]( "[https://www.darrenfang.com/2015/01/setting-up-http-cache-and-gzip-with-nginx/](https://www.darrenfang.com/2015/01/setting-up-http-cache-and-gzip-with-nginx/)")

Cache 缓存
--------

使用 Nginx 加cache 这个之前在前面的帖子已经有提到过，这里换汤不换药。下面也po出配置文件：

    proxy_cache_path /tmp/ramdisk levels=1:2 keys_zone=my_zone:64m inactive=24h max_size=64m;
    proxy_cache_key "$scheme$request_method$host$request_uri";
    
        location ~ .*\.(gif|jpg|png|html|htm|css|js|ico|swf|pdf)$ {
            #Proxy 
            proxy_set_header        Host $host;
            proxy_ignore_headers    Set-Cookie Cache-Control;   #这句代码很关键，尤其要忽略set-cookie
    
            proxy_set_header            X-real-ip $remote_addr;
            proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
    
            proxy_pass https://con; # upstream的
    
            #Use Proxy Cache
            #定义的缓存池名称
            proxy_cache my_zone;
            proxy_cache_key "$host$request_uri";   #定义缓存存放目录的子目录命名规则
            add_header Cache "$upstream_cache_status"; #缓存的HTTP头部加了一个Cache
            proxy_cache_valid  200 304 301 302 8h;
            proxy_cache_valid  404 1m;
            proxy_cache_valid  any 2d;
        }

整体不复杂，指定了静态文件的类型的正则，这些文件将不会在源站进行请求，会现在Nginx的缓存里面取。当然，这里的请求量不大，所以就使用了较小的空间 64M, 也是**热数据**的思想，这里的缓存内容是放在内存盘里面的。

* * *

配置之后，继续查看响应头： ![file](/wp-content/uploads/2019/04/image-1554273465243.png) 可以看到，这里的缓存已经是 HIT 的状态。说明配置成功。使用 `df -h` 查看缓存的分区，已经使用部分空间。

* * *

安全！ 全站HTTPS
-----------

由于 Wordpress 的管理界面是直接账号密码登陆的，所以如果直接使用 HTTP 在页面上进行登陆，还真的是有些不放心，所以这里的 HTTPS 必须是得加上。HTTPS 的服务器配置，对于 HTTPS 的配置，可以直接取标准模板里面进行拷贝，这里就不在赘述，一样的可以本站搜索 NGinx 得到相关的结果。

这里给出一段比较精巧的代码，用来实现全站的 Https 的，这里直接把 80 端口的请求进行重定向到目标站点（https）。

    server {
         listen 80 default_server;
         #listen [::]:80;
         server_name _;
         return 302 https://$host$request_uri;
         #rewrite ^(.*)$  https://$host$1 permanent;
    
         error_page 404 /404.html;
            location = /40x.html {
            }
    }
    

KCP 弱网络加速
---------

这里使用了比较 LOW 的方法，使用 FRP 把本地的主机映射到外网，由于前面使用的境外的 VPS，所以网络环境是相当的不好，请求的 TTFB 往往长达十几秒，所以必须是使用 KCP 来解决弱网络下面的问题。

关于KCP 的介绍：

> [https://github.com/skywind3000/kcp](https://github.com/skywind3000/kcp)

更快 ! redis ！
------------

没想到是在这种情况下接触到了 redis。先做一个基础的了解吧，这个是一个基于内存的非关系数据库，由于其强大的性能。多用于做 对象缓存。

由于之前 wordpress 的官方镜像是直接打包 php-fpm 的，其中默认没有 redis 的moudule。 所以需要安装组件，构建新的镜像。构建命令下面已经给出：

    FROM wordpress:latest
    ENV PHPREDIS_VERSION 3.1.3
    RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    &amp;&amp; tar xfz /tmp/redis.tar.gz \
    &amp;&amp; rm -r /tmp/redis.tar.gz \
    &amp;&amp; mkdir -p /usr/src/php/ext \
    &amp;&amp; mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    &amp;&amp; docker-php-ext-install redis \
    &amp;&amp; rm -rf /usr/src/php

构建过程的Dockerfile 参考链接：

> [https://blog.csdn.net/xxx9001/article/details/81914074](https://blog.csdn.net/xxx9001/article/details/81914074)

* * *

在镜像本身有了 php 的 redis 插件之后，我们需要在我们的stack里面添加redis的服务。所以我们修改 compose文件，添加redis 的配置:

       redis:
         image: redis
         restart: always

之后直接进行更新这个 stack 即可。连接时候默认的`host:port` 是 `redis:6379`。

* * *

在wordpress里面直接进行redis的插件安装即可。 这里使用的是 `LiteSpeed Cache` ![file](/wp-content/uploads/2019/04/image-1554307607046.png)

之后直接使用命令使用 shell 连上 redis 的容器： `docker exec -i -t mynginx /bin/bash redis-cli` 之后来使用 `keys *` 来显示所有的缓存键。

后
-

经过一番折腾之后，站点，也就是本站的访问体验也算是得到了较大程度的加强，使用 Google 的 pagespeed-insight 进行测速：

> \[PageSpeed Insights\]( [https://developers.google.com/speed/pagespeed/insights/?hl=zh-cn](https://developers.google.com/speed/pagespeed/insights/?hl=zh-cn))

得到的评分已经达到了70+的水平，在TTFB达到 2-3 秒的时间下，算是比较OK的了，还是比较开心的。 至此，优化的操作到此告一段落啦

> with man nothing is possible but with god all things are possible