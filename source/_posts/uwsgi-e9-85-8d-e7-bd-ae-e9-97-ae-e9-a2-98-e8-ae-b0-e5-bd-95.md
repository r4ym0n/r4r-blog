---
title: Uwsgi 配置问题记录
url: 498.html
id: 498
categories:
  - 未分类
date: 2018-10-13 00:00:00
tags:
---

前
-

使用 Nginx 和 uwsgi 一起搭建提供 CGI 服务。本来时很顺利的搭建完成了环境。发现到另一台主机上就出现了葛总问题，这里记录一下

Nginx
-----

在 Nginx 的配置文件中的配置如下 ，在官方的模板配置里面也已经给出。

    location ^/CGI {
        rewrite ^ /cgi-bin/x.py last;
    }
    
    location /cgi-bin {
        # internal;
        include $nginx_root/nginx/conf/uwsgi_params;
        uwsgi_modifier1 9;
    
        uwsgi_pass 127.0.0.1:9000;
    
    }

> **uwsgi协议魔术变量**， 你可以通过使用web服务器（或一般使用一个uwsgi兼容的客户端）传递的专用的变量来动态调整或配置uWSGI服务器的各个方面。

**uwsgi_param**　文件的具体内容如下

    uwsgi_param  QUERY_STRING       $query_string;
    uwsgi_param  REQUEST_METHOD     $request_method;
    uwsgi_param  CONTENT_TYPE       $content_type;
    uwsgi_param  CONTENT_LENGTH     $content_length;
    
    uwsgi_param  REQUEST_URI        $request_uri;
    uwsgi_param  PATH_INFO          $document_uri;
    uwsgi_param  DOCUMENT_ROOT      $document_root;
    uwsgi_param  SERVER_PROTOCOL    $server_protocol;
    uwsgi_param  REQUEST_SCHEME     $scheme;
    uwsgi_param  HTTPS              $https if_not_empty;
    
    uwsgi_param  REMOTE_ADDR        $remote_addr;
    uwsgi_param  REMOTE_PORT        $remote_port;
    uwsgi_param  SERVER_PORT        $server_port;
    uwsgi_param  SERVER_NAME        $server_name;

这里的uwsgi_modifier1 9;

这里所谓的魔术变量 可以理解为 Nginx 对 uwsgi 发送命令的操作类型， 具体的指令可见：

> [uwsgi协议魔术变量](https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/Vars.html)

Uwsgi
-----

### default & cgi

这个问题，真的是巨大的坑，由于资料较少，最后是在官方文件发现的这一个问题。

在Uwsgi 进行编译安装之后执行 CGI 请求的时候返回一以下内容

> no python application found, check your startup logs for errors

* * *

在进行问题的查证，看了手册后发现，uwsgi 的不同编译参数对应了不同的版本，有默认版本 ，和CGI 版本。

编译命令如下

    curl http://uwsgi.it/install | bash -s default /tmp/uwsgi.cgi   # 这个版本需要和web应用联系

    curl http://uwsgi.it/install | bash -s cgi /tmp/uwsgi.default   # 这个版本的才能用于 nginx

### 参考

> 在官方 的文档里提供了详尽的 配置，已经优化的各种方法
> 
> *   [在uWSGI上运行CGI脚本](https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/CGI.html)

其他Q
---

### MIME 的问题

**MIME**(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。

    print ("Content-type:text/html")
    print 

在 CGI 脚本的执行过程中，必须有 MIME 的头，否则脚本执行时 发生 502

### MIME 的问题 2

没错，这个 MIME 又出问题了， 在同样环境的主机上面，做了平行迁移，于是就出了问题。测试其他的脚本没有问题，测试这个 功能脚本就有问题了，进行对比发现了一个诡异的规律：

这个 CGI 脚本是可以正常执行的。 返回 200

    import cgi
    import cgitb
    cgitb.enable()
    
    print ("Content-type:text/html")
    print
    print ('pass')

然后下面这个，就直接 报错 502 ， invalid CGI response!!!.

    import cgi
    import cgitb
    cgitb.enable()
    
    import os
    os.chdir('/')
    import sys
    sys.path.append("..")
    from util import util, db_mysql
    
    print("Content-type:text/html")
    print

综上,发现问题，MIME 头需要在用户的自定模块前打印，否则导致脚本 502！