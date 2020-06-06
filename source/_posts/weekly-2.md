---
title: Weekly.2
url: 698.html
id: 698
categories:
  - 每周分享
date: 2019-04-04 20:25:52
tags:
---

前面的话
----

这周事相当咸鱼的一周，也不不知道做了些什么，就浑浑噩噩的过去唉，可能做的最多的就是这个小破站的迁移和优化吧（就是您现在打开的这个），每天折腾折腾，有了现在的精美的界面和比较优秀的访问速度，还是比较有成就感的事情。 从最开始的自己写的简单的 compose 到后面的加上 Redis ，整个的工程还是蛮综合的。后面一步是把反代给换掉，换成 Traefik。

事
-

1.  关于\[996.icu\]( [https://996.icu](https://996.icu)) 🤺链接在此： 这种最热门的，也是不得不说的事件就是这个996了，上面的链接内容便是本质，上班九九六以后 ICU。这里给出一个图大家品品咯 ![](/wp-content/uploads/2019/04/B8C4XF4ZMA9FKSPQVAK.jpg) 狗都困了，所谓一个互联网人，其实对于加班这个事实也是颇有感觉的。朋友老是说：**那你早点走啊** 事实呢是:你发现身边的同时都在疯狂输出的时候，你怎么（敢）能走呢？？？
    
    > 国家实行劳动者每日工作时间不超过八小时、平均每周工作时间不超过四十四小时的工时制度。 ----知法懂法学法用法
    
    Website
    
    DESC
    
    955.WLB
    
    is a repo that maintains a whitelist of 955 work–life balanced companies. It promotes people to flee 996 and join 955.
    
    996.LAW
    
    is a repo that collects useful information about cases between employees and enterprise.
    
    996.TSC
    
    is a repo designed to let more people know and join the activity of 996.ICU.
    
    996.LIST
    
    is a repo of a rank list of 996 companies and 955 companies.
    
2.  Google 的门户搜索框被挖到 XSS 这里的XSS 当然是反射型的 XSS。不过，想着这个每天UV上十亿的门户站上存在这样的洞。还是觉得 **网络安全** 真的还有很远很远的路要走，因为 矛和盾是一起进化的。 当然，这个漏洞已经被fix了，下面是这个 Github上面的修复的commit
    
    > [Automated g4 rollback of changelist 214621663.](https://github.com/google/closure-library/commit/c79ab48e8e962fee57e68739c00e16b9934c0ffa "Automated g4 rollback of changelist 214621663.")
    
    简单的介绍以下何为反射型XSS，别人可以使用自己构造的链接，通过受害者点击，来被执行一些交互操作，简单的有删除，登陆，Cookie等等，负载的可能可以给你的联系人发邮件。
    
3.  AI有意思： ![](/wp-content/uploads/2019/04/MTSL7TJG31S269Z76HW-1-300x208.png) 偶然在 Twitter 上看见了一个不得了的东西，上面的这段英语就不多做翻译了。的确像有一句话里面说到：`在宏观世界里，当群体人数众多的时候，同性相斥异性相吸的强相互作用力其效果将会明显的显现出来。`。对于技术的普及还是这些原始驱动力来的快。你说这个 facesweep 可以帮人拍电影，屁民心想，和我啥关系。可是你一旦前面加了个小字。这技术就是蓬勃发展了。

文章
--

*   [跟着“创业“三年半的日子](https://pjf.name/blogs/three-years-life-in-startup-company.html) # 一篇蛮有感触的文章，虽然不曾经历
    
    > 马云说，一个人辞职只有两个原因：一是钱太少，二是干得不开心了。而我不幸的是两者皆有。
    > 
    > 最受打击的应该是看清楚了CEO的本质。从最开始相信他是一个做实事，靠谱的CEO到最后发现只不过**是一个赌徒**，满足个人私欲的“骗子”。自己的情怀，寄托在这里能实现财富自由的梦也被无情的击碎。除了心酸外，再也找不到其他词语能够表达我的情感了。
    
    讲到为什么呆在这里，作者写了一段话：`“我说是情怀的话，肯定会被笑死吧。然而真相就是这样，这个项目大部分代码都是我无数个通宵，无数杯咖啡一行一行码出来，看着就像是自己的孩子一样。”` 当自己还有这梦想的是后何尝不是这样子的呢。相信他人，相信未来，是可以像《硅谷》这个剧里面的那样激动人心。但是试试往往更有可能是寒心的结果，这个让我想到当时在车站给一个素不相识的路人转了两百块钱的事情。现在想想心里还是不舒服。
    
    > 最重要的，我明白了创业最最重要的东西，那便是坚持。很多人，走着走着就散了。很多事情，做着做着就忘记了原本的目的。创业也是一样，如果不能时刻记得最初的愿景，死亡只是眼前的事情。作为一家公司的CEO，如果只会给别人画大饼，却长时间连员工的基本保障都不能给予的时候，应该感到羞愧，也是一个人不负责的表现。当我未来某一天决定创业时，这些经验一定能让我少走很多弯路。如此想来，也不算是一件坏事了。
    

PICKS
-----

这次推荐点什么好东西呢？想想，那就看书吧，推荐看书的方法。 如何快速的取了解一个新的技术，或者是新的东西？那就去看书吧，厚的薄的就看，一两本翻完基本上也就有了基础的了解了。没错这里是翻完：

所以这里推荐 Gitbook 平台，准确是一种发布方式，可以让作者取创建属于自己的书呢。

> [wizardforcel](https://legacy.gitbook.com/@wizardforcel) 像是这个，我就在上面扒过不少好书。通俗易懂，简短干货

另外直接使用 Google 的高级搜索，我们也可以很快找到类似的书比如：

    site:gitbook.com 软件测试

![file](/wp-content/uploads/2019/04/image-1554380538995.png) 达拉，很容易找到自己想看的书了XD

Tech
----

*   [Zsh下配置Docker命令补全](https://escapelife.github.io/posts/1c151aac.html)
    
*   [Docker安装typecho](https://pjf.name/blogs/install-typecho-by-docker.html)
    
*   [Traefik 详解](https://my.oschina.net/guol/blog/2209678)
    
*   [Traefik 另类的服务暴露方式](https://mritd.me/2018/05/24/kubernetes-traefik-service-exposure/)
    
*   如果想让自己的东西从能用就行，到变得优雅。最好的建议就是取看看他的官方的文档吧
    
*   [Traefik 实战(traefik+docker swarm)](https://my.oschina.net/guol/blog/1942373) 关于Treafik 的一批很好的初步实践
    
*   [將你的網站加上 https — Cloudflare](https://medium.com/tool-s/%E5%B0%87%E4%BD%A0%E7%9A%84%E7%B6%B2%E7%AB%99%E5%8A%A0%E4%B8%8A-https-cloudflare-6569f5365c7d) 用cloudflare 给网站轻松 CDN
    
*   [用Typecho Redis Cache来为Typecho提供全站超高速缓存](http://ju.outofmemory.cn/entry/336117)

后
-

这周的内容的确显得太敷衍了，如果您作为一个读者，我就先说声抱歉了。 因为这周琐事太多也不知道自己做了啥就过去了。 `给自己的话，就是继续完成标签收敛的任务`