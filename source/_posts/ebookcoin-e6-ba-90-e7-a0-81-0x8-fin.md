---
title: EbookCoin 源码 0x8 FIN
url: 524.html
id: 524
date: 2018-05-30 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E5%89%8D "前")前
----------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

这章是本书的，区块链项目的最后的一个部分了

[](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#DPoS%E6%9C%BA%E5%88%B6 "DPoS机制")DPoS机制
---------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E7%AE%80%E4%BB%8B "简介")简介

很多人说，整个区块链就是个分布式的软件嘛。实际不然，区块链的灵魂，就是存在于共识这两个字，共识才是区块链的灵魂。

* * *

前面的博文其实也是讲过共识的这个东西。在区链里的这个概念太重要了。 拜占庭将军与共识算法  
Py源码_挖矿

* * *

“拜占庭将军问题” 就是针对分布式公式算法提出的，这个问题也是区块链产品 的核心问题.

> 比特币是如何解决拜占庭将军问题的：
> 
> *   维持周期循环，保持节点步调一致
> *   通过算力竞赛，确保网络单点广播
> *   通过区块链使用一个共同账本

上面的三点就是实现一个比特币的PoW机制解决 **拜占庭将军问题的答案**。这也给其他的答案提供了重要的参考答案：

> 只要保证时间统一，步调一致，单点广播，一个链条 就可以解决分布式系统的拜占庭将军问题。

* * *

**DPoS** 机制 亿书区块链由受托人来创建区块，受托人是来自于用户节点的信任，通过推广和投票实现前 101名。这一就可以被系统接纳作为为真正的可以处理新区块的节点，并且可以得到区块奖励。至之于PoW的共识算法，需要最大算力。DPoS算是发挥了社区力量。不过实际上，通过票选这样的东西，我不是很看好，这样极大的提高了准入门槛。使得，投票，拉票这样的模式，又一次搬上了BC，区链的节点被固定在了这百来个节点之上。 在DPoS的过程中是存在了这样的几个过程：

*   注册受托人，接收投票
    *   用户申请自己成为受托人，可以想做就是被选举人
    *   接收投票
*   维持循环，调整受托人
    *   块周期：时段周期，也就是出块时间
    *   受托人周期：称为循环周期，每101个区块，重新产生了受托节点的名单。
    *   奖励周期：由区块链的高度，来不断的调整区块奖励的量  
        块周期最短 10s，受托人周期 16min， 区块奖励周期 不确定
*   循环产生新区块，广播
    *   源码

### [](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB "文件包含")文件包含

*   ./modules/delegates.js
*   ./modules/round.js
*   ./modules/accounts.js
*   ./helper/slots.js

delegates.js 受托人的相函数，申请，投票等 round.js 受托人的循环周期 accounts.js 实现用户的投票功能 slots.js

### [](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E5%8A%9F%E8%83%BD%E5%AE%9E%E7%8E%B0 "功能实现")功能实现

*   注册受托人 在网络上注册成为一个受托人 的函数实现
    
        "put /": "addDelegate"
    
    上面是绑定的路由接口，下面的是具体的功能实现
    
        shared.addDelegate = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {        ... // 对象结构验证        },        required: ["secret"]    }, function (err) {        if (err) {            return cb(err[0].message);        }        var hash = crypto.createHash('sha256').update(body.secret, 'utf8').digest();        var keypair = ed.MakeKeypair(hash);        // 验证密码        if (body.publicKey) {            if (keypair.publicKey.toString('hex') != body.publicKey) {                return cb("Invalid passphrase");            }        }        library.balancesSequence.add(function (cb) {            if (body.multisigAccountPublicKey && body.multisigAccountPublicKey != keypair.publicKey.toString('hex')) {                modules.accounts.getAccount({publicKey: body.multisigAccountPublicKey}, function (err, account) {                    if (err) {                        return cb(err.toString());                    }                            ... //                    modules.accounts.getAccount({publicKey: keypair.publicKey}, function (err, requester) {                        ... //                        try {                            // 这里创建一个交易，里面的type就是委托                            var transaction = library.logic.transaction.create({                                type: TransactionTypes.DELEGATE,                                username: body.username,                                sender: account,                                keypair: keypair,                                secondKeypair: secondKeypair,                                requester: keypair                            });                        } catch (e) {                            return cb(e.toString());                        }                        modules.transactions.receiveTransactions([transaction], cb);                    });                });            } else {                ... //
    

* * *

*   投票 同样的这也是一种交易方式，最早是出现在了 0x6 里面，作为一种 TYPE 出现。在 accounts 里面实现的。
    

* * *

*   块周期 这个周期可能是类比与 PoW 里面的一个块的计算周期，比特币的周期是 10分钟左右，因为 PoW 是基于概率出块的。所以是左右。但是在DPoS的机制下，在 ebook 里面是准确的 10秒钟的出块时间。
    
    *   Epoch 时期；新纪元；新时代；阶段
    *   Slot 槽，窗口 时间的计算是通过这两个函数得到的时间节点，第一个是硬编码的创世时间，第二个是当前的时间 function beginEpochTime() {
        
            var d = new Date(Date.UTC(2016, 5, 20, 0, 0, 0, 0)); //Testnet starts from 2016.6.20return d;
        
        } function getEpochTime(time) {
        
            if (time === undefined) {    time = (new Date()).getTime();    // 这里存在的问题是，这个是直接获取本地时间，如果不同的机器之间的本地时间是存在偏差的，那么可能导致时间不准}var d = beginEpochTime();var t = d.getTime();return Math.floor((time - t) / 1000);
        
        } 前面讲到亿书的。出块时间是 10s 所以这里的块数，直接是返回了快创世时间，对区块周期的取整 getSlotNumber: function(epochTime) {
        
            if (epochTime === undefined) {    epochTime = this.getTime()}// 这里就是直接返回了return Math.floor(epochTime / constants.slots.interval);
        
        }, **参数的意义** 这些值，出块时间必须10S吗？节点个数必须是 101个吗？ 如果我们对这个模式进行合理演绎，只有一个节点，这样的效率就得到了极大的提高，不过实际上，这就是一个单点系统，安全性就是 0 了，所以这个是一个折衷的过程。
        

* * *

*   受托人周期 作为中心节点的受托人的循环周期，前面有讲是 101 个区块周期，发生一次调整。及时更换公信节点。网络是随机的在101个节点中选取产出者，不过每个节点在一个周期内都是会有一次机会产出新的块的(只是顺序随机)。 这样在每个区块的信息里面，会有 区块高度 (height) 和 产生机器的公钥(Generator Public Key)是严格对应的.在运行的节点，会有一个表，是用来维护当前的 受托人的内容的，具体的实现如下，看起来有点难受
    
        Round.prototype.tick = function (block, cb) {    function done(err) {        cb && setImmediate(cb, err);    }    modules.accounts.mergeAccountAndGet({        ... //        // 计算当前的委托人周期轮数，是可以通过其进行计算的 r=h/101        var nextRound = self.calc(block.height + 1);
    

            if (round !== nextRound || block.height == 1) {            // 如果已经开始了下一轮的受托人循环            if (privated.delegatesByRound[round].length == constants.delegates || block.height == 1 || block.height == 101) {                // 确认满足了，新的一轮的条件                var outsiders = [];                async.series([                    function (cb) {                        // 创世区块                        if (block.height != 1) {                            // 这里是对受托人进行查表。                            modules.delegates.generateDelegateList(block.height, function (err, roundDelegates) {                                if (err) {                                    return cb(err);                                }                                // 遍历查到的                                for (var i = 0; i < roundDelegates.length; i++) {                                    // 在当前的列表中没有新的一轮的节点地址                                    if (privated.delegatesByRound[round].indexOf(roundDelegates[i]) == -1) {                                        outsiders.push(modules.accounts.generateAddressByPublicKey(roundDelegates[i]));                                    }                                }                                cb();                            });                        } else {                            cb();                        }                    },                    function (cb) {                    ... //                    },                    function (cb) {                        // 在后面进行票数更新                        self.getVotes(round, function (err, votes) {                            if (err) {                                return cb(err);                            }                            async.eachSeries(votes, function (vote, cb) {                                // 查询数据库，获取当前状态                                library.dbLite.query('update mem_accounts set vote = vote + $amount where address = $address', {                                    address: modules.accounts.generateAddressByPublicKey(vote.delegate),                                    amount: vote.amount                                }, cb);                            }, function (err) {                                self.flush(round, function (err2) {                                    cb(err || err2);                                });                            })                        });                    },                    function (cb) {                        // 对读出的节点票选数进行更新                        var roundChanges = new RoundChanges(round);                        async.forEachOfSeries(privated.delegatesByRound[round], function (delegate, index, cb) {                            var changes = roundChanges.at(index);                            modules.accounts.mergeAccountAndGet({                                publicKey: delegate,                                balance: changes.balance,                                u_balance: changes.balance,                                blockId: block.id,                                round: modules.round.calc(block.height),                                fees: changes.fees,                                rewards: changes.rewards                            }, function (err) {                                if (err) {                                    return cb(err);                                }                                if (index === privated.delegatesByRound[round].length - 1) {                                    modules.accounts.mergeAccountAndGet({                                        publicKey: delegate,                                        balance: changes.feesRemaining,                                        u_balance: changes.feesRemaining,                                        blockId: block.id,                                        round: modules.round.calc(block.height),                                        fees: changes.feesRemaining                                    }, cb);                                } else {                                    cb();                                }                            });                        }, cb);                    },                    function (cb) {                        // 这里是对每个票选项的数据库回写                        self.getVotes(round, function (err, votes) {                            if (err) {                                return cb(err);                            }                            async.eachSeries(votes, function (vote, cb) {                                library.dbLite.query('update mem_accounts set vote = vote + $amount where address = $address', {                                    address: modules.accounts.generateAddressByPublicKey(vote.delegate),                                    amount: vote.amount                                }, cb);                            }, function (err) {                                library.bus.message('finishRound', round);                                self.flush(round, function (err2) {                                    cb(err || err2);                                });                            })                        });                    }                ], function (err) {                    // 删除临时空间                    delete privated.feesByRound[round];                    delete privated.rewardsByRound[round];                    delete privated.delegatesByRound[round];                    done(err);                });            } else {                done();            }        } else {            done();        }    });}

* * *

*   奖励周期 在亿书的项目中，类似于BTC的四年一减产1/2的的情况，亿书的区块奖励一样的是，随着不同的时间进行衰减。在亿书中，最后ed区块奖励是衰减到一个定值的，意味着在后面的时期，会保持着不断的同比增发。引起适当的通胀。这也是DPoS的一个特点。 这种，不同于 BTC 这是一种通胀货币，作者也有提到，说是担心对降低代币的价值。实际上，如果有大量的侧链，必须是要有一定的代币的产生来提供各种侧链产品的使用。不会导致**主链和侧链绑定紧密，互相掣肘**
    
    > **以太坊的运行，侧链应用使用主链币进行众筹时候，此消彼长，价格波动**
    

    具体的实现细节很简单，就是维护一个 reward 的值就好了。    function Milestones() {    var milestones = [        500000000, // Initial Reward        400000000, // Milestone 1        300000000, // Milestone 2        200000000, // Milestone 3        100000000  // Milestone 4    ];这里就是奖励衰减的情况的五个里程碑

### [](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E6%80%BB%E7%BB%93 "总结")总结

从共识机制上理解了DPoS这种方法，之于PoW的确有很大的不同，更像是一种民主制度。通过票选我们的代表人,来实现区块的自治. **时间统一，步调一致，单点广播，一个链条** 便是一个分布式系统的特性. 而且有 块周期,受托人周期,奖励周期这样几个周期

[](https://www.diglp.xyz/2018/05/30/EbookCoin_0x8_FIN/#%E5%90%8E "后")后
----------------------------------------------------------------------

这一篇,就是关于亿书的这个工程的最后一篇源码的文章了,很美可能会有奇迹对于项目总体的一篇总结文. 一个区块链的体系,从各个部分功能的实现,和一个总体的构成,的确是一个精妙的过程. 这里对 PoW和PoS这种共识的方式,做点自己的见解. 如我们熟知的黄金一样,对于金矿的开采, 处理这一过程,实际上会消耗大量的资源和成本,实际上我们的BTC体系也是如此,会消耗大量的资源.来产生这一的一个东西.所以不赞成,向部分人说的那样,把资源消耗在了没有意义的计算上 . 像是POS这种机制,更多的是人们相信这个币是有价值的,是一种信任.和我们手上的纸币一样,其成本的价格实际上远远不足其本身的价值,只不过,是国家赋予了他的价值,我们信任它,我们认为他是有价值的,所以我们可以使用它,可是如果我们用人民币在美利坚大陆上,可能就是没有那么好使了.所以说,PoS的代币的价值是产生于我们的信任,如果存在着信任危机,那么这种币的价值是岌岌可危的. …