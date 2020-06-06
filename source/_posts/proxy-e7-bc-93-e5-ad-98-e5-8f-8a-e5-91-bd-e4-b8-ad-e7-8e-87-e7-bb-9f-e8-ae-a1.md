---
title: Nginx Proxy缓存 及 命中率统计
tags:
  - cache
  - Nginx
  - proxy
url: 464.html
id: 464
categories:
  - OP之路
date: 2019-02-28 00:00:00
---

前
-

车能跑之后，就希望跑的快点对吧？所以就想办法开始对其性能进行优化。 这里就使用 CDN的思想，对代理的静态文件进行缓存，实现访问速度的提升。

所以这篇的主要目的，就是配置主机的 CDN 功能，实现对页面的静态文件缓存。

proxy_cache 配置
--------------

由于Nginx里面默认是编译了 Cache 的功能， 所以可以很方便的通过配置，来实现功能。这里直接贴出conf的内容。这里需要指定缓存空间，以及缓存配置。

    proxy_cache_path /tmp/ramdisk levels=1:2 keys_zone=my_zone:64m inactive=24h max_size=64m;
    proxy_cache_key "$scheme$request_method$host$request_uri";

这里是在 HTTP 域里面进行的缓存空间的配置。

* * *

下面，就是通过正则匹配来实现，不同静态资源的统一缓存。

    location ~ .*\.(gif|jpg|png|html|htm|css|js|ico|swf|pdf)$ {
        # 部分不需要走缓存
        # proxy_cache_bypass  $http_cache_control;
        proxy_set_header            Host $host;
        proxy_ignore_headers Set-Cookie Cache-Control;   
        proxy_set_header            X-real-ip $remote_addr;
        proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
    
        proxy_pass https://frpcon;
    
        #Use Proxy Cache
        proxy_cache my_zone;    # 缓存空间指定
        proxy_cache_key "$host$request_uri";    # 键存储方式  
        add_header Cache "$upstream_cache_status"; # 返回头添加字段，说明命中状态
        proxy_cache_valid  200 304 301 302 8h;  # 不同返回码的有效时间
        proxy_cache_valid  404 1m;
        proxy_cache_valid  any 2d;
    }

> [nginx proxy cache配置参数解读](https://segmentfault.com/a/1190000012680453) 这里是一篇 相关配置参数的说明

* * *

如上配置完成之后，reload server ，缓存功能理应启动。

* * *

### **于 2019-11-09 17:33:58 星期六 重大补充说明**：

上面的配置可能出现配置之后，缓存文件夹为空的情况。**一定要注意这里的proxy_cache** 不能有下划线。所以上面的命名修正为：

    proxy_cache_path /tmp/ramdisk levels=1:2 keys_zone=myzone:64m inactive=24h max_size=64m;

### **于2019-11-11 00:21:09 星期一 补充**

当下级服务器出现错误返回码时，默认使用缓存来提升容错能力

        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;

热数据思想
-----

冷热数据分离的思想，常伴在身，虽然这里的请求量是微乎其微，不过还是建一个概念功能。Cache 的请求量一定是非常高的，所以才会被 cache。涉及到IO的时候，最快的设备可能就是我们的内存了。

所以这里直接使用**内存盘** 对数据来进行缓存，这样的化，大大的降低了物理磁盘的IO量，而且也大大提升了缓存性能。

* * *

在linux下实现一个内存盘是相当的简单：

    mkdir /tmp/ramdisk  # 创建挂载点
    mount  -t tmpfs -o size=64m  myramdisk /tmp/ramdisk # 实现内存盘挂载
    
    ####### 测速 #####
    ➜  /tmp sudo dd if=/dev/zero of=/tmp/ramdisk/zero bs=4k count=100
    100+0 records in
    100+0 records out
    409600 bytes (410 kB) copied, 0.000243897 s, 1.7 GB/s

这里顺便进行了测速，可见速度还是相当的满意的。至此内存盘的配置完毕。

* * *

当然如果要实现自动挂载，需要在 `fstab` 里面进行配置.

    myramdisk   /tmp/ramdisk    tmpfs   defaults    0   0

至此配置完毕。

命中率统计
-----

### 日志配置

关于命中率的统计这里还是通过在日志里面实现的。和之前的一篇一样，通过 shell 脚本对日志进行提取得到我们需要的数据。

不过在这里，需要对日志个事进行设定，使其能提供我们需要的信息：

    log_format  proxy   '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"'
                        '"$request_time" "$upstream_response_time" "$upstream_cache_status" "$upstream_addr"';
    
    ...
    location / {
        ...
        access_log xxxx proxy;  # 这里对格式进行指定。
    }

这里在我们的HTTP域里面自定义日志类型

### 统计脚本

学好 awk ，走遍天下都不怕。。。

    awk '{if($(NF-1)=="\"HIT\"") hit++} END {printf "%.4f%",hit/NR*100}' /var/log/nginx/xxx.log

这里实现了对缓存命中率的统计。后面可以直接合并入之前的接口之中即可。

后
-

慢慢装起来一辆能跑快的车，还是不错的。

更新
--

如果遇到了在挂载缓冲分区之后，明明是有目录的e，但是nginx报错：

    2019/05/04 21:56:26 [notice] 20504#0: signal process started
    2019/05/04 21:56:26 [emerg] 4284#0: mkdir() "/tmp/ramdisk" failed (2: No such file or directory)
    2019/05/04 22:02:39 [notice] 21522#0: signal process started
    2019/05/04 22:02:39 [emerg] 4284#0: mkdir() "/tmp/ramdisk" failed (2: No such file or directory)
    2019/05/04 22:03:14 [emerg] 21628#0: "proxy_cache" zone "my_zone" is unknown in /etc/nginx/nginx.conf:96
    2019/05/04 22:03:54 [notice] 21743#0: signal process started
    2019/05/04 22:03:54 [emerg] 4284#0: mkdir() "/tmp/ramdisk/" failed (2: No such file or directory)
    2019/05/04 22:04:19 [notice] 21827#0: signal process started

这里重启一下Nginx即可，注意是重启，不是reload。应该是一个小BUG