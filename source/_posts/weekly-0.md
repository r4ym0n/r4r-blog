---
title: Weekly.0
url: 442.html
id: 442
categories:
  - 未分类
date: 2019-03-22 00:00:00
tags:
---

前面的话
----

总会遇见一些奇奇怪怪但是给自己带来一些新奇的想法的东西。有时候给偷偷转发，或者自己收藏。

但是，很遗憾的是总是没有进行统一的管理。 一会丢了，一会不见了。突然想到的时候，找啊找，总是找不到它了。

所以，决定再在这里开坑吧，每周一更，时间佛系。也算是对自己的监督了。

正文
--

### 句子

*   是不是每个人年轻的时候都有这样一段日子，鸿鹄志高却难遂，迷茫地过着，昏昏噩噩地耗，最终不是妥协泯然众人，就是找不到出口被生活围困。这时候家人朋友，看在眼里，哪怕不说，心里想的也是“小镇青年何必心怀远方”这样的想法吧。（J·M·库切《青春》）
    
*   当发现自己的才会撑不起野心的时候，就安静下来学习吧。
    
*   使用人月作为一项工程的规模，是一个危险和带有欺骗性的神话。（佛瑞德·布鲁克斯《人月神话》）
    
*   卡辛斯基曾经提到过这种情况。未来生产力大发展，物质极大丰富，人类无所事事，只能"把时间花在互相擦皮鞋上面，或者用出租车带着彼此到处瞎转，互相为对方做手工艺品，互相给对方端盘子等等。"说实话，我看不出来，大家互相拍视频，直播吃饭、购物、打游戏，跟互相擦皮鞋，有什么本质的不同。
    
    （对未来AI带来的生产力解放的一个猜想）
    

### 文章

*   [化工男的十二年的化学遭遇之路](https://www.kechuang.org/t/62325) 真真实实的好文章
    *   保证不伤害自己，保证不伤害他人，保证自己不被他人伤害
*   [世界上只有一种病，那就是穷病](http://www.yelook.com/2261.html) 穷，大概是我现在最害怕的一个东西
    *   一个得了穷病的人，永远着眼于眼前的资源，惧怕风险，不舍得投入，做事畏首畏脚，注定不会有大的成就。道理一套一套的摆在那边，但是真正的让自己去行动，确实难上加难，否则我也不会有昨天那般狼狈不堪了。真的需要痛定思痛的去改变自己了。

### TECH

1.  [什么是 Docker -- V2EX](https://www.v2ex.com/t/267641#reply7)
    1.  **Docker不是虚拟机**，所形容的集装箱，更多的指的是应用的标准。这也就是一个容器一个进程的最好的解释。
    2.  那么 Docker 的实质是什么？在我看来就是个针对 PAAS 平台的自动化运维工具而已。众所周知（当然如果你不知道，那么我来告诉你）：自动化运维的大前提就是标准化。
    3.  当你开始诟病Docker的变更需要提交太麻烦的时候，要开始反思自己的需求是否真的需要 Docker了。
2.  [Linux 内存中的 Cache 真的能被回收么？](https://www.v2ex.com/t/278921#reply22)
    1.  buffers/cache 占用的较多，说明系统中有进程曾经读写过文件，但是不要紧，这部分内存是当空闲来用的
3.  [Kubernetes实战 – 谈谈微博应对春晚等突发峰值流量的经验](https://www.kubernetes.org.cn/5036.html)
4.  [计算机原理 —— 计算机是如何启动的](http://liaoph.com/how-computers-boot-up/)
    1.  硬件初始化 =\> 加载BIOS => 加载Loader(win/Linux Loader) => （Stage2 加载）=> 加载内核代码
5.  [计算机原理 —— 主板与内存映射](http://liaoph.com/motherboard-and-memory-map/)
    1.  不是所有的地址都会被进行内存寻址，会在北桥进行地址转换，到一些IO设备。
6.  [Etcd 分布式配置共享](https://blog.mallux.me/2017/02/08/etcd/)
    1.  从本质上说，`分布式配置共享服务就是一种分布式的键值数据库`。
    2.  Etcd 主要注重于应用配置数据的保存，它能够确保数据在多个节点上的高度一致性，并在集群半数以下成员节点出现故障时，继续保持正常运转
    3.  用来做主机的配置的统一管理
7.  [Docker 三剑客](https://blog.mallux.me/2017/02/03/docker-orchestrating/)
    1.  Docker Compose
        1.  Docker 的最佳实践是一个容器只运行一个进程，因此要运行多个组件则必须运行多个容器。在一个由多容器构成的应用里，我们需要一个有效的工具来定义一个应用由哪些容器组成，以及定义这些容器之间如何关联。为了解决以上问题，Compose 便应运而生。
        2.  从本质上来讲，Compose 把 YML 文件解析成 docker 命令的参数，然后调用相应的 docker 命令行接口，从而把应用以容器化的方式管理起来
    2.  Docker Swarm
        1.  它可以把多个 Docker 主机组成的系统转换成为单一的虚拟 Docker 主机。
        2.  一个服务由多个任务组成，一个任务即一个运行的容器。
        3.  The swarm manager uses `ingress load balancing ( 基于 ipvs 的四层代理 )` to expose the services you want to make available externally to the swarm。使用四层负载均衡实现可用性拓展
        4.  Docker Swarm（With an external key-value store）推荐使用 `etcd` 进行统一管理
            1.  官方不推荐基于 token 的 Docker Swarm 来创建 Swarm 集群（仅适用于开发测试环境），而应该基于 KV 键值配置共享来实现 Swarm 集群。
            2.  使用 docker-machine + etcd 的方式来直接创建 swarm 集群，etcd 使用单台主机（clean-Host）提供服务。
            3.  `swarm manager` 和 `swarm agent` 的创建，只要指定 discovery 服务为 Etcd 等配置共享服务即可。例如：`--swarm-discovery etcd://172.16.0.21:2379`。
    3.  Docker Machine
        1.  一个 Docker Machine 就是一个 Docker host 主机和经过配置的 Docker client 的结合体。
        2.  方便的进行跨主机的多Docker 的服务管理
8.  [负载均衡集群 LVS 详解](http://liaoph.com/lvs/)
    1.  垂直拓展/水平拓展 = 高性能/多数量
    2.  四层负载均衡：根据请求报文中的目标地址和端口进行调度
    3.  七层负载均衡：根据请求报文的内容进行调度，这种调度属于「代理」的方式

### PickS

*   [Notion](https://www.notion.so/) \- 好像是一款很好用的全功能的笔记工具

后
-

后面自己专注收集吧，也希望成为每周的 Tab-Closing 计划。