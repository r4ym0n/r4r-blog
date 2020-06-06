---
title: 代码即架构--从一行run到Yaml
tags:
  - Docker
  - HA
  - K8S
  - kubernetes
url: 1419.html
id: 1419
categories:
  - OP之路
date: 2019-11-09 23:08:22
---

任何东西都应该代码化，有了Docker的技术之后，服务架构本身也夸一被代码化了。 这一篇帖子记录的是从最简单的 `docker run` 到一个基本的K8S的高可用的服务。

背景
--

这次的背景接着前面的 [kiwix 离线个人wiki部署](https://blog.12ms.xyz/2019/06/02/%e6%9c%ac%e5%9c%b0%e7%be%a4%e6%99%96%e6%90%ad%e5%bb%ba%e7%a6%bb%e7%ba%bfwiki/) ，在那篇文章中，最终使用 `docker run`命令来进行服务的运行。 但是我们需要知道的是，在微服务的思想中，容器不是HA的，但是服务是HA的。

    docker run -p 32768:80 -v /volume1/DataBank/Wiki/zims/:/data --name="kiwix-server" kiwix/kiwix-serve wikipedia_zh_all_novid_2018-07.zim

所以，这里的目的就是把上面的这句 命令转化为一个服务。

标准YAML
------

在之前也写过K8S的yaml 文件，但是都是copy paste 出来的，没有一个标准，所以这里就来搞一个标准模板。下面是从官方拿到的一个部署实例。

    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2 # tells deployment to run 2 pods matching the template
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80

编写我们自己的YAML
-----------

### 基础应用

可以参考上面的示例，我们需要创建一个属于自己的应用 YAML。照葫芦画葫芦，我一我们得到来下面的内容

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kiwix-dept
    spec:
      selector:
        matchLabels:
          app: kiwix
      replicas: 2 # 使用两个实例
      template:
        metadata:
          labels:
            app: kiwix
        spec:
          containers:
          - name: kiwix
            image: kiwix/kiwix-serve:latest
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 80

### 端口暴露

在K8S中，service层可以理解为网络的调度器。由service 来控制网络的负载均衡，所以这里需要进行端口的暴露，一样的参考官方的说明文档，照葫芦画葫芦。

    apiVersion: v1
    kind: Service
    metadata:
      name: kiwix-svc
      labels:
        app: kiwix
    spec:
      ports:
      - port: 30002
        targetPort: 80
        nodePort: 30002
        protocol: TCP
        name: http
      type: NodePort
      selector:
        app: kiwix

### 持久卷

K8S上的存储，一直是个老大难的问题。因为服务是运行在多个节点上，但是存储又是相互独立的，所以这里需要使用 可共享的文件系统来实现文件共享。最简单易得的就是我们的 `NFS`。

* * *

**这里涉及到一个新的 概念 `PV/PVC`，看了技术的简介，可以理解为 PV是存储设备的通用抽象层，而PVC是在其上的通用控制层。**这里也是直接搬移官方的yaml；

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: nfs
    spec:
      capacity:
        storage: 64Gi
      accessModes:
        - ReadWriteMany
      nfs:
        server: sync.gov
        path: "/volume1/docker/wiki/"
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs
    spec:
      accessModes:
        - ReadWriteMany
      storageClassName: ""
      resources:
        requests:
          storage: 64Gi

在pod内部使用卷的且进行挂载的时候的yaml 先是指定挂载点，之后来下载需要挂载的 PVC

            volumeMounts:
              # name must match the volume name below
              - name: nfs
                mountPath: "/mnt"
          volumes:
          - name: nfs
            persistentVolumeClaim:
              claimName: nfs

### 最终整合

在对上面的 应用，网络，存储这里的三大部分进行整合之后，就得到了下面的内容

    apiVersion: v1
    kind: Service
    metadata:
      name: kiwix-svc
      labels:
        app: kiwix
    spec:
      ports:
      - port: 30002
        targetPort: 80
        nodePort: 30002
        protocol: TCP
        name: http
      type: NodePort
      selector:
        app: kiwix
    
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: nfs
    spec:
      capacity:
        storage: 64Gi
      accessModes:
        - ReadWriteMany
      nfs:
        server: sync.gov
        path: "/volume1/docker/wiki/zims/"
    
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs
    spec:
      accessModes:
        - ReadWriteMany
      storageClassName: ""
      resources:
        requests:
          storage: 64Gi
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kiwix-dept
    spec:
      selector:
        matchLabels:
          app: kiwix
      replicas: 2 # 使用两个实例
      template:
        metadata:
          labels:
            app: kiwix
        spec:
          containers:
          - name: kiwix
            image: kiwix/kiwix-serve:latest
            imagePullPolicy: IfNotPresent
            command: ["/usr/local/bin/kiwix-serve"]
            args: ["--port", "80", "--library","/data/library.xml","-r","wiki","-v"]
            ports:
            - containerPort: 80
              protocol: TCP
            livenessProbe:
              httpGet:
                scheme: HTTP
                path: /wiki/
                port: 80
              initialDelaySeconds: 30
              timeoutSeconds: 30
            volumeMounts:
              # name must match the volume name below
              - name: nfs
                mountPath: "/data"
          volumes:
          - name: nfs
            persistentVolumeClaim:
              claimName: nfs

直接在使用 `kubuctl`进行部署即可

后
-

这次的 yaml 的部署显得前所未有的顺利，遇到了port 配置的一点的问题，之外在卷挂载以及使用上面十分的顺利。目前本站的 wiki 就是使用上述的 yaml 文件来进行部署。

K8S平台使得架构可以使用 代码进行编写，是对应用本身的极大的提高。这里使用的简单的工程中，分为`逻辑层，存储层，接入层`清晰明了，都可以保证HA。

参考资料
----

*   [为容器设置启动时要执行的命令及其入参](https://kubernetes.io/zh/docs/tasks/inject-data-application/define-command-argument-container/)
*   [Kiwix Docker file](https://hub.docker.com/r/kiwix/kiwix-serve/dockerfile)
*   [k8s-nfs-example](https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs)
*   [存储-Volumes](https://kubernetes.io/zh/docs/concepts/storage/volumes/#nfs)