---
title: 发行自己的ERC20 Token
url: 586.html
id: 586
date: 2018-02-12 00:00:00
tags:
---

这次主要实现 发行一个自己的ERC20 标准的 Token. (然后进行ICO, 圈钱跑路, 走上人生巅峰,XD)

> 其实把Token 叫做代币, 容易让人产生误解.实际上是一种凭证, 只是被现在狂热的人们搅浑了.
> 
> Token的持有人可以完全控制资产，遵守ERC20的token可以跟踪任何人在任何时间拥有多少token.

[](https://www.diglp.xyz/2018/02/12/BC_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#%E5%89%8D "前")前
------------------------------------------------------------------------------------------------------------------

现在币圈的狂热之势四起, 昨天一个朋友, 发来一串神秘代码, 说转账有 3000多的糖果.  
这里偷偷的放一下地址, 可以去玩玩..

> ENU: 0x275b69AA7c8C1d648A0557656bCe1C286e69a29d

![Tx](https://www.diglp.xyz/images/tx.png)

这个转账记录还真是来势汹汹. 然后就好奇的 用浏览器 看了看合约代码

    string public constant name = "Enumivo";string public constant symbol = "ENU";uint public constant decimals = 8;uint256 public totalSupply = 1000000000e8;uint256 public totalDistributed = 100000000e8;uint256 public totalRemaining = totalSupply.sub(totalDistributed);uint256 public value;

这个 `totalSupply` 是不是相当惊人… 这个估计是用来测试的合约, 不知道被谁发现了来.

所以, 这玩意这么火, 这次就自己实现一个!

[](https://www.diglp.xyz/2018/02/12/BC_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#ERC20-Token%E6%A0%87%E5%87%86 "ERC20 Token标准")ERC20 Token标准
--------------------------------------------------------------------------------------------------------------------------------------------------------------

实现这样的一个token 实际上还是使用Solidity语言编写的合约来实现, 只是合约的形式符合了ERC20 的标准. 这样就可以被区块浏览器识别成一个 Token.

> 官方标准 ERC-20 Token Standard

手册里说明了代码的应有的函数和成员 (看E文 还是难受, 找到了一篇译文, 不过是机翻的, 看着玩吧)

> 以太坊ERC20 Token标准完整说明

合约的框架如下

    MARKDOWN_HASH76cf27064fa80102a145ffa96bc36aefMARKDOWNHASH

_<a href="[https://www.diglp.xyz/2018/02/12/BC](https://www.diglp.xyz/2018/02/12/BC)_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#Token%E5%AE%9E%E7%8E%B0%E4%BB%A3%E7%A0%81" class="headerlink" title="Token实现代码">Token实现代码
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

下面就是自己的token的实现代码, 其实我们根据官方的合约框架进行填充, 这里对代码进行注解.

    MARKDOWN_HASH8db24718257a5dd22ed19a62f7eeff9aMARKDOWNHASH

_通过以上代码,就算是实现了一个符合ERC20 标准的Token, 通过读代码,也是学到了其具体实现, 代码的总结,在后面实现吧.  
这次主要是实现这个token的部署._

_<a href="[https://www.diglp.xyz/2018/02/12/BC](https://www.diglp.xyz/2018/02/12/BC)_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#Token%E5%90%88%E7%BA%A6%E9%83%A8%E7%BD%B2" class="headerlink" title="Token合约部署">Token合约部署
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

上面实现了一个token的代码, 现在需要的是把他部署到以太坊网络上去.  
这里我们使用官方的IDE REMIX

> 环境:
> 
> *   firefox
> *   REMIX
> *   mateMask

由于实际上我们的合约部署是需要消耗ether的, 所以这里我们选择以太坊的测试网络, 这样不会消耗主网资源, 利国利民

### [](https://www.diglp.xyz/2018/02/12/BC_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#%E5%88%9B%E5%BB%BA%E6%B5%8B%E8%AF%95%E9%92%B1%E5%8C%85 "创建测试钱包")创建测试钱包

![ropsten](https://www.diglp.xyz/images/ropsten.png)

这里已经在ropsten网络上创建了一个钱包, 注意左上角的网络选择是 Ropsten网络就好, 然后我们创建自己的钱包, 这个很简单, 就不多讲

现在钱包是有了, 没币呀. 不慌! 没币,我们要去, 由于是测试网络, 所以这些东西就很随便了, 有水龙头,给我们免费发放!

> Ethereum Ropsten Faucet

直接在上面的地址直接领就好, (要是现在的行情, 点一下7000块呢 嘿嘿嘿)

### [](https://www.diglp.xyz/2018/02/12/BC_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#%E7%BC%96%E8%AF%91%E5%90%88%E7%BA%A6%E4%BB%A3%E7%A0%81 "编译合约代码")编译合约代码

打开我们的在线IDE, 贴上代码

> REMIX

![RUN](https://www.diglp.xyz/images/run.png)

上面会自动识别成 injected web3 (实际上这个js 是matemask在网页进行注入了)

`Create`前面就是我们创建合约的构造函数的参数, 从左到右是发行量, 名字, 和 符号

    1000, "AnFun", "AFC"

之后点击创建,会弹出mateMask的支付请求

![pay](https://www.diglp.xyz/images/pay.png)

> 画外音:这里可见, 整个过程只用消耗GAS的费用, 可谓相当低 , 然而就这样的东西, 堂而皇之的被各种利用, 成空气币!!!

点击提交(submit), 我们会看到我们的支付记录,

![Txhash](https://www.diglp.xyz/images/txlog.png)

可见, 合约已经创建!

### [](https://www.diglp.xyz/2018/02/12/BC_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#Token%E7%9A%84%E6%B7%BB%E5%8A%A0%E5%92%8C%E4%BA%A4%E6%98%93 "Token的添加和交易")Token的添加和交易

打开我们的小狐狸, 在token的选项中, 点击添加token, 把我们的合约地址(见上图)填入, 自动的识别我们的信息, 再次刷新可见, 我的token 已经到手了!(圈钱跑路去 2333).

> 0xc67Cc41C6517d97df3DD1eCC7885c209fE04bFa8 这里给出地址, 想要的免费空投 2333

![tokens](https://www.diglp.xyz/images/tokens.png)

现在我们有了token了, 下面进行交易, 这里是使用

> MY_WALLET_ _这个可不是钱包, 只是一个在线的接口_

_同样的确认网络一直, 找到Token 即可!_

_<a href="[https://www.diglp.xyz/2018/02/12/BC](https://www.diglp.xyz/2018/02/12/BC)_%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84ERC20%20Token/#%E5%90%8E%E9%9D%A2%E7%9A%84%E8%AF%9D" class="headerlink" title="后面的话">后面的话
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

也是通过这样的一篇文章可见, 有些所谓的token是多么的黑暗.  
参考出处

> 代码参考

这里自己也是学习, 并且加上了注释. 在下一篇, 详细的写写上述代码的 Solidity的知识