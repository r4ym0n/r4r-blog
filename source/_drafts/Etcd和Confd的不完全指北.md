---
title: Etcd和Confd的不完全指北
url: 1278.html
id: 1278
categories:
  - 未分类
tags:
---

etcd 类似于 Win系统下的注册表，以键值的形式来实现配置文件，或者其他内容的统一管理。

> Distributed reliable key-value store for the most critical data of a distributed system

这一篇对etcd 和 confd这两个组件来进行初步的使用和避坑指北。

简介
--

etcd 和 win中我们熟知的注册表很相似。使用path来作为key，在可以中保存着value的值。用的时候直接对键进行写入即可，内容任意使用字符串来进行保存。

etcd的客户端
--------

在使用 python 的etcd 的模块的时候遇到的几个坑：

### version_prefix

    client = etcd.Client(host='api.example.com', protocol='https', port=443, version_prefix='/etcd')

这个是给出的官方示例，看到这里prefix就觉得这个是键值前面的prefix，如果使用键`/123` 那么对应的就是`/etc/123`，然而在进行尝试之后发现来了这样操作的话返回大量的404的错误。

* * *

在进行资料查询之后了解到，由于 etcd 是基于http的无状态链接。所以这里的version prefix 的确是version相关的，如该前级进行了路由转发，这里的prefix就需要进行调整，类似于一般配置中的 basepath。

### write

    etcd_cli.write('/confcenter/ips/'+ip, json.dumps([]))