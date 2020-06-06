---
title: EbookCoin 源码 0x5
url: 532.html
id: 532
date: 2018-05-14 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

[](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E7%AD%BE%E5%90%8D "签名")签名
--------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E7%AE%80%E4%BB%8B "简介")简介

现实中的签名是有法律效益的，每个人的签名痕迹都是独一无二的，所以签名是用来证明这个东西是否和你相关的最好的东西。在现在的数字时代，签名一样是必不可少的而且是是数字化的。…又啰嗦了好多。 所以在区块链上，的数字资产的证明，一样是离不开数字签名的，当然在拓展需求的情况下还存在着多重签名。

* * *

数字签名技术，来自于 PKI 体系下的东西。我把某一数据的摘要值，使用我的私钥进行加密，接收方拥有我的公钥，即可以进行验证，对其加密摘要进行解密，之后与原文的同类型摘要进行对比，对比一致，就可以证明，数据没有被修改，且数据源是正确的发送方。这样就像等于对数据进行了签名。其具体的实现，在网上找了链接。

> 数字签名原理及其应用-CSDN 数字签名、数字证书关系-知乎

* * *

前面我们讲了，在区块链的世界，一切的操作或多或少都离不开交易，那么如果我要转账操作，那么如何说明这个操作是合法的呢？这里就是数字签名在区块链里的作用，**进行交易广播的签名**， 这样大家都知道和认为，这个交易是合法的。

### [](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB "文件包含")文件包含

实现签名操作的模块如下

*   ./modules/signatures.js
*   ./modules/multisignatures.js

signatures.js 实现了对交易进行签名的功能 multisignatures.js 如其名曰，实现了消息的多重签名的功能。

### [](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0 "代码实现")代码实现

*   API 实现外部功能，就少不了API接口，一样的是对EXPRESS框架的拓展
    
        router.map(shared, {    "get /fee": "getFee",    "put /": "addSignature"});library.network.app.use('/api/signatures', router);
    
    得到的路由接口的对应关系是
    
        get /api/signatures/fee    ->    getFeeput /api/signatures/    ->    addSignature
    

*   添加签名
    
        签名的添加在文中实现的代码如下。不知为何这个功能在整个项目中是只是留出了接口，而且和原本想想的作用不是十分一致。shared.addSignature = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {        type: "object",            ... // 合法性检验        },        required: ["secret", "secondSecret"]    }, function (err) {        if (err) {            return cb(err[0].message);        }            ... // 密码检测部分，签名出现多次        library.balancesSequence.add(function (cb) {            // 多重签名判断            if (body.multisigAccountPublicKey && body.multisigAccountPublicKey != keypair.publicKey.toString('hex')) {                modules.accounts.getAccount({publicKey: body.multisigAccountPublicKey}, function (err, account) {                    ... // 合法性检验                    modules.accounts.getAccount({publicKey: keypair.publicKey}, function (err, requester) {                        if (err) {                            return cb(err.toString());                        }                        ... // 合法性检验                        var secondHash = crypto.createHash('sha256').update(body.secondSecret, 'utf8').digest();                        var secondKeypair = ed.MakeKeypair(secondHash);                        ... //                     });                });            } else {                modules.accounts.getAccount({publicKey: keypair.publicKey.toString('hex')}, function (err, account) {                    if (err) {                        return cb(err.toString());                    }                    ... // 合法性检验                    var secondHash = crypto.createHash('sha256').update(body.secondSecret, 'utf8').digest();                    var secondKeypair = ed.MakeKeypair(secondHash);                    try {                        var transaction = library.logic.transaction.create({                            type: TransactionTypes.SIGNATURE,                            sender: account,                            keypair: keypair,                            secondKeypair: secondKeypair                        });                    } catch (e) {                        return cb(e.toString());                    }                    modules.transactions.receiveTransactions([transaction], cb);                });            }        }, function (err, transaction) {            if (err) {                return cb(err.toString());            }            cb(null, {transaction: transaction[0]});        });    });}
    
    代码的实现，实际上是创建了一个是签名类型的交易。感觉在此处的功能可能不是那么完整。这里又secondSecret，像是注册了第二密码。实际上在书中所描述的，是亿书的支付密码功能。
    

*   多重签名 简单的理解来说，就好比一个文件需要几个人一起签名了之后才会生效，这个就是所谓的多重签名。就像是核弹的发射按钮一样，需要多个授权码才可以进行操作。 完成应用的场景，我们可以设想成我们常见的担保交易。AB之间的交易，需要C进行担保，所以先A签名，B签名，确认之后C进行签名，使得交易有效，即可实现功能 所以，多重签名的应用场景，在于 电子商务，财产分割，资金管理。
    
    * * *
    
        router.map(shared, {    "get /pending": "pending", // Get pending transactions    "post /sign": "sign", // Sign transaction    "put /": "addMultisignature", // Add multisignature    "get /accounts": "getAccounts"});library.network.app.use('/api/multisignatures', router);
    
    多重签名其对应着自己的一套独立的API。，一样的格式，实现组建功能的拓展
    
        required: ['min', 'lifetime', 'keysgroup', 'secret']
    
    这里是多重签名所需要的参数。
    

[](https://www.diglp.xyz/2018/05/14/BC_EbookCoin_0x5/#%E5%90%8E "后")后
---------------------------------------------------------------------

实际上，这一部分的源码，感觉也只是从概念上再次理解了签名的实现。不过，这里的源码实现的方式可能和心中想的不太一样， 个人认为，签名是一个交易所必须的部分，因为这样才能证明交易的发起者，是正确的账户所有者。在文章所示的代码中，只是进行账户认证后，生成了secondKeypair。并没有实际上的使用其 prikey 对数据进行加密的部分 weird。可能具体细节是在交易里面实现的。