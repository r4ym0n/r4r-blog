---
title: Python 的几个小tip
tags:
  - python
url: 788.html
id: 788
categories:
  - DEV
date: 2019-04-14 15:29:56
---

前
-

这个是翻 《深度实践OpenStack》这本书里面看到的，了这里总结一下。原书写的太松散了，没有重点

xrange
------

一般都习惯于使用 range ，但是pyhton的原则是 在数据数量小于5的时候使用 `range()` ，当数据量大于5的时候使用 `xrange()` ，后者应该是个生成器，大幅降低内存消耗。

推导试
---

`[x for x in range]`

SQLAlchemy
----------

使用 ORM 来进行数据库的操作，比那种裸写 SQL 是要优雅不少的\[\]()

logging
-------

Python 的一个很好的实现日志分级和输出的模块

Eventlet
--------

高性能网络库，实现异步网络通信