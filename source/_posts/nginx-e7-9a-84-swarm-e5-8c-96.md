---
title: Nginx 的 Swarm 化
url: 454.html
id: 454
categories:
  - 未分类
date: 2019-03-13 00:00:00
tags:
---

前
-

这个题目可能显得比较不严谨，但是也想不到什么好的名字。这篇主要就是实现，在一个 Swarm 集群里面的 Nginx 的服务部署。且实现可以很快的进行 文件变更以及配置变更的统一提交。

这里至于之前的对于Swarm 的服务功能的测试不同了，更加偏向应用。再往后一部分，**将会使用stack** 来实现一个真正可用的服务架构。

正文
--

使用Swarm 很容易的实现了一个服务：

    docker service create \
    --replicas 2 \  # 两个副本（一共）
    --network ngx_net \     # 虚拟网络
    --name my-test \        # 服务名
    -p 80:80 nginx          # 端口映射（本地到容器）

但是实际上要对这些服务进行以下内容的变更，就需要挂载**外部卷**了。

### 外部卷的挂载

在创建服务的时候，可以使用以下，命令进行 一个数据卷的创建，以及挂载。而且在服务被创建的时候，这些容器会被拷贝到多个节点上去，命令如下

    # 创建一个挂载卷
    docker volume create --name myvolume
    # 在容器内挂载卷
    docker service create \
    --replicas 2 \
    --network ngx_net \
    --mount type=volume,src=myvolume,dst=/wangshibo \
    --name test-nginx nginx

但是问题来了，如果部署完成之后，对配置文件进行变更，这文件并不会同步到每个节点上。这里打算使用 NFS 进行部署

### NFS 的部署

为了实现多节点的配置文件的同步，以及数据卷的统一管理，这里就直接使用NFS

使用 apt 直接进行服务的安装 ：

    apt-get install nfs-common      # 这个在所有节点进行安装
    apt-get install nfs-kernel-server   # 这里是NFS的服务，安装在服务节点上

配置共享目录 `/etc/exports` ，添加新行。

    /srv/nfs/  bgb1(rw,sync)  bgb2(rw,sync)  bgb3(rw,sync)

重启服务端的nfs服务，并且尝试挂载：

    > service nfs-kernel-server start
    # b 主机上进行挂载
    > mount bgb0:/srv/nfs /mnt/
    > chmod a+x /srv/nfs -R

* * *

测试完成，NFS 部署完成。

### NFS 共享卷挂载

为了实现多个节点之间的文件共享。**这里遇到了很坑的问题，不知道是自己的理解问题还是BUG什么，具体症状是 NFS卷 不进行自动的挂载**。

新建一个NFS的卷，下面命令进行参数修改即可。

    docker volume create --driver local \
    --opt type=nfs4 \
    --opt o=addr=,rw \
    --opt device=: \
    share

* * *

由于上面的问题折腾了很久的时间，只好暂时使用手动进行挂载，后面再进行逐步的研究。

    mount bgb0:/srv/nfs /var/lib/docker/volumes/share/_data
    # 用作页面 /usr/share/nginx/html/
    mount bgb0:/srv/nfs/nginx/html /var/lib/docker/volumes/Nginx_html/_data
    # 用作配置 /etc/nginx/
    mount bgb0:/srv/nfs/nginx/conf /var/lib/docker/volumes/Nginx_conf/_data

和第一部分的创建服务一样，这里直接使用Portainer来进行创建各个节点的卷，**然后手动执行上面的命令，进行NFS的挂载，没有美感**

（应该又是一个 BUG ，在创建服务的时候初始配置，如果直接使用了当前的卷，其默认的的是新建新的卷。而不是对原有的卷来进行拷贝，这点待会可能去提个 Issue 。）**需要在服务创建之后，后面进行第二次的挂载，才能正确的挂载 nginx_html 这个卷**

当全部进行手动挂载之后，所有的配置文件都可可以从NFS主机处获取。实现配置的统一管理

* * *

记得映射 **80 和 443** 完成主机功能配置。

### Nginx 配置

当前面的基于 NFS 的共享配置完成之后，可以直接对之前的配置进行平行迁移即可，（copy-paste）。

完成配置文件的迁移之后，可以直接在终端里面进行平滑升级。

    docker service update WebTest

### frpc 的部署

这里这里可以直接在 DockerHub上面找到相应的镜像，前面还以为没有，打算自己写 Dockerfile的，现在既然遇上了就直接使用了。

> [xddxdd/frpc](https://hub.docker.com/r/xddxdd/frpc/tags) 这个是 frpc 的image ，里面刚好有 armv7 的版本 。

    entry point     /usr/bin/frpc
    working Dir     /frp
    

这里，直接 bind 绑定 host 的目录，到容器内部

    --mount type=bind,src=/conf,dst=/frp
    

主机目录配置文件 `frpc.ini` ，其配内容如下

    [common]
    server_addr = *******
    server_port = 7000
    token = *******
    
    [web_swarm]
    type = tcp
    local_ip = 192.168.***.***
    local_port = 443
    remote_port = ******
    

之后直接对容器进行重启即可。

### 后

这里是在云服务器上使用 Nginx 做一个 upstream ， 具体的细节前面有讲 不在赘述。至此功能测试完毕。

续
-

后面将会使用 Stack 和 network 功能， 实现结构的整体部署。还得解决 NFS 的不自动挂载的问题

> 是不是每个人年轻的时候都有这样一段日子，鸿鹄志高却难遂，迷茫地过着，昏昏噩噩地耗，最终不是妥协泯然众人，就是找不到出口被生活围困。这时候家人朋友，看在眼里，哪怕不说，心里想的也是“小镇青年何必心怀远方”这样的想法吧。（J·M·库切《青春》）