---
title: TabClosing 3
tags:
  - Docker
  - helm
  - kubernetes
  - Nginx
url: 1534.html
id: 1534
categories:
  - 每周分享
date: 2019-12-15 03:18:13
---

一样的建立一个 cron 吧，每周一次来把有意思的链接收集归档一下。

好玩的
---

*   [网页时光机](https://archive.org/web/)
*   [pimeyes-根据照片找视频](https://pimeyes.com/en/)

书
-

*   [Welcome to Heptio Docs!](http://docs.heptio.com/index.html) Welcome to Heptio’s documentation, a resource for Kubernetes admins and users. 一些基础实践的指引，比如run a custom LAMP application on kubenates. 可以很好的体验什么是代码即架构
*   [Helm User Guide - Helm 用户指南](https://whmzsu.github.io/helm-doc-zh-cn/)
*   [Nginx开发从入门到精通](http://tengine.taobao.org/book/index.html)

K8S
---

*   [Example: Deploying WordPress and MySQL with Persistent Volumes](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/) 官方上提供了wordpress和mysql的示例的deployment
*   [Kubernetes中的Volume介绍](https://jimmysong.io/posts/kubernetes-volumes-introduction/)
*   [Docker MySQL最佳实践？](https://liangjun.work/2019/08/03/1448bd5b/)
*   [kubernetes service external ip pending](https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending) It looks like you are using a custom Kubernetes Cluster (using minikube, kubeadm or the like). In this case, there is no LoadBalancer integrated (unlike AWS or Google Cloud). With this default setup, you can only use NodePort or an Ingress Controller.
*   [五分鐘 Kubernetes 有感](https://medium.com/@evenchange4/%E4%BA%94%E5%88%86%E9%90%98-kubernetes-%E6%9C%89%E6%84%9F-e51f093cb10b)
*   [Tutorial: Run WordPress with Helm on Kubernetes](http://docs.heptio.com/content/tutorials/aws-qs-helm-wordpress.html) 比较标准的helm部署wordpress的文章，可以少不少坑
*   [Kubernetes 的基本概念和术语](https://www.sunmite.com/kubernetes/basic-concepts-and-terms-of-kubernetes.html) Kubernetes 为每个 Pod 都分配了一个唯一的 IP 地址，称之为 Pod IP，一个 Pod 里的多个容器共享 Pod IP 地址。 StatefulSet 里的每个 Pod 都有稳定、唯一的网络标识，可以用来发现集群内的其他成员 Kubernetes 的 Service 定义了一个服务的访问入口地址 Annotation 与 Label 类似，也使用 key/value 键值对的形式进行定义。不同的是 Label 就有严格的命名规则，它定义的是 Kubernetes 对象的元数据，并且用于 Label Selector。Annotation 则是用户任意定义的附加信息，以便于外部工具查找
*   [Kubernetes的三种外部访问方式：NodePort、LoadBalancer 和 Ingress](http://dockone.io/article/4884) ClusterIP 集群内部节点可直接访问 NodePort 每个节点需要暴露统一的端口 LoadBalance 有自己独立的IP Ingress 针对HTTP层面进行转发
*   [玩K8S不得不会的HELM](https://juejin.im/post/5d5b8dba6fb9a06b1b19c21b) 一篇很好的helm的从入门到进阶的文章
*   [k8s v1.13.2 安装排错日志](https://www.jianshu.com/p/98c22ca2d9ab)

其他
--

*   [星际文件系统](https://zh.wikipedia.org/wiki/%E6%98%9F%E9%99%85%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F) 提前关注一下吧，感激他又开始火起来了。
    
    > 它是一种内容可寻址的对等超媒体分发协议。在IPFS网络中的节点将构成一个分布式文件系统。