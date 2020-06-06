---
title: Nginx 折腾笔记（LB 和 Tunnel）
url: 480.html
id: 480
categories:
  - 未分类
date: 2019-01-20 00:00:00
tags:
---

前面的话
----

突然又是折腾了一波nginx，其中一点点的配置和过程在这里记录一下。

下午又把 《社交网络》这部片看了一遍，又是一种熟悉的热血沸腾的感觉。

思考自己仿佛也是在同样的年龄，但是，为什么却是那种的遥不可及的的感觉，

也许这个就是文化吧。一定去西海岸！

> drop the The, just Facebook

* * *

1.  这世道，绅士做不了。海盗才是王。
2.  签任何协议之前至少看3遍以上。
3.  出卖你的往往是你最好的朋友

之前的场景
-----

内网不通，怎么打洞，使得外网可以访问。内网打洞。

原始方案，两次端口转发，加上 VPN。这样实现协议栈倒是没有加其他的东西，都团原生的。

后面 想想，使用 `frp` 或者 `ngrok` 直接实现内网的转发，到还是一个更好的方法了。之前的只是赞成做了个理论性的尝试，没想到，ei？还真是能用。

* * *

方案的两次转发，主要在于，不同网段之间的转发。

1.  由服务器 外网的网段， 转发到 VPN 的子网网段的内网主机 IP
2.  在内网主机上，由VPN的子网网段，转发到内网网卡
3.  并且设置内网的网卡路由走内网网关

* * *

Nginx Port forwarding and Load Balance
--------------------------------------

    stream {
        server {
            listen 80;
            proxy_pass 127.0.0.1:7000;
        }
    }

配置文件很容易的实现了 **80到7000端口** 的转发，

下面的配置很容易实现转发的 **LB** 这里就会有很好的用处

    stream {
        server {
            listen 80;
            proxy_pass frp;
        }
        upstream frp{
            server 127.0.0.1:7000;
            server 127.0.0.1:8000;
        }
    }

* * *

这里一个point 是 **七层均衡** 还是 **四层均衡**。这里的层指的是在协议栈中的层级 ，OSI 的七层结构，其中的七层均衡指的就是在应用层实现的（HTTP），四层就是在传输层（tcp）。

这里的配置段是存在于 stream 的不是在 Http中。

* * *

这里本来有个突然很荒谬的想法：能不能使用 server_name 用来对不同的 **referer** 进行都不同端口的转发。

（事实上，当然可以通过虚拟主机，在同一个端口上，通过 server_name 字段来解析到不同的端口，配置如下）

    upstream baidu{ 
        server 127.0.0.1:8081; 
    } 
    
    upstream google { 
        server 127.0.0.1:8082; 
    } 
    
    server { 
        listen 80; 
        server_name www.baidu.cn baidu.cn;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        location / { 
            proxy_pass http://baidu; 
        } 
    } 
    
    server { 
        listen 80; 
        server_name www.google.cn google.cn;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        location / {
            proxy_pass http://google; 
        } 
    } 

这样就可以，实现在同一端口的来自已不同站点的请求，分配到各自不同的 server。

* * *

当时就想，能不能 我的 Frp 连接也可以实现这样的域名绑定呢？当时有点懵，显然回答是不行的。

回到**Domain Name** 的解析原理，最终是在 Dns server 解析成为真实的 IP 。在进行 HTTP 的forward 的时候，其源站实际上是包含在 HTTP 的请求包里面的，所以，Nginx 可以对其不同的源站解析的不同的端口。

那么这里回到前面 的情况，如果是使用了nginx 来对一个 tcp 连接进行分发。可能实现吗？

不可能，因为，根本无法获得源站的地址呀。这里就是问题所在了。

关于 Nginx LB 的配置
---------------

这里就顺便，巩固一下 LB 的相关配置：

### Nginx 的 LB 类型

**nginx 的 upstream目前支持 4 种方式的分配** 引用自（[https://www.cnblogs.com/microtiger/p/7623858.html](https://www.cnblogs.com/microtiger/p/7623858.html)）

1.  轮询（默认）

　　每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，**能自动剔除**。

1.  weight

　　指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

1.  ip_hash

　　每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

1.  fair（第三方）

　　按后端服务器的响应时间来分配请求，响应时间短的优先分配。 **这个感觉不错，后面可以试试**

1.  url_hash（第三方）

* * *

ip\_hash是容易理解的，但是因为仅仅能用ip这个因子来分配后端，因此ip\_hash是有缺陷的，不能在一些情况下使用：

nginx不是最前端的服务器。ip_hash要求nginx一定是最前端的服务器，否则nginx得不到正确ip，就不能根据ip作hash。譬如使用的是squid为最前端，那么nginx取ip时只能得到squid的服务器ip地址，用这个地址来作分流是肯定错乱的。

这里就最好使用 Url_hash 的方式，根据 不同的 Url 进行分流。

* * *

不过这里又有了一个严重的问题 ：**Session 和 Cookie**，这里直接贴出文章。本来想总结一下 ， 已经有写好的了，自己也学习一下

> [cookie 和session 的区别详解](https://www.cnblogs.com/shiyangxt/articles/1305506.html)

cookie 和session 的区别：

1.  **cookie数据存放在客户的浏览器上，session数据放在服务器上**
    
2.  cookie不是很安全，**别人可以分析存放在本地的COOKIE**并进行COOKIE欺骗考虑到安全应当使用session。
    
    httpOnly ：JS 无法读取到 cookie
    
3.  session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用COOKIE。
    
4.  单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。
    
    **这里比较好玩， 可以将 Cookie 设置为 大于 4096 的kv，这样的话，将会直接导致 客户端的 Deny**
    
5.  所以个人建议：
    
    将登陆信息等重要信息存放为SESSION （存在 Session 注入的问题？） 其他信息如果需要保留，可以放在COOKIE中
    

知识树
---

这个比较好玩， 以后每次类似的 blog 的时候就加上 吧，做个自己的思路方向的记录：

*   nginx 的 LB
    *   通过 域名 的TCP 转发可行性
    *   nginx 的 LB 的种类
    *   IP_hash 的问题
    *   URL_hash 的 session问题
        *   session和Cookie
        *   httpOnly

* * *