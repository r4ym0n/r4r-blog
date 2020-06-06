---
title: OpenFaas 进阶
tags:
  - openfaas
url: 1256.html
id: 1256
categories:
  - DEV
---

在前面一篇基础尝试与部署的的post之后，现在进一步用函数服务的思想来构建功能。

> [http://blog.12ms.xyz/2019/03/19/openfaas-的部署](http://blog.12ms.xyz/2019/03/19/openfaas-的部署)

访问物理主机的端口
---------

在Swarm网络中，物理主机上有网卡`docker0`，该网卡上所分配的IP，和本地的环回`127`一致。 一般分配的ip为`172.17.0.1`，当作和`127.0.0.1`一致进行访问即可

handle 怎么获取Http的请求头
-------------------

具体的内容可以看这篇：

> [Lab 4 - Go deeper with functions](https://github.com/openfaas/workshop/blob/master/lab4.md#inject-configuration-through-environmental-variables)

Http的请求在全局上以及被解析为环境变量，获取请求头的紫的，直接getenv即可：

    path = os.getenv("save_path", "/home/app/")