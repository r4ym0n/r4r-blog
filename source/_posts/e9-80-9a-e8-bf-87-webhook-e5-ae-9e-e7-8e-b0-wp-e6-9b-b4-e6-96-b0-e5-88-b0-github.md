---
title: 通过 WebHook 实现 WP 更新到 Github
tags:
  - git
  - python
  - wordpress
url: 804.html
id: 804
categories:
  - DEV
date: 2019-04-16 22:38:22
---

前
-

博客从前面的静态站点，搬到现在的动态站点了，虽说方便了很多，但是，之前是在 **Github上**托管的，所以由个更新，感觉还可以看见的填色游戏，迁走了之后，感觉少了点什么东西，没有色块了感觉自己没啥动静了。

* * *

所以这篇就是使用 **WP webhook** 来实现到 Github 的自动更新。完成commit。这样就可以继续填色游戏了，并且，后面打算优化脚本实现，文章的自动同步。

WebHook 的设置
-----------

在 WP 端使用的是 **WP Webhook 这个插件** 支持 Send 和 Recieve 的方式实现。这里由于是实现自动的 push 所以自然使用的是 send。在 插件页面配置如下

![file](https://i.loli.net/2019/04/16/5cb5ea0fc407b.png)

* * *

这里可能是属于一个 BUG。很难受忍不住吐槽。 在这个 URL 填写的时候，试了很多种形式来写。IP的，IP端口的，域名端口的，域名的，发现只有单单域名或者hostname的时候可用，其他情况下通通地都是 **URL不可用**。

由于web服务地限制，只能使用非标端口，但是用了非标端口，又不能被识别，真是麻烦。所以被迫的，后面又把 Nginx 跑了起来，用 443 + SSL 一样的放方，强制过备案。

WebHook Server 的實現
------------------

解决了WP的WebHook的请求的问题，这里就要实现自己的 webhook 的服务器了，这里这里直接用SimpleHTTPServer 这个 module 自己实现一个 。

* * *

下面直接贴出来这个python写的小工具。

    #!/usr/bin/env python
    # coding=utf-8
    
    import os, time
    from wsgiref.simple_server import make_server
    import logging
    
    __author__ = 'lau'
    
    logging.basicConfig(level=logging.INFO,
                        filename='./log.out',
                        filemode='a',
                        format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')
    
    def newPost(environ):
        if environ['REQUEST_METHOD'] == "POST":
            print(environ)
            logging.info('new post')
            os.system('git add .')
            os.system('git commit -m "merge"')
            os.system('git push origin master')
            logging.info('git push finish :new')
    
    def update(environ):
        if environ['REQUEST_METHOD'] == "POST":
            logging.info('post update')
            os.system('git add .')
            os.system('git commit -m "%s"' % time.asctime(time.localtime(time.time())))
            os.system('git push origin master')
            logging.info('git push finish :update')
    
    # 这里路由和函数进行绑定
    route_map = {
        '/new': newPost,
        '/update': update
    }
    
    def application(environ, start_response):
        start_response('200 OK', [('Content-Type', 'text/html')])
        # 由于这里进行了一级的路由，因为不能直接占根，所以这里，就直接取最后一级
        environ['PATH_INFO'] = '/' + environ['PATH_INFO'].split('/')[-1]
        print(environ['PATH_INFO'])
        if environ['PATH_INFO'] in route_map:   # 路由匹配之后，这里直接进行调用其处理方法
            route_map[environ['PATH_INFO']](environ)
        return [b'Hello!']
    
    # 创建一个服务器，IP地址为空，端口是8000，处理函数是application:
    httpd = make_server('', 8000, application)
    print('Serving HTTP on port 8000...')
    httpd.serve_forever()

* * *

上面的代码直接跑起来应该问题不大了。没有什么核心的，简单的直接使用 system 来调用shell。

后面需要在 nginx 的配置里面添加内容：

    location // {
       proxy_pass http://127.0.0.1:8000;
    }

后
-

后面打算把请求体作为文章顺便保存下来。优化一一下功能，顺便解决更新一下 push 多次的现象