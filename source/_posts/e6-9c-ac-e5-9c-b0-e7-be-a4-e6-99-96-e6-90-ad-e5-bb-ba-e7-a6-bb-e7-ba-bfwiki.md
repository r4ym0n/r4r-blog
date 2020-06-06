---
title: 群晖NAS搭建离线wiki -- 基于Docker
url: 1021.html
id: 1021
categories:
  - 玩点什么
date: 2019-06-02 22:30:27
tags:
---

> Kiwix让您能够随身携带完整的维基百科！无论您搭乘船只，还是身处偏僻的地区，抑或身陷囹圄，Kiwix都使您能够接触到全人类的知识。您不需要连接因特网，因为所有的资料都储存在您的电脑，优盘或者DVD中！

身陷囹圄，却渴望“正确”和“真相”。这样做不一定“正确”，但是是有意义的。 这一篇文章主要是记载如何使用手上的群晖 DS216 来实现部署一个自己的离线的wiki。

前面的问题
-----

这里使用的自己的群晖，但是环境问题，有下面几个坑，大家没准也遇得上

### wget不支持HTTPS

由于群晖默认安装的wget没有编译https的支持，惊了。在下载时候以下报错。

    root@Sync-NG:~/qspace# wget https://bootstrap.pypa.io/get-pip.py
    https://bootstrap.pypa.io/get-pip.py: HTTPS support not compiled in.

* * *

解决方案是卸载原有`wget`，安装支持https的版本。涉及命令如下：

    root@Sync-NG:~/qspace# ipkg remove wget
    Removing package wget from root...
    Successfully terminated.
    root@Sync-NG:~/qspace# ipkg install wget-ssl
    Installing wget-ssl (1.12-2) to root...
    Downloading http://ipkg.nslu2-linux.org/feeds/optware/syno-i686/cross/unstable/wget-ssl_1.12-2_i686.ipk
    Installing libidn (1.25-1) to root...
    Downloading http://ipkg.nslu2-linux.org/feeds/optware/syno-i686/cross/unstable/libidn_1.25-1_i686.ipk
    Configuring libidn
    Configuring wget-ssl
    Successfully terminated.

### 群晖上的pip的安装

由于本身的python是不带 pip模块的，所以需要给它装上，这里有很好的一键安装的脚本。

    wget https://bootstrap.pypa.io/get-pip.py
    python3 get-pip.py

一路绿灯，完成pip安装

### 群晖上使用 supervisord

supervisor 来实现对进程的管理以及 自动拉起，直接使用pip进行安装。`python -m pip install supervisor`，之后生成配置文件`echo_supervisord_conf > /etc/supervisord.conf`，在配置文件进行如下修订，实现一个配置的目录：

    [include]
    files = /etc/supervisord.d/*.ini

部署记录
----

### 下内容

在 kiwixi 上来下载离线的wiki的包

### 上Docker

这里直接上Docker，忽略所有的麻烦的环境问题了。[kiwix-serve](https://hub.docker.com/r/kiwix/kiwix-serve/dockerfile)，但是很坑的是，**为什么做好的镜像，没有对应的说明**，所以使用只能靠自己来摸索。

* * *

这里就是很熟悉的语法，进行卷挂载，这里app默认的目录是 `/data`，启动的参数在后面，指定的是 zim 的文件。之后就OK啦

    docker run -p 32768:80 -v /volume1/DataBank/Wiki/zims/:/data --name="kiwix-server" kiwix/kiwix-serve wikipedia_zh_all_novid_2018-07.zim

* * *

运行效果，界面就OK了： ![file](https://i.loli.net/2019/06/02/5cf3da7e2f23a33084.png)

### 最后穿透

一样是使用`frp`来实现反代，Nginx的反代配置。

    server {
            listen 443 ssl;
            server_name wiki.diglp.xyz;
    
            ssl_certificate "/etc/nginx/ssl/_.diglp.xyz/_.diglp.xyz_chain.crt";
            ssl_certificate_key "/etc/nginx/ssl/_.diglp.xyz/_.diglp.xyz_key.key";
            ssl_session_cache shared:SSL:1m;
            ssl_session_timeout  10m;
            ssl_ciphers HIGH:!aNULL:!MD5;
            ssl_prefer_server_ciphers on;
    
            location / {
                proxy_set_header        host $host;
                proxy_ignore_headers    set-cookie cache-control;   #这句代码很关键，尤其要忽略set-cookie
                auth_basic "Is You?";
                auth_basic_user_file /etc/nginx/.htpasswd;
    
                proxy_set_header            x-real-ip $remote_addr;
                proxy_set_header            x-forwarded-for $proxy_add_x_forwarded_for;
                proxy_set_header            x-forwarded-proto https;
                client_max_body_size        100m;
                proxy_pass http://127.0.0.1:1111111;
            }
    }

* * *

reload之后，直接就生效了，然后在本站上直接配置了链接。Done

Fix
---

### 使用referer限制直接访问

这里使用Nginx的配置，对referer进行限制，避免直接访问，造成不必要的麻烦。

    valid_referers blocked www.diglp.xyz blog.diglp.xyz ray.i124u.cf;
    if ($invalid_referer) {
        return 403;
    }

* * *

通过对referer 的限制实现对访问的过滤。

### ht文件的直接deny

    location ~/\.ht {
        deny all;
    }

后续
--

在原有的部署基础上，解决了几个问题

### 使用 basepath 进行访问

使用basepath进行访问，这样就不需要一个新的域名了。在启动参数中使用 `-r` 来进行指定。这里使用`-r wiki` 来实现basepath 的设置。

* * *

搭配 nginx 来进行规则转发

            location ^~ /wiki/ {
                auth_basic            "Please input password";
                auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
                proxy_pass http://127.0.0.1:32768;
            }

### 使用 library

如果不使用 lib文件的话，一次启动只能指定一个zim库，很不爽，所以这里使用生成的lib来进行配置，这样可以同时使用多个 zim库。

    #!/bin/bash
    FILES=./*.zim
    LIBRARY_PATH=./library.xml
    
    for f in $FILES
    do
      echo $f
      ./kiwix-manage $LIBRARY_PATH add $f
    done