---
title: OpenFaaS 的部署
url: 446.html
id: 446
categories:
  - 未分类
date: 2019-03-19 00:00:00
tags:
---

前面的话
----

本来说项目完结的转身离去，不过这样的一个平台带来的 **从未体验过的全新感觉** 实在是震惊呢。这一片再开新坑，来部署一个 **serverless** 思想的 OpenFaaS平台

> FaaS = Function as a Service

这篇操作主要参考文章：

> *   [Your Serverless Raspberry Pi cluster with Docker](https://blog.alexellis.io/your-serverless-raspberry-pi-cluster/)
> *   [使用 Docker 构建你的 Serverless 树莓派集群](https://linux.cn/article-9007-1.html)（上文译版）
> *   [OpenFaaS快速入门指南](https://jimmysong.io/posts/openfaas-quick-start/)
> *   [什么是Serverless（无服务器）架构？](https://jimmysong.io/posts/what-is-serverless/)

部署过程使用一键安装脚本，感觉有点点惭愧，不过体验功能才是重点。不重复造轮子问题不大。这篇打算主要写，这个 OpenFaaS 的架构的组成，以及后面的基础使用。

关于 OpenFaaS
-----------

### 什么是OpenFaaS

和我们熟知的 **IaaS，PaaS，SaaS** 类似，这里指的是云的三大层次。如果不是很清楚可以看这个 ：

> *   [IaaS，PaaS，SaaS 的区别 - ruanyf](http://www.ruanyifeng.com/blog/2017/07/iaas-paas-saas.html)
> 
> 今天大多数公司在开发应用程序并将其部署在服务器上的时候，无论是选择公有云还是私有的数据中心，都需要提前了解究竟需要多少台服务器、多大容量的存储和数据库的功能等。并需要部署运行应用程序和依赖的软件到基础设施之上。假设我们不想在这些细节上花费精力，是否有一种简单的架构模型能够满足我们这种想法？

函数及服务，在这里是Serverless的一种实现，这里的 Serverless 可以简单的理解为，服务器的服务应用都被**打包成了一个个独立的容器**。这种情况下可以很容易的进行全自动的横向扩容。

> 传统的服务器端软件不同是经应用程序部署到拥有操作系统的虚拟机或者容器中，一般需要长时间驻留在操作系统中运行，而FaaS是直接将程序部署上到平台上即可，当有事件到来时触发执行，执行完了就可以卸载掉。

* * *

就可以使用 类似于 `http://bgb0:8080/function/nslookup` 这样的URL来对容器服务产生一个请求，而不用再考虑后端的各种的体系和流程。

OpenFaaS部署
----------

### FaaS 服务部署

    # 这里使用现有的项目一键部署
    git clone https://github.com/alexellis/faas/
    cd faas
    ./deploy_stack.sh
    
    # 客户端安装 fssa-cli 用于构建
    curl -sSL cli.openfaas.com | sudo sh

对于 OpenFaaS 的部署，上面有给出的一键部署的的脚本，在x86和Arm上都可以进行快速的部署。

    root@vm0:~# docker stack services func
    ID                  NAME                MODE                REPLICAS            IMAGE                                 PORTS
    joj44r8vpaf4        func_faas-swarm     replicated          1/1                 openfaas/faas-swarm:0.6.1-armhf       
    mjnrabl3d8sj        func_nats           replicated          1/1                 nats-streaming:0.11.2                 
    mo8klvy4id9n        func_queue-worker   replicated          1/1                 openfaas/queue-worker:0.6.0-armhf     
    phqan7977dgi        func_alertmanager   replicated          1/1                 functions/alertmanager:0.15.0-armhf   
    sjbxputi1xe8        func_prometheus     replicated          1/1                 functions/prometheus:2.7.0-armhf      *:9090->9090/tcp
    z6pc60t4b684        func_gateway        replicated          1/1                 openfaas/gateway:0.11.0-armhf         *:8080->8080/tcp

在部署完成之后，可以看到有这些服务组成了这个 Stack

*   func_gateway 管理 Function 的前端页面
*   func_prometheus 数据统计，以及告警的平台，用于检测服务状态来来实现系统扩容。

### Grafana 监控部署

这里使用了prometheus作为数据收集和数据源和Grafana很好的兼容。直接可以作为Dashboard 的数据源。这里已经有了 FaaS 的模板的Json配置文件：

> [https://grafana.com/dashboards/3526](https://grafana.com/dashboards/3526)

在对Dashboard导入后，有统计调用频率，函数副本数，调用次数，调用比例。

OpenFaaS.HelloWorld()
---------------------

由于是 ARM 的平台，所以导致可以直接拿来使用的镜像不是很多。其自带的 **nodeinfo** 和 **nslookup** 。这里如果要实现一个hello world 需要自己去构建函数镜像。直接使用 `faas-cli` 先生成模板。

    faas-cli new --lang python hello-python

然后回自动的构建一个python 的函数容器的模板：

    root@bgb0:~/functions# tree . -L 2
    .
    |-- hello-python
    |   |-- handler.py          # 修改
    |   `-- requirements.txt
    |-- hello-python.yml        # 修改
    |-- pass
    `-- template
        |-- csharp
        |-- csharp-armhf
        |-- dockerfile
        |-- go
        |-- go-armhf
        |-- java8
        |-- node
        |-- node-arm64
        |-- node-armhf
        |-- php7
        |-- python
        |-- python-armhf        # 拷贝到上级
        |-- python3
        |-- python3-armhf
        `-- ruby

然后开始写接口，这里直接编辑 `handler.py`。

    def handle(req):
        """handle a request to the function
        Args:
            req (str): request body
        """
        print("Hello! This is a test: " + req)
        return req

修改之后，执行构建命令，有二十几个step。。。

    faas-cli build -f ./hello-python.yml

> If you're trying thing out on a single host, then you don't need to push your images to a registry, they'll just be used from the local Docker library.

为了让其他的容器拉到镜像，这里需要 push 镜像上去。

后
-

今天变成 `高盐度低湿度常见水产品`