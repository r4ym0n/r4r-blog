---
title: Kafka 简单部署使用指北
tags:
  - ELK
  - kafka
url: 1080.html
id: 1080
categories:
  - DEV
date: 2019-08-11 20:04:53
---

又有接触到新的东西还是蛮开心的，这次整上的开源组件是Kafka。读音卡夫卡。 一眼看起来，是一个高端玩意，刚刚接触的时候有些害怕自己驾驭不住，后面发现，还行，简单好用。 所以这篇文章，就来介绍一下卡夫卡，以及简简单单的进行一个指北。

什么是kafka
--------

kafka 简单来说是一个消息队列服务，主要的功能是把随机写转化为顺序写，或者可以理解为缓存？ 和我们的linux中的queue差不多，另外提本身是分布式的，所以可以保持系统的高吞吐和高可用。

另外，比较重要的一点，Kafka可以很大的降低系统复杂性。 怎么说呢？做标准的系统中，上一个组件和下一个组件互连，随着系统的越来越复杂之间的连接越来越多。整个系统的内部，就变成了一张大网。 所以引入了kafka，在这一层对各个模块之间的连接进行抽象。 写消息只需要上报到kafka即可，读消息只需要到kafka中根据对应的`topic`来进行读即可。 这样由分到总再到分的结构，很大的简化了原来系统的复杂结构。

怎么测试？
-----

kafka的基本使用是相当的简单了： 配置文件写好之后，可以直接运行服务

    ./zookeeper-server-start.sh -daemon /data/home/kafka/config/zookeeper.properties
    ./kafka-server-start.sh -daemon /data/home/kafka/config/server.properties

然后进行本地的消息读写测试：

    ./kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test
    > 123
    ./kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test --from-beginning
    123

* * *

上面的流程呢？向消息队列写数据，再读出数据。就这样就完成了kafka的基本使用。

如何部署
----

在部署的这部分，还是踩了一些坑，因为是开着官方文档来的，结果坑就在官方文档里面！ kafka 严格意义上是一个队列服务的软件，那么实际上他要实现分布式部署的话，还需要一个组件`zookeeper`， 官方的quicklystart[文档](https://kafka.apache.org/quickstart) 这里有一句话：

> Kafka uses ZooKeeper so you need to first start a ZooKeeper server if you don't already have one. You can use the convenience script packaged with kafka to get a quick-and-dirty single-node ZooKeeper instance.

也就是说kafka的下载包中的配置，只是能启动一个单一的实例，如果需要启动一个集群，那么需要先有一个zookeeper的网络。

* * *

所以最重要的是需要先有一个zookeeper的网络，所以得有两份配置，kafka和zookeeper。 kafka 的配置缺省差不多可以使用。大坑在zookeeper。这里先贴出配置：

    dataDir=/data/home/ramonesliu/kafka/data
    clientPort=2181
    maxClientCnxns=0
    
    server.1=xxxx:2888:3888
    server.2=xxxx:2888:3888
    server.3=xxxx:2888:3888
    
    tickTime=2000
    initLimit=10
    syncLimit=5

这里的配置看起来部署很简单，但是大坑存在与最后面这三行。默认配置是没有的。 但是一旦缺了这几个配置，启动的时候竟然会报一个`parse error`的问题。这个问题当时是排查了一个小时。。。

还有一个关键步骤是在上面指定的Dir里面需要写入一个myid的文本文件，里面的数值，需要和server.x这里的x相同，作为一个zookeeper节点的唯一标识。之后就使用命令行直接启动，即可。

* * *

使用zookeeper的本身的shell来检验集群状态

    > ./zookeeper-shell.sh  127.0.0.1:2181 ls /brokers/ids
    
    Connecting to 127.0.0.1:2181
    WATCHER::
    WatchedEvent state:SyncConnected type:None path:null
    [0, 1, 2]

怎么使用
----

kafka只是作为一个使用过程中的中间件，所以使用起来是十分简单的，配置输入，配置输出即可： 这里的使用场景是用于filebeat和logstash之间。示例配置文件如下：

    #filebeat上报示例配置：
    output.kafka:
      hosts: ["kfk:19093"]
      topic: nginx
      required_acks: 1
    
    #logstash 拉取示例配置
    input {
        kafka{
            bootstrap_servers => ["kfk:19093"]
            group_id => "nginx"
            auto_offset_reset => "earliest"
            decorate_events => "true"
            topics => ["nginx"]
            codec => json
        }
    }

filebeat 作为数据的输入，logstash作为数据的输出，这样从原来的fb直接向logstash 的输出，成为了logstash到kfk的主动拉取。所以logstash 的负载得到了较大的舒缓。

总结
--

kafka这个中间件的使用，降低了原系统直接的复杂性，并且实现了两个服务间的解耦。当logstash服务不可用的时候，一样可以进行数据的缓冲。

参考
--

*   [我个人的kafka broker和zookeeper集群实践（★firecat推荐★）](https://blog.csdn.net/libaineu2004/article/details/79655537)