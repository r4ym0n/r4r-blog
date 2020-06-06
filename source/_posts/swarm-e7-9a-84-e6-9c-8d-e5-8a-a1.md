---
title: Swarm的服务
url: 456.html
id: 456
categories:
  - 未分类
date: 2019-03-12 00:00:00
tags:
---

前面的话
----

上一篇简单快速的进行swarm集群的部署，这可以开始使用了。这里试着部署一个 Nginx 的服务，并且一步步实现前面的高可用的服务架构。

> [Nginx image 地址](https://hub.docker.com/_/nginx)

这里推荐一个博客：

> [散尽浮华 \- 安寻安放，不卑不亢；重剑无锋，大巧不工！](https://www.cnblogs.com/kevingrace/)

里面很多很多的干活，实用型的文章

概念
--

*   多个容器的统一为服务
    
*   多个服务的统一为栈
    
*   服务内自动有VIP机制，VIP 为各个节点的IP
    
*   > docker service其实不仅仅是批量启动服务这么简单，而是在集群中定义了一种状态。Cluster会持续检测服务的健康状态并维护集群的高可用性。
    

基础
--

Swarm 作为 Docker 的原生功能，其主要的工具有以下：

*   docker swarm 用于集群管理
*   docker service 用于服务创建
*   docker node 用于节点管理

实践
--

### 部署服务

这里简单的基于 Swarm 部署一个 Nginx 的服务。（其实这里本来应该有详细的说明的，留在下篇吧）

    docker network create -d overlay ngx_net
    # 查看docker网络
    docker network ls
    
    docker service create \
    --replicas 2 \  # 两个副本（一共）
    --network ngx_net \     # 虚拟网络
    --name my-test \        # 服务名
    -p 80:80 nginx          # 端口映射（本地到容器）
    
    docker service ps   # 查看运行的所有的服务
    
    docker ps           # 查看节点的运行的容器

### 服务扩缩容

Swarm 的一大亮点，就是可以很灵活的进行服务的扩缩容。可以很方便的调整服务的对应的容器的数量。

    docker service scale my-test=1
    docker service scale my-test=5

### 数据的持久化

容器和镜像的区别在与其可读性，镜像可以理解为只读的容器。为了数据的持久化，数据应该挂在在硬盘之上，而不是容器智能，否则容器销毁，数据将会消失。

这里主要有两种办法：

*   bind 这里是直接绑定硬盘上的目录和容器内的目录。
*   volumes 在主机上建立相应的目录，可以进行统一的管理。

    # 创建一个挂载卷
    docker volume create --name myvolume
    # 在容器内挂载卷
    docker service create \
    --replicas 2 \
    --network ngx_net \
    --mount type=volume,src=myvolume,dst=/wangshibo \
    --name test-nginx nginx
    
    # 这里在容器内执行shell，可以看到文件已经同步
    docker exec -ti 3618e3d1b966 /bin/bash

* * *

这里再折腾的时候，遇到了很大的问题，后面看到发现是swarm 的特性。当服务有多个副本的时候，在创建的时候，的确会在manager界面对目录进行拷贝。但是，后续再对manager上的文件进行修改，这些修改将不会被同步到各个节点上。需要一个个的进行修改很是麻烦，后面可以试着使用 **NFS** 实现多个镜像的配置文件的统一。

参考
--

> *   [Docker管理工具-Swarm部署记录](https://www.cnblogs.com/kevingrace/p/6870359.html)

后面的话
----

> 当你发现自己的才华撑不起野心时，就请安静下来学习吧