---
title: 容器内部网络转发 Ingress-Nginx部署
tags:
  - Docker
  - Ingress
  - kubernetes
url: 1441.html
id: 1441
categories:
  - OP之路
---

Services "ingress-nginx" not found #2599 [https://github.com/kubernetes/ingress-nginx/issues/2599](https://github.com/kubernetes/ingress-nginx/issues/2599)

建立 Ingress
----------

    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: host-ingress
      annotations:
        kubernetes.io/ingress.class: "nginx"
    spec:
      rules:
      - host: www.12ms.xyz
        http:
          paths:
          - path: /wiki/
            backend:
              serviceName: kiwix-svc
              servicePort: 80
      - host: second.foo.com
        http:
          paths:
          - backend:
              serviceName: service2
              servicePort: 80
      - http:
          paths:
          - backend:
              serviceName: service3
              servicePort: 80

> ingress-nginx-readme.md [https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md](https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md)

部署 Ingress-Controller
---------------------

有了前面的ingress层，还需要一个 controller 来对流量进行调度，这里使用的是 nginx。官方已经给出了oneclick 的部署的yaml的文件，

### pod 无法被调度到 master

在部署的过程中发现 Nginx Pod不会被调度到 Master 节点上。导致 其状态一致在 `pendding` 的状态下。通过 `describe`来查看实际的内容。看到会有一个warning，大意是：主机 tain 以及没有满足条件的主机。这是由于 K8S 默认master节点是不允许 pod部署的。所以我们设置 master 的taint 使得其可以被调度 pod。

    kubectl taint nodes k8s-master key=value:PreferNoSchedule

官网上也有很好的 `trouble Shooting` 的文章

> [https://kubernetes.io/zh/docs/tasks/debug-application-cluster/debug-application/#pod%E5%81%9C%E7%95%99%E5%9C%A8pending%E7%8A%B6%E6%80%81](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/debug-application/#pod%E5%81%9C%E7%95%99%E5%9C%A8pending%E7%8A%B6%E6%80%81)

### 503 错误

在顺利的在 master 运行之后，发现所有的请求返回的都是 503。来检查 Nginx-controller 的日志。

    kubectl -n  ingress-nginx logs nginx-ingress-controller-6d87447d67-r5gjb -f

发现有大量的错误日志：

    W1111 12:22:15.278922       6 queue.go:130] requeuing &ObjectMeta{Name:sync status,GenerateName:,Namespace:,SelfLink:,UID:,ResourceVersion:,Generation:0,CreationTimestamp:0001-01-01 00:00:00 +0000 UTC,DeletionTimestamp:,DeletionGracePeriodSeconds:nil,Labels:map[string]string{},Annotations:map[string]string{},OwnerReferences:[]OwnerReference{},Finalizers:[],ClusterName:,ManagedFields:[]ManagedFieldsEntry{},}, err services "ingress-nginx" not found
    W1111 12:22:15.478746       6 queue.go:130] requeuing &ObjectMeta{Name:sync status,GenerateName:,Namespace:,SelfLink:,UID:,ResourceVersion:,Generation:0,CreationTimestamp:0001-01-01 00:00:00 +0000 UTC,DeletionTimestamp:,DeletionGracePeriodSeconds:nil,Labels:map[string]string{},Annotations:map[string]string{},OwnerReferences:[]OwnerReference{},Finalizers:[],ClusterName:,ManagedFields:[]ManagedFieldsEntry{},}, err services "ingress-nginx" not found
    W1111 12:22:15.678442       6 queue.go:130] requeuing &ObjectMeta{Name:sync status,GenerateName:,Namespace:,SelfLink:,UID:,ResourceVersion:,Generation:0,CreationTimestamp:0001-01-01 00:00:00 +0000 UTC,DeletionTimestamp:,DeletionGracePeriodSeconds:nil,Labels:map[string]string{},Annotations:map[string]string{},OwnerReferences:[]OwnerReference{},Finalizers:[],ClusterName:,ManagedFields:[]ManagedFieldsEntry{},}, err services "ingress-nginx" not found

设置body-max
----------

在进行文件传送的时候出现`413`的报错

        nginx.ingress.kubernetes.io/proxy-body-size: "60M"

参考
--

> *   [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)