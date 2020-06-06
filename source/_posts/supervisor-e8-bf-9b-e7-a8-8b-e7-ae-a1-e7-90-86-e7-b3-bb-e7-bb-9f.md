---
title: Supervisor 进程管理系统
url: 466.html
id: 466
categories:
  - 未分类
date: 2019-02-23 00:00:00
tags:
---

前
-

一台机器东西跑多了，就开始各种混乱了。如果是标准安装的注册了service 的还可以比较容易的进行管理。然而很多组件是后面直接下载二进制文件的。这里一个 `nohup` ，那里一个。最后只能 ps 找出来。如没运行了，有时候甚至找不到了。自启动也是。

所以这里就是用 Supervisor 进行统一的进程管理，对添加的进程，的启停，自启等都能有很方便的管理。这篇就做个 basic 的部署使用说明。

部署及配置
-----

debian 和 redhat 的风格各自不同，debian直接通过 `apt-get` 安装即可。redhat 使用pip安装

    sudo pip install supervisor

以上完成，就实现了安装过程。

* * *

一共有两个部分 `supervisorctl` 和 `supervisord`。根据其命名可以看出来，一个是控制端，另一个是守护进程。先启动守护进程，并注册开机启动。

    systemctl enable supervisor.service     # redhat
    update-rc.d supervisor enable           # debian

之后把进程跑起来

    ➜  ~ systemctl start supervisord.service               
    ➜  ~ supervisorctl 
    supervisor> 

至此,配置没有问题，部署完成。

进程配置
----

在 `/etc/supervisord.d` 里面通过 ini文件（conf）进行统一的注册。示例注册如下：

    [program:frps]
    command=/***/frp/frps -c frps_full.ini --log_file frps.log
    directory=/***/
    autostart=true
    startsecs=5
    autorestart=true
    startretries=3
    user=root
    redirect_stderr=true
    stdout_logfile_maxbytes=20MB
    stdout_logfile_backups=5
    stdout_logfile=/data/logs/frps_stdout.log                              

这样就完成了一个进程的注册。后面在 ctl 里面进行更新，和操作了，十分方便

    ➜  ~ supervisorctl 
    frps                             RUNNING   pid 6607, uptime 6:14:01
    uwsgi                            RUNNING   pid 6606, uptime 6:14:01
    supervisor> 

至此，一个进程注册完成

后
-

这篇文章感觉水水的，不过，挺偏实用向，下面给出 supervisor 的官方手册

> [Supervisor: A Process Control System](http://supervisord.org/index.html)