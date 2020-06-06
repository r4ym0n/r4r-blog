---
title: Kubernetes 主机端口暴露以及内部访问
tags:
  - K8S
  - logstash
  - network
url: 1212.html
id: 1212
categories:
  - OP之路
date: 2019-09-21 19:47:44
---

在 K8S 上部署的服务，由于性能的问题，需要从主动的数据拉去，成为被动的端口暴露。 所以怎么暴露端口就是成了一个问题。 和之前使用的swarm的不同，k8s的层次分离的更细一些。不过网络这一部分都是依托在`service`的层次之上

通过service进行端口的暴露
----------------

K8S中，是通过service 这一层次进行网络控制的。可以理解为这个是容器网络和主机之间共有的中间层。 通过service 可以将容器pod的端口暴露在 容器网络内部，或者主机网络。

* * *

这里的配置和说明，主要目的是暴露在主机网络上，（实际上容器网络的暴露方法类似）。 这里直接贴出来配置：

    kind: Service
    apiVersion: v1
    metadata:
      name: logstash-app-fb
      namespace: default
    spec:
      type: NodePort
      ports:
        - port: 7001            # 容器网络的端口
          targetPort: 5045      # 容器内部的端口
          nodePort: 17001       # 主机网络的端口
      selector:
        app: logstash-app-fb    # 关键的selector

通过配置的里的注释可以直接看到，这里有几个关键的配置。关于三个端口就不多讲了，需要进行一一的对应，`5045`需要和容器的开放的端口一致，`7001` 是在容器网络内部开放的端口（他的使用在后面会写上），nodePort就是我们的主机网络中需要进行暴露的端口了（也就是直接使用物理节点的IPport可以进行访问的）

这里最为关键部分，就是这里的selector，作为选择器，他决定了哪些deployment会被加入这个网络。在这里例子里，所有的metadata中app和 `logstash-app-fb` 一致的部署，将会被加入这个网络。metadata 配置如下：

    metadata:
      name: logstash-app-fb
      namespace: default
      labels:
        app: logstash-app-fb

一旦加入网络之后，所有的请求会被k8s的内部的调度器根据配置的策略来进行负载的均衡。

* * *

**这样的好处：** 这里引用官方的文档中的话，具体的网络，单大意如下：

> 在这种微服务的架构下，不要希望容器是稳定的，因为他可能随时因为异常而退出，但是加入了service这一个层次，可以把容器抽象为一个服务。这个服务对外来说是稳定的。

通过 servicename 来进行内部网络访问
------------------------

有了上面的那句话的思想，实际上可以推导出来，在k8s的内部网络中，其访问的单元也应该是大于单个容器的。 以上述的配置为例，暴露的7001的端口，就是k8s网络中的内部端口。

k8s有自己的名字服务，管理着所有的注册的service的IP。所以，我们可以直接使用 servicename:port 来对我们的服务进行k8s网络内的访问，很大的简化了各个模块之间的调用。**在容器内部使用 bash 执行**，示例：

    bash-4.2$ curl -v logstash-app-fb:7001
    * About to connect() to logstash-app-fb port 7001 (#0)
    *   Trying 10.97.93.0...
    * Connected to logstash-app-fb (10.97.93.0) port 7001 (#0)
    > GET / HTTP/1.1
    > User-Agent: curl/7.29.0
    > Host: logstash-app-fb:7001
    > Accept: */*
    > 
    * Recv failure: Connection reset by peer
    * Closing connection 0
    curl: (56) Recv failure: Connection reset by peer

可见，其端口在容器网络内部，是之间可以通过service来进行访问的。