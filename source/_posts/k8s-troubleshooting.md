---
title: k8s-TroubleShooting
url: 1485.html
id: 1485
categories:
  - 未分类
date: 2020-01-16 17:43:27
tags:
---

这一篇拿来来记录这些在 k8s的使用过程中的技巧以及踩的坑。

应用故障排查 [https://kubernetes.io/zh/docs/tasks/debug-application-cluster/debug-application/](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/debug-application/)

删掉那些 terminaling
----------------

pod死掉的时候，有时候就卡在 terminating 的的状态，虽然没啥影响，但是看的强迫症犯了，着实难受。看着官方的手册研究了一下，有办法解决，如下命令：

    kubectl delete pod wp-ray-wordpress-9674f4c69-z6xrh --grace-period=0 --force

当master节点不会被调度 pod 的时候 [https://docs.lvrui.io/2018/11/14/%E4%B8%BAk8s-master%E8%8A%82%E7%82%B9%E6%B7%BB%E5%8A%A0%E6%B1%A1%E7%82%B9taints/#book:mark](https://docs.lvrui.io/2018/11/14/%E4%B8%BAk8s-master%E8%8A%82%E7%82%B9%E6%B7%BB%E5%8A%A0%E6%B1%A1%E7%82%B9taints/#book:mark)

Docker 的磁盘悬挂的问题快速解决
-------------------

`docker system prune` 磁盘清理 该指令默认会清除所有如下资源： 已停止的容器（container） 未被任何容器所使用的卷（volume） 未被任何容器所关联的网络（network） 所有悬空镜像（image）。