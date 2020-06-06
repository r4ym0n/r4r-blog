---
title: TabClosing 2
tags:
  - Docker
  - elasticsearch
  - Nginx
url: 1494.html
id: 1494
categories:
  - 每周分享
date: 2019-12-08 23:47:29
---

> 为什么总是停留在什么都想，可是什么都不想做的状态呢？ 我的大好周末好可惜，一定需要找到驱动自己的动力。或许是工作日的事情消耗了自己的太多的活力，导致周末需要用重获自由般的休息来refill 自己？

有意思
---

这里收集一些有意思的小东西

*   [http://protobot.org/#zh](http://protobot.org/#zh) 创意缺乏的时候，来随机一下自己的灵感

书们
--

*   [kubernetes 从入门到实践](https://www.kancloud.cn/huyipow/kubernetes/531982) 不错的书，mark
    
*   [Kubernetes Handbook——Kubernetes中文指南/云原生应用架构实践手册](https://jimmysong.io/kubernetes-handbook)
    
*   [Linux 学习笔记](https://www.huweihuang.com/linux-notes/) 学习笔记系列，东西很多涉及也很广。
    
*   [Serverless Handbook——无服务架构实践手册](https://jimmysong.io/serverless-handbook/) 本书是本人学习和实践 Serverless 过程中所整理的资料，主要关注的 Serverless 开源项目是 Knative。
    
*   [OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/lua/brief.html) openresty 的一篇入门的文章，写的很不错，涉及到Lua的愈发，以及在WAF方面的应用等。 粗略的翻了一下，后面如果要切的话可以作为参考
    
*   [http://www.lingocn.com/](http://www.lingocn.com/) 一个高质量电子书的分享站点
    
*   [The Linux Kernel](https://www.kernel.org/doc/html/latest/admin-guide/index.html) linux 内核的文档，对内核接口介绍详细
    
*   [太极书馆](http://www.8bei8.com/app/bookFace/default20190701/index_default.php?back_url=http://www.8bei8.com/book/wenku/mengdejiexi_dmao.html) 偶然发现的一个藏书室
    

其他
--

*   [RssHub](https://rsshub.app/) 这个之前有推过的一个很不错的项目，后面可以成为资讯来源 考虑结合 keydatas 或者 [wpematico](http://www.wpematico.com/)这种自动抓取的plugin
    
*   [7 Best Auto Blogging Plugins for WordPres](https://blog.hubspot.com/website/best-auto-blogging-plugins-for-wordpress)
    
*   [19 个很有用的 ElasticSearch 查询语句](https://n3xtchen.github.io/n3xtchen/elasticsearch/2017/07/05/elasticsearch-23-useful-query-example) 一些比较常用的ES搜索的命令，以便后面可以进行查阅
    
*   [CGroup 介绍、应用实例及原理描述](https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html) Cgroup介绍以及实操的不糙文章（IBM社区真的可以）
    
*   [twitter的数据下载](https://twitter.com/settings/your_twitter_data) twitter是可以下载个人数据的。
    
*   [Sherlock的博客git](https://github.com/SherlockHomer/blog) 大佬的一些杂文收集
    
*   [sonarqube](https://www.sonarqube.org/) 一个代码静态分析的静态分析的工具
    
*   [flickr](https://www.flickr.com/) 知名的在线相册，用于存储各种各样的精美照片
    
*   [端媒体](https://theinitium.com/) 主流读者为具有全球经验的华人精英，包括： 律师、银行家、基金管理人等专业人士 企业所有者、高级管理人员与董事 政府官员、大学教授等意见领袖
    
*   [Nginx+Lua 实现灰度发布](https://i4t.com/4070.html) LUA 由于其环境极其简单的特性，可以很好的作为胶水到各个组件
    
*   [可能是全网把 ZooKeeper 概念讲的最清楚的一篇文章](https://segmentfault.com/a/1190000016349824) ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。 服务生产者将自己提供的服务注册到Zookeeper中心，服务的消费者在进行服务调用的时候先到Zookeeper中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。
    
*   [Mysql大表删除](http://www.yoonper.com/post.php?id=72) 通过把现有的表的IDB文件通过硬链接成另一个文件。进行快速删除
    
        ln 123.ibd 123.bak
    
*   [日志系统实践 \- 搭建开发环境x](http://www.yoonper.com/post.php?id=63)
    
    *   [http://www.yoonper.com/post.php?id=63](http://www.yoonper.com/post.php?id=63)
    *   id++
    *   [http://www.yoonper.com/post.php?id=71](http://www.yoonper.com/post.php?id=71) 一个日志手收集系统的自主开发过程系列  
        ![file](https://blog.12ms.xyz/wp-content/uploads/2019/12/image-1575814219683.png)
*   [什么是Serverless（无服务器）架构？](https://jimmysong.io/posts/what-is-serverless/)
    
    > Karl Marx说的好，生产力决定生产关系，云计算的概念层出不穷，其本质上还是对生产关系和生产力的配置与优化，生产者抛开场景意味追求高大上的技术将譬如“大炮打蚊子”，小题大做，鼓励大家为了满足大家的好奇心进行折腾，毕竟那么多科学发现和重大发明都是因为折腾出来的，不想要一匹跑的更快的马，而是发明汽车的福特，捣鼓炸药的诺贝尔，种豌豆的孟德尔……同时还是要考虑将技术产业化（或许能改变生产关系），提高生产力
    
*   [kubernetes dashboard 升级之路](https://www.qikqiak.com/post/update-kubernetes-dashboard-more-secure/)
    
*   [算法源于生活](https://www.jianshu.com/p/f0dddfa11457) 这是我突然想到的一句话，也是我突然遇到的一篇文章
    
*   [Kubernetes 安装 Helm 并使用 Helm 安装 wordpress](https://www.sunmite.com/docker/kubernetes-install-helm-and-wordpress.html) Helm 的入门篇，自己参照这个部署成功
    
*   [Nginx - 启动流程](http://cxd2014.github.io/2019/01/24/nginx-init/) 在用Nginx 就需要去了解 Nginx
    
*   [24岁，我是这样给自己和父母配置保险的](https://blog.shuziyimin.org/526)
    

> 瑞克的口头禅“Wubba Lubba Dub-Dub”最早出现在“米西魔术盒”中。这在鸟人（Birdperson）世界的母语中意味着“请帮帮我，我好痛苦。”\[7\]\[8\]