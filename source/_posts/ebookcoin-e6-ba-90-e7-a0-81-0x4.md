---
title: EbookCoin 源码 0x4
url: 534.html
id: 534
date: 2018-05-13 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/13/BC_EbookCoin_0x4/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

最近的拖延症严重到极致，我都是有点佩服自己了，应该一周前的是现在还没有解决。

[](https://www.diglp.xyz/2018/05/13/BC_EbookCoin_0x4/#%E5%9C%B0%E5%9D%80 "地址")地址
--------------------------------------------------------------------------------

前：前面的部分，从篇幅都可以看出是个极简单的功能，一句话概括就是通过一个初始密码，生成一个密钥对，然后保存在account结构中。当然，我们的一个账户，将会实现一个 全面的api，用来显示账户的信息，像是余额，地址，私钥。。。

* * *

### [](https://www.diglp.xyz/2018/05/13/BC_EbookCoin_0x4/#%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB "文件包含")文件包含

*   ./modules/accounts.js
*   ./logic/account.js
*   ./modules/contracts.js

accounts.js 实现了账户功能 account.js 实现了账户属性 contracts.js 实现了联系人的功能

### [](https://www.diglp.xyz/2018/05/13/BC_EbookCoin_0x4/#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0 "代码实现")代码实现

*   API 已知，ebook是基于HTTP实现的区块链项目，所以，其功能是由其api实现的。所以这里从其实现的功能入手。这个称之为中间件，拓展路由的。
    

            router.map(shared, {            "post /open": "open",            "get /getBalance": "getBalance",            "get /getPublicKey": "getPublickey",            "post /generatePublicKey": "generatePublickey",            "get /delegates": "getDelegates",            "get /delegates/fee": "getDelegatesFee",            "put /delegates": "addDelegates",            "get /username/get": "getUsername",            "get /username/fee": "getUsernameFee",            "put /username": "addUsername",            "get /": "getAccount"        });通过些拓展的API我们可以发现，一个账户的具体要实现的功能.打开账户,余额,公钥,等等.在后面的代码里,实现了中间件功能的添加,可以实现了在EXPRESS的 框架里的功能拓展.        library.network.app.use('/api/accounts', router);如果在http的请求层次,我们和得到如以下的映射        get /api/getBalance -> shared.getBalance        post /api/open -> shared.open

*   账户地址 在比特币系统中地址实际上有多种, 1 开头的 账户地址,以及 3 开头的测试地址.所以根据地址的形式可以实现不同种类的地址分类.地址的生成过程在前面的地方有涉及,这里再贴出代码.
    
        Accounts.prototype.generateAddressByPublicKey = function (publicKey) {    var publicKeyHash = crypto.createHash('sha256').update(publicKey, 'hex').digest();    var temp = new Buffer(8);    for (var i = 0; i < 8; i++) {        temp[i] = publicKeyHash[7 - i];    }    var address = bignum.fromBuffer(temp).toString() + 'L';    // 注意这里是在后面添加了L    if (!address) {        throw Error("wrong publicKey " + publicKey);    }    return address;};
    

    在亿书的项目里面,使用了地址后面的后缀对不同的地址进行区分.上面的 **L** 是用于生成常规地址的.还有一类的地址,是块地址(这个好像在传统体系里面没有).其使用的是结尾的**C作为标识**    privated.getAddressByPublicKey = function (publicKey) {        var publicKeyHash = crypto.createHash('sha256').update(publicKey, 'hex').digest();        var temp = new Buffer(8);        for (var i = 0; i < 8; i++) {            temp[i] = publicKeyHash[7 - i];        }        var address = bignum.fromBuffer(temp).toString() + "C";        return address;    }    generatorId: privated.getAddressByPublicKey(raw.b_generatorPublicKey),

*   别名地址 在亿书的项目中很好的加入了别名地址这个东西.比如我们在BTC体系下的转账需要输入很长的账户地址,往往,这个地址是十分难以记忆的,座椅在亿书的体系中引入了别名系统(ALIAS),就像支付宝的用户名一样,我们不需要一大堆冗长的字符串,只需要用户名一样的东西,就可以进行转账.具体的实现原理是在数据库中同时保存地址和别名. 由于别名通常是用于转账的时候,所以别名的引用代码,在trsnaction的模块里
    
        shared.addTransactions = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {        type: "object",        properties: {            secret: {                ... // 用于检验合法性            }        },        required: ["secret", "amount", "recipientId"]    }, function (err) {        if (err) {            return cb(err[0].message);        }        // 这里是对密码的检验        var hash = crypto.createHash('sha256').update(body.secret, 'utf8').digest();            var keypair = ed.MakeKeypair(hash);        if (body.publicKey) {            if (keypair.publicKey.toString('hex') != body.publicKey) {                return cb("Invalid passphrase");            }        }        var query = {};        // 神奇的正则表达式        var isAddress = /^[0-9]+[L|l]$/g;        // 这里里的IF 就是显得很重要的了,实际上这里是对其进行的一次判断,        // 如果是合法的地址,那么就是使用地址查询, 如果不是地址就是使用别名查询        if (isAddress.test(body.recipientId)) {            query.address = body.recipientId;        } else {            query.username = body.recipientId;        }        library.balancesSequence.add(function (cb) {            modules.accounts.getAccount(query, function (err, recipient) {                ... // 合法性判断                var recipientId = recipient ? recipient.address : body.recipientId;                var recipientUsername = recipient ? recipient.username : null;                    ... // 一系列的正则判断                            try {                                // 创建交易结构                                var transaction = library.logic.transaction.create({                                    type: TransactionTypes.SEND,                                    amount: body.amount,                                    sender: account,                                    recipientId: recipientId,                                    recipientUsername: recipientUsername,                                    keypair: keypair,                                    requester: keypair,                                    secondKeypair: secondKeypair                                });                            } catch (e) {                                return cb(e.toString());
    

    具体部分注释中标出. 这里的 recipientID 可以是地址,也可以是用户名.

* * *

*   别名注册 别名在地址生成的本身是不需要的,需要的只是 secret ,所以,别名的注册是另一个功能.
    
        shared.addUsername = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {        ... // 格式检测            }        },        required: ['secret', 'username']    }, function (err) {        if (err) {            return cb(err[0].message);        }        var hash = crypto.createHash('sha256').update(body.secret, 'utf8').digest();        var keypair = ed.MakeKeypair(hash);        // 打开钱包的密码判断        if (body.publicKey) {            if (keypair.publicKey.toString('hex') != body.publicKey) {                return cb("Invalid passphrase");            }        }        library.balancesSequence.add(function (cb) {            // 判断是否多重签名                    if (body.multisigAccountPublicKey && body.multisigAccountPublicKey != keypair.publicKey.toString('hex')) {                    modules.accounts.getAccount({publicKey: keypair.publicKey}, function (err, requester) {                        ... // 合法性                         var secondKeypair = null;                        if (requester.secondSignature) {                            var secondHash = crypto.createHash('sha256').update(body.secondSecret, 'utf8').digest();                            secondKeypair = ed.MakeKeypair(secondHash);                        }                        try {                            var transaction = library.logic.transaction.create({                                type: TransactionTypes.USERNAME,                                username: body.username,                                sender: account,                                keypair: keypair,                                secondKeypair: secondKeypair,                                requester: keypair                            });                        } catch (e) {                            return cb(e.toString());                        }                        modules.transactions.receiveTransactions([transaction], cb);                    });                });            } else {                self.getAccount({publicKey: keypair.publicKey.toString('hex')}, function (err, account) {                    if (err) {                    ... // 合法性检测                    var secondKeypair = null;                    if (account.secondSignature) {                        var secondHash = crypto.createHash('sha256').update(body.secondSecret, 'utf8').digest();                        secondKeypair = ed.MakeKeypair(secondHash);                    }                    try {                        var transaction = library.logic.transaction.create({                            type: TransactionTypes.USERNAME,                            username: body.username,                            sender: account,                            keypair: keypair,                            secondKeypair: secondKeypair                        });                    } catch (e) {                        return cb(e.toString());                    }                    ... //
    

    在这部分的代码中,实际上的主要函数,是在上面创建的一个交易,这个交易是写在链上的,`type: TransactionTypes.USERNAME,`,这样就可以建立一个别名,实现地址的映射.

[](https://www.diglp.xyz/2018/05/13/BC_EbookCoin_0x4/#%E6%80%BB%E7%BB%93 "总结")总结
--------------------------------------------------------------------------------

看完这部份的功能, 其实发现了一个共性我们可以很简单的概述我们上面所有的 操作的功能.就是 **交易** ,**没错一切就是交易.** 这里的交易实际上是一种泛化,其实是指的在区块链上所有的记录,也就是上链.一旦数据上链,那么就是产生一个不可改变的记录值. 这里 的api 实际上是对特殊交易种类的不同封装.