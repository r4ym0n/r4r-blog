---
title: 基于Helm的wordpress的快速部署
tags:
  - Docker
  - helm
  - K8S
  - wordpress
url: 1445.html
id: 1445
categories:
  - OP之路
date: 2019-11-25 00:51:20
---

这篇文章是很实战的一篇，使用 `helm`来快速搭建一个 wordpress 站点。

> 这篇文章已经在draft 里面待了快俩星期了。最近又是各种人生迷茫？

很好的参考文章：

> [https://www.sunmite.com/docker/kubernetes-install-helm-and-wordpress.html](https://www.sunmite.com/docker/kubernetes-install-helm-and-wordpress.html)

这一篇文章记录了主要的过程 但是也是存在这不少的坑。

准备
--

### 搜索相关的chart

使用 helm 来进行 相关 的Chart 的搜索，可以看到当期的chart 的发行的版本

    ➜  k8s helm search wordpress
    NAME                CHART VERSION   APP VERSION DESCRIPTION
    stable/wordpress    7.6.7           5.2.4       Web publishing platform for building blogs and websites.

### 查看 chart的Doc

helm 的部署的过程实际上是使用预设的模板以及配置来进行整个服务的部署，使用 `helm inspect` 来查看这个 chart 的默认参数。

     helm inspect value stable/wordpress

看到 `wordpress` 的默认的配置

    | wordpressUsername                       | User of the application                                                       | user                                                       |
    | wordpressPassword                       | Application password                                                          | _random 10 character long alphanumeric string_               |
    | wordpressEmail                          | Admin email                                                                   | user@example.com                                           |
    | wordpressFirstName                      | First name                                                                    | FirstName                                                  |
    | wordpressLastName                       | Last name                                                                     | LastName                                                   |
    | wordpressBlogName                       | Blog name                                                                     | User's Blog!                                               |
    

### 特色问题 网络

由于无法和 `helm`默认的仓库通信，所以这里需要在 `helm`的出适合的时候指定国内的镜像源。

    helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.12.2 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

### tiller 容器服务安装

Error: could not find a ready tiller pod #2064

> [https://github.com/helm/helm/issues/2064](https://github.com/helm/helm/issues/2064)

### tiller 服务 端口转发报错

    E1112 20:50:07.137974    8960 portforward.go:400] an error occurred forwarding 36910 -> 44134: error forwarding port 44134 to pod 7cec34191bfb05d735558f58c6aee548fd2185ad01f0096835afc5042cfe5ee2, uid : container not running (7cec34191bfb05d735558f58c6aee548fd2185ad01f0096835afc5042cfe5ee2)
    ###########

    ➜  ~ kubectl -n kube-system port-forward svc/tiller-deploy 44134:44134
    Forwarding from 127.0.0.1:44134 -> 44134
    Forwarding from [::1]:44134 -> 44134

安装
--

### service用户角色问题

[https://github.com/helm/helm/issues/3130](https://github.com/helm/helm/issues/3130)

在直接使用`helm`来对wordpress来进行安装，发现会返回权限错误。

    ➜  k8s helm install --name wp-test stable/wordpress
    Error: release wp-test failed: namespaces "default" is forbidden: User "system:serviceaccount:kube-system:default" cannot get resource "namespaces" in API group "" in the namespace "default"

这里是没有创建HELM对应的role导致的权限谁是，所以需要给helm 来创建角色，参看的yaml文件如下：

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: tiller-clusterrolebinding
    subjects:
    - kind: ServiceAccount
      name: tiller
      namespace: kube-system
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: ""

在进行初始化的时候指定`sesrvice-account` 为我们建立的 tiller，这样就可以解决前面的 `forbidden` 的问题。

解决方法在init的时候指定 `--service-account`

    helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts --service-account tiller

### 建立PV卷

在helm安装的默认配置里面的是没有配置 Pv 的，所以导致我们在进行挂载的时候会保持在pending的情况。descrice 之后可以在event中看到是在等待合适的挂载的PV

    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mariadb-pv
      labels:
        app: mariadb
        component: master
        heritage: Tiller
        release: wordpress
    spec:
      capacity:
        storage: 10Gi
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      storageClassName: nfs
      nfs:
        server: sync.gov
        path: "/volume1/docker/wp-post/mariadb-pv/"
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: wordpress-pv
      labels:
        app: wordpress
        chart: wordpress-7.6.7
        heritage: Tiller
        release: wordpress
    spec:
      capacity:
        storage: 10Gi
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      storageClassName: nfs
      nfs:
        server: sync.gov
        path: "/volume1/docker/wp-post/wordpress-pv/"
    

### helm 安装 WP

在前面的配置完成之后，直接使用 helm来进行安装，等待 pod 的状态

    helm install --name wordpress stable/wordpress --set global.storageClass="nfs"

在部署完成来desc一线这个service ，可以看到默认的nodeport来转发规则

     kubectl describe svc  wordpress

参考
--

*   [helm 的其他chart](https://github.com/helm/charts/tree/master/stable/wordpress)
*   [是时候使用Helm了：Helm, Kubernetes的包管理工具](https://www.kubernetes.org.cn/3435.html)
*   [Helm User Guide - Helm 用户指南](https://whmzsu.github.io/helm-doc-zh-cn/)
*   [Connection refused error when issuing helm install command #3460](https://github.com/helm/helm/issues/3460)