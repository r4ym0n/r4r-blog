---
title: 使用 Sealos 的Kubernetes快速部署指南
tags:
  - Docker
  - K8S
url: 1357.html
id: 1357
categories:
  - OP之路
date: 2019-11-02 23:26:43
---

在自己使用 esxi 来进行多节点虚拟化的基础上来进行 k8s 的部署。其实早都想从原始的 swarm来切换到 k8s来，最近才完成这个，这里做一个记录。以便后面可以按图索骥。

具体方法
----

这里一样使用的是开包即用的方法，使用 [sealos](https://github.com/fanux/sealos) 来进行一件的快速部署。

    # 下载离线包
    wget https://github.com/fanux/sealos/releases/download/v2.0.7/sealos && \
        chmod +x sealos && mv sealos /usr/bin 
    # 直接进行安装
    sealos init --passwd YOUR_SERVER_PASSWD \
        --master 192.168.0.2  --master 192.168.0.3  --master 192.168.0.4  \
        --node 192.168.0.5 \
        --pkg-url https://sealyun.oss-cn-beijing.aliyuncs.com/37374d999dbadb788ef0461844a70151-1.16.0/kube1.16.0.tar.gz \
        --version v1.16.0

这里依托于了工具本身，所以部署的过程显得和谐切顺利。

如果需要移除 k8s 的本身，执行以下命令：

    sealos clean \
        --master 192.168.0.2 \
        --master 192.168.0.3 \
        --master 192.168.0.4 \
        --node 192.168.0.5 \
        --user root \
        --passwd your-server-password