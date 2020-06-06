---
title: TabClosing
tags:
  - istio
  - Jenkins
  - Linux
  - Nginx
  - nodejs
  - OPS
url: 1432.html
id: 1432
categories:
  - 每周分享
date: 2019-11-10 19:05:20
---

每周的标签关闭计划，还自己一个干净的浏览器。 每篇舍不得关闭的博文，总会有一点值得你记录的东西。

好书
--

*   [Mastering Elasticsearch(中文版)](https://doc.yonyoucloud.com/doc/mastering-elasticsearch/index.html) es的书写的不错
*   [Kubernetes Handbook——Kubernetes中文指南/云原生应用架构实践手册](https://jimmysong.io/kubernetes-handbook/) 一本开源图书免费阅读，jimmy 出品。

正文
--

*   [摘选 《人人都是工程师》前言](https://i4t.com/2132.html) 论什么是自学
    
*   [老男孩Shell企业面试题30道 \[答案\]](https://i4t.com/965.html)
    
*   [Apache Kafka基准测试：每秒写入2百万（在三台廉价机器上）-并发编程网 - ifeve.com](http://ifeve.com/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines/) 在三台 `Xeon 2.5 GHz处理器六核` 的机器上可以实现 2M 的每秒写入量。对吞吐参数也做了详细的实验。
    
*   [Linux命令大全](https://man.linuxde.net/) 对于命令的功能和参数可以直接进行查询
    
*   [OpenResty最佳实践](https://legacy.gitbook.com/book/moonbingbing/openresty-best-practices) 后面可以从nginx到 openResty来进行过度，更便捷的方法来实现简单的WEB功能
    
*   [snippets-代码片段收集板](http://snippets.barretlee.com/) 代码片段站点包含 HTML，HTACCESS，NGINX 等等
    
*   [Go语言充电站](http://lessisbetter.site/) 一个主要是Go语言的站点
    
*   [技术分享之service mesh (k8s&istio)的那些事儿-峰云就她了](http://xiaorui.cc/2019/08/24/%e6%8a%80%e6%9c%af%e5%88%86%e4%ba%ab%e4%b9%8bservice-mesh-k8sistio%e7%9a%84%e9%82%a3%e4%ba%9b%e4%ba%8b%e5%84%bf/)
    
    > 推荐大家把一些有状态的服务，比如postgres、mysql、elasticsearch，放在k8s的pod之外。为啥？ 我问了一圈在各公司搞云平台开发的，他们k8s每次出问题，大概率都是因为存储。 😅 这个很考验基础架构团队的能力。 一篇不错的 istio 的介绍文章，也给出了PDF的下载链接 [http://xiaorui.cc/static/service_mesh.pdf](http://xiaorui.cc/static/service_mesh.pdf)
    
*   [技术分享之http2和quic的那些事儿--峰云就她了](http://xiaorui.cc/2019/09/17/%e6%8a%80%e6%9c%af%e5%88%86%e4%ba%ab%e4%b9%8bhttp2%e5%92%8cquic%e7%9a%84%e9%82%a3%e4%ba%9b%e4%ba%8b%e5%84%bf/) Http2的讲解文章，干货生动。
    
*   [Raft共识算法的官方页面](https://raft.github.io/) 有动画生动的介绍共识过程，生动形象
    
*   [etcd 使用入门](https://cizixs.com/2016/08/02/intro-to-etcd/) ETCD 的入门基本的使用简介
    
*   [Jenkins入门指南](https://jenkins.io/zh/doc/pipeline/tour/getting-started/) Jenkins的官方入门智能。
    
*   [用Python的交易员--知乎](https://www.zhihu.com/people/traderusingpython/activities) VN.py的作者
    
*   [Greenwicher's Wiki](http://wiki.greenwicher.com/) 涵盖金融以及阅读的个人wiki
    
*   [一个新手面试 Linux 运维工作至少需要知道哪些知识？](https://www.zhihu.com/question/19855127)
    
    > 运维岗位不像其它岗位，如研发工程师、测试工程师等，有非常明确的职责定位及职业规划，比较有职业认同感与成就感；而运维工作可能给人的感觉是哪方面都了解一些，但又都比上专职工程师更精通、感觉平时被关注度比较低（除非线上出现故障），慢慢的大家就会迷惘，
    
*   [Project HashClash - MD5 & SHA-1 cryptanalysis](https://github.com/cr-marcstevens/hashclash) 实现MD5碰撞的工具，基于CUDA可以自己跑
    
*   [在线工具](https://tool.lu/) 就是在线工具
    
*   [Node初学者入门，一本全面的NodeJS教程](http://ourjs.com/detail/529ca5950cb6498814000005) 一个**不错**的NodeJS的书，介绍来是怎么使用 JS 来实现各种的开发，web/应用
    
*   [机器战胜人类了，伺候机器的运维呢? \-\- 三斗室](http://chenlinux.com/2016/03/19/machine-vs-ops/)
    
    > 『跟其他运维工程师觉得这个职业将消失不同。我是对运维职业是持极端乐观态度的，也许运维职业将是人类最后一个职业。很可能祂们在能自理之前还需要我们伺候。。。也说不定，某几个运维工程师因为某种不知道的原因还会被祂们当宠物留下来，成为人类的最后的延续。』 『我终于明白这个图片的寓意了，它其实预示了人类的未来命运。』
    
*   [Docker源码分析系列文章](https://guanjunjian.github.io/category/#%E9%98%85%E8%AF%BB)
    
*   [Nginx源码解读系列](https://ivanzz1001.github.io/records/)
    
*   [Being壁纸](https://bing.ioliu.cn/ranking) M$的壁纸说实话好看