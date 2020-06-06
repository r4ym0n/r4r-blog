---
title: Docker集群搭建
url: 458.html
id: 458
categories:
  - 未分类
date: 2019-03-05 00:00:00
tags:
---

快速部署
----

### Docker 的安装

Docker 的安装，这里直接使用官方提供的一键安装的脚本。一路绿灯

    curl -sSL https://get.docker.com/ | sh
    
    curl -sSL https://get.daocloud.io/docker | sh

### 创建Swarm集群

直接使用 Docker 的 `swarm` 来进行集群的创建。直接通过help可以查看相关的参数

    # 初始化 Swarm 集群
    docker swarm init --advertise-addr 192.168.xxx.xxx  # 这里是主机的地址 注意保持静态 IP
    
    # 得到 Worker 的join-token
    docker swarm join-token worker
    
    # 输入join之后直接加入集群
    docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    6km4644t62w3h3g7iujceerl1 *   bgb1         Ready               Active              Leader              18.06.3-ce
    kwi9t231ulc2dry75zzg52580     bgb2         Ready               Active                                  18.06.3-ce

### Portainer 服务部署

    #拉镜像
    docker pull portainer/portainer
    # 创建挂载卷
    docker volume create portainer_data
    
    # 创建挂载目录,不然后面挂载会报错
    mkdir -p /opt/portainer
    
    docker service create \
    --name portainer \
    --publish 9000:9000 \
    --replicas=1 \
    --constraint 'node.role == manager' \
    --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
    --mount type=bind,src=//opt/portainer,dst=/data \
    portainer/portainer \
    -H unix:///var/run/docker.sock
    
    # 列出所有的服务
    docker service ls
    ID                  NAME                MODE                REPLICAS            IMAGE                        PORTS
    plmkdifqivcs        mAgent_agent        global              4/4                 portainer/agent:latest       

* * *

这里注意的是，这里的部署是服务部署，而不是单个容器的部署。

容器的部署是针对与单个节点的。在上面的部署方式也可以使用：

    docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

这里的问题是，这个命令只是在单主机的 Node 上对该镜像进行了部署。如果进行服务的访问，**只能通过该主机的Hostname** 进行访问。

但是一旦进行了一个服务的部署，那么所有的节点IP，都会做一个统一的映射，继而都可以对该服务进行访问。（这里选择的网络接口是 Ingress ，直接和主机网络共享，并且在服务的子网内，这些容器是自动进行负载均衡的。全自动的！

至此，整个 **Swarm** 集群搭建完成，并且使用了轻量级的容器管理系统 **Portainer** 。

笔记s
---

这里，自己折腾了一下简单的部署，实际的应用后面还有很长很长的路要走。这个section记录一下途中遇上的坑：

*   Manager节点需要配置静态 IP ，因为 advertise 的地址是固定的，否则会发现IP改变之后节点全部 Down 掉。
*   在跑 Portainer 的注意检查其挂载的目录是否真实存在，否则报错。（bind，和mount）
*   Agent 的部署好像很吃内存。
*   脚本安装Docker 的时候，被CDN坑，没办法

资料：
---

*   [Docker Swarm 深入浅出](https://www.bookstack.cn/read/docker-swarm-guides/README.md)
*   [Use bind mounts](https://docs.docker.com/storage/bind-mounts/)