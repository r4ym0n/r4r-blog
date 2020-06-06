---
title: 智能合约 HelloWorld
url: 590.html
id: 590
date: 2018-02-09 00:00:00
tags:
---

![CPP](http://www.ethdocs.org/en/latest/_images/cpp_35k9.png) ![logo](http://www.ethdocs.org/en/latest/_images/ETHEREUM-ICON_Black.png)

**自己的菜鸟级的起步教程,也算是给自己给自己长记性**

[](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%87%86%E5%A4%87 "准备")准备
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E4%BB%80%E4%B9%88%E6%98%AF%E4%BB%A5%E5%A4%AA%E5%9D%8A "什么是以太坊")什么是以太坊

以太坊白皮书_ZH

[以太坊白皮书_EN](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/)

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E7%8E%AF%E5%A2%83%E4%BB%8B%E7%BB%8D "环境介绍")环境介绍

这里使用了,以下两个开发工具

*   truffle
*   testrpc

Truffle 是一个基于js 开发的 以太坊开发框架,其集成很多开发功能及一身, 能够在本地编译, 部署智能合约, 并且可以通过console 对节点进行 rpc 。

testrpc 严格意义上是一个节点模拟工具(调试环境), 打开本地端口后, 其数据存在内存中, 不在硬盘的数据库内(不同于 geth ,mist ) 用于测试合约很方便,  
(如果在geth 上测试合约,需要自己开私链,还是方便了不少)

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%B7%A5%E5%85%B7%E5%AE%89%E8%A3%85 "工具安装")工具安装

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#truffle-%E5%AE%89%E8%A3%85 "truffle 安装")truffle 安装

truffle 是node.js工程  
所以先安装 node.js

安装环境后 console 执行

    npm install -g truffle        #可能需要权限

安装完成后执行

    truffle version

回显

    Truffle v4.0.5 (core: 4.0.5)Solidity v0.4.18 (solc-js)

如上安装成功

**这个框架在激烈的开发中,所以不同版本,可能出入大,(反正我是支持支持最新版!)**

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#testrpc-%E5%AE%89%E8%A3%85 "testrpc 安装")testrpc 安装

    npm install -g ethereumjs-testrpc

也是js 开发,所以,一键安装它  
执行后,回显如下

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  
33  
34  
35  

  
EthereumJS TestRPC v6.0.3 (ganache-core: 2.0.2)  
  
Available Accounts  
==================  
(0) 0x819d0cce264d8c7028f079f828ec44ad50ab6f1f  
(1) 0xa0eb8d663514aed055c26fdfa02082f283e3814b  
(2) 0x083c2e3debbd83e7193d430c95cb65dfac38be2e  
(3) 0xa268de20bc84f2a371f46e71dd51a437fd0b2b8a  
(4) 0x6a183c520fac524984ff620f6962e758b0a72d3c  
(5) 0xe365cc8ef2ab2bc1bc4d6bbda83a8abb589239cc  
(6) 0x179aa33ae7918af15c956d8b2c8e784004da30e7  
(7) 0x596a67f06884c15ffe9c8767b698730791d6a80d  
(8) 0x318bb06f46a24a322a2a0a173712b630a41f8755  
(9) 0xe1b157fed1cbd523538ed0785bbafef2bb5b5aa5  
  
Private Keys  
==================  
(0) 660f587668641256e4ad353e0a2df52f02524c41641ea3e55aa4ad28300f3c63  
(1) 1ff553362355f36b2d2c3bcef0487cbf4a0db041eeab71e00c6bfb5b43a82e67  
(2) bd10b1b76af76ab8a02a9299efd9ff3cafff68a6af0adf4b7e1ff83abc50b479  
(3) 673ee36f18ee7d58ce0ddfd78b4a584c9b2f12be4c7860d813e9e906582aba15  
(4) 794e336225d007e6a3fbd06a60d087fd0a0380fb04a39adbc066e765c676a105  
(5) c7cd161d995caf782f90ac51e86aa9acece2dbf2f95e1fa0ddd286c98dcbe6ec  
(6) 9058ad2690b7db693675badc61388fe563b6b35e800e02e760e8b3a681660470  
(7) 5b343a275911d1cb644ef55081feabc5d40fce12e1036b2f2eb3c639763e2f7a  
(8) fb237a23e02e29711e7050f394c09bdee3d7818ab91dfcb82501773a80b61c5a  
(9) f6f62f1d1ef0811a9f2c63cde211fc2a57032a4752505cee8e18caa85ff94339  
  
HD Wallet  
==================  
Mnemonic:      rose april chef waste mule setup coffee icon upper news amused lecture  
Base HD Path:  m/44'/60'/0'/0/{account_index}  
  
Listening on localhost:8545  

会自动的分配我们十个地址,用于测试, 打开8545 rpc端口

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E7%8E%AF%E5%A2%83%E6%B5%8B%E8%AF%95 "环境测试")环境测试

先启动节点 testrpc

    testrpc

之后启动truffle的rpc命令行

    truffle console        #可能会有网络问题,见后

当终端1出现

    truffle(development)>

说明已经,正常接入rpc控制台

执行命令

    truffle(development)> web3.eth.accounts[ '0x819d0cce264d8c7028f079f828ec44ad50ab6f1f',  '0xa0eb8d663514aed055c26fdfa02082f283e3814b',  '0x083c2e3debbd83e7193d430c95cb65dfac38be2e',  '0xa268de20bc84f2a371f46e71dd51a437fd0b2b8a',  '0x6a183c520fac524984ff620f6962e758b0a72d3c',  '0xe365cc8ef2ab2bc1bc4d6bbda83a8abb589239cc',  '0x179aa33ae7918af15c956d8b2c8e784004da30e7',  '0x596a67f06884c15ffe9c8767b698730791d6a80d',  '0x318bb06f46a24a322a2a0a173712b630a41f8755',  '0xe1b157fed1cbd523538ed0785bbafef2bb5b5aa5' ]

回显恰好是分配我们的测试地址

余额查询

    ruffle(development)> web3.eth.getBalance(web3.eth.accounts[0])BigNumber { s: 1, e: 20, c: [ 1000000 ] }

(好像js 对大数支持不好???)

[](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#Q-amp-A "Q&A")Q&A
=============================================================================================================================================================================

因为这个版本迭代太快,所以发现网上有些教程,存在各种报错  
所以算是自己慢慢摸索考证,解决了部分问题

    Q: 运行truffle console 时 报错,说是网络有问题相关    Error: No network specified. Cannot determine current network.A: 修改目录下的truffle.js的内容 用于指定RPC的地址

​  
module.exports = {  
networks: {  
development: {  
host: “localhost”,  
port: 8545,  
network_id: “*”  
}  
}  
};

至此环境搭建完毕

[](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%BC%80%E5%A7%8B "开始")开始
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

上面算是搭建好了开发的环境  
下面开始写一个hello world的智能合约

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6 "什么是智能合约")什么是智能合约

> 其实在我的理解上 EVM 以太坊虚拟机,就是我们允许智能合约的平台, 我们使用solidity 编写的合约,然后,经过编译器,将其编译成字节码(op) , (真的神奇), 当部署,和使用合约之后,就会被执行  
> 就是一个函数,签订合同的就可以使用(估计错误)

这样援引一段话

> 智能合约只是一些运行在电脑（或其他节点）的区块链加密货币网络的特定代码，一旦节点执行了这个代码，合约就会更新总账（ledger）。所以如果你们熟悉我的话，我喜欢在一些已经存在的概念上(notion)，做一些类比和抽象上的尝试。实际上这可以构建的知识结构，比如模式(schema)。  
> 什么是智能合约

这里是部分EOS(基于eth的去中心操作系统的部分合约代码)

    contract DSAuthority {function canCall(    address src, address dst, bytes4 sig    ) constant returns (bool);}contract DSAuthEvents {    event LogSetAuthority (address indexed authority);    event LogSetOwner     (address indexed owner);}

下面是部分的evm机器码(汇编!!!)

    PUSH1 0x60PUSH1 0x40MSTORECALLDATASIZEISZEROPUSH2 0x011bJUMPI

这里是的EOS合约内容

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%90%88%E7%BA%A6%E9%83%A8%E7%BD%B2 "合约部署")合约部署

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%B7%A5%E7%A8%8B%E6%A8%A1%E6%9D%BF "工程模板")工程模板

truffle 很方便的给我们提供了,便捷的工程模板的搭建

建立目录hello 执行

    truffle init

等待片刻

    Downloading...Unpacking...Setting up...Unbox successful. Sweet!Commands:  Compile:        truffle compile  Migrate:        truffle migrate  Test contracts: truffle test

以上回显 表示模板建立OK 目录结构

    hello├── contracts│   └── Migrations.sol├── migrations│   └── 1_initial_migration.js├── test├── truffle-config.js└── truffle.js3 directories, 4 files

模板初始化完毕

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%90%88%E7%BA%A6%E7%BC%96%E8%AF%91 "合约编译")合约编译

solidity是编写合约的语言,和js相似(奈何没学过),所以从度娘那里抄了一个helloworld

    pragma solidity ^0.4.4;contract hello {    function sayHello() returns (string) {            return ("Hello World");    }}

contract {} 说明是合约, 定义函数,说明返回值类型,函数体返回字符串,所以最后应该会显示该字符串(应该吧)

保存为hello.sol ,保存在contract 目录下  
(模板本身自带一个合约,可以删除,可以不动,没影响)

执行 完成编译

    truffle compile Compiling ./contracts/Migrations.sol...Compiling ./contracts/hello.sol...Compilation warnings encountered:/Users/r4y/Misc/tmmp/contracts/hello.sol:4:6: Warning: No visibility specified. Defaulting to "public".        function sayHello() returns (string) {     ^Spanning multiple lines.,/Users/r4y/Misc/tmmp/contracts/hello.sol:4:6: Warning: Function state mutability can be restricted to pure        function sayHello() returns (string) {     ^Spanning multiple lines.Writing artifacts to ./build/contracts

警告先忽略, 编译内容在我们的 **build/contract** 下的json文件中

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%90%88%E7%BA%A6%E9%83%A8%E7%BD%B2-1 "合约部署")合约部署

修改部署脚本 **igrations/1\_initial\_migration.js**

    ar Migrations = artifacts.require("./hello.sol");module.exports = function(deployer) {  deployer.deploy(Migrations);};

**部署合约前先要启动节点**  
执行 以进行合约部署

    truffle migrate    --reset    #出现网络问题,要注意网络部署 见前

注意 : 当增加或者删除了某个合约后，可以执行“truffle migrate –reset”命令重新部署合约。  
终端回显

    Running migration: 1_initial_migration.js  Deploying hello...  ... 0xf4f68259d28b84c26ce9bdfb5b246c40441f2733b7810d555fbcdeda24249b6e  hello: 0x84c33260f8085d2b236184734072755cd661dcebSaving artifacts...

同时 节点终端dbg  
有我们的TX gas

    Transaction: 0xf4f68259d28b84c26ce9bdfb5b246c40441f2733b7810d555fbcdeda24249b6eContract created: 0x84c33260f8085d2b236184734072755cd661dcebGas usage: 142468Block Number: 1Block Time: Thu Jan 25 2118 xx:xx:21 GMT+0x00 (CST)

合约部署完毕

#### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%90%88%E7%BA%A6%E6%89%A7%E8%A1%8C "合约执行")合约执行

    truffle consloe     #进入rpc 命令行ruffle(development)> var contractundefinedtruffle(development)> hello.deployed().then(instance => contract = instance)......truffle(development)> contract.sayHello.call()'Hello World'

至此HELLOworld 总算是hel出来了

### [](https://www.diglp.xyz/2018/02/09/ETH%20%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%20helloworld/#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98 "常见问题")常见问题

Q&A