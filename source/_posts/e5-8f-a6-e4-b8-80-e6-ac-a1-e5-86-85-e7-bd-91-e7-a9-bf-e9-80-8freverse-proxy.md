---
title: 另一次内网穿透(reverse proxy)
url: 468.html
id: 468
categories:
  - 未分类
date: 2019-02-21 00:00:00
tags:
---

前
-

很快这就是第二篇了，上篇信誓旦旦说的更好的方案，实际上，这里较比而已只是有了一点点改进XD

上一篇：[愚蠢的内网穿透方案(tunnel)](https://blog.diglp.xyz/2019/02/20/OP_%E6%84%9A%E8%A0%A2%E7%9A%84%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F%E6%96%B9%E6%A1%88(tunnel)/) <\-\- 在此

其目标和目的是一样的，实现页面的内网穿透，以及内网主机的反代。

方案以及可行性
-------

### 可行性

如上面所说，实际上分了两个部分，内网穿透，以及反向代理。这里实际上有点混淆了一个概念。反向代理实际上是内网穿透的一个实现过程。之于正向代理不同，反向代理是代理服务器在上次，有请求之后，会把请求转发都后排的客户端，所以这就是反向代理的过程。

那么内网穿透呢？内网穿透是在反向代理的功能上更多了一层。上面提到反向代理，在有连接的时候，会把它直接转发到后面的客户端。但是在穿透这个情况下，内网主机是**无法被公网主机所访问**，从而无法传递请求了。所以我们在反向代理的前提下，**使用客户端向服务器发起TCP长连接**，这样就打通了，公网主机到内网主机之间的通道。从而实现了内网的穿透。

> 内网穿透 = 反向代理 \+ 正向连接

* * *

下面又是一个反向代理，这个和前面不一样，是七层代理（也就是应用层）HTTP 的七层代理。对Http的请求进行转发。这个转发和上面不同。转发的服务是运行在内网的主机上的。其作用是对公网穿透的主机请求进行到同一内网的主机的转发。

这里就是实现的是对HTTP的请求的转发。转发主机与lb主机是在同一个可以访问的网段内的。

### 方案

之于上次的傻傻的方法所不同，这次使用现成的轮子了，内网穿透这里直接使用 `FRP` （Fast reverse proxy）实现，FRP是一个简单易用的内网穿透工具。在公网主机上进行配置。实现内网主机的透传连接。把你穿透到公网主机的开放端口。实现内网主机在公网上的端口访问。

内网主机使用 `Nginx` 对目标主机进行 反代，并且把web端口开放在 `FRP` 的转发上。

至此方案设计完毕

部署过程
----

### FRP的部署

[FRP项目Readme](https://github.com/fatedier/frp/blob/master/README_zh.md) 这里面对工具的部署和使用做了详尽的说明，我们直接下载对应架构的 FRP 进行使用即可。其中的功能很多。这里我们只需要用到其中的 TCP 透传的功能。

    git clone https://github.com/fatedier/frp.git

之后直接 make ，就OK了，当然也是推荐使用现有的 Release的版本。

### Nginx 部署

Nginx 是在大多数的发行版里默认带有的组件，简单性能轻量。

### FRP配置 以及测试

#### FRP 配置

这里只是用到了 FRP 的内穿功能，通过简单的配置文件实现：

服务端：

    [common]
    server_addr = 0.0.0.0
    server_port = *000
    
    token = *******
    
    allow_ports = 2000-3000,3001,3003,4000-50000

配置简单通俗易懂，配置监听端口，以及连接 token、

* * *

客户端：

    [common]
    server_addr = ****.diglp.xyz
    server_port = *000
    token = *******
    [web]
    type = tcp
    local_ip = 127.0.0.1
    local_port = 443
    remote_port = ****

这里是客户端的配置文件，比较核心的配置在其指定本地端口以及远程端口的两行。指本地的localhost的443端口透传到远程主机的 `****` 端口。

#### FRP 测试连接

服务端部署，使用 Nohup 使其在后台运行，当然可以注册一个 supervisor。

    ➜  ~ nohup ./frps -c frps_full.ini --log_file frps.log &

客户端连接，运行：

    nohup ./frpc -c frpc_wan.ini &

* * *

测试连接，这里使用 `nc -l` 对端口进行监听：

客户端：

    nc -l 443

服务端：

    curl -XGET localhost:**80

以此模拟一个 GET 请求。之后在客户端得到如下结果，说明透传成功：

    root@vm:~/qspace $ nc -l 443
    GET / HTTP/1.1
    User-Agent: curl/7.29.0
    Host: localhost:**80
    Accept: */*

* * *

或者：客户端

    root@vm:~/qspace $ sudo python -m SimpleHTTPServer 443
    Serving HTTP on 0.0.0.0 port 443 ...
    127.0.0.1 - - [20/Feb/2019 23:29:47] "GET / HTTP/1.1" 200 -
    

服务端：

    ➜  frp_0.23.2 curl -XGET localhost:**80
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
        <title>Directory listing for /</title>
        <body>
            <h2>Directory listing for /</h2>
            <hr>
                <ul>
                    <li><a href="frp_0.24.1_linux_arm/">frp_0.24.1_linux_arm/</a>
                    <li><a href="frp_0.24.1_linux_arm.tar.gz">frp_0.24.1_linux_arm.tar.gz</a>
                </ul>
            <hr>
        </body>
    </html>
    

至此连接测试完成。FRP 功能测试正常。

* * *

更新:

这里可以注册为 **服务** :

    # 复制文件
    cp frps /usr/local/bin/frps
    mkdir /etc/frp
    cp frps.ini /etc/frp/frps.ini
    
    # 编写 frp service 文件，以 centos7 为例
    vim /usr/lib/systemd/system/frps.service
    # 内容如下
    [Unit]
    Description=frps
    After=network.target
    
    [Service]
    TimeoutStartSec=30
    ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini
    ExecStop=/bin/kill $MAINPID
    
    [Install]
    WantedBy=multi-user.target
    
    # 启动 frp 并设置开机启动
    systemctl enable frps
    systemctl start frps
    systemctl status frps

这里参考了这篇文章

> [https://mritd.me/2017/01/21/use-frp-for-internal-network-wear/](https://mritd.me/2017/01/21/use-frp-for-internal-network-wear/)

Nginx 配置以及测试
------------

这里用到了 Nginx 的反代功能，在写正确的配置之时，后面也写自己在过程中的尝试和想法XD。

这里直接把配置贴上来了就

    server {
        listen       443 ssl default_server;
        server_name  _;
        root         /usr/share/nginx/html;
    
        ssl_certificate     /etc/nginx/ssl/mine.diglp.xyz.crt;
        ssl_certificate_key /etc/nginx/ssl/mine.diglp.xyz.key;
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
    
        # Load configuration files for the default server block.
        location = / {
            #internal;
            #return 403;
            empty_gif;
        }  
        error_page 404 /404.html;
            location = /40x.html {
        }
    
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    
    server {                                                                     
        listen       443 ssl;                             
        server_name  xxxk.****.diglp.xyz;                                            
        root         /usr/share/nginx/html;                                     
    
        ssl_certificate     /etc/nginx/ssl/mine.diglp.xyz.crt;
        ssl_certificate_key /etc/nginx/ssl/mine.diglp.xyz.key;
        ssl_session_cache shared:SSL:1m;                                        
        ssl_session_timeout  10m;                                               
        ssl_ciphers HIGH:!aNULL:!MD5;                                           
        ssl_prefer_server_ciphers on;                                           
    
        location / {
            proxy_pass http://172.***.64.***;
        }
    } 
    
    server {                                                                     
        listen       443 ssl ;                             
        server_name  bkjw.****.diglp.xyz;                                            
        root         /usr/share/nginx/html;                                     
    
        ssl_certificate     /etc/nginx/ssl/mine.diglp.xyz.crt;
        ssl_certificate_key /etc/nginx/ssl/mine.diglp.xyz.key;
        ssl_session_cache shared:SSL:1m;                                        
        ssl_session_timeout  10m;                                               
        ssl_ciphers HIGH:!aNULL:!MD5;                                           
        ssl_prefer_server_ciphers on;                                           
    
        location / {
            proxy_pass http://172.***.13.***;
        }                                                                         
    }              
    

关于这里的说明：

*   由于域名没有备案，所以随随便便弄个自签证书，顶上去，先用着了。所以会显示不安全
*   配置文件从 模板 copy-paste 上来的，所以有些冗长
*   这里还撞上了名字服务的问题

* * *

配置完成后使用

    /usr/sbin/nignx -s reload 
    

对其进行热重启就好了。理论上问题不大。

系统测试
----

这个架构十分的简单，也没有什么问题可言吧。好像是的，直接公网访问。即可

可用性保证
-----

由于条件问题，内网客户端并不能保证稳定以及可用性，为了可用性的保证，这里使用 云拨测，对目标 URL 进行定时请求，当有服务不可用时，及时的进行告警。

> [腾讯云-云拨测](https://console.cloud.tencent.com/cat/availTaskList) 现在是免费使用 Recommend

过程问题笔记
------

这部分，没什么太大营养。。。

### rewrite

其实，在这整个搭建的过程中，遇到了一个很大的问题。什么呢？就是通过名字对页面进行访问和区分。比如 `adb.com/1` 和 `abc.com/2` **分别转发到不同的**。乍一看是很好实现的，直接 rewrite 就好了比如这样：

    location ^~ /phpMyAdmin {
        root /var/services/web;
        rewrite ^(.*)$ /phpMyAdmin/disabled.html break;
    }
    

这样的确可以实现隐式的重定向，但是这里的核心问题，**不是同源**。所以这个方法直接 pass 掉了。

* * *

### 直接proxy_pass的问题

    location /bkjw/ {
        proxy_set_header Host $host;
        proxy_pass http://172.*.13.*;
        #proxy_pass http://bkjw.guet.edu.cn;
    }
    

这样的配置看似是没有问题，但是在测试的时候，会发现非常非常多的 `404`。具体什么情况呢？因为页面的请求是相对与我们的路由路径 `/bkjw/` 来加载静态文件的。所以请求的路径是 `abc.com/bkjw/` 直接导致了静态文件的 **404**。

那么就想嘛，我把静态文件单独代就行了？所以有这样的：

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css|html)$
    {
        proxy_pass http://172.*.13.*;
        expires      30d;
    }
    

显然这样也是不行的，有悖了初衷，还是占用了 `/`。

### Set-Cookie 和 referer 的脑洞

那么归根结底，我们要确认的是这个静态文件的请求是从哪里来的。于是就想到了 referer。

    location / {
        if ($http_referer ~* '.*/bkjw/' ){
            proxy_pass http://172.#.13.#;
        }
        return 403;
    }
    

这样一试，还可以！打开了首页里面所有的东西都加载出来了。不过问题又出现了，在打开第二个页面的时候，`referer` 已经发生了变化，同样导致大量 404。

* * *

这样，我就给用户一个标识，在请求资源的时候我就可以判断啦。这里就出现了 `set-cookie` 的方法。希望通过cookie的方式给用户一个标识符，说明其源站从哪里来。配置如下：

    location /bkjw/ {
        #add_header Set-Cookie 'bussid=bkjw' ;
        proxy_pass http://172.16.13.22/;
    }
    
    location / {
        if ($cookie_{bussid} ~* 'bkjw' ){
            proxy_pass http://172.#.13.#;
        }
        return 403;
    }
    

结果发现，并没有什么用。F12 看了看，发现了奥秘，**请求静态资源的时候是不进行Cookie传递的**，一想也是，这样才安全嘛。

> 这里回顾一个经典的 phishing 的手段，在邮件中插入一个外部图片，透过请求图片的请求头，获得已读回执，和基本的系统状态

* * *

至此没有办法，只能使用 `server_name` 来对其进行区分，最终的配置如上上的示例所示。显得不是那么有美感了。

后面的话
----

> 人工智能的发展一旦上路了将会是飞速的，不要想着如何对付和人一样聪明的电脑，要么它永远不如你，要么它就把你远远的甩在身后。和你齐头并进只不过是一个瞬间而已。