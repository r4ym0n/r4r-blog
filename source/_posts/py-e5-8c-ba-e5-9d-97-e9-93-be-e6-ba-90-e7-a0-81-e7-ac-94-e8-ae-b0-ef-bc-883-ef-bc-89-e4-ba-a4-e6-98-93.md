---
title: Py区块链源码笔记 （3）交易
url: 564.html
id: 564
date: 2018-03-12 00:00:00
tags:
---

怎么学习区块链知识呢? 各种的资料看的头大，还是晕晕乎乎。所以那不如自己实现一个吧？？

说是自己实现，实际上想先对源码进行解读。这里给出的源码，是一个基于Python实现的一个功能较为健全的区块链。下面给出项目地址

> 项目地址

[](https://www.diglp.xyz/2018/03/12/BC_pyMyBC_3/#%E4%BA%A4%E6%98%93-transaction "交易(transaction)")交易(transaction)
-----------------------------------------------------------------------------------------------------------------

交易，是区块链里面实现价值传递的核心体现。如果在BTC的体系下，交易的本身是一个叫做`UTXO (Unspent Transaction Output)`的模型

这里分析两篇好的文章，了解一下UTXO的模型

> 其实并没有什么比特币，只有 UTXO
> 
> 比特币UTXO的原理

实际上我们的每笔币的转移，都是一条由我们的私钥进行签名的一条数据

> 区块数据

由上面的数据我们可见，去数据是由输入和输出两部分。

* * *

### [](https://www.diglp.xyz/2018/03/12/BC_pyMyBC_3/#%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90 "源码分析")源码分析

**Web后端代码**

这里贴出的是后端代码，由函数名可以很容易的知道，这个是发起一个新的交易。是用户使用post提交的请求。之后创建新的交易。

    @app.route('/transactions/new', methods=['POST'])def new_transaction():    # 取得用户提交的json    values = request.get_json()    # 检查用户的Post的json是否合法    required = ['sender', 'recipient', 'amount']    if not all(k in values for k in required):        # 是否包含这里的所有的键        return 'Missing values', 400    # 这里开始创建一个交易    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])    response = {'message': f'Transaction will be added to Block {index}'}    return jsonify(response), 201

* * *

**新建交易**

这个函数比较重要，意在添加一笔交易在当前的区块中，当区块中存在了多笔交易，或者是BTC网络中的 十分钟一次的HeartBeat，之后，我们的节点开始一次Mine，当计算出了Proof的值之后，就把这个区块入链。这样我们的交易记录就永远的存在于区块值之中了。这样的一次次的极记录也就构成了我们的UTXO的模型。当一个用户开始转账时候，由节点负责这个交易的合法性校验。

**UTXO中，你的输出是不能大于你的输入的，你输出多少，你就减去多少。**

    def new_transaction(self, sender, recipient, amount):    """    :param sender: Address of the Sender    :param recipient: Address of the Recipient    :param amount: Amount    :return: The index of the Block that will hold this transaction    """    self.current_transactions.append({        # 这里在我们当前的区块数据中追加我们的交易        'sender': sender,        'recipient': recipient,        'amount': amount,    })    return self.last_block['index'] + 1        # 区块数据索引加一

这里的交易记录是十分简单的，可以看到有三个键 **`sender, recipient, amount`**.发送者，接收者，和总额。

​  
last\_block = blockchain.last\_block  
proof = blockchain.proof\_of\_work(last_block)

    # 这里是我们的mine的区块奖励，这个recipient也就是我们的CoinBase账户，他将得到我们的区块奖励blockchain.new_transaction(    sender="0",    recipient=node_identifier,    amount=1,)# 这里很重要，整个区块的内容进行一个Hash。保持数据不变previous_hash = blockchain.hash(last_block)block = blockchain.new_block(proof, previous_hash)

上面的代码的功能可以去看前面的**Mine**部分，实际上这里没有进行交易的合法性检验，不过代码简易~

### [](https://www.diglp.xyz/2018/03/12/BC_pyMyBC_3/#%E5%B0%8F%E7%BB%93 "小结")小结

代码简易，所以还是忽略了许许多多的细节问题，比如交易的合法性，和实际的账户等等。不过我们可以看见，区块的本质实际上就是这些TX的不断Append而构成的。

* * *

**BTC 区块数据**  
这里贴出的是随便找到的比特币的第123个区块的数据

> 区块内容

早期区块里的内容是十分的少的，所以我们可以很清楚的看见区块中的数据，现在的区块的大小在2M（比特币扩容问题，后面会再讲）。  
每条交易的数据大小大概在200B左右，所以每个区块大概有10000个这样的交易存着。

下面尝试着看看内容~

    {        // 块hash  "hash": "00000000a3bbe4fd1da16a29dbdaba01cc35d6fc74ee17f794cf3aab94f7aaa0",  "ver": 1,      // 前块hash  "prev_block": "000000008d98d186565441057e87cc03251b95b9042956c9fb11325e2d4a847a",  "mrkl_root": "b944ef8c77f9b5f4a4276880f17256988bba4d0125abc54391548061a688ae09",  "time": 1231677823,    // unix的时间戳转换为北京时间是 2009/1/11 20:43:43  "bits": 486604799,      "nonce": 4094077204,    // nonce 类比于我们的Proof  "n_tx": 1,            // 一个 tx  "size": 216,            // 总大小 216B7  "tx": [    {      "hash": "b944ef8c77f9b5f4a4276880f17256988bba4d0125abc54391548061a688ae09",      "ver": 1,      "vin_sz": 1,      "vout_sz": 1,      "lock_time": 0,      "size": 135,      "in": [        {          "prev_out": {            "hash": "0000000000000000000000000000000000000000000000000000000000000000",            "n": 4294967295          },          "coinbase": "04ffff001d02df00"        }      ],      "out": [        {          "value": "50.00000000",          "scriptPubKey": "04b715afd59b31be928e073e375a6196d654a78d9aa709789665dd4aecf1b85ebc850ffb90a1c04f18565afe0be4a042ff6629c398f674a5c632b017d793dc8e04 OP_CHECKSIG"        }      ],      "nid": "3445a53aefbc184c37229153c5b619759f9836458d69ffa1874b79614e4f6c7b"    }  ],  "mrkl_tree": [    "b944ef8c77f9b5f4a4276880f17256988bba4d0125abc54391548061a688ae09"  ],  "next_block": "00000000ceae2b1cb578f066bd08c672fe87814880671c205febb2d624184f21"}

[](https://www.diglp.xyz/2018/03/12/BC_pyMyBC_3/#%E5%90%8E%E9%9D%A2%E7%9A%84%E8%AF%9D "后面的话")后面的话
-------------------------------------------------------------------------------------------------

到此，自己一共写了这是第三篇文章了。也是已经慢慢的读完了，并理解了这个简易的 区块链的demo。实话，真的收益匪浅。

虽然代码简易，一些具体的实现细节并不存在，不过也正是因为这个，才更容易理解。所以自己还要进一步的理解区块链的源码，下一个目标，就是BTC源码把！！！！加油年轻人。