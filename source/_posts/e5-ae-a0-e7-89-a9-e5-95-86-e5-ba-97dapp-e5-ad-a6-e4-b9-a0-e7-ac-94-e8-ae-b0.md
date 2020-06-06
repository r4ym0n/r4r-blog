---
title: 宠物商店Dapp 学习笔记
url: 588.html
id: 588
date: 2018-02-11 00:00:00
tags:
---

truffle 可以算是一个超级强大的 Ethereum 开发工具集, 集各种的功能集一身, 今天, 照着官方的文档, 和 手把手的教程, 完成了其中提供的一个demo.

[](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#truffle%E7%9A%84%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84 "truffle的目录结构")truffle的目录结构
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E7%9B%AE%E5%BD%95%E6%A0%91 "目录树")目录树

    demo├── build│   └── contracts│       ├── Migrations.json│       └──  Adoption.json├── contracts│   ├── Migrations.sol│   └── Adoption.sol├── migrations│   └── 1_initial_migration.js├── truffle-config.js├── truffle.js├── test│   └──1_initial_migration.js└── src    └── ...

​

*   contract 此目录就是我们的编写的智能合约所存在的目录, 使用solidity语言编写
*   migrations 此目录下是用于迁移部署合约的JS的脚本
*   test 测试合约时所用的测试脚本
*   src 一个前端的实现, 主要是调用 wed3 的库, 与节点服务器进行RPC

[](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E4%BB%A3%E7%A0%81%E5%88%86%E6%9E%90 "代码分析")代码分析
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E5%90%88%E7%BA%A6%E4%BB%A3%E7%A0%81 "合约代码")合约代码

    pragma solidity ^0.4.17;contract Adoption {  address[16] public adopters;  // 保存领养者的地址  /**func: 领养宠物 para: 领养宠物ID */  function adopt(uint petId) public returns (uint) {      require(petId >= 0 && petId <= 15);  // 确保id在数组长度内    adopters[petId] = msg.sender;        // 保存调用这地址     return petId; //返回当前宠物ID  }  // 返回领养者  function getAdopters() public view returns (address[16]) {    return adopters;  }}

以上是实现宠物领养的合约代码.

*   指定编译器版本
*   定义 Adoption 的合约结构
    
        contract {...}
    
*   定义一个存放地址的定长数组
*   定义合约函数 adopt
    *   Public类型, 可以被外部访问, uint参数 为调用时传入的要领养的宠物的ID, 返回值就是当前领养的宠物id
    *   `require()` 用于检查变量值是否满足当前条件, 不满足条件则立即抛出异常, 并且对所有的已做修改进行回滚(revert)
    *   使用数组的对应的 ID index 来保存领养者的地址
*   定义合约函数 `getAdopters()`
    *   直接返回 存储领养者的定长数组

### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E6%B5%8B%E8%AF%95%E4%BB%A3%E7%A0%81 "测试代码")测试代码

truffle 作为一个集成的环境, 也是很好的提供了合约测试的功能

    pragma solidity ^0.4.17;import "truffle/Assert.sol";   // 引入的断言文件import "truffle/DeployedAddresses.sol";  // 用来获取被测试合约的地址// 上面两个文件是Truffle框架提供, 本身并没有import "../contracts/Adoption.sol";      // 被测试合约, 这样才能调用它//这个合约是用来测试合约的, 每个用例都会被执行, 通过断言判断是否有问题contract TestAdoption {  Adoption adoption = Adoption(DeployedAddresses.Adoption());  // 领养测试用例  function testUserCanAdoptPet() public {    uint returnedId = adoption.adopt(8);    // 这里传入是8 返回也应该是8    uint expected = 8;    Assert.equal(returnedId, expected, "Adoption of pet ID 8 should be recorded.");  }  // 宠物所有者测试用例  function testGetAdopterAddressByPetId() public {    // 期望领养者的地址就是本合约地址，因为交易是由测试合约发起交易，    address expected = this;    address adopter = adoption.adopters(8);    // 当前地址和返回地址的判断, adopters明明是Array啊, 还能这样?    Assert.equal(adopter, expected, "Owner of pet ID 8 should be recorded.");  }    // 测试所有领养者  function testGetAdopterAddressByPetIdInArray() public {  // 领养者的地址就是本合约地址    address expected = this;    address[16] memory adopters = adoption.getAdopters(); // 内存分配,不是storage    Assert.equal(adopters[8], expected, "Owner of pet ID 8 should be recorded.");  }}// 关于 this的使用, this代表当前合约, 也是ADDRESS

代码分析写在了注释里面 , 下面是执行结果

    > truffle testUsing network 'development'.Compiling ./contracts/Adoption.sol...Compiling ./test/TestAdoption.sol...Compiling truffle/Assert.sol...Compiling truffle/DeployedAddresses.sol...

​  
TestAdoption  
✓ testUserCanAdoptPet (113ms)  
✓ testGetAdopterAddressByPetId (110ms)  
✓ testGetAdopterAddressByPetIdInArray (196ms)

​  
3 passing (1s)

可见, 通过测试命令, Truffle 对每个函数进行自动的 测试, 对运行结果进行assert(断言)分析, 如果合约代码存在问题, 测试过程会把错误显示出. 这里是全部测试通过的情况.

### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E5%89%8D%E7%AB%AF%E5%BA%94%E7%94%A8%E4%BB%A3%E7%A0%81 "前端应用代码")前端应用代码

奈何 没接触过JS 所以看起来有些吃力, 也算是借这个学习一下了!

代码主体存在于 `src/js/app.js` , 从功能上讲就是对 我们的合约进行调用, 把我们领养的宠物这个信息记录在区块里.

#### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#InitWeb3 "InitWeb3")InitWeb3

    initWeb3: function () {  // 是否当前浏览器提供web3(如 metaMask)?  if (typeof web3 !== 'undefined') {    App.web3Provider = web3.currentProvider;    //如果是就直接使用当前的  } else {    // 如果没有插件提供的web3, 就向本地的节点要一个    App.web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');  }  web3 = new Web3(App.web3Provider);  return App.initContract();    // 调用后续},

这个是web3这个JS包的初始化代码,代码中优先使用Mist 或 MetaMask提供的web3实例，如果没有则从本地环境创建一个。

#### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#InitContract "InitContract")InitContract

    initContract: function () {  $.getJSON('Adoption.json', function (data) {    // 用Adoption.json数据创建一个可交互的TruffleContract合约实例。    var AdoptionArtifact = data;    App.contracts.Adoption = TruffleContract(AdoptionArtifact);    // Set the provider for our contract    App.contracts.Adoption.setProvider(App.web3Provider);    // Use our contract to retrieve and mark the adopted pets    return App.markAdopted();    //进行回调,所以先执行这个  });  return App.bindEvents();},

这里实现了合约的初始化, 这里加载了Adoption.json，保存了Adoption的ABI（接口说明）信息及部署后的网络(地址)信息，它在编译合约的时候生成ABI，在部署的时候追加网络信息.

> 画外音: 这里也是展现了NodeJs的一切皆回调的 异步属性 , 中间部分的代码就是我们执行GetJson的 回调代码.(不同于同步(顺序)编程)

#### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#MarkAdopted "MarkAdopted")MarkAdopted

    markAdopted: function(adopters, account) {var adoptionInstance;App.contracts.Adoption.deployed().then(function(instance) {  adoptionInstance = instance;      // 这里部署了合约, 并且保存该合约的实例      // 调用合约的getAdopters(), 用call读取信息不用消耗gas, 返回领养者的array  return adoptionInstance.getAdopters.call();}).then(function(adopters) {    // 得到领养者列表之后, 这里进行遍历, 如果发现有存在合理的领养者(if), 改变前端的按钮样式吧(应该)  for (i = 0; i < adopters.length; i++) {    if (adopters[i] !== '0x0000000000000000000000000000000000000000') {      $('.panel-pet').eq(i).find('button').text('Success').attr('disabled', true);    }  }}).catch(function(err) {  console.log(err.message);      //这里捕获异常, 并且log出来});},

> 画外音 : then 也是一种回调用法, 在前一函数执行完之后, 才会进行 then 内的函数, 这样就确保了数据的完整获得

这里实现了 合约的部署, 和对已经领养的dog进行标记, 遍历数组得到, 这个判断方式值得学习.

#### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E7%BB%91%E5%AE%9A%E4%BA%8B%E4%BB%B6 "绑定事件")绑定事件

    bindEvents: function() {   $(document).on('click', '.btn-adopt', App.handleAdopt); },

这个地方就是通俗易懂了,把按钮点击的事件, 和它的服务函数(handle), 绑定起来.

> 画外音: JavaScript 是一种事件驱动的语言, 所以, 这里和QT很相似, 也是事件驱动, 用户的点击, 产生事件, 之后调用事件的处理函数,区别于 消息驱动(如MFC)

#### [](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E6%9C%8D%E5%8A%A1%E5%87%BD%E6%95%B0 "服务函数")服务函数

    handleAdopt: function(event) {   event.preventDefault();   var petId = parseInt($(event.target).data('id'));   var adoptionInstance;   // 获取用户账号.调用Web3   web3.eth.getAccounts(function(error, accounts) {     // 出常的回调     if (error) {       console.log(error);     }     var account = accounts[0];   // 创建合约实例             App.contracts.Adoption.deployed().then(function(instance) {       adoptionInstance = instance;       // 发送交易领养宠物       return adoptionInstance.adopt(petId, {from: account});    执行领养函数     }).then(function(result) {       return App.markAdopted();    // 标记所领养宠物     }).catch(function(err) {       console.log(err.message);     });   }); }

这里就是, 鼠标点击事件的服务函数. 当鼠标点击Button的时候事件触发, 服务函数被回调.  
>  
>  
>

> *   event.preventDefault()
>     
>     该方法将通知 Web 浏览器不要执行与事件关联的默认动作（如果存在这样的动作）。避免服务函数被中断
>     
> *   parseInt()
>     
>     该方法将通知 Web 浏览器不要执行与事件关联的默认动作（如果存在这样的动作）
>     

[](https://www.diglp.xyz/2018/02/11/%E5%AE%A0%E7%89%A9%E5%95%86%E5%BA%97(pet-shop)%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E5%86%99%E5%9C%A8%E5%90%8E%E9%9D%A2 "写在后面")写在后面
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------

这次是个简单的demo的实现和分析,也得上是第一个Dapp的实现

这个Demo 的实现参照博文

一步步教你开发、部署第一个去中心化应用(Dapp) - 宠物商店

也感谢博主的OFS的开源精神!(保留版权声明)