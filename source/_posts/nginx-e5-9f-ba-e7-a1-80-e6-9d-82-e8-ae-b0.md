---
title: Nginx 基础杂记
url: 506.html
id: 506
categories:
  - 未分类
date: 2018-09-09 00:00:00
tags:
---

前
-

> 实践是最好的复习方式，书中所学的东西很快的投入应用。得到的远超过理论本身

再硅谷百年史中有一句话，大意是这样：

> 硅谷并不一定是新技术产生的地方，但是新的技术得以在这片土地得到快速的传播，和应用，技术的本身得到快速迭代，展现出了无尽的生命力。

上周，一个基础的Nginx入门， 这周很多地方都有用上都能用上。幸哉

正文
--

### CGI的配置

这部分，简单的记录一个的基于 Nginx 的一个 CGI 服务器的配置过程。

#### Nginx 的编译安装

这部分在 Linux 的环境下，拥有了一个 GNU 的完整编译链的话，可以很容易的直接

    ./configure

使用 AUTOCONF 和 AUTOMAKE 直接生成和当前系统环境下的 配置的 Makefile 。

    make 
    make install

这两个命令，直接对 Nginx 进行编译安装。得到的文件在

    /usr/local/nginx/
    $nginx_root/conf # 这个里面是它的配置文件所在的地方

#### uWSGI 编译安装

uWSGI 官方提供了一键安装的操作，用起来oneClick ：

    curl http://uwsgi.it/install | bash -s default /tmp/uwsgi

后面是编译输出的路径。这样一个回车键就可以

在 `/tmp/uwsgi` 下面的就是我们编译的bin文件

#### Nginx CGI 的配置

Nginx 可以根据不同的配置文件，以进程为单位启动多个。

    ./nginx [-t # 检查配置文件] [-c 指定配置文件路径]

这里需要注意的是 ，nginx 的配置文件默认在 `/usr/local/conf`。所以这里，我们可以把配置文件夹拷贝到当前目录，之后使用 c 参数进行指定就好

* * *

下面这部分是 Nginx 的配置，直接在预设的幕布就配置中进行修改就好，这里贴出配置文件：

    worker_processes  1;
    
    error_log  logs/error.log;  # 这里可以配置 log 所记录的日志级别
    
    pid        logs/nginx.pid;  # Pid 文件，用于保存当前服务进程的PID
    
    events {
        worker_connections  1024;
    }
    
    http {
        include       mime.types;   # 多用途互联网邮件扩展（MIME，Multipurpose Internet Mail Extensions）
        default_type  application/octet-stream; # 这里设置类型
    
        #access_log  logs/access.log  main; # 访问日志
    
        sendfile        on;
        #keepalive_timeout  0;
        keepalive_timeout  65;
        #gzip  on;
    
        server {                    # 一个虚拟服务器的定义
            listen       8000;
            server_name  localhost; 
            charset  utf-8;         # 设置页面编码
            #access_log  logs/host.access.log  main;    # 当前虚拟服务器的访问日志
    
            location / {            # 根路由配置
                root   html;        # 注意，如果在编译的时候没有指定 root
                                    # 这里的 html 是相对路径在 /usr/local/nginx/html/
                index  index.html index.htm;
            }
    
            # error_page  404              /404.html;
            # redirect server error pages to the static page /50x.html
    
            error_page   500 502 503 504  /50x.html; # 定向错误码，到路由
            location = /50x.html {  # 精确匹配路由
                root   html;        # 在 /html 里面找到 50x.html （alias 和 root）
            }
    
            location /cgi-bin {     # 这里就是一个 CGI 的路由配置了
                    include uwsgi_params;   # 引入 uwsgi 的所有可用参数
                    uwsgi_modifier1 9;
                    uwsgi_pass 127.0.0.1:9000;  # CGI 请求传入本地监听 的9000 端口
                                                # 交给 uWSGI 执行
            }
    
    }

可见，在Nginx 里的主要的配置内容，是直接 使用 uWSGI 监听的端口， 进行命令传递，至此Nginx 的配置部分就好了，这里的重点，就是在与对应 路由的配置，

#### uWSGI 配置

在 uWSGI 直接使用 ./ 便可以将服务，运行起来。不过会出一堆挺吓人的警告。具体是其没有进行正确的配置。启动命令：

    sudo -u root ./uwsgi ./uwsgi.ini

这里有一点是：uWSGI 服务，用于相应 WEB 请求，所以安全考虑，这里应该有严格的权限控制。使用

`sudo -u nobody` 指定用户来启动 CGI 服务，这样可以控制CGI 脚本的权限，避免出现风险。不过这里由于需要对部分文件进行读写， 所以图方便使用了root。

uWSGI 的配置文件如下

    [uwsgi]
    processes = 4   # 指定worker数，是uWSGI 的worker
    master = true   # 这个配置重要，启用 master_process 不然将会工作在单线程模式
                    # 而且，只可以运行在前台
    socket = 127.0.0.1:9000     # 连接类型，以及监听端口
    chdir = /data/_dir/nginx/cgi-bin        # CGI 脚本们的路径
    cgi=/cgi-bin=/data/_dir/nginx/cgi-bin   
    cgi-helper=.py=python       # 对不同的文件类型指定不同的解释器
    
    daemonize = /data/_dir/nginx/uwsgi/uwsgi.log    # 后台运行的日志
    pidfile = /data/_dir/nginx/uwsgi/uwsgi.pid  # PID 文件
    

对上面的配置文件做了简单的解释，有了这样的配置文件之后，一个 基于 Nginx 的服务就已经搭建起来了。

#### CGI 脚本

CGI 脚本们存在于 cgi-bin 这个目录下，如果有 web 对CGI 的路由进行了访问，那么这个http 请求将会被下发到 CGI 服务（uwsgi） 之后，它创建解释器进程，对我们的 脚本进行执行，并最终接收返回结果，传递回前端。

这里列出几个简单的脚本

*   shell
    
        #!/bin/bash
        echo "Content-Type:text/html"
        echo ""       # 空行分隔
        echo "hello world!"
    
*   Python
    
        #!/usr/bin/env python
        print("Content-type: text/html\n\n")
        print()   # 空行分隔
        print("<h1>Hello World</h1>")
    

又是教科书式的 **helloworld**。 这里我们就直接使用输出函数，对html 内容进行输入，最终他将返回到前端。

* * *

##### 带上参数

对于 CGI 请求，当然是可以带上了参数，这里只 说 GET 这种请求方式

使用 cgi 模块，可以很容易的提取参数

    import cgi, cgitb
    form = cgi.FieldStorage()
    context  = form.getvalue('keyword')
    
    print "Content-type: text/html\n\n"
    print() # 空行分隔
    print("<h1>Hello World</h1>")
    print("<h2>context</h2>")

#### 小结

##### Q: Uwsgi 只能前台运行的问题

在配置中 没有指定 Master_process 所以工作在单线程调试模式，**./uwsgi** 的过程中，发现其只能在前台以单进程的模式运行。在中发现是配置文件中，没有指定Master_process 的数量

所以导致，其是在单进程情况下运行在前台，多半这个为调试模式下的状态，单进程

* * *

##### Q: uwsgi 错误invalid request block size

这个其实不算问题，是自己在调试过程中发现的。使用：

    curl "127.0.0.1:9000" -V

因为 curl 是发送的 http 的请求，uwsgi 默认绑定的是 socket。

实际上这个和 UWSGI 的绑定的协议有关。在测试时请求的 **uwsgi.ini** 配置文件里面内容把socket=:8000替换成http=:8000，可以切换 所绑定的协议

* * *

##### Q: **PID** 文件使用

在 nginx 和 Uwsgi 的配置文件中，添加 PID 的参数，可以是的其在运行的时候生成 PID 文件

这样可以快速方便的给相关的进程发信号，进行重启或者是终止

    Kill -HUP `cat ./xxx.pid`

### 一个简单的下载页面

这里是 Nginx 的另一组配置文件，实现的是开放一个页面可以进行文件俩蓝和下载

    http {
        include       mime.types;   # 多用途互联网邮件扩展（MIME，Multipurpose Internet Mail Extensions）
        default_type  application/octet-stream; # 这里设置类型
        #access_log  logs/access.log  main; # 访问日志
        sendfile        on;
        keepalive_timeout  65;
        #gzip  on;
    
        server {
            listen       80;
            charset utf-8;
    
            location / {
                root   html;
                index  index.html index.htm;
            }
    
            location /confs {       # 路由
                alias   /data/_dir/nginx/www/mon_conf;
                autoindex on;             #开启索引功能
                autoindex_localtime on;   # 显示本机时间而非 GMT 时间
                auth_basic "Enter your name and password";
                # 指定验证文件
                auth_basic_user_file /data/_dir/nginx/www/mon_conf/.htpasswd;
                index  index.html index.htm;
            }
    
            location /query {   
                rewrite ^ /cgi-bin/query.py last;   # 这里的路由重写
            }
    
            location /cgi-bin {
                internal;                           # 内部访问，外部访问时 404
                include uwsgi_params;
                uwsgi_modifier1 9;
                uwsgi_pass 127.0.0.1:9000;
            }
        }
    }

* * *

#### 页面的登入认证

由于这里可以直接进行文件访问，所以添加了页面的登陆验证

    auth_basic "Enter your name and password";
    auth_basic_user_file /data/_dir/nginx/www/mon_conf/.htpasswd;

> *   [在线 htpasswd 生成器](http://tool.oschina.net/htpasswd)
> *   htpasswd 是开源 http 服务器 [apache httpd](http://www.oschina.net/p/apache+http+server) 的一个命令工具，用于生成 http 基本认证的密码文件。

这里简单的测试， **admin:1234 (MD5)** 得到密文

`admin:$apr1$PqhZgBDT$pAwEYVAKKXdHIXxqvbEYU/`

* * *

#### URL 的重写

URL 的重写，这里实现的是隐式的重定向。就是我们的页面发生跳转，但是URL 不发生变化。

这里使用 nginx 的 rewrite 模块实现相关功能。具体配置如下：

    location /query {   
        rewrite ^ /cgi-bin/query.py last;   # 这里的路由重写
    }
    
    location /cgi-bin {
        internal;                           # 内部访问，外部访问时 404
        include uwsgi_params;
        uwsgi_modifier1 9;
        uwsgi_pass 127.0.0.1:9000;
    }

下面的CGI 部分和之前是没有什么区别，不过值得注意的是 ， 这里的internal 的修饰。说明这刚路由路径，不可以从外部进行访问。

所以在 这里进行了重定向 `rewrite ^ /cgi-bin/query.py last;` 对 query 进行重写，相对的就是对重写后的地方进行访问。

##### rewrite 的标识

不写last和break - 那么流程就是依次执行这些rewrite

1.  rewrite break - url重写后，直接使用当前资源，不再执行location里余下的语句，完成本次请求，地址栏url不变
2.  rewrite last - url重写后，马上发起一个新的请求，再次进入server块，重试location匹配，超过10次匹配不到报500错误，地址栏url不变
3.  rewrite redirect – 返回302临时重定向，地址栏显示重定向后的url，爬虫不会更新url（因为是临时）
4.  rewrite permanent – 返回301永久重定向, 地址栏显示重定向后的url，爬虫更新url

*   引用自 [Nginx中的rewrite指令(break,last,redirect,permanent)](https://blog.csdn.net/zhanlanmg/article/details/49684803)

### 一些总结

#### 502 和 504

*   502 Bad Gateway 504 Gateway time-out 503 Service Unavailable

正如其 英译过来，

*   Bad Geteway 一般发生在当前的HTTP请求，在进行 CGI 分发的时候，发生错误，比如 :9000 端口连接失败。或者 CGI 服务没有启动 Gateway time-out Nginx 在进行任务分发之后，超过预定时间，没有得到 CGI 服务器的 echo，可能是 CGI 服务的进程阻塞 Service Unavailable 由 CGI 服务直接返回，无法处理任务的请求，一般是流量太大导致的问题

参考内容
----

> *   [The uWSGI project](https://uwsgi-docs-cn.readthedocs.io/zh_CN/latest/index.html)