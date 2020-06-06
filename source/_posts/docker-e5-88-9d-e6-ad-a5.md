---
title: Docker 初步
url: 482.html
id: 482
categories:
  - 未分类
date: 2019-01-08 00:00:00
tags:
---

前
-

这篇，是一个Docker 的实战篇，使用Docker实现一个文件上传的web页面。ps：本来是要实现上传后自动编译并且允许返回结果的。因为各种原因就烂尾了，就这样吧先。

因为这篇，是比较基础的操作，很多东西还是有悖了Docker的 **Build，ship，and run** 的思想。这里还需要进行手动，所以显得比较有悖初衷。不过做一个过程的记录还是可以的。

* * *

后面就打算实现一个标标准准的Docker 的工程，体现出 **微服务** 的思想。

> Docker 的一个容器理应是只跑一个进程（服务）的。

Basic
-----

先从Dockerhub上拉镜像，这里需要 Nginx 和 Uwsgi，这里选取了[tiangolo/uwsgi-nginx](https://hub.docker.com/r/tiangolo/uwsgi-nginx) 这个镜像。

在启动镜像的时候使用交互模式：

    docker run -itd -p 80:80 --name test2 docker.io/tiangolo/uwsgi-nginx:latest /bin/sh

这个时候，Dockerfile 里面的CMD命令将会被重载为后面的 `/bin/sh`

在交互模式下，attach 了容器之后，可以使用 `ctrl+p，ctrl+q` 组合键进行 detach

    docker attach bf00008eee04
    docker attach test2

* * *

创建挂载卷容器，在对应的环境 编译 uwsgi， 之后可以之间拷贝到本地

    docker run -tdi -v /tmp/tttmm/:/tmp/mm --name share docker.io/tiangolo/uwsgi-nginx:latest /bin/bash 

这里使用 bash 好用得多。另外这里附上 uwsgi 的CGI版本的一键编译的script：

    curl http://uwsgi.it/install | bash -s cgi /tmp/uwsgi

* * *

CGI 部署
------

跑一个综合的容器， 有端口映射以及卷挂载功能：

    docker run -tdi -p 80:80 -v /tmp/tttmm/:/tmp/mm --name share docker.io/tiangolo/uwsgi-nginx:latest /bin/bash 

由于 镜像自带的uwsgi 的版本是app的，其没有编译 CGI 功能。所以在这里需要进行替换。之间在容器内使用上述命令进行编译。对 `/usr/local/bin/uwsgi` 进行替换，对配置文件进行修改

    [uwsgi]
    socket = /tmp/uwsgi.sock
    chown-socket = nginx:nginx
    chmod-socket = 664
    hook-master-start = unix_signal:15 gracefully_kill_them_allroot@251a2c8dc0b2:/usr/local/bin# 

这里是用户自定义的：

    [uwsgi]
    plugins = cgi
    cgi = /app
    cgi-allowed-ext = .py
    cgi-helper = .py=python

指定CGI根目录，以及对应的后缀文件的解析器。后面再修改Nginx配置即可。

* * *

编写实例 CGI 脚本。使用其标准输出作为页面内容：

    print("Content-Type: text/html")
    print("")
    print("<html>")
    print("<h2>First CGI server Base on Docker</h2>")
    print("<p>Hello ann</p>")
    print("<p>cause you are my world XD</p>")
    print("</html>")

* * *

至此，启动 `/usr/bin/supervisord` 便可以启动相关进程

> Supervisor ([](http://supervisord.org/)[http://supervisord.org](http://supervisord.org)) 是一个用 Python 写的进程管理工具，可以很方便的用来启动、重启、关闭进程（不仅仅是 Python 进程）。除了对单个进程的控制，还可以同时启动、关闭多个进程，比如很不幸的服务器出问题导致所有应用程序都被杀死，此时可以用 supervisor 同时启动所有应用程序而不是一个一个地敲命令启动。

在客户端使用 浏览器请求便可以得到返回结果

* * *

Post 文件以及编译执行
-------------

前面实现了一个简单的CGI, 后面开始完善整个功能，实现文件的POST 上传以及在线的编译。

先需要对 Nginx 配置进行修改：

    server {
        listen 80;
    
        location / {
            root /app/html;         # 这里是页面的目录
        }
    
        location /cgi-bin {
            root /app/cgi-bin;      # cgi 的目录
            include uwsgi_params;
            uwsgi_modifier1 9;
            uwsgi_pass unix:///tmp/uwsgi.sock;
        }
    }

重启 Nginx 的服务，使其配置生效，测试 页面 以及 CGI 脚本。

这里先构建文件的上传入口，简易的上传 页面：

    <html>
        <head>
            <meta charset="utf-8">
            <title>update</title>
        </head>
        <body>
            <h1>UploadPoint</h1>
            <form id="upload-form" action="/cgi-bin/upload.py" method="post" enctype="multipart/form-data" >
            　　　<input type="file" id="upload" name="upload" /> <br />
            　　　<input type="submit" value="Upload" />
            </form>
        </body>
    </html>

以及后端的实现文件保存以及编译的功能

下面是CGI代码：

    import cgi, cgitb 
    
    form = cgi.FieldStorage() 
    fileitem = form['upload']
    
    if fileitem.filename:
        print("Content-type:text/html")
        print()
        print("<html>")
        print("<head>")
        print("<meta charset=\"utf-8\">")
        print("<title>succeed</title>")
        print("</head>")
        print("<body>")
        print("<h2> 输入的内容是：%s</h2>")
        with open('tmpfile/' + fileitem.filename, 'wb+') as f:
            f.write(fileitem.file.read())
            print('<p>文件已保存</p>')
        print("</body>")
    
    else:
        text_content = "没有内容"
        print("Content-type:text/html")
        print()
        print("<html>")
        print("<head>")
        print("<meta charset=\"utf-8\">")
        print("<title>failed</title>")
        print("</head>")
        print("<body>")
        print("<h2> 输入的内容是：%s</h2>" % text_content)
        print("</body>")
        print("</html>")

参考
--

*   [在uWSGI上运行CGI脚本](https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/CGI.html)
*   [使用 supervisor 管理进程](http://liyangliang.me/posts/2015/06/using-supervisor/)
*   [镜像地址 tiangolo/uwsgi-nginx](https://hub.docker.com/r/tiangolo/uwsgi-nginx)