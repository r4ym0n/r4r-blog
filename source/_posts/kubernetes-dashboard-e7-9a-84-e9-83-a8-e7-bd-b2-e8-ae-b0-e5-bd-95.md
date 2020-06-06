---
title: Kubernetes dashboard 的部署记录
tags:
  - Docker
  - kubernetes
url: 1359.html
id: 1359
categories:
  - OP之路
date: 2019-11-06 22:19:15
---

K8S 搭建之后，当然需要使用 dashboard 这样的强有力的管理工具来实现对整个平台的控制，这篇就记录部署 dashboard 过程中踩的坑，以及解决方法。

镜像拉取以及部署
--------

**如果你有很好的网络环境，那么这一步是相当的容易**，根据官方的手册，你连左右俯冲都不需要： 当然如果你是又特色的网络环境的话，需要参考后面的网络的解决方案

    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
    
    # api 暴露访问
    kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' &

这样直接就可以在本机的地址，来直接进行 dashboard访问来

    http://<换成你的IP>:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.

调整NodePort
----------

我们当然希望我们的dashboard 需要想 一个服务一样暴露出来，这样我们需要修正配置文件了，找到最后的service的部分：

    # ------------------- Dashboard Service ------------------- #
    kind: Service
    apiVersion: v1
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      name: kubernetes-dashboard
      namespace: kube-system
    spec:
      ports:
        - port: 443
          targetPort: 8443
          nodePort: 30001
      type: NodePort
      selector:
        k8s-app: kubernetes-dashboard

在这里需要进行修改，添加 `NodePort`的配置，这样的话就可以直接在主机的网络上进行服务的访问。

登录看看
----

由于最新版的dashboard默认了minimal的权限，所以登上去基本什么都做不了，单是好歹能看把：

    # 列出所有的 secret
    kubectl get secret -A
    # 查看指定的 secret 的token
    kubectl -n kube-system get secret kubernetes-dashboard-token-<你自己的不一样>

得到token之后，直接去上面暴露出来的服务进行访问登录即可。在登录的时候选择使用 令牌来进行登录。

网络问题
----

如果你的网络很有特色，那么你需要这部分的内容。主要的思路是到其他地方来拉取镜像。之后重新tag即可

    docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
    docker tag mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1

另外有一点很坑的是，需要在yaml里指定，不需要拉取最新的镜像，否则在部署的过程中，kube还是会尝试去拉取最新的镜像导致部署失败，**修改yaml文件**

          - name: kubernetes-dashboard
            image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
            imagePullPolicy: IfNotPresent

权限问题
----

这个问题头疼了很久，官方的yaml默认配置的是minimal的权限，如果使用默认的token去进行访问的话，很多功能是用不了的。所以这里需要创建自己的admin来对集群的所有内容来进行统一管理。

由于到处都是教程，又很少有说怎么去解决配置的问题，这里简单的讲一下了。一码胜千言，这里直接把能用的yaml贴出来：

    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cluster-admin
    rules:
      # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin
      namespace: kube-system
      labels:
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: admin
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: rbac.authorization.k8s.io
    subjects:
    - kind: ServiceAccount
      name: admin
      namespace: kube-system
    

* * *

这里面分为三个部分：

*   ClusterRole
*   ClusterRoleBinding
*   ServiceAccount

这里理解为，创建集群角色，创建集群用户，绑定集群用户。这样就完成创建了一个具有所有权限的用户了。enjoy it。

后
-

最后祝各位玩的愉快

续
-

由于兼容性的问题需要升级为2.0 的dashboard，更加易用和方便：

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml

> 官方Dashboard链接 [https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)