---
title: Docker Registry 搭建
tags:
  - Docker
url: 1354.html
id: 1354
categories:
  - OP之路
date: 2019-11-02 17:53:41
---

在本地进行镜像构建的话，由于特色的互联网政策，与 docker hub 的连通率是在的糟糕，所以只能想办法自己搭建一个 Docker的私有仓库，用来保存自己构建的镜像，也可以作为外部镜像的中转站。在拉取 k8s 的 dashboard 的时候，镜像根本拉不下来，所以只能在 海外的主机上进行中转。

这一篇就记录一下搭建的过程，使用官方的镜像，很快的搭建一个 私有的 仓库。

拉取镜像
----

docker hub 的直连速度实在不行，这里推荐使用 [Daocloud 的加速方案](https://www.daocloud.io/mirror#accelerator-doc) ，配置过程十分简单，一行命令搞定，实际上是修改了默认的 registry

    curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

在修改完了deamon的文件之后，重启docker的服务使配置的 默认源生效。 之后直接拉取官方的镜像即可

    docker pull registry

镜像部署
----

由于已经打包好了应用的镜像，在进行部署的时候只需要考虑两点，端口的映射和数据的持久化。所以直接执行启动命令：

     docker run -d \
        -p 15000:5000 \
        -v /opt/data/registry:/var/lib/registry \
        myregistry

（容器本身是不可靠的，但是服务是可靠的所以，最好的方法是使用服务的方式进行部署） 这样很快的在本机的 15000端口开启了docker的registry。

推送镜像
----

由于这样直接部署的仓库是 http的，所以还需要在docker的配置中指定是不安全的 http，否则默认的话会使用 https 来进行操作，返回 400 bad request。修改之后的domean 的文件如下：

    {"registry-mirrors": ["http://f1361db2.m.daocloud.io"],
        "insecure-registries": ["127.0.0.1:15000"]
    }

这里可以对 配置 hosts，这样的话，美观一点。（实际上这里可以用 host加上nginx来把端口降为80） 这些都是可优化的地方。

* * *

现在就开始 push 镜像了，首先需要把 官方的镜像重新tag一下，之后直接进行推送：

    # 重新tag 镜像，指定仓库
    docker tag k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1 127.0.0.1:15001/kubernetes-dashboard-amd64:v1.10.1
    # 对镜像进行推送
    docker push 127.0.0.1:15001/kubernetes-dashboard-amd64:v1.10.1
    
    ➜  ~ curl 127.0.0.1:15001/v2/_catalog
    {"repositories":["kubernetes-dashboard-amd64"]}

拉取镜像
----

完成上面的操作之后，镜像已经被推送到了我们的自己的仓库中了，这样我们回到本地主机进行拉取。 一样的先对docker的配置进行修改，和上面一样，信任http。

    {"registry-mirrors": ["http://f1361db2.m.daocloud.io"],
        "insecure-registries": ["127.0.0.1:15000"]
    }

直接拉取镜像，并且重新的打tag

    docker pull sync.gov:15000/kubernetes-dashboard-amd64:v1.10.1
    docker tag sync.gov:15001/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1

至此完成了 私有docker镜像仓库的搭建以及 镜像的中转。

EOF