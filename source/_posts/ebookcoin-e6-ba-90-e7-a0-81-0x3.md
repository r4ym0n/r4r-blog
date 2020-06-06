---
title: EbookCoin 源码 0x3
url: 536.html
id: 536
date: 2018-05-12 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

[](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E5%8A%A0%E5%AF%86%E5%92%8C%E9%AA%8C%E8%AF%81 "加密和验证")加密和验证
-----------------------------------------------------------------------------------------------------------------

其实慢慢的发现，如果把区块链给拆解了，那么一个P2P网络的上一层，就是一个 **PKI(Public Key Infrastructure)** ，一个完整的加密体系。使得加密网络的实现。

### [](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E6%A8%A1%E5%9D%97%E5%8C%85%E5%90%AB "模块包含")模块包含

*   ./modules/accounts.js
*   ./logic/account.js

accounts.js 实现了账户功能 account.js 实现了账户属性

### [](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E6%8A%80%E6%9C%AF%E6%A6%82%E5%BF%B5 "技术概念")技术概念

在书中讲解的主要是公钥加密，此处略去。 在比特币的环境下，一个密钥对的公钥可以近似的看作这个账户的地址。私钥是用于生成比特币支付的必须的签名，用来证明对该账户的所有权，即地址中的所有的资金对视取决于其对应的密钥进行的所有权和控制权。 **私钥必须严格保密**，一旦发生丢失，其保护的比特币永远不会找回。 在ebookcoin的项目里，也是直接使用了公钥地址作为了用户的ID。 加密过程，使用了NodeJs的**Crypto**的模块，提供了安全凭证的方式，用于HTTPS/HTTP的连接，有对OoenSSL，HMAC，Hash，加密，解密，签名，验证，都是是进行了封装。 在项目中实现的hash算法是 HASH256.

    var hash = crypto.createHash('sha256').update(data).digest();

上面的语句实际上是实现了一个，先创建 256 的实例，之后对数据进行处理，最后使用digest，获得加密字符串(密文)。之后可以使用Ed25518实现密钥对 **签名验证** 使用Ed25519组件，封装了数字签名算法，签名验证速度很快。7100 Signature/s

    var res = ed.Verify(hash, signatureBuffer || ' ', publicKeyBuffer || ' ');

### [](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0 "代码实现")代码实现

使用过钱包都知道，很多软件，在创建钱包的实话，只是需要使用一次输入的密码就好了，另外可能会加上混淆。实际上，就是这个密码，会生成我们的公钥和私钥。（一般我们注册的时候是不是用户名密码都有来着？？） 在这个项目中，其公钥生成代码如下

    shared.generatePublickey = function (req, cb) {    var body = req.body;    library.scheme.validate(body, {            // 检查结构合法，多次出现        type: "object",        properties: {            secret: {                type: "string",                minLength: 1            }        },        required: ["secret"]    }, function (err) {        if (err) {            return cb(err[0].message);        }        privated.openAccount(body.secret, function (err, account) {        // 通过合法性检测，就开户            var publicKey = null;            if (!err && account) {                publicKey = account.publicKey;            }            cb(err, {                publicKey: publicKey            });        });    });};

上面的代码如何产生一个地址公钥的过程，前面的部分检测合法性，后面的就是使用就对用户的secret进行处理。后面调用 `openAccount`

    privated.openAccount = function (secret, cb) {    var hash = crypto.createHash('sha256').update(secret, 'utf8').digest();    var keypair = ed.MakeKeypair(hash);    self.setAccountAndGet({publicKey: keypair.publicKey.toString('hex')}, cb);    // 以键值的形式传入了参数};

调用的时候，把 secret 参数传入。在这里对密码进行 **sha256** 处理，和前面描述一样，先建立实例，之后使用进行散列。 这里得到 hash 之后，使用 `MakeKeypair` 进行生成密钥对.

    var    ed = require('ed25519');

这里的 `MakeKeypair` 是ed25519自带的模块,不展开了.其返回的值实际上是一个 密钥对.而后使用内部成员.`setAccountAndGet`

    Accounts.prototype.setAccountAndGet = function (data, cb) {    var address = data.address || null;    if (address === null) {        if (data.publicKey) {            address = self.generateAddressByPublicKey(data.publicKey);        } else {            return cb("Missing address or public key");        }    }    if (!address) {        throw cb("Invalid public key");    }    library.logic.account.set(address, data, function (err) {        if (err) {            return cb(err);        }        library.logic.account.get({address: address}, cb);    });};

这里的一句 `var address = data.address || null;`显得很神奇.在网上看到了如下解释

* * *

> *   js 这种写法是什么意思 var a= b || c
> 
> 在js中，这相当于一个赋值语句，如果b的值大于0或为true，那么就把b的值赋给a，否在就把c的值赋给a

* * *

所以,这里就是判断,这个data的address是否是空,不空,就赋值addr,空就幅值 null. 如果是空,那我们就直接进入if,给其分配addr.我们知道,**地址是其公钥通过特定算法得到的**

    Accounts.prototype.generateAddressByPublicKey = function (publicKey) {    var publicKeyHash = crypto.createHash('sha256').update(publicKey, 'hex').digest();    var temp = new Buffer(8);    for (var i = 0; i < 8; i++) {        temp[i] = publicKeyHash[7 - i];    }    var address = bignum.fromBuffer(temp).toString() + 'L';    if (!address) {        throw Error("wrong publicKey " + publicKey);    }    return address;};

使用这段函数,我们从函数名可以看出,是通过 pubkey得到我们拥有的地址。从函数得知，**其地址，是取其hash的前8位倒序。** 返回上级函数，一次判断生成地址是否为空。之后调用logic。保存账户地址参数。

[](https://www.diglp.xyz/2018/05/12/BC_EbookCoin_0x3/#%E5%90%8E "后")后
---------------------------------------------------------------------

这里的代码篇幅很大，本书中只是讲解了部分简单的功能，这里做个推广的总结，我们的地址是来自于公钥的特殊处理。在一系列操作之后，地址，公钥，私钥，都会存入我们的 logic的参数里面，实现了一个钱包的创建 最近，拖延症又是严重的时候。