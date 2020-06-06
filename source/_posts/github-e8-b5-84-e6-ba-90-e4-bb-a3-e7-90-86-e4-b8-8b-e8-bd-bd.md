---
title: GitHub 资源代理下载
tags:
  - Docker
  - faas
url: 1287.html
id: 1287
categories:
  - 未分类
date: 2019-10-03 15:56:08
---

代理下载这个词用的可能不是那么准确，这次的问题是解决 github 的资源下载过于缓慢的问题。 思路是常规思路绕过特色的保护，使用海外的服务器来进行代理操作，比如进行文件的下载，之后再在本地进行文件的拉取。

法一
--

说到代理之类的东西，果断就是想到了nginx 的这种天生的http服务器的特性。直接直接对github的下载的链接做 proxy，规则配置如下：

            location ^~ /gitmirror/ {
                #proxy_http_version 1.1;
                #proxy_set_header Host $host;
                proxy_pass https://www.github.com;
                rewrite ^/gitmirror/(.*)$ /$1 break;
            }

在应用配置之后发现自己的想法是很天真的，原因很简单，**github访问链接都不是直连，都是有经过301来进行跳转** 这样最终还是跳了出去，通过 直接 proxy的方法显然是不行的。

法二
--

通过openapi来进行代理下载，之后再在本地进行拉取下载。使用 faas 来对接口进行调用 To be countinued...