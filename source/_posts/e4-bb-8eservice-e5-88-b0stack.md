---
title: 从Service到Stack
url: 452.html
id: 452
categories:
  - 未分类
date: 2019-03-14 00:00:00
tags:
---

前
-

本来想着集群稳定呢，结果早上六点docker 的网络就突然挂了。默认网关除了问题，ping 外网不可达。导致远程主机域名无法解析，`8.8.4.4:53` 不可达，整体服务不可用。

这篇，接着上面的内容，实现 frp 和 Nignx 的服务的Stack 化。前面遇到的问题，后面在找个整体的时间进行解决吧。

架构描述 以及 Compose实现
-----------------

### 描述

这里主要是使用 frp 和 Nginx ，前者用于穿透，后者进行页面的反代。所以这里的就是两层的服务。应该还有一级提供负载均衡的server，不过Swarm 的本身已经提供了 VIP 以及负载均衡的功能。

### Docker-Compose

docker-compose 是用来做docker 的多容器控制，可以通过一个描述文件，来对服务，或者容器，进行整体的搭建，大大简化了操作流程以及可维护和可拓展性。这里的内容参考了网上的例子，进行了简单的修改。实现了前面所描述的架构。

具体的 `.yml` 文件如下

    version: "3.2"
    services:
      nginx:
        image: nginx
        ports:
          - "80"    # 对外暴露端口
        volumes:
          - nginx_conf:/etc/nginx
        networks:
          - backend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
    
      frp:
        image: xddxdd/frpc:arm32v7
        networks:
          - backend
        volumes:
         - frpc_ini:/frp
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
    networks:
      backend:
    # 这里是应该建立的卷
    volumes:
      nginx_conf: 
        driver_opts:
          type: "nfs4"
          o: "addr=192.168.1.130,nolock,soft,rw"
          device: ":/srv/nfs/nginx/conf"
      frpc_ini: 
        driver_opts:  
          type: "nfs4"
          o: "addr=192.168.1.130,nolock,soft,rw"
          device: ":/srv/nfs/frpc"

一个好消息：在这种配置的情况下比较奇迹的是可以进行 **NFS 的自动挂载**了。解决了前面一个很大的问题。也不清楚是之前的BUG 还是什么情况。

> Compose下面使用nfs : [How to directly mount NFS share/volume in container using docker compose v3](https://stackoverflow.com/questions/45282608/how-to-directly-mount-nfs-share-volume-in-container-using-docker-compose-v3)

* * *

这里**有一个坑**。在Portainer 上面直接进行 Stack 部署的话，会有报错，说是平台不支持，但是的确是拉的arm的。报错信息如下：

    State Message   pending task scheduling
    Error message   no suitable node (unsupported platform on 3 nodes; 1 node not available for new tasks)

查证之后，得到以下的回答

> 参考链接 = [no suitable node - unable to deploy image using docker service](https://stackoverflow.com/questions/48962399/no-suitable-node-unable-to-deploy-image-using-docker-service)

在命令行里进行创建，带一个 `--resolve-image never` 的参数，这样可以解决报错：**镜像平台不支持的问题** 但前提是，镜像的架构必须相同。

    docker stack deploy --compose-file str.yml --resolve-image never home

这里有一个 Issue ,在进行 nfs 的挂载部分,可能会出现 ,`chmod` 的permission deny,试着以下命令。

    chmod a+x /srv/nfs -R

项目细节
----

这里需要解决的问题，就是网络的问题。在前面的方法里面是直接把端口暴露在了 HOST 上面，这样占用主机资源显然是不符合**容器化的思想**。这里就使用Docker 的强大的网络功能，来实现一个真正的服务。swarm 的服务网络其自动带有了 负载均衡 以及 **VIP**。所以，这里实现目标就是 FRP 到 VIP的映射

### Stack的内部网络

Swarm 的自建 Layout 网络默认的是VIP模式，可以inspect服务得到服务所在网络的VIP。这个服务，跑了 Nginx 镜像的多个副本。这些副本共用一个 VIP。实现了负载均衡以及高可用，当单容器挂了之后，自动的进行 IP 的漂移。

获取 service 的VIP 如下：

    docker service  inspect nginxs
    
    "Endpoint": {
        "Spec": {
            "Mode": "vip"
        },
        "VirtualIPs": [
            {
                "NetworkID": "ktsw8uaob2n0ppsh93p5upils",
                "Addr": "10.0.14.9/24"
            }
        ]
    }
    

由于前面已经配置了，这个 Stack ，两个服务是在同一个子网（backend）中。所以 ，frp服务是和nginx 的网络是联通的。所以可以直接访问 其VIP。由于，好像使用stack了NFS 的问题自动解决了。这里直接修改主机上的配置文件即可。

    root@bgb0:/tmp# cat /srv/nfs/frpc/frpc.ini 
    [common]
    server_addr = ****.****.xyz 
    server_port = ****
    token = myfrptest
    [web_swarm]
    type = tcp
    local_ip = 10.0.14.9    # 这里是 Nginx服务的VIP
    local_port = 443
    remote_port = ****

之后，更新该服务，重新加载配置。发现外网服务可达，已实现目标功能。开心

参考
--

> *   [Docker Swarm 入门：Service Network 管理](https://www.jianshu.com/p/60bccbdb6af9) 通过这篇，对网络有了基本了解 （小姐姐好看）
> *   [Docker Swarm 服务发现和负载均衡原理](https://www.jianshu.com/p/dba9342071d8)
> *   [docker swarm 集群及多主机overlay网络测试](http://www.huilog.com/?p=1038)
> *   [“三剑客”之Swarm应用数据持久化管理（volume 、bind 、 nfs）](https://blog.51cto.com/ganbing/2091292)

后面的话
----

构思基本已经实现了，一个基于容器技术的服务部署。后面，想着对系统的整体的日志监控，依旧使用前面的ELK。完成之后，算是项目收工，后面开始Golang了