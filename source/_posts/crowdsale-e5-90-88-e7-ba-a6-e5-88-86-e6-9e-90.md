---
title: CrowdSale 合约分析
url: 582.html
id: 582
date: 2018-02-14 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E5%89%8D "前")前
--------------------------------------------------------------------------------------------------

前面的文章实现了自己创建的 Token 在以太坊网络的发布, 这一篇, 接着来, 也是涉及到了更多的东西. 这次实现一个合约, 实现我们可以进行自动的 Token发放.

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E5%90%88%E7%BA%A6%E4%BB%A3%E7%A0%81 "合约代码")合约代码
-----------------------------------------------------------------------------------------------------------------------------------

此段代码选自官方的教程, 这里凭着个人的学习和理解, 加上注释

> 官方代码

    pragma solidity ^0.4.16;interface token {    function transfer(address receiver, uint amount);   // 这里调用你的Token合约的函数接口}contract Crowdsale {    address public beneficiary;     // ICO的目标账户    uint public fundingGoal;        // 筹集的目标金额    uint public amountRaised;       // 当前募集的金额    uint public deadline;           // 时间限制    uint public price;              // 单token的定价    token public tokenReward;       // 这里是token的地址    mapping(address => uint256) public balanceOf;       // 地址和Token的映射关系    // 以上所有的数据都是Public的    bool fundingGoalReached = false;                    // 是否筹够    bool crowdsaleClosed = false;                       // 是否进行    event GoalReached(address recipient, uint totalAmountRaised);       // 达标事件    event FundTransfer(address backer, uint amount, bool isContribution);       // 转款事件    // 事件用于记录信息, Dapp读取事件    /**     * Constrctor function     *     * Setup the owner     */    function Crowdsale(                                 // 构造函数        address ifSuccessfulSendTo,                     // 参数表        uint fundingGoalInEthers,        uint durationInMinutes,        uint etherCostOfEachToken,        address addressOfTokenUsedAsReward    ) {        beneficiary = ifSuccessfulSendTo;        fundingGoal = fundingGoalInEthers * 1 ether;        deadline = now + durationInMinutes * 1 minutes;        price = etherCostOfEachToken * 1 ether;        tokenReward = token(addressOfTokenUsedAsReward);        // 对上面的数据变量进行赋值    }    /**     * Fallback function     *     * The function without name is the default function that is called whenever anyone sends funds to a contract     * 无名函数用于任何时候有人转钱了的回调, 这里是分配token的重点     */    function () payable {        require(!crowdsaleClosed);      // 保证Ico是没有结束        uint amount = msg.value;        balanceOf[msg.sender] += amount;            amountRaised += amount;        tokenReward.transfer(msg.sender, amount / price);   // 这里调用接口,把等价的Token分配给msg        FundTransfer(msg.sender, amount, true);             // 产生事件, 已经转Token了!    }    modifier afterDeadline() { if (now >= deadline) _; }    // 修饰符, 是不是已经过了时间    /**     * Check if goal was reached     *     * Checks if the goal or time limit has been reached and ends the campaign     */    function checkGoalReached() afterDeadline {     // 注意这里的修饰符, 如果已经超时了, 直接关闭ICO,         if (amountRaised >= fundingGoal){            fundingGoalReached = true;              // 判断是否筹齐, 齐了就发事件            GoalReached(beneficiary, amountRaised);        }        crowdsaleClosed = true;    }

​  
/**

         * Withdraw the funds (退钱的)     *     * Checks to see if goal or time limit has been reached, and if so, and the funding goal was reached,     * sends the entire amount to the beneficiary. If goal was not reached, each contributor can withdraw     * the amount they contributed.     * 检查时间, 和总额是否集齐, 如果达到 , 就把合约中的所有的钱款转入 受益人的账户. 如果没有达成,投钱的可以拿回自己的钱     */    function safeWithdrawal() afterDeadline {        if (!fundingGoalReached) {                  // 没达成            uint amount = balanceOf[msg.sender];    // 投资人有多少Token            balanceOf[msg.sender] = 0;              // 把他的token 清零            if (amount > 0) {                                       if (msg.sender.send(amount)) {                          // 这里应该是有个发送请求(后面自己看看)                    FundTransfer(msg.sender, amount, false);                } else {                    // 保持总额不变                    balanceOf[msg.sender] = amount;                 }            }        }        if (fundingGoalReached && beneficiary == msg.sender) {          // 如果已经集齐,而且是发起人调用了合约            if (beneficiary.send(amountRaised)) {                       // 这里和上面一样, 应该是Address的一个成员函数                FundTransfer(beneficiary, amountRaised, false);         // 发送 转款事件            } else {                //If we fail to send the funds to beneficiary, unlock funders balance                fundingGoalReached = false;            }        }    }}

其实可见, 一个实现token 众售的合约实际上还是比较容易理解的, 主要是 token 的发放, 退钱的, 和owner 用来提钱的这几个部分组成

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#Point "Point")Point
------------------------------------------------------------------------------------------------------

这里对上面的合约出现的新的Solidity要点进行说明

### [](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E6%8E%A5%E5%8F%A3-interface "接口(interface)")接口(interface)

接口这个东西 , 其实在上一篇文章中已经有说过 , 哪里没有用到. 是个很重要的东西 , 就是实现可以在合约中调用其他合约的 external 的函数.

    interface token {    function transfer(address receiver, uint amount);   // 这里调用你的Token合约的函数接口}

这里定义 的一个接口, 主要是是实现调用 我们Token合约中的合约发放函数. 这里有个很棒的例子: 这个是著名的以太坊的加密猫, 这个项目当时可是导致了以太坊网络的严重堵塞. 这里选取里其中一段合约代码.

    function getKitty(uint256 _id)    external    view    returns (    bool isGestating,    bool isReady,    uint256 cooldownIndex,    uint256 nextActionAt,    uint256 siringWithId,    uint256 birthTime,    uint256 matronId,    uint256 sireId,    uint256 generation,    uint256 genes) {    Kitty storage kit = kitties[_id];    // if this variable is 0 then it's not gestating    isGestating = (kit.siringWithId != 0);    isReady = (kit.cooldownEndBlock <= block.number);    cooldownIndex = uint256(kit.cooldownIndex);    nextActionAt = uint256(kit.cooldownEndBlock);    siringWithId = uint256(kit.siringWithId);    birthTime = uint256(kit.birthTime);    matronId = uint256(kit.matronId);    sireId = uint256(kit.sireId);    generation = uint256(kit.generation);    genes = kit.genes;}

由名字和返回值可见 , 这个是blahblah 一大堆,用于获取某只Kitty 的所有信息的函数. 这里如果我们的合约突然想使用一下这里的喵的DNA怎么办? 很好, 我们可以使用接口了!

    interface Kitty {      function getKitty(uint256 _id) external view returns (        bool isGestating,        bool isReady,        uint256 cooldownIndex,        uint256 nextActionAt,        uint256 siringWithId,        uint256 birthTime,        uint256 matronId,        uint256 sireId,        uint256 generation,        uint256 genes    );}

上面我们就定义了一个对应的接口, 如何使用呢?

    address ckAddr = 0x06012c8cf97bead5deae237070f9587f8e7a266d;Kitty kittyInterface = Kitty(ckAddress);     // 这里实例化这个接口!// 下面就是调用了uint dna;,,,,,,,,dna = kittyInterface.getKitty(0)        // 假定是 0 号Kitty// 这里是多返回值

这样我们就可以获取dna了, 使用接口, 是不是很精妙?

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E5%9B%9E%E9%80%80%E5%87%BD%E6%95%B0 "回退函数")回退函数
-----------------------------------------------------------------------------------------------------------------------------------

> 每一个合约有且仅有一个没有名字的函数。这个函数无参数，也无返回值。如果调用合约时，没有匹配上任何一个函数(或者没有传哪怕一点数据)，就会调用默认的回退函数。

在示例代码中, 我们使用到了回退函数,(可见只有一个修饰符, 没有函数名的)

    function () payable {    require(!crowdsaleClosed);      // 保证Ico是没有结束    uint amount = msg.value;    balanceOf[msg.sender] += amount;        amountRaised += amount;    tokenReward.transfer(msg.sender, amount / price);   // 这里调用接口,把等价的Token分配给msg    FundTransfer(msg.sender, amount, true);             // 产生事件, 已经转Token了!}

这个也是我们ICO合约的重要函数, 实现Token的分发. 根据定义, 我们如果直接对合约地址转账, 那么默认就会调用了回退函数, 这里很巧妙的利用来转账!

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E5%9C%B0%E5%9D%80-Address "地址(Address)")地址(Address)
---------------------------------------------------------------------------------------------------------------------------------------

上面的代码有一处是没看懂

    if (msg.sender.send(amount)) {          // 这里应该是有个发送请求(后面自己看看)    FundTransfer(msg.sender, amount, false);} else {    // 保持总额不变    balanceOf[msg.sender] = amount; }

这里感觉是一个请求, 会有返回值.于是下面就查证官方的Wiki看到

> 地址类型的成员 属性：balance 函数：send()，call()，delegatecall()，callcode()。

*   地址字面量(literal) 其实我们可以理解成一个常量, 在Solidity 里如果有字面量
    
        0x06012c8cf97bead5deae237070f9587f8e7a266d
    
    这个会直接被编译器理解为Address类型.
    
*   balance 这个如其定义一样 `myAddress.balance`, 这个值就是我们的当前余额
    
*   send 这个比较重要, 之前没理解,怎么是下面这张形式
    
        beneficiary.send(amountRaised)
    
    最后Wiki’中知道了, 这个理解顺序是 `this to beneficiary`, 就是由合约向其他的账户发送Ether.
    

[](https://www.diglp.xyz/2018/02/14/CrowdSale%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0/#%E5%90%8E%E9%9D%A2%E7%9A%84%E8%AF%9D "后面的话")后面的话
-----------------------------------------------------------------------------------------------------------------------------------

这次的crowdsale 结合上一次的token , 就可以发动一场轰轰烈烈的ICO了. 不过 希望能够只是以学习为目的, 尊重技术. 不要让这种没有意义的代码充斥 以太坊网络 **Don’t be evil** 后面还会有一篇,crowdsale在测试网络上的部署. 自己记录本身, 也是学习, 和大家共勉!