---
title: Py区块链源码笔记 （1）挖矿
url: 568.html
id: 568
date: 2018-03-09 00:00:00
tags:
---

怎么学习区块链知识呢? 各种的资料看的头大，还是晕晕乎乎。所以那不如自己实现一个吧？？

说是自己实现，实际上想先对源码进行解读。这里给出的源码，是一个基于Python实现的一个功能较为健全的区块链。下面给出项目地址

> 项目地址

[](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#%E6%8C%96%E7%9F%BF%EF%BC%88mine%EF%BC%89 "挖矿（mine）")挖矿（mine）
-------------------------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%8C%96%E7%9F%BF "什么是挖矿")什么是挖矿

当然进入币圈或者链圈的人当然对挖矿这个词不会陌生。区块链的核心问题就是解决了拜占庭将军问题，实现了全网的可信。在比特币网络里面，每十分钟产生的数据成为一个block。这个块被所有的矿工一起进行计算。其中有一个nonce的值。一旦有一个矿工挖到了这个特定值。那么就挖到了这个block。可以得到coinbase的奖励。

> 拜占庭将军问题

### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#%E6%BA%90%E7%A0%81%E7%9A%84%E5%AE%9E%E7%8E%B0 "源码的实现")源码的实现

#### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#Web%E7%AB%AF "Web端")Web端

这里是使用了python的Flask模块作为一个web的服务端。

    app = Flask(__name__)# 这里的@说明是函数修饰符，后面有说明@app.route('/mine', methods=['GET'])def mine():    ...    return jsonify(response), 200app.run(host='0.0.0.0', port=5000)

这里就可以实现后端的对前端的GET请求的响应，执行挖矿操作后，返回Json的信息。

* * *

#### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#Python%E7%9A%84%E5%87%BD%E6%95%B0%E4%BF%AE%E9%A5%B0%E7%AC%A6 "Python的函数修饰符")Python的函数修饰符

这里的 `@app.route('/mine', methods=['GET'])`是一个对函数的修饰符，和solidity里面的modifier差不多。其意在执行完了修饰符的函数之后，才会继续执行下面修释的函数

    def test(f):      print "before ..."      f()  # 这里指代的就是所修饰函数    print "after ..."  @test  def func():      print "func was called"  # 直接运行，输出结果：before ...  func was called  after ...  # 所以这里可以看出上述代码等价于test(func)    # func 是个pointer

#### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#Miner "Miner")Miner

这里是miner的代码，这里实现了一个简单的挖矿过程，虽然和实际的挖矿过程还是有些差距，不过也正是简单易读，所以我们好理解。

#### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#miner%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86 "miner的请求处理")miner的请求处理

    @app.route('/mine', methods=['GET'])def mine():    # 这里使用PoWPoW共识机制。    # 这里的返回了一个ARRAY的tail其定义是 chain = []    last_block = blockchain.last_block                # 这里使用PoW，代码后面展开    proof = blockchain.proof_of_work(last_block)    

​  
​ # We must receive a reward for finding the proof.  
​ # The sender is “0” to signify that this node has mined a new coin.  
​ # 翻译过来，这个就是区块奖励，也就是CoinBase,  
​ # 作为一个区块的第一笔交易，所以发送者是零地址。  
​  
blockchain.new_transaction(  
sender=”0”,  
recipient=node_identifier,  
amount=1,  
)

    # 把这个    区块加到链尾，实现了到主链的融合previous_hash = blockchain.hash(last_block)block = blockchain.new_block(proof, previous_hash)# 这里把新的区块数据反馈到前端response = {    'message': "New Block Forged",    'index': block['index'],    'transactions': block['transactions'],    'proof': block['proof'],    'previous_hash': block['previous_hash'],}return jsonify(response), 200

#### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#PoW%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81 "PoW的实现代码")PoW的实现代码

    def proof_of_work(self, last_block):    """    一段简单的工作量证明的算法:    """    last_proof = last_block['proof']    last_hash = self.hash(last_block)    proof = 0    while self.valid_proof(last_proof, proof, last_hash) is False:        proof += 1    return proof

​  
上述代码的实际上的工作原理： 先使得 proof的值为零，之后慢慢的递增。直到找到

**找到一个数字`P`，使得`Hash(P,P')`的哈希值是前面包含了4个0的，这里的`P’是上一个区块的P值**

实际上我们的挖矿的过程和这个差不多，这个P在实际上是一个nonce(number once)的值。这里一样的进行了大量的简化

* * *

**valid_proof**

    # point ： 这里是py的特性F-strings，分别计算花括号的值，并且进行拼接guess = f'{last_proof}{proof}{last_hash}'.encode()    guess_hash = hashlib.sha256(guess).hexdigest()print(guess_hash)return guess_hash[:4] == "0000"

这里就是对我们的Proof的值的**合法性判断**，

*   先把上一个的proof，本次的proof，和上一个区块的hash进行一个拼接(F-string)之后进行编码。
*   对其进行sha256的hash。得到hash值。
*   之后判断该hash的值是否是以四个0结尾
*   返回是否找到合法的proof值，如果找到，返回True
*   返回上层函数PoW的过程结束

这里是部分的打印结果：挖出第一个块的打印（第0块，是创世区块，设置的proof是100）

    f49bd479bb433aa37bcb01b36cc6e4f3f8881ae4cdfeecfc3fc84a2a69a29951d71f73e0d52ad34fbc8848d85890a0951773fafc6c7214fe94794cc9c2dca904312cc1a85835727e29b7d85ae0a781a35ab57376ae56e29f8e6a70e1f76eb1390000b1964e2a279761ab62cf0d52272f540867aee83bb22ffc6eb2e9bf63f3b1100 16623 ac018635f614a44ab203ef49fcb7887b36de048fd5d5a286a06c9b32666bd618

我们可以看见，最后一行，我们的last_proof是100，这次我们尝试了 16623 次，得到了proof，使得他们的hash是以四个0开头的  
**0000b1964e2a279761ab62cf0d52272f540867aee83bb22ffc6eb2e9bf63f3b1**

同样的下面是第二块

    42d9c8f25d3b438157c2e4cd06fff288600eb78ef9536aeec60dea13c46fb41d00006b342191c828ecfeeb21ad3cfe3320ded31d4a0bc64fcf5c103c5a8806cb16623 187207 58f183fc794d087ccce036e25ca039099af9738d7cdf5c23564def4254eb1281

### [](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#%E5%B0%8F%E7%BB%93 "小结")小结

可见这里的代码很简单的实现了一个PoW的模型，proof可以类比于我们寻找的 nonce 的这个值，实际上，关于BTC的难度系数的动态调整，就是和这得到的hash的零的个数，和后面数值的有效的大小的要求而确定的。

这里我们在BTC的区块链浏览器可以直接看到我们的区块数据。

> BTC里面的随便一个区块

*   高度 512,589  
    确认数 2  
    大小 1,096,630 Bytes  
    **Nonce** **0x185d9c75**  
    时间 2018-03-08 22:05:10  
    块哈希
*   000000000000000000474697c175dadd12b11f9736c10e2a632aa52d7a555a0f  
    前一个块
*   00000000000000000000021c043e439b5f4b632389b0062306bf2d4e0b657c7c  
    后一个块
*   000000000000000000434de347737700f50cacde89f956e08ed4a39dddd23bf0

可以看到，其实，差不多2333.

[](https://www.diglp.xyz/2018/03/09/BC_pyMyBC_1/#%E5%90%8E%E9%9D%A2%E7%9A%84%E8%AF%9D "后面的话")后面的话
-------------------------------------------------------------------------------------------------

打算潜心学习，不能浮躁，慢慢的学习这些底层的原理和实现，后面应该会有关于交易（加密签名），组网（P2P）网络的内容