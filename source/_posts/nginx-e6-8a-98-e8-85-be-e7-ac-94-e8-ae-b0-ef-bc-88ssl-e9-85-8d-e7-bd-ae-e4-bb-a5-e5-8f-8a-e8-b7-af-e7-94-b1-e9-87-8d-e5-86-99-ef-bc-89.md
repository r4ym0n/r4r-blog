---
title: Nginx 折腾笔记（SSL配置以及路由重写）
url: 474.html
id: 474
categories:
  - 未分类
date: 2019-02-08 00:00:00
tags:
---

前面的话
----

需求分析，由于 搭好了日志平台，本想着把Blog迁移到主机上，最后想想，算了

这里的博客的域名，本来是通过 在解析配置里面使用了一个 `CNAME`到 [quartz010.github.io](quartz010.github.io) 实现一次访问，这里对架构进行一次调整。原 `blog`的二级先解析到服务器，留下日志之后，进行一次隐式跳转，实现一次访问。不过，这样会受限于服务器的带宽。但是也算是接入了日志平台。所以这里需要使用 Nginx 实现重定向。

* * *

其二，域名不备案，web一起来，就被工信部给 ban 了。可是备案太麻烦了，真的是懒。所以这里跑 HTTPS ，做一个短期的检测绕过。因为 HTTPS 其报文内容是加密的，所以流量只要不是很大，应该也不至于单独去做侧信道的分析。所以这里顺便给部署全站的 HTTPS 访问暂时的撑着。

显式重定向 和 隐式重定向
-------------

这里的一点调整在于 Github page 的命名

当repo 命名为 `quartz010.github.io` 可以直接通过 其进行访问，如果其他命名的话，需要进行 quartz010.github.io/xxx 进行访问，所以修改之后导致了其找不到静态文件。导致样式消失以及一大堆404 的问题。

这里给出 基本配置：

    server {
        listen       80;
        listen       [::]:80;
    
        server_name blog1.diglp.xyz;
    
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
    
        location / {
            #rewrite  ^/(.*)$  /$1 break;
            proxy_pass https://quartz010.github.io;
        }
        # rewrite "^/(.*)$" https://quartz010.github.io/$1 last;
    }

这里的重点在 Location 的这一段：

    location / {
        #1# rewrite  ^/(.*)$  /$1 break;                # 隐式重定向
        #2# proxy_pass https://quartz010.github.io;     # 显式重定向
    }

这里的重定向方式有两种，不过严格意义上讲，通过`proxy_pass`实现的只能叫做反向代理。不过这里用到了实现 Url不改变的重定向，也权且这样叫吧。 后面讲路由重写，这里先看这个正则表达式 `^/(.*)$` 代表以 `/` 开头中间匹配 `.*` (也就是任意个单字符)，之后再进行结尾。综上是匹配所有的路由。

路由重写
----

路由的重写，这里算得上是很重要的一个模块了。一般 Nginx 默认编译需要 加上 Pcre 的正则的库，其基本语法如下：

    server {
        rewrite 规则 定向路径 重写类型;
    }

*   规则：用于匹配的 URL 或者是正则表达式。
*   重定向之后的链接，可以带参数
*   重写类型：
    *   last ：表示完成rewrite，浏览器地址栏URL地址不变
    *   break：本条规则匹配完成后，终止匹配，不再匹配后面的规则，浏览器地址栏URL地址不变
    *   redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址
    *   permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
*   **作用域:** _server, location, if_

    rewrite ^(.*)$  https://$host$1 permanent

这里的重写类型是可选项，也可以不进行指定。这里值得注意的是，last 和 break 类型的区别。last 可以理解为，对url改变之后，继续对url进行重写匹配，知道最后没有相关规则时候便停止。break 在完成本次的重写之后，就不进行后继的匹配重写。eg

    server {
        location / {
            rewrite /111/ /555/ last;
            rewrite /222/ /555/ break;
        }
        location ^~ /555/ {
            internal;
            empty_gif;
            return 403;
        }
    }

在这样的配置下。得到的结果是 访问 `/111/` 的时候，直接返回了403的页面。而访问 `/222/` 的时候，得到了404的返回。不过，值得注意的是， `111` 返回了 403 的内容，但是实际上的 URL 并没有发生改变。即：

**xxx/111/ 的 URL 实际上是返回了 /555/ 的内容**

SSL 配置
------

### 基础知识

SSL 在http的具体应用就是 HTTPS （**HTTP over TLS**）。

> HTTPS的主要思想是在不安全的网络上创建一安全信道，并可在使用适当的加密包和_服务器证书可被验证且可被信任时_，对[窃听](https://zh.wikipedia.org/wiki/%E7%AB%8A%E8%81%BD)和[中间人攻击](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)提供合理的防护。

一样回到PKI（Public Key Instruction）的体系里面，其中一个重要的概念就是 HTTPS 的证书。

在普通的点对点加密里面，另个客户端可以直接进行秘钥的商定。

但是在 BS 里面，服务器将面临多个客户端的请求，如果使用自签发证书，会出现证书可能会被伪造。可能会有其他的客户端使用伪造的证书进行HTTPS通信。

常见的应用在于使用 **SSLstrip** 进行证书伪造攻击，以实现把HTTPS流量降级到HTTP。

为了**避免**这样的情况出现，这里就出现了 CA机构（Certificate Authority）用于进行一个可信的第三方证书保管。证书由可信第三方提供，不是直接从服务器获得，所以保护了证书被伪造的可能。

### 证书配置

直接在云服务商的website上面申请域名证书，这里申请的是针对 \[mine.diglp.xyz\]() 的单域名证书。可以免费申请。不过 CA 机构是 **TrustAsia** 据说这个真的不值得Trust。。。这就另说了。

这里了解一下文件分布：

    .
    |-- Apache
    |   |-- 1_root_bundle.crt
    |   |-- 2_mine.diglp.xyz.crt
    |   `-- 3_mine.diglp.xyz.key
    |-- IIS
    |   `-- mine.diglp.xyz.pfx
    |-- Nginx
    |   |-- 1_mine.diglp.xyz_bundle.crt
    |   `-- 2_mine.diglp.xyz.key
    `-- Tomcat
        `-- mine.diglp.xyz.jks

tree 一下目录，可见已经根据不同的Server给分成了不同的类型。内容都是一样，进行了一定程度的合并和拆分而已

    ssl_certificate "/xxx/1_mine.diglp.xyz_bundle.crt";                    
    ssl_certificate_key "/xxx/2_mine.diglp.xyz.key";           
    ssl_session_cache shared:SSL:1m;                                             
    ssl_session_timeout  10m;                                                    
    ssl_ciphers HIGH:!aNULL:!MD5;                                                
    ssl_prefer_server_ciphers on;

这样就很轻松的完成了Https的配置。

全站使用 HTTPS 的实现
--------------

这里很巧妙的使用了重定向，实现了全站的HTTPS。通过对HTTP流量进行重定向实现

    server {
        listen 80 default_server;
        #listen [::]:80;
        return 301 https://$host:9943$request_uri;
        #rewrite ^(.*)$  https://$host$1 permanent;
     }

这里对所有的请求，直接重定向到HTTPS。很是巧妙

后面的话
----

很多事，很多事，面对审判吧。

用的就记录下来