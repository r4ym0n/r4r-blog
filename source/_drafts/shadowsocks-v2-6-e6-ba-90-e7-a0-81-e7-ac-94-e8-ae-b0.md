---
title: Shadowsocks v2.6 源码笔记
url: 1361.html
id: 1361
categories:
  - 未分类
tags:
---

Shadowsocks 是和生活比较相关的工具了，用过之后不妨去了解一下他的原理，为后面对抗AI流量规则识别来打下基础。

写在前面的话
------

在开始看源码之前，对结果进行可能的猜想，主题的功能应该是分为两块，正向代理，以及加密。如果没有 GFW的存在，http 的透明代理，以及可以解决我们的需求了。但是由于审查策略，我们的流量必须伪装再伪装，让它无法知道我是谁，在干什么，这样就实现了对墙本身的穿越。

开始
--

这里的源码是选择的在github看到的最老的 python的版本 ：

> github: [https://github.com/shadowsocks/shadowsocks/archive/2.6.zip](https://github.com/shadowsocks/shadowsocks/archive/2.6.zip) 本地下载: [https://blog.12ms.xyz/wp-content/uploads/2019/11/shadowsocks-2.6.zip](https://blog.12ms.xyz/wp-content/uploads/2019/11/shadowsocks-2.6.zip)

先对文件的目录结构研究一下，主要的app文件在 shadowsocks这个目录下 ：

    shadowsocks
    ├── __init__.py
    ├── asyncdns.py
    ├── common.py
    ├── crypto
    │   ├── __init__.py
    │   ├── ctypes_libsodium.py
    │   ├── ctypes_openssl.py
    │   ├── m2.py
    │   ├── rc4_md5.py
    │   ├── salsa20_ctr.py
    │   ├── table.py
    │   └── util.py
    ├── daemon.py
    ├── encrypt.py
    ├── eventloop.py
    ├── local.py
    ├── lru_cache.py
    ├── server.py
    ├── tcprelay.py
    ├── udprelay.py
    └── utils.py

根据 `test.py` 这个测试用例，得到main在 `local.py`和`server.py`中，分别是我们的客户端和服务端。