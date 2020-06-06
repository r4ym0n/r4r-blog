---
title: Nginx 实现HTTP/HTTPS的透明代理
url: 1645.html
id: 1645
categories:
  - OP之路
date: 2020-06-06 03:09:46
tags:
---

使用透明代理来解决，存在的某些站点的不可达。这里直接贴出来 ng的配置

对于 80 端口这里使用 直接 proxypass的方式

    server {
        resolver 114.114.114.114;
        listen 80;
        location / {
            proxy_pass http://$http_host$request_uri;
            proxy_set_header HOST $http_host;
            proxy_buffers 256 4k;
            proxy_max_temp_file_size 0k;
            proxy_connect_timeout 30;
            proxy_send_timeout 60;
            proxy_read_timeout 60;
            proxy_next_upstream error timeout invalid_header http_502;
        }
    }

对于 443 端口，不能使用 http 的形式，因为是涉及到了包的内容，使用4层转发来解决

    stream {
        resolver 114.114.114.114;
        server {
            listen 443;
            ssl_preread on;
            proxy_connect_timeout 5s;
            proxy_pass $ssl_preread_server_name:$server_port;
        }
    }

使用时候，直接在 主机上配置 `/etc/hosts`里写上域名的host即可

    192.168.111.252 xxx.example.com