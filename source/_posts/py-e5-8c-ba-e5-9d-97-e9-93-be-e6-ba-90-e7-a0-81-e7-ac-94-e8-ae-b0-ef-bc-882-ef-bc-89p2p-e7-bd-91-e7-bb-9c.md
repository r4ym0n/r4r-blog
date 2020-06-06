---
title: Py区块链源码笔记 （2）P2P网络
url: 566.html
id: 566
date: 2018-03-11 00:00:00
tags:
---

怎么学习区块链知识呢? 各种的资料看的头大，还是晕晕乎乎。所以那不如自己实现一个吧？？

说是自己实现，实际上想先对源码进行解读。这里给出的源码，是一个基于Python实现的一个功能较为健全的区块链。下面给出项目地址

> 项目地址

[](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#P2P%E7%BD%91%E7%BB%9C "P2P网络")P2P网络
------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8P2P "为什么使用P2P")为什么使用P2P

我们日常里用的磁链，迅雷之类的东西都是P2P技术实现的，所以P2P这个名词听起来不会太陌生。

其实在这里是对P2P技术的一种泛化，指的是这一类的去中心化的网络。所以P2P在区块链里面就是显得十分重要了。

一个P2P网络的功能基本大致可以有一下三个：

*   路由拓展 可以在自己的路由表里，发现或者初始化添加其他路由节点  
    节点路由 请求向其他节点路由  
    节点保存 把已知节点写入我们的路由表

* * *

### [](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90 "源码分析")源码分析

这里是其项目里的测试代码

> 这个测试模块可以参考python的unittest这个module。

这里的测试类是用于节点注册测试  
​  
class TestRegisterNodes(BlockchainTestCase):

    def test_valid_nodes(self):    blockchain = Blockchain()    blockchain.register_node('http://192.168.0.1:5000')        # 可见该函数是实现节点注册    self.assertIn('192.168.0.1:5000', blockchain.nodes)def test_malformed_nodes(self):        # 畸形节点测试    blockchain = Blockchain()    blockchain.register_node('http//192.168.0.1:5000')        # 应该自行排除畸形节点    self.assertNotIn('192.168.0.1:5000', blockchain.nodes)def test_idempotency(self):            # 对等性测试    blockchain = Blockchain()    blockchain.register_node('http://192.168.0.1:5000')    blockchain.register_node('http://192.168.0.1:5000')    assert len(blockchain.nodes) == 1

#### [](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E8%8A%82%E7%82%B9%E6%B3%A8%E5%86%8C "节点注册")节点注册

核心函数`blockchain.register_node()`

    def register_node(self, address):    """    Add a new node to the list of nodes    :param address: Address of node. Eg. 'http://192.168.0.5:5000'    """    parsed_url = urlparse(address)    if parsed_url.netloc:        self.nodes.add(parsed_url.netloc)    elif parsed_url.path:        # Accepts an URL without scheme like '192.168.0.5:5000'.        self.nodes.add(parsed_url.path)    else:        raise ValueError('Invalid URL')

**urlparse()** 的Doc

    >>> from urllib.parse import urlparse>>> o = urlparse('http://www.cwi.nl:80/%7Eguido/Python.html')>>> o   ParseResult(scheme='http', netloc='www.cwi.nl:80', path='/%7Eguido/Python.html',            params='', query='', fragment='')>>> o.scheme'http'>>> o.port80>>> o.geturl()'http://www.cwi.nl:80/%7Eguido/Python.html'

可见这个方法可以很容易的解析url。所以在工程代码里。是对域名，和路径的提取。如果不是合法值，就直接Raise一个异常

**添加node**  
​  
在构造函数中有`self.nodes = set()`所以可见这个是一个集合。

    self.nodes.add(parsed_url.path)        # 这行代码就是添加节点的地址

这里在这个项目里有些过度简化。只是实现了节点添加的功能。

    # Flask 的route修饰符@app.route('/nodes/register', methods=['POST'])def register_nodes():values = request.get_json()nodes = values.get('nodes')if nodes is None:    return "Error: Please supply a valid list of nodes", 400for node in nodes:    blockchain.register_node(node)response = {    'message': 'New nodes have been added',    'total_nodes': list(blockchain.nodes),}return jsonify(response), 201

这里代码可见，把node的list以json的形式直接post上去。解析之后直接append。

* * *

### [](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E8%8A%82%E7%82%B9%E8%A7%A3%E6%9E%90-resolve "节点解析(resolve)")节点解析(resolve)

这段代码是请求的节点更新的代码，因为一个去中心的网络，需要实施的维护个更新自己的数据。也就是区块链中的账本。所以这段代码算是实现了简单的**DAO**(Distributed Autonomous Organization),翻译过来是**分布式自治组织**。

节点之间进行互相的请求，以确保自己的区块高度是目前的最高区块，如果当前区块不是最高，则在其他的路由节点获取当前的区块信息。

    @app.route('/nodes/resolve', methods=['GET'])    # 请求响应def consensus():replaced = blockchain.resolve_conflicts()        # 区块冲突解决if replaced:                response = {        'message': 'Our chain was replaced',    # 如果其他节点区块高度大于我们，则更新我们的节点        'new_chain': blockchain.chain    }else:    response = {        'message': 'Our chain is authoritative(当局的)',    # 如果我们是最新，就保持        'chain': blockchain.chain    }return jsonify(response), 200

下面这段就是我们的区块更新的代码，同等简单的方式，更新获取相邻节点的区块高度。进行append判断。这里实现简单的共识算法，使用使用最高网络区块直接替换来解决区块的冲突。

实际上，这里的共识的实现是十分重要和复杂的一个要点。因为在全部网络上，我们不可避免的存在着**拜占庭节点**（byzantine）来作恶。如果我们使用直接最高区块的简单的共识算法。那么网络上的的拜占庭节点，可随随便便的编造区块数据，并且使用PoS把他的伪造交易数据添加到链上。大家发现他是最高的，所以纷纷认为他是对的。这样对网络是毁灭性的打击

    def resolve_conflicts(self):    """    :return: True if our chain was replaced, False if not    """    neighbours = self.nodes    new_chain = None    # 找到我们自己的区块高度    max_length = len(self.chain)    # 获取并且验证我们在网络上得到的区块    for node in neighbours:        response = requests.get(f'http://{node}/chain')        # 获取网络节点的完整链，        if response.status_code == 200:            length = response.json()['length']        # 得到区块高度            chain = response.json()['chain']        # 和当前的完整链            # 检查是否最长链，并且检验链的有效性            if length > max_length and self.valid_chain(chain):        # 最长链，并进行合法性检验                max_length = length                    new_chain = chain    #  如果有效且最长，我们替换我们的本地链    if new_chain:        self.chain = new_chain        return True    return False

可见这里就实现了一个简易的区块共识协议，我们可以称之为最长就是最好 2333

* * *

**合法区块检测**

用于判断获取的链的合法性，简单的说就是进行；链上区块遍历，判断其hash是否成链，即 `block['previous_hash'] != self.hash(last_block)` ,并且对区块的 proof ，last\_proof，previous\_hash ，进行一次hash计算，判断是否是四个0 开头的

> PoS 的过程可以看上一篇的内容 挖矿/)`

    def valid_chain(self, chain):    """    :param chain: A blockchain    :return: True if valid, False if not    """    last_block = chain[0]    current_index = 1    while current_index < len(chain):        # 遍历链上区块        block = chain[current_index]        print(f'{last_block}')        print(f'{block}')        print("\n-----------\n")        # 检查hash是否成链        if block['previous_hash'] != self.hash(last_block):            return False        # 检查proof（nonce）是否合法        if not self.valid_proof(last_block['proof'], block['proof'], last_block['previous_hash']):            return False        last_block = block        current_index += 1    return True

所以我们可以看到，区块链之所以安全，其内核基本就是靠这里所描述的功能保护的，虽然这里的代码是十分简易，不过大体上也是描述出了灵魂所在。可以使用hssh链，进行整个链的合法性校验，所以，使得链上的数据修改基本成为**不可能！！！**

* * *

### [](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E5%B0%8F%E7%BB%93 "小结")小结

这里的源码实现的P2P网络是比较简单，这里只是实现了节点的添加，并没有实现节点路由和拓展，及路由表的查询。所以后面打算自己可以实现一个

[](https://www.diglp.xyz/2018/03/11/BC_pyMyBC_2/#%E5%90%8E%E8%AE%B0 "后记")后记
---------------------------------------------------------------------------

这篇可能比较水，因为源码本身实在太过简化，后面考虑自己添加部分功能，可以及时PR÷