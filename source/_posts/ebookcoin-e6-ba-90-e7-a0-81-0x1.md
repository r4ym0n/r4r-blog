---
title: EbookCoin 源码 0x1
url: 540.html
id: 540
date: 2018-05-02 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址
> 
> [](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#app-js "app.js")app.js
> ----------------------------------------------------------------------------

这个 **app.js** 是在node.js 中的入口程序文件，在其他情况下也可能是 **server.js** 。类似以python的顺序执行的模式。没有(可以用)一个main。

### [](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#%E6%A8%A1%E5%9D%97%E4%BE%9D%E8%B5%96 "模块依赖")模块依赖

下面的代码是 **app.js 1~15** 主要的是导入的模块。这里，一个个的search这些模块的功能

    var program = require('commander');                // 命令行框架开源包 commander.jsvar packageJson = require('./package.json');    // 定义了这个项目所需要的各种模块var Logger = require('./logger.js');            // 日志模块var appConfig = require("./config.json");        // 全局默认配置var genesisblock = require('./genesisBlock.json');    // 创世区块的设置var async = require('async');                    //    异步编程包var extend = require('extend');                    // 实现Obj的重载var path = require('path');                        // 处理路径的包var https = require('https');                    // TLS/SSL的包var fs = require('fs');                            // 文件系统包var z_schema = require('z-schema');                // 同步异步？var util = require('util');                        // 模块支持包var Sequence = require('./helpers/sequence.js');    // 

*   **config.json** 在导入文件中，`config.json` 是全局配置文件。当参数少的时候，可以硬编码到代码里，当参数多的时候就是需要 全局配置文件。
    
        {    "port": 7000,    "address": "0.0.0.0",    "serveHttpAPI": true,    "serveHttpWallet": true,    "version": "0.1.3",    ...
    
*   **commander.js**
    
        program    .version(packageJson.version)    .option('-c, --config <path>', 'Config file path')    .option('-p, --port <port>', 'Listening port number')    .option('-a, --address <ip>', 'Listening host name or ip')    .option('-b, --blockchain <path>', 'Blockchain db path')    .option('-x, --peers [peers...]', 'Peers list')    .option('-l, --log <level>', 'Log level')    .parse(process.argv);    // 参数在这里
    
    这里是使用 commander 模块对输入的启动参数(process.argv)进行解析。
    
        $ node app.js -p 8080 # 这里是启动进程的参数# 这样的话参数就被按序的保存在了program里program.port == '8080' // True
    

### [](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#%E5%8A%9F%E8%83%BD%E5%AE%9E%E7%8E%B0 "功能实现")功能实现

*   **20~70** 实现了一个对于启动参数的解析和保存功能。
    
*   **35** 周期性的调用，`setInterval` 可以对设置的函数进行周期性调用。
    
        if (typeof gc !== 'undefined') {    setInterval(function () {        gc();    }, 60000);    // 1 min}
    
*   **72** **103** 异常处理 这里是对于这个app的异常处理的函数，一样的是回调。报一个 fatal 并且发送消息
    
        process.on('uncaughtException', function (err) {    // handle the error safely    logger.fatal('System error', { message: err.message, stack: err.stack });    process.emit('cleanup');    // 发送信号给handler});
    
    这里，应该应该也是个异常处理，应该是同时会log异常产生的域
    
        var d = require('domain').create();d.on('error', function (err) {    logger.fatal('Domain master', { message: err.message, stack: err.stack });    process.exit(0);});
    
*   **101** logger的初始化
    
*   **78** 缺省的硬编码配置，应该挺重要所以贴出
    
        var config = {    "db": program.blockchain || "./blockchain.db",    "modules": {        "server": "./modules/server.js",        "accounts": "./modules/accounts.js",        "transactions": "./modules/transactions.js",        "blocks": "./modules/blocks.js",        "signatures": "./modules/signatures.js",        "transport": "./modules/transport.js",        "loader": "./modules/loader.js",        "system": "./modules/system.js",        "peer": "./modules/peer.js",        "delegates": "./modules/delegates.js",        "round": "./modules/round.js",        "contacts": "./modules/contacts.js",        "multisignatures": "./modules/multisignatures.js",        "dapps": "./modules/dapps.js",        "sia": "./modules/sia.js",        "crypto": "./modules/crypto.js",        "sql": "./modules/sql.js"    }};
    
    这里基本上是描述了功能所对应的模块
    
*   **108** 这里的一个 `d.run()` 可能是打开了大门 前面得知，d是我们创建的一个域，这里应该就是我们的域开始的时候了。这部分书中称为是模块加载。  
    这里有个十分重要的东西 `async`。
    
    > Async is a utility module which provides straight-forward, powerful functions for working with asynchronous JavaScript.
    
    官方的描述，是该模块提供了一个处理异步的功能。这里的`async.auto`意味着代码的顺序执行.
    
        function auto<R extends async.Dictionary<any>, E>(tasks: async.AsyncAutoTasks<R, E>, concurrency?: number, callback?: async.AsyncResultCallback<R, E>): void
    
    这个是auto函数的定义,可以看见, 在后面的如同 `logger: function (cb) {`  
    这种形式的,实际上 **logger** 就是成为了一个任务task(这里本来就是要发生调度的)。那么这里就相当于定义了一组task。
    
*   **222** 网络的初始化 在书中，是直接跳过了前面的调度说明(虽说有点理解)，直接到了网络这个任务。 下面是网络加载的部分代码：
    
        var express = require('express');var app = express();var server = require('http').createServer(app);var io = require('socket.io')(server);
    
    这里使用了 **express** 模块，这个是一个web 应用的开发框架.上面这段代码,算是对于网络服务的初始化,框架绑定 **服务(HTTP)**,服务绑定 **io(SOCKET)**. 这里的Scope到底是怎么来的???可能是对这个执行原理是不清楚.不过书中是直接让理解成为从 **`config.js`** 的内容包含(即前面的所有模块).
    
    *   **228** 这里是对是否使用 SSL 的一个判断
        
            if (scope.config.ssl.enabled) {
        
        可以在 Config.json 里找到对应的配置
        
            "ssl": {    "enabled": false,    "options": {        "port": 443,        "address": "0.0.0.0",        "key": "./ssl/ebookcoin.key",        "cert": "./ssl/ebookcoin.crt"    }
        
*   **277** 构建链接 实际上是网络模块直接的链接 这里可以看到使用的模块,就会很多了.所以这里是实现模块间的功能链接
    
        connect: ['config', 'public', 'genesisblock', 'logger', 'build', 'network', function (cb, scope) {
    
    这部分,是真的有点模糊,什么中间价之类的…实际上这里的app是我们的express的框架,那么这里的use,理应是对框架所使用的功能的配置.
    
    *   **310** 这里可能是对我们的http请求的解析
        
            var parts = req.url.split('/');
        
        这里分割 url ,把请求分词存在`part`里面.这里是解析的函数.
        
            if (parts.length > 1) {    if (parts[1] == 'api') {        if (scope.config.api.access.whiteList.length > 0) {            if (scope.config.api.access.whiteList.indexOf(ip) < 0) {                    res.sendStatus(403);                } else {                    next();                }            } else {                next();            }        } else if (parts[1] == 'peer') {            if (scope.config.peers.blackList.length > 0) {                if (scope.config.peers.blackList.indexOf(ip) >= 0) {                    res.sendStatus(403);                } else {                    next();                }            } else {                next();            }        } else {            next();        }    } else {        next();    }});
        
    *   **343** 开始服务监听 这个 Listen 我当然懂..终于由开的明白的了,在指定地址,和端口进行监听
        
            scope.network.server.listen(scope.config.port, scope.config.address, function (err) {
        
        > “Ebookcoin started”
        

*   **386** 逻辑加载 ,可能是针对模块间的逻辑,进行的 New.
*   **418** 模块加载, 卒

### [](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#Todo "Todo")Todo

[](https://www.diglp.xyz/2018/05/02/BC_EbookCoin_0x1/#%E6%96%87%E4%BB%B6%E6%80%BB%E7%BB%93 "文件总结")文件总结
------------------------------------------------------------------------------------------------------

app.js 是第一个启动文件, 也是最重要的一个,这里实现了模块的加载,功能的初始化,最重要的是实现了全部的 任务的调度, 由于对于async的不熟悉,和网络框架的不了解,对全局上的实现,还是理解较弱,不过,随着后面的学习,理解程度肯定会加深,来补充 TODO