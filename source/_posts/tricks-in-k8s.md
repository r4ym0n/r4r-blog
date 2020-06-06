---
title: Tricks in k8s
tags:
  - Docker
  - K8S
  - OPS
url: 1096.html
id: 1096
categories:
  - DEV
date: 2019-08-18 00:53:48
---

这一篇主要收集k8s使用过程中的一些小技巧

pod获取主机的信息
----------

怎么在把主机的信息传给pod里面呢？第一直觉就是环境变量，也的确是这样。

            env:
              - name: "HTTP_HOST"
                valueFrom:
                  fieldRef:
                    fieldPath: status.podIP

可以通过环境变量，和yaml的配置来直接传值

怎么保证多个副本的命名不同
-------------

和上面的思路相似，这里可以在metadata里面来获取pod的名字，直接作为环境变量传入容器内， 这样进行的配置，各个节点的命名就各自不同啦。

            env:
              - name: "NODE_NAME"
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name

容器内部的时间
-------

在有些 镜像的打包过程中时区不一样，所以导致在运行的时候，容器的时区和物理主机的时区不相同。 为了解决这个问题，就直接把本机的时区配置挂载到里面就可以啦。 在容器的后面的挂载卷配置：

            volumeMounts:
            - name: timezone
              mountPath: /etc/localtime
          volumes:
            - name: timezone
              hostPath:
                path: /usr/share/zoneinfo/Asia/Shanghai