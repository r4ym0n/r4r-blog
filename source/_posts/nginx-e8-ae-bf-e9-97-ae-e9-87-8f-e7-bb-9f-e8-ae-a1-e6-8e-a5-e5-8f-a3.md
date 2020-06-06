---
title: Nginx 访问量统计接口
url: 460.html
id: 460
categories:
  - 未分类
date: 2019-03-03 00:00:00
tags:
---

前
-

这个post来自于之前偶遇的一个页面上的组件：

> [Welcome to RSSHub!](https://rsshub.app/) 万物皆可订阅的RRSHUB

上面的那个Debug 的统计，就是显得十分的帅气了，想着自己也搞一个？

查询语句
----

当然,这里也没什么数据库的，直接是用shell出来的，根据日志的格式来进行解析，参考页面：

> [统计Nginx访问量 - 简书](https://www.jianshu.com/p/537a0bddda94)
> 
> [nginx cache查看缓存命中率](https://www.jianshu.com/p/800e168e2463)

* * *

这些操作都是基于 Nginx 日志进行操作的，这里列举一条：

    111.59.124.138 - - [25/Feb/2019:15:31:26 +0800] "GET / HTTP/2.0" 200 12628 "-" "Mozilla/5.    0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.10    9 Safari/537.36" "-"

使用awk对参数进行提取，默认以空格进行分割。

    awk '{print $1}'  /var/log/nginx/access.log|sort | uniq -c |wc -l
    
    awk '{print $7}' /var/log/nginx/access.log|wc -l
    
    awk '{print $7}' /var/log/nginx/access.log|sort | uniq -c |sort -n -k 1 -r|head 10
    
    awk '{print $1}' /var/log/nginx/access.log|sort | uniq -c |sort -n -k 1 -r|head 10

> UV 和 PV：**UV（Unique visitor）** 24小时内的单个自然人， **PV（Page View）** 页面点击量。

* * *

这里学好 `awk` 和 `sed` 和 `xargs` 后面我要开专题。 通过 Shell 的天然的特性。就可以得到响应的内容。

CGI脚本
-----

得到统计提取的脚本之后，这里直接进行通过wsgi实现接口。供前端调用

    import cgi
    import os
    import json
    
    print("Content-Type: application/json")
    print("")
    
    query = {
        'uv':       "awk '{print $1}'  /var/log/nginx/xxxx.xxx.cf.log|sort | uniq -c |wc -l",
        'pv':       "awk '{print $7}' /var/log/nginx/xxxx.xxx.cf.log|wc -l",
        'hoturl':   "awk '{print $7}' /var/log/nginx/xxxx.xxx.cf.log|sort | uniq -c |sort -n -k 1 -r|head",
        'hotip':    "awk '{print $1}' /var/log/nginx/xxxx.xxx.cf.log|sort | uniq -c |sort -n -k 1 -r|head"
    }
    
    resp = dict(zip(query, map(lambda x: os.popen(x).read(), query.values())))
    result = json.dumps(resp)
    print(result)

* * *

前端使用 `AJAX` 进行请求，拉取数据，进行基本处理即可，这里的问题存在：`CC攻击` 的问题，这里直接使用 Shell 调用，如果接口被压测，可能占用大量的 IO资源。所以优化方案是 数据入库。

前端处理
----

前端通过 `AJAX` 对接口动态调用，拉取数据。之后进行处理和动态刷新。

    
        $.get('/get_dbg/').done(function(data) {
                $('#PV').text('站点PV:  '+ data.pv)
                $('#UV').text('站点UV:  '+ data.uv)
                $('#hotip').html('<br>' + data.hotip.replace(/\n/g, '<br>'))
                $('#hoturl').html('<br>' + data.hoturl.replace(/\n/g, '<br>'))
            })；
    

这里进行全局的 `\n` 替换，使内容换行。对页面进行刷新即可。

后
-

这里做了 nginx 的日志的统计，进行对部分服务质量的反馈。