---
title: ELK 日志可视化平台 搭建
url: 472.html
id: 472
categories:
  - 未分类
date: 2019-02-14 00:00:00
tags:
---

架构选型
----

这里使用ELK 平台主要是实现了单主机的日至手机以及可视化，吞吐量比较小，所以直接进行单点部署。由以下几个组件组成：

*   FileBeat
*   Elasticsearch
*   Kibana

Filebeat 这里实现日志文件的手机以及格式化，并且通过管道直接存入ES

ES 作为主体，实现对索引条目进行保存

Kibana 实现了ELK系统的整体监控，通过 API与ES交互，实现数据可视化，以及设置。

*   logstash

logstash 作为一个对数据进行流处理的module，这里由于不需要对日志进行进一步处理，所以并没有使用此组件。**关键是内存真的不够了，跑不起来了是真的Q**

安装及部署
-----

安装以及部署的过程实际上并不是十分复杂，在[ELK项目官网](https://www.elastic.co/)上实际上已经给出了相关的 `rpm包`，所以直接使用 rpm 进行安装即可：

    -rw-r--r-- 1 root root  12M Jan 29 19:31 filebeat-6.6.0-x86_64.rpm
    -rw-r--r-- 1 root root 177M Jan 29 19:34 kibana-6.6.0-x86_64.rpm
    -rw-r--r-- 1 root root 163M Jan 29 19:35 logstash-6.6.0.rpm
    rpm -ivh xxx.rpm

* * *

这里涉及到的一个 point 是`systemV init` 还是 `systemd` 下面给出一篇相关文章，(IBM Developer 还真是个好地方。)

> [浅析 Linux 初始化 init 系统，第 3 部分](https://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/index.html)

这里提到的是Linux里最重要的启动进程，作为**第一个用户态进程** 其PID为1 `ps 1` 可以看到进程的运行。他负责在内核启动之后，完成余下的引导过程，比如加载加载服务，启动Shell/图形化界面等等。在 CentOS 7 之后，使用 Systemd 风格取代了原来的 init的风格。（使得启动速度有的提升）

    ➜  ~ ps 1
      PID TTY      STAT   TIME COMMAND
        1 ?        Ss     2:50 /usr/lib/systemd/systemd --switched-root --system --deserialize 21

可以看到，这里的 ps1 已经是 systemd 了。

所以这里又涉及到了，Ubuntu 和 CentOS 的一大差别，自己也是用了才发现，之前Ubuntu的服务启动与管理使用 `service` 命令，并且在 `/etc/inittab` 里面可以对初始化进行一系列的配置。在Cent里面写着一句话：

* * *

总之在目前的环境中，使用 `systemctl` 命令对服务进行控制。

    ➜  ~ systemctl status elasticsearch.service
    ● elasticsearch.service - Elasticsearch
       Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; vendor preset: disabled)
       Active: active (running) since Tue 2019-02-05 02:39:39 CST; 1 weeks 1 days ago
         Docs: http://www.elastic.co
     Main PID: 27965 (java)
       CGroup: /system.slice/elasticsearch.service
               ├─27965 /bin/java -Xms256m -Xmx256m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=6...
               └─28026 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

* * *

之后修改各自的配置文件，使得其端口对应，一般就可以跑起来了。

为了安全起见，这里使用 Nginx 做了一个 Web 的认证，对路由进行一个转发。

环境调整
----

### ES的JVM 的堆大小调整 （heap）

启ES的时候，直接报错 `unable to alloca` ，一想应该是内存太小了导致的问题。查看配置文件 `/etc/elasticsearch/jvm.options` 并且进行参数调整

    -Xms256m    # 这里缺省为 1g
    -Xmx256m

由于本机的RAM也只有 **1G** 还得跑其他的奇怪的东西，所以这里只能非常的乞丐设置。分出一点宝贵内存。

### 虚拟内存 Swap 的设置

由于VM的机能，所以1G的内存显然是不够用的。只能改变参数。使得`JVM`分配的内存只有 256M。且，启用虚拟内存，即swap。

[Linux SWAP 交换分区配置说明 - CSDN](https://blog.csdn.net/tianlesoftware/article/details/8741873)

* * *

     dd if=/dev/zero of=/swap-file bs=1M count=1024 # 新建白文件 1G
     mkswap /swap-file  # 格式化
     swapon /swap-file  # 启用swap

为了使swap自动挂载使用 `/etc/fstab` 里面进行配置：

    UUID=653bbeb5-4abb-4295-b110-5847e073140d   swap    swap    defaults    0   0
    /swap-file                                  swap    swap    defaults    0   0

* * *

顺便调整了 `swappiness` 使得机器更加积极的使用 虚拟内存

    ➜  ~ sysctl vm/swappiness=60

### 设置副本数 0

由于只是单点，且性能受限，这里发现的问题es的节点状态一直是 `yellow`。归根发现就是性能问题，所以这里进行配置，**设置其分片副本为 0**。

    curl -H "Content-Type: application/json" -XPUT 'http://localhost:9202/_settings' -d '
    {
        "index" : {
            "number_of_replicas" : 0
        }
    }'
    

后面的话
----

**Happy Valentine's Day**