---
title: EbookCoin 源码 0x6
url: 530.html
id: 530
date: 2018-05-16 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

[](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E4%BA%A4%E6%98%93 "交易")交易
--------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E7%AE%80%E4%BB%8B "简介")简介

通过前面的文章知道了，交易在区块链体系下实际上是一个泛化。其实指的是数据在链上的状态，和操作过程。所以交易这个功能，是相当于前面的所以的东西的一种综合显得尤为重要。在书中也是摊牌，说第13/14章的确是存在着细节的隐瞒。这里要深入的学习了。

* * *

在比特币体系下的一个简单的交易其实现的功能，简单说，就是把比特币从一个账户转移到另一个账户，并且其中的部分金额被作为矿工费用给了负责进行交易校验的矿工。 更为深入的讲解其过程实际上是，一笔已经被签名了的 **交易** 被用户广播到区块链网络上去。之后由矿工进行收集和封包，到区块中去，最终进行PoW过程，得到正确的Hash后，区块将会永远的被写入到链上。 在区块链中的交易，实际上只是一串字节码，其中是没有任何的机密信息，比如私钥或者密码。所以他可以是任何的记录形式，只要这个数据记录在了区块链的网络上。**发送者不需要信任任何广播该交易的节点，同时节点也不需要信任任何广播该交易的发送者**

### [](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB "文件包含")文件包含

*   ./modules/transactions.js
*   ./logic/transaction.js
*   ./helper/transaction-types.js

transactions.js 交易过程进行所需要的模块 transaction.js 实现一个交易的对象结构 transaction-types.js 一个辅助定义，用于类似的宏的的一个交易类型定义。

### [](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E9%87%8D%E8%A6%81%E7%90%86%E8%AE%BA "重要理论")重要理论

*   **交易的生命周期** 在链上的吊椅其目的是，正确的生成，传播和验证，并且最终入链。所以在其开发的角度讲：一次交易有以下的过程
    
    *   生成交易，如前面提到的一条交易信息
    *   广播到网络，所有网络节点都会获得交易数据
    *   验证交易，验证交易的合法性，不合法的交易信息是不予打包的
    *   写入区块链中 所以，可见交易时整个区块链系统的灵魂，一笔交易的实现，综合了前面的所有的内容，p2p， 签名， 加密。。。等等。
        

* * *

*   **交易类型** 实际上交易在区块链时泛化的，其不一定时特指的时价值的传递和转移。 不过不严谨的说 BTC 只是实现了BTC可以在不同的账户之间流动的交易过程，所以其交易种类时比较单一的。 不过我们既然是要拓展区块链的功能，那么我们在这里可以定义更多的交易过程。实际上，交易的实指，就是一次合法的信息被计入区块链的过程。
    
    * * *
    
    这里可能称之其为宏可能不是十分的合适，不过从功能上讲的确是实现了宏的功能。
    
        module.exports = {    SEND : 0,    SIGNATURE : 1,    DELEGATE : 2,    VOTE : 3,    USERNAME : 4,    FOLLOW : 5,    MULTI: 6,    DAPP: 7,    IN_TRANSFER: 8,    OUT_TRANSFER: 9,    ARTICALE: 10,    EBOOK: 11,     BUY: 12,    READ: 13}
    
    其实这个也不陌生，前面的 `transaction.create` 里面已经包含了这个交易的种类。像是签名(SIGNATURE).SEND是基本的转账交易，。。。
    

* * *

*   **交易流程** **生成,签名,广播,校验,入链,这个是区块链上一个交易的声明周期** 这里是前面常见的一个东西，就是我们的交易创建，这里也向我们展示了一笔交易的实际结构.
    
        var transaction = library.logic.transaction.create({    type: TransactionTypes.SEND,    // 交易类型    amount: body.amount,            //     sender: account,                // 发送者地址    recipientId: recipientId,        // 接收方地址    recipientUsername: recipientUsername,    keypair: keypair,    requester: keypair,    secondKeypair: secondKeypair}
    
    对一个合法的交易进行 **签名**,其中要记录其交易时间戳,用于追溯,接着就生成了交易ID,这个ID是及其复杂的,不会像数据库一样是生成的顺序的序号,是一种默克尔树的索引结构. 合法性校验,每笔被广播到网络上的交易,会被节点收集,进行合法性校验,不合法的交易信息是无法被打包的,合法的交易信息,会加入当前的区块.等待**网络心跳** 从而开始求解当前区块. 这里涉及到**双花**的问题, 双花,说明了就是一笔钱花两次.在比特币历史上是出现过双花的问题,不过在至今网络稳定的情况下.这种可能性发生的就会很小了. 其实实现的过程如下,我们在一个区块时间内**对一笔钱**进行两次交易的广播,这交易信息,可能是会被不同的节点记录和接收,这里我们把这两个节点之间的差异,叫做形成了分叉. 所以,当区块产生之后,就会由另个不同的区块.当一个区块已经被确认之后,比如说,A收到了B的钱.可是此时,A拥有更强的算力,使得另一个分叉变成了最长链,这样.给B的转账就是变成无效化的了.这就是双花攻击 笔者,其实也在一个 新的 PoW 项目中遭遇了类似的问题,不过其主要原因不是,节点作恶e,而是网络分区.
    

* * *

*   **广播点到点网络** 一但有交易广播,我们的接收到的节点,就把他分送到所有的 peerlist 的节点,具体的过程,是把信息 post 到节点的 /api/peer/transaction 这个接口去.
    

### [](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0 "代码实现")代码实现

*   **API** 这个地方的代码就是十分重要和综合的了,这里实现了一个完整的交易. 先从交易的API入手,来拆解一个交易过程的实现原理
    
        router.map(shared, {            "get /": "getTransactions",            "get /get": "getTransaction",            "get /unconfirmed/get": "getUnconfirmedTransaction",            "get /unconfirmed": "getUnconfirmedTransactions",            "put /": "addTransactions"        });        library.network.app.use('/api/transactions', router);
    

*   **添加交易** 从请求方式我们么一看出,实际上 这个**put** 是上传数据的,就是写数据,所以这里贴上其代码,进行深入分析
    
        shared.addTransactions = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {        type: "object",        properties: {            ...    // 对象结构的合法性检测            },        required: ["secret", "amount", "recipientId"]    }, function (err) {        if (err) {            return cb(err[0].message);        }        // 出现了很多遍的验证密码的过程        var hash = crypto.createHash('sha256').update(body.secret, 'utf8').digest();        var keypair = ed.MakeKeypair(hash);        if (body.publicKey) {            if (keypair.publicKey.toString('hex') != body.publicKey) {                return cb("Invalid passphrase");            }        }        var query = {};        // 进行地址正则, 不是地址,就是用户别名 (接收方 reception)        var isAddress = /^[0-9]+[L|l]$/g;        if (isAddress.test(body.recipientId)) {            query.address = body.recipientId;        } else {            query.username = body.recipientId;        }        // 在序列中进行添加        library.balancesSequence.add(function (cb) {            // 注意这个 query,这里是获取(**验证**)接收方的账户            modules.accounts.getAccount(query, function (err, recipient) {                if (err) {                    return cb(err.toString());                }                // 如果不存在接收方,之间返回交易时不会发生的                if (!recipient && query.username) {                    return cb("Recipient not found");                }                var recipientId = recipient ? recipient.address : body.recipientId;                var recipientUsername = recipient ? recipient.username : null;                //这里对发送方的账户合法性进行检验                if (body.multisigAccountPublicKey && body.multisigAccountPublicKey != keypair.publicKey.toString('hex')) {                    // 如果时多重签名的情况                    modules.accounts.getAccount({publicKey: body.multisigAccountPublicKey}, function (err, account) {                        if (err) {                            return cb(err.toString());                        }                        // 账户不存在                        if (!account || !account.publicKey) {                            return cb("Multisignature account not found");                        }                        if (!account || !account.multisignatures) {                            return cb("Account does not have multisignatures enabled");                        }                        // 账户不存在于签名组                        if (account.multisignatures.indexOf(keypair.publicKey.toString('hex')) < 0) {                            return cb("Account does not belong to multisignature group");                        }                        // 继续验证发送方                        modules.accounts.getAccount({publicKey: keypair.publicKey}, function (err, requester) {                            if (err) {                                return cb(err.toString());                            }                            //                             if (!requester || !requester.publicKey) {                                return cb("Invalid requester");                            }                            if (requester.secondSignature && !body.secondSecret) {                                return cb("Invalid second passphrase");                            }                            if (requester.publicKey == account.publicKey) {                                return cb("Invalid requester");                            }                            var secondKeypair = null;                            if (requester.secondSignature) {                                var secondHash = crypto.createHash('sha256').update(body.secondSecret, 'utf8').digest();                                secondKeypair = ed.MakeKeypair(secondHash);                            }                            try {                                // 这里创建交易,                                // 其中的信息有以上描述的许多                                var transaction = library.logic.transaction.create({                                    type: TransactionTypes.SEND,                                    amount: body.amount,                                    sender: account,                                    recipientId: recipientId,                                    recipientUsername: recipientUsername,                                    keypair: keypair,                                    requester: keypair,                                    secondKeypair: secondKeypair                                });                            } catch (e) {                                return cb(e.toString());                            }                            // 这里的处理很重要                            modules.transactions.receiveTransactions([transaction], cb);                        });                    });                } else {                    // 如果时普通签名的情况                    modules.accounts.getAccount({publicKey: keypair.publicKey.toString('hex')}, function (err, account) {                    // 基本同上                    });                }            });        }, function (err, transaction) {            if (err) {                return cb(err.toString());            }            cb(null, {transactionId: transaction[0].id});        });    });}
    
    当我们完成了一系列的操作后产生了一个交易的条目(`var transaction`),在最后,使用了
    
        modules.transactions.receiveTransactions([transaction], cb);
    
    对这个产生的交易对象进行处理
    
        Transactions.prototype.receiveTransactions = function (transactions, cb) {    // 使用串行调度, 对transactions 的所有的 transaction 进行遍历    async.eachSeries(transactions, function (transaction, cb) {        // 未经确认的交易处理函数        self.processUnconfirmedTransaction(transaction, true, cb);    }, function (err) {        cb(err, transactions);    });}
    
    上面的是交易处理函数,处理遍历交易列表每笔交易,下面是单笔交易的处理函数
    
        Transactions.prototype.processUnconfirmedTransaction = function (transaction, broadcast, cb) {    // 获取交易发送者的账户公钥    modules.accounts.setAccountAndGet({publicKey: transaction.senderPublicKey}, function (err, sender) {        // 注意,这里是定义的一个函数,实际上是在最后进行的操作,闭包函数第一次遇到,后面有总结        function done(err) {            if (err) {                return cb(err);            }            // 这里是完成后的过程,在最后...            // 这里执行的是把这个交易加入当前的区块            privated.addUnconfirmedTransaction(transaction, sender, function (err) {                if (err) {                    return cb(err);                }                // 在总线上发送一个未确认的交易的消息                library.bus.message('unconfirmedTransaction', transaction, broadcast);                cb();            });        }        if (err) {            return done(err);        }        // 一样的发送者的合法性,以及检验多重签名检验        if (transaction.requesterPublicKey && sender && sender.multisignatures && sender.multisignatures.length) {            modules.accounts.getAccount({publicKey: transaction.requesterPublicKey}, function (err, requester) {                //同下            });        } else {            // 开始交易检验和处理            library.logic.transaction.process(transaction, sender, function (err, transaction) {                if (err) {                    return done(err);                }                // 这里进行双花检验,同一个ID的交易,不允许出现两次                if (privated.unconfirmedTransactionsIdIndex[transaction.id] !== undefined || privated.doubleSpendingTransactions[transaction.id]) {                    return cb("Transaction already exists");                }                // 该交易已经通过校验,之后对 DONE 进行回调实现了交易的打包                library.logic.transaction.verify(transaction, sender, done);            });        }    });}
    
    * * *
    
    > **闭包函数**:有权访问另一个函数作用域内变量的函数都是闭包 我们知道，js的每个函数都是一个个小黑屋，它可以获取外界信息，但是外界却无法直接看到里面的内容。将变量 n 放进小黑屋里，除了 inc 函数之外，没有其他办法能接触到变量 n，而且在函数 a 外定义同名的变量 n 也是互不影响的，这就是所谓的增强“封装性”。
    
    简单说,在一个函数的包里,实现了另一个函数,之前在一个python里面也是看见了这种闭包的写法 这里还的调用栈中出现了 `process` 和 `verify` Process 函数的实现就比较简单
    
        this.process = function (trs, sender, cb) {    setImmediate(cb, null, trs);}    
    
    在上面的进行的是立即执行回调,可能是便于重用 verify 函数 进行交易的 recipient 的地址和法检测,并且判断,转账的金额小于 0
    
        this.verify = function (trs, sender, cb) {    var isAddress = /^[0-9]+[L|l]$/g;    if (!isAddress.test(trs.recipientId.toLowerCase())) {        return cb("Invalid recipient");    }    if (trs.amount <= 0) {        return cb("Invalid transaction amount");    }    cb(null, trs); // 完成之后回调,那个闭包的 done}
    
    * * *
    
    这里贴出 Done 函数, 分析交易校验完成之后的操作
    
        function done(err) {    if (err) {        return cb(err);    }    // 这里是完成后的过程,在最后...    // 这里执行的是把这个交易加入当前的区块    privated.addUnconfirmedTransaction(transaction, sender, function (err) {        if (err) {            return cb(err);        }        // 在总线上发送一个未确认的交易的消息        library.bus.message('unconfirmedTransaction', transaction, broadcast);        cb();    });}
    

    privated.addUnconfirmedTransaction = function (transaction, sender, cb) {    self.applyUnconfirmed(transaction, sender, function (err) {        if (err) {            self.addDoubleSpending(transaction);            return setImmediate(cb, err);        }        privated.unconfirmedTransactions.push(transaction);    // 这里在当前的区块里把交易压入,即打包        var index = privated.unconfirmedTransactions.length - 1;         privated.unconfirmedTransactionsIdIndex[transaction.id] = index; // 添加交易索引        setImmediate(cb); //立即执行后面的回调    });}

[](https://www.diglp.xyz/2018/05/16/BC_EbookCoin_0x6/#%E5%90%8E "后")后
---------------------------------------------------------------------

在这个部分,是真正的实现了一次链上的交易, 整个过程从交易的产生,到处理,校验,和打包广播.基本上是实现了一个交易的生命周期.整个过程是十分精彩的. 从客户端调用API向节点发送 交易信息,到交易被创建,检验,打包,发送.这几个过程.都很好的展现了出来