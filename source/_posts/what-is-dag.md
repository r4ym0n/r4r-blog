---
title: What is DAG?
url: 520.html
id: 520
date: 2018-06-03 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/06/03/What_is_DAG/#What-is-DAG "What is DAG")What is DAG
--------------------------------------------------------------------------------------

> DAG意思是有向无环图，所谓有向无环图是指任意一条边有方向，且不存在环路的图。\[百科\]

DAG 是一种在图论中定义的结构，DAG 是**（Directed acyclic graph）**的缩写，翻译过来的意思是有向无环图。如下图所表示的。 ![](https://upload-images.jianshu.io/upload_images/796321-6bd22461289a94bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/680) 这样有向无环是指它是由集合的顶点和有向边构成，每条边连接一个顶点到另一个，这样，假如顶点A开始，沿着有序的边，是无法回到最初的起点的。

[](https://www.diglp.xyz/2018/06/03/What_is_DAG/#DAG-%E4%B8%8E-%E5%8C%BA%E5%9D%97%E9%93%BE "DAG 与 区块链")DAG 与 区块链
----------------------------------------------------------------------------------------------------------------

实际上，在严格意义上讲，DAG已经是不能算作区块链了。因为这玩意，**没有区块，也没有链的** 在我们的传统BTC的体系下，衍生出了区块链这个词。正如其名，**区块构成的链**，如下图。 ![](https://upload-images.jianshu.io/upload_images/796321-fa58f376b01c7061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700) 在图中，把DAG也作为了对比，相比较而言，DAG显得是没有那么的工整了，有点乱糟糟的感觉，不过，我们都能观察到一个特点，上图的两个结构，所表现的都是一个有向的结构，左边的区块链的从起点，到最长的端点。在DAG里，一样的是从起点，指向最远的端点。 在经典链式结构里，每个区块的产生，需要进行PoW的过程，符合要求的块，将会被加入链中，当发生分叉的时候，最长链是付出最多工作量的链，所以是以最长链为准。 然而在DAG的体系里，区块是不存在的了，每个新加入的单元，我们是称之为**交易(TX)**，当一个新的交易产生的时候，不单单是作为一个单位，接在另一个单位的后面，**而是加入到之前的所有(N个)单位的后面**。所以当你发布新交易时，前面有两个有效交易，那么你的区块会主动同时链接到前面两个之中， DAG 中的每个新单元，验证并确认其父辈单元，父辈单元的父辈单元，慢慢可达创世单元，并将其父辈单元的哈希包含到自己的单元里面。随着时间递增，所有交易的区块链相互连接，形成图状结构，如若要更改数据，那就不仅仅是几个区块的问题了，而是整个区块图的数据更改。

* * *

那么相比之下呢？

*   单元：在区块链体系下基础单元是Block（区块），而在DAG组成单元是TX(Transaction)（交易）；
*   拓扑结构：区块链是由Block区块组成的**单链**，只能按出块时间同步依次入链，单线程的处理网络的广播消息；DAG是由交易单元组成的网络，交易和交易可以异步并发，同时的写入交易，多线程的进行网络的处理；
*   粒度：区块链每个区块单元记录多个用户(10Min)的多笔交易，DAG每个单元记录单个用户交易。

相比之下，DAG这种结构的应用，比传统区块链，是比较复杂的。不过以交易作为最小的粒度的模式，使得区块的效率得到了很大的提高。

* * *

传统区块链技术的几个问题

1.  TPS：指的是每秒处理的交易数，在区链的体系里，每个区块是单位，包含着很多的交易，这样经过一个长达十分钟的区块周期，比特币的效率一直比较低，由于BlockChain链式的存储结构，整个网络同时只能有一条单链，基于POW共识机制出块无法并发执行；十分钟出一个块，6个出块才能确认(pow(1/2,6)即0.015625)，大约需要一个小时；以太坊大幅改善，出块速度也要十几秒。
    
2.  **确定性问题**：这个是当前的区链系统最大的问题，就是确定性问题。 在一条链上，通过PoW形成的链是没有一个最终的确定状态，比特币和以太坊存在51%算力攻击问题，基于POW共识的最大问题隐患；只要有隐藏着的极为强大的算力，和一个隐藏的足够长的链，这样的孤岛连接主网，瞬间可能巅峰目前的一切；考虑到现实中的ASIC，以及量子计算机，这种危险现实存在。
    
3.  中心化问题：基于区块的POW共识中， 矿工一方面可以形成集中化的矿场集团，另一方面，获得打包交易权的矿工拥有巨大权力，可以选择哪些交易进入区块，哪些交易不被处理，甚至可以只打包符合自己利益的交易，这样的风险目前已经是事实存在。
    

* * *

DAG 的特性 在DAG结构中，为了避免双花问题，当然还是有所谓的”主链“概念，不过实际上是经过验证认定的一条最短的交易路径。 不想单单的一个是节点产生的区块，在DAG的条件下，每个产生的交易，都会引用过去的 N 个交易，这样，就会证明前一个交易的合法性，从而间接证明所有交易的合法性。 这样的DAG就会有了很大的并行计算的能力，每个交易之间不再是打包成一个区块而是根据高度进行引用。没有手续费，交易产生的越多，处理速度反而是更快 所以看起来DAG的确是有不少优点的存在。

* * *

> 分布式系统中的一大难题就是，在可分区、一致性、可及性三者间，只能满足其二。 区块链在高效率低能耗、去中心化和安全三个方面，只可选其二，存在“不可能三角”悖论。

[](https://www.diglp.xyz/2018/06/03/What_is_DAG/#%E5%B1%95%E6%9C%9B%E6%9C%AA%E6%9D%A5 "展望未来")展望未来
-------------------------------------------------------------------------------------------------

> “十到二十年之后，区块链技术发展到极致，彻底解决了可扩展性，安全性，易用性的问题之后，终局会是什么样子？ 世界上八十亿人，上万亿台机器，可以瞬间（迟滞低于一秒）通过区块链网络进行无需中介，无需许可，自动编程，费用极低的，可以信赖的价值交换，价值交换的单位可以低到一分，一厘，甚至更低。 这是一个可以期待的发展局面之一。 这样的价值交换网络铺开后，会催生以前不可能有的应用…… 率先在扩展性和易用性上达到一小时可以支持上百万，上亿笔交易的价值区块链，将会成为各种无法想象的应用的基石，吸引更多的用户和开发者。而各种应用的丰富，又将使这个区块链更加有价值，导致良性循环，强者亦强。”
> 
> —\- 引用自王川