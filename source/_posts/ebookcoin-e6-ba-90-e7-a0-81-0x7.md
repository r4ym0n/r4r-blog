---
title: EbookCoin 源码 0x7
url: 526.html
id: 526
date: 2018-05-28 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址 要把不。。

[](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E5%8C%BA%E5%9D%97%E9%93%BE "区块链")区块链
-------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E7%AE%80%E4%BB%8B "简介")简介

书中的这章实际上是一个狭义上的区块链，把区块链作为一个数据结构的，把它存储为一个文件形式的，不过大多数是存储在一个数据库中的。每个区块的记录都是时间上的上一个区块。所以作为一个链，可以直接回溯到第一个区块。一个区块的ID，可就是作为区块的检验和认证。 与区块链对应的直接的关系的是交易表，在cryptoCurrency体系中存在着大量的交易，在区块中包含着我们的资产信息的交易表，这样就可以查到我们公信的交易记录。

* * *

在区块链中的写操作，实际上就是在区块链上记录信息 所以在区块链上记录信息，比如这个

> Peking University teachers and classmates

或者又是一个**智能合约** 可以使得区块链系统，在特定的情况下执行一个目的的合约条件. 一个节点的操作:有创建新区块，和同步区块并解决分叉。

### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB "文件包含")文件包含

*   ./modules/blocks.js
*   ./logic/block.js
*   ./module/loader.js

blocks.js 定义区块操作函数的具体实现 block.js 定义区块这个基本的对象结构 loader.js 用来处理区块链同步事件

### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5 "基本概念")基本概念

栈用来表示一个有前后顺序的结构,在btc里面把链作为了栈的演化,第一个区块作为栈底,后面的区块按时间顺序一次的在其上面堆叠,这样 就是区块高度的这词的由来. 分叉:新区块传播时间差导致/人为分叉(系统规则改变)/软硬分叉 **堆栈方式理解数据结构,并且采用自引用的关联方式设计数据库模型,和加密解密技术,而且运行在一个P2P的网络上,有大家共同的进行维护才是一个区块链** 区块链产品的特点:

*   分布式存储
*   公开透明
*   无法篡改
*   方便追溯

区块链设计实际要解决的问题

*   加载本地的区块文件
    *   保存创世区块
    *   加载本地区块链
    *   验证本地区块链
*   处理新区块
    *   创建新区块
    *   写入区块交易
    *   区块入链
    *   处理区块链分叉
*   同步区块链
    *   保持本地的区块链与网络中的区块链同步

### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0 "代码实现")代码实现

#### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E6%9C%AC%E5%9C%B0%E5%8C%BA%E5%9D%97%E6%96%87%E4%BB%B6 "本地区块文件")本地区块文件

*   **创世区块** 这里就是区块链高度是0的区块的构造函数,在启动区块链的服务的实话,会创建这个实例.其中是指定了创世区块的编码.
    
        function Blocks(cb, scope) {    library = scope;    genesisblock = library.genesisblock;    self = this;    self.__private = privated;    privated.attachApi();    privated.saveGenesisBlock(function (err) {        setImmediate(cb, err, self);    });}
    
    其具体的内容在GensisBlock里面保存着.创世去看也是一个标准区块,和普通的区块没有什么实质性的差别,其中也保存着一些交易,是对账户的初始化,和其中测试的一些内容
    
        {  "version": 0,  "totalAmount": 10000000000000000,  "totalFee": 0,  "payloadHash": "f14eafcab4ce055b04f369fcc9c1afd31dd8903f1fc7cf6674a1b6e833d200d1",  "timestamp": 0,  "numberOfTransactions": 103,  "payloadLength": 20326,  "previousBlock": null,  "generatorPublicKey": "c79750916ed4c2b6dc4a3fe9e30efbc9cf58607f4420dc64454ece02a08e1fb5",  "transactions": [    ...                  },      "signature": "e1712ea3ea45534b582c9d42b4e0dafb65615d569cd7ffd6b886533d2c3a60ec24f715126dfc0e1d54b6d98cf22fb1698fa9e6e57b28e1e1770c39f74f2e160c",      "id": "3491871562583833419"    }  ],  "height": 1,  "blockSignature": "b7312773191f876cd69b31a2b4815755a77b05df49e23b3c10dad2e3ff220478b46d1825df552c9ea48771a19a6efa1bf5882c92917c9b65324875c831515c06",  "id": "15719620766428602938"}
    
*   **区块文件加载** 加载本地的数据库文件,先是查询数据库,之后进行解析
    
        Blocks.prototype.loadBlocksOffset = function (limit, offset, verify, cb) {    var newLimit = limit + (offset || 0);    var params = {limit: newLimit, offset: offset || 0};    library.dbSequence.add(function (cb) {        library.dbLite.query("SELECT " +            "b.id, b.version, b.timestamp, b.height, b.previousBlock, b.numberOfTransactions, b.totalAmount, b.totalFee, b.reward, b.payloadLength, lower(hex(b.payloadHash)), lower(hex(b.generatorPublicKey)), lower(hex(b.blockSignature)), " +            ...    // 很长的数据库查询语句            "", params, privated.blocksDataFields, function (err, rows) {            // Notes:            // If while loading we encounter an error, for example, an invalid signature on a block & transaction, then we need to stop loading and remove all blocks after the last good block. We also need to process all transactions within the block.            if (err) {                return cb(err);            }            // 查询结果保存在 row里.后面进行解析            var blocks = privated.readDbRows(rows);            async.eachSeries(blocks, function (block, cb) {                async.series([                    function (cb) {                        // 判断是不是创世区块                        if (block.id != genesisblock.block.id) {                            if (verify) {                                if (block.previousBlock != privated.lastBlock.id) {                                    return cb({                                        message: "Can't verify previous block",                                        block: block                                    });                                }                                try {                                    // 这里对整个链进行交易                                    var valid = library.logic.block.verifySignature(block);                                }                                 ...    // 这里如果区块校验失败.那么删除当前区块,和后面的区块                                if (!valid) {                                    // Need to break cycle and delete this block and blocks after this block                                    return cb({                                        message: "Can't verify signature",                                        block: block                                    });                                }                                modules.delegates.validateBlockSlot(block, function (err) {                                    if (err) {                                        return cb({                                            message: "Can't verify slot",                                            block: block                                        });                                    }                                    cb();                                });                            } else {                                setImmediate(cb);                            }                        } else {                            setImmediate(cb);                        }                    }, function (cb) {                        // 对交易内容进行排序,使得投票和签名交易排在前面                        block.transactions = block.transactions.sort(function (a, b) {                            ... //                            return 0;                        });                        async.eachSeries(block.transactions, function (transaction, cb) {                            if (verify) {                                modules.accounts.setAccountAndGet({publicKey: transaction.senderPublicKey}, function (err, sender) {                                    if (err) {                                        ... //                                     }                                    if (verify && block.id != genesisblock.block.id) {                                        library.logic.transaction.verify(transaction, sender, function (err) {                                            if (err) {    ...                                            }                                            privated.applyTransaction(block, transaction, sender, cb);                                        });                                    } else {                                        // 不需要验证,或者是创世区块                                        privated.applyTransaction(block, transaction, sender, cb);                                    }                                });                            } else {                                setImmediate(cb);                            }                        }, function (err) {                                if (err) {    // 这里是发生错误的时候                                library.logger.error(err);                                var lastValidTransaction = block.transactions.findIndex(function (trs) {                                    return trs.id == err.transaction.id;                                });                                var transactions = block.transactions.slice(0, lastValidTransaction + 1);                                // 进行数据回滚                                /// ...                            } else {                                privated.lastBlock = block;    // 更新区块                                modules.round.tick(privated.lastBlock, cb);                            // ...
    

    这里是区块的签名校验的函数,可以清楚的看到,对区块的合法性的校验,是基于对取 hash 的值的的校验.    Block.prototype.verifySignature = function (block) {        var remove = 64;        try {            var data = this.getBytes(block);            var data2 = new Buffer(data.length - remove);            for (var i = 0; i < data2.length; i++) {                data2[i] = data[i];            }            var hash = crypto.createHash('sha256').update(data2).digest();            //保存其hesh            var blockSignatureBuffer = new Buffer(block.blockSignature, 'hex');            var generatorPublicKeyBuffer = new Buffer(block.generatorPublicKey, 'hex');            // 验证hash            var res = ed.Verify(hash, blockSignatureBuffer || ' ', generatorPublicKeyBuffer || ' ');        } catch (e) {            throw Error(e.toString());        }        return res;    }

#### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E6%96%B0%E5%8C%BA%E5%9D%97 "新区块")新区块

*   创建新区块 在创建一个新的区块，对待打包的交易数据进行校验，合法的校验将会被打包到区块中去，后面的process函数，负责区块信息的后面的处理。
    
        Blocks.prototype.generateBlock = function (keypair, timestamp, cb) {    var transactions = modules.transactions.getUnconfirmedTransactionList();    var ready = [];    // 这里是一个each的用法,用于遍历这个待处理的交易    async.eachSeries(transactions, function (transaction, cb) {        // 先检验sender的合法性        modules.accounts.getAccount({publicKey: transaction.senderPublicKey}, function (err, sender) {            if (err || !sender) {                return cb("Invalid sender");            }            if (library.logic.transaction.ready(transaction, sender)) {                // 这里对交易进行交易,图个合法之后我们把他压入已确认的区块中                library.logic.transaction.verify(transaction, sender, function (err) {                    ready.push(transaction);                    cb();                });            } else {                setImmediate(cb);            }        });    }, function () {        try {            // 之后就是创建这个区块了            var block = library.logic.block.create({                keypair: keypair,                timestamp: timestamp,                previousBlock: privated.lastBlock,                transactions: ready // 已准备好的交易            });        } catch (e) {            return setImmediate(cb, e);        }        // 处理其他字段        self.processBlock(block, true, cb);    });};
    
    在这里，没有使用像btc一样的**默克尔树**的结构，看样子是直接的一个list的形式。 Process 是对区块的其他的属性进行的处理的函数。从上面的代码可以看出，主要的部分有：keypair，timestamp，previous，和已经OK了的交易记录 ready。函数中是有大量的判断语句，所以这里从简分析
    
        Blocks.prototype.processBlock = function (block, broadcast, cb) {    ...    privated.isActive = true; // 区块在处理    library.balancesSequence.add(function (cb) {        try {            block.id = library.logic.block.getId(block);        }     // 链高度增长    block.height = privated.lastBlock.height + 1;    // 这里命名可能有点不合理，实际上应该是判断是不是该对当前块内容进行撤销    modules.transactions.undoUnconfirmedList(function (err, unconfirmedTransactions) {        // 这里是发生撤销的闭包回调        function done(err) {            modules.transactions.applyUnconfirmedList(unconfirmedTransactions, function () {                privated.isActive = false;                setImmediate(cb, err);        });        // 这里是对于错误情况的判断        // 区块前驱不合法        if (!block.previousBlock && block.height != 1) {            return setImmediate(done, "Invalid previous block");        }        var expectedReward = privated.milestones.calcReward(block.height);        // 区块奖励不合法        if (block.height != 1 && expectedReward !== block.reward) {            return setImmediate(done, "Invalid block reward");        }        // 本地数据库中查询        library.dbLite.query("SELECT id FROM blocks WHERE id=$id", {id: block.id}, ['id'], function (err, rows) {            try {                // 验证区块合法性                var valid = library.logic.block.verifySignature(block);            }             // 如果区块的前驱区块(相同的高度)不相同，说明发生了分叉            if (block.previousBlock != privated.lastBlock.id) {                // Fork same height and different previous block                modules.delegates.fork(block, 1);                return done("Can't verify previous block: " + block.id);            }            ... //            // 这里是对区块的时间戳的验证，其产生的时间，不能小于前驱，也不能太长            var blockSlotNumber = slots.getSlotNumber(block.timestamp);            var lastBlockSlotNumber = slots.getSlotNumber(privated.lastBlock.timestamp);            if (blockSlotNumber > slots.getSlotNumber() || blockSlotNumber <= lastBlockSlotNumber) {                return done("Can't verify block timestamp: " + block.id);            ... //        }    }    ...}
    

    > 问: 怎么叫做日趋一致和准确,如果节点自己取，也许会取到一个完全错的时间> 答: 类似时间之矢（时间戳的链）。如果你挖矿成功，可以自己定义这个区块的时间戳，但是但不能早于上一个区块，也不能过于太晚，否则当区块广播出去后，其他节点发现，时间晚于自己的时间，就有可能拒接该区块。作者：大圣2017链接：https://www.jianshu.com/p/4fbcb87e05b8來源：简书著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

#### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E5%88%86%E5%8F%89%E5%92%8C%E5%90%8C%E6%AD%A5 "分叉和同步")分叉和同步

**分叉** 当分叉发生的时候，这个是一个事件函数，绑定的是收到区块广播。

    // EventsBlocks.prototype.onReceiveBlock = function (block) {    //  正在同步，就直接忽略    if (modules.loader.syncing() || !privated.loaded) {        return;    }    library.sequence.add(function (cb) {        if (block.previousBlock == privated.lastBlock.id && privated.lastBlock.height + 1 == block.height) {            // 前驱块高度相等，而且当前块的高度是本地块的后继，即标准区块            library.logger.log('Recieved new block id: ' + block.id + ' height: ' + block.height + ' slot: ' + slots.getSlotNumber(block.timestamp) + ' reward: ' + modules.blocks.getLastBlock().reward);            self.processBlock(block, true, cb);        } else if (block.previousBlock != privated.lastBlock.id && privated.lastBlock.height + 1 == block.height) {            // Fork right height and different previous block  <<<---发生了分叉 同高度，父块不同            modules.delegates.fork(block, 1);            cb("Fork");        } else if (block.previousBlock == privated.lastBlock.previousBlock && block.height == privated.lastBlock.height && block.id != privated.lastBlock.id) {            // Fork same height and same previous block, but different block id <<<---发生了分叉 同父块同高度，交易内容(ID)不同             modules.delegates.fork(block, 4);            cb("Fork");        } else {            cb();        }    });};

**区块链同步** 节点定时的轮询网络的最新区块高度，从而及时的进行同步

    privated.loadBlocks = function (lastBlock, cb) {    // 这里就是向数据库中的随机节点发送区块高度的请求    modules.transport.getFromRandomPeer({        api: '/height',        method: 'GET'    }, function (err, data) {    //  得到区块高度后        var peerStr = data && data.peer ? ip.fromLong(data.peer.ip) + ":" + data.peer.port : 'unknown';        if (err || !data.body) {            library.logger.log("Failed to get height from peer: " + peerStr);            return cb();        }        library.logger.info("Check blockchain on " + peerStr);        data.body.height = parseInt(data.body.height); // parse解析        // 老规矩的对象正则        var report = library.scheme.validate(data.body, {            type: "object",            properties: {                "height": {                    type: "integer",                    minimum: 0                }            }, required: ['height']        });        if (!report) {            library.logger.log("Failed to parse blockchain height: " + peerStr + "\n" + library.scheme.getLastError());            return cb();        }        // 目前的本地高度，比接收到的数据要低        if (bignum(modules.blocks.getLastBlock().height).lt(data.body.height)) { // Diff in chainbases            privated.blocksToSync = data.body.height; // 将会同步到            // 判断本地的最新的区块是不是创世区块，如果是，就完整更新，不是就只是部分更新            if (lastBlock.id != privated.genesisBlock.block.id) { // Have to find common block                privated.findUpdate(lastBlock, data.peer, cb);            } else { // Have to load full db                privated.loadFullDb(data.peer, cb);            }        } else {            cb();        }    });};

这里就是部分更新的函数实现。

    privated.findUpdate = function (lastBlock, peer, cb) { // 传入了目标同步节点的地址    var peerStr = peer ? ip.fromLong(peer.ip) + ":" + peer.port : 'unknown';    library.logger.info("Looking for common block with " + peerStr);    // 找到差异的区块高度    modules.blocks.getCommonBlock(peer, lastBlock.height, function (err, commonBlock) {        if (err || !commonBlock) {            return cb(err);        }        library.logger.info("Found common block " + commonBlock.id + " (at " + commonBlock.height + ")" + " with peer " + peerStr);        // 最新的区块长度，减去共有的区块长度        var toRemove = lastBlock.height - commonBlock.height;        // 差的太大就不从这里更新了，提高稳定性        if (toRemove > 1010) {            library.logger.log("long fork, ban 60 min", peerStr);            modules.peer.state(peer.ip, peer.port, 0, 3600);            return cb();        }        var overTransactionList = [];            //...            async.series([                function (cb) {                    if (commonBlock.id != lastBlock.id) {                        modules.round.directionSwap('backward', lastBlock, cb);                    } else {                        cb();                    }                },                function (cb) {                    library.bus.message('deleteBlocksBefore', commonBlock);                    // 删除共同块之后的侧链(分叉链)                    modules.blocks.deleteBlocksBefore(commonBlock, cb);                },                function (cb) {                    if (commonBlock.id != lastBlock.id) {                        modules.round.directionSwap('forward', lastBlock, cb);                    } else {                        cb();                    }                },                function (cb) {                    library.logger.debug("Loading blocks from peer " + peerStr);                    // 删除侧链之后，从peer加载区块数据，实现自身数据的同步                    modules.blocks.loadBlocksFromPeer(peer, commonBlock.id, function (err, lastValidBlock) {

### [](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#summary "summary")summary

这一部分，实现了一个区块链的运行时的操作处理，是对比前面所有部分的一个综合应用。 实现了本地区块的加载，新区块的生成，以及分叉的处理，及区块同步。 原理，在注释和代码阅读时候，可以基本的理解，不过，其具体的实现细节，的确还是比较复杂的。

[](https://www.diglp.xyz/2018/05/28/BC_EbookCoin_0x7/#%E5%90%8E "后")后
---------------------------------------------------------------------

竟然把两周过的如此之快。悔之