---
title: 使用Certbot来部署HTTPS
tags:
  - Nginx
url: 1324.html
id: 1324
categories:
  - OP之路
date: 2019-10-26 14:30:57
---

站点使用的https是来自freessl申请的通配证书，免费🆓是关键。但是奈何几个月的有效期太短自己又懒得续费，所欲出找了找自动续费的方案，发现`certbot`是个不错的选择，而且可以自动的根据webserver的配置，来获得域名以及配置。很方便，这篇就介绍一下这个工具

安装部署
----

项目的官网已经提供了很好的支持，在安装和部署的方面基本上可以实现 oneclick。[https://certbot.eff.org/](https://certbot.eff.org/)

    #安装源
    yum -y install yum-utils
    yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional
    
    #安装应用
    sudo yum install certbot python2-certbot-nginx
    
    # 执行证书更新
    sudo certbot --nginx
    
    # 设置自动更新
    echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew" | sudo tee -a /etc/crontab > /dev/null
    # 这里使用pyhton做了随机的延迟，避免用户都使用这条命令之后导致的请求尖峰的问题，使用shell的更优雅的写法是：
    echo "0 0,12 * * * root sleep $[$RANDOM/1024] && certbot renew" | sudo tee -a /etc/crontab > /dev/null
    

注意的问题
-----

*   由于使用 pyhton2 编写的工具，所以nginx的配置文件中不能存在中文。
    
*   如果存在报错 `pyOpenSSL' module missing required functionality`：
    
            raise ImportError("'pyOpenSSL' module missing required functionality. "
        ImportError: 'pyOpenSSL' module missing required functionality. Try upgrading to v0.14 or newer.
    
    尝试卸载`urllib3`，并且使用`yum`来进行重装
    
        yum -y install python-urllib3
    

后
-

这个配置很贴心的甚至可以做到自动的配置域名的 https的跳转，实例的配置如下：

    server {
        if ($host = share.diglp.cn) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
        if ($host = life.diglp.cn) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    
        listen       80;
        server_name  life.diglp.cn share.diglp.cn;
        return 404; # managed by Certbot
    }

根据对应的host进行301跳转。