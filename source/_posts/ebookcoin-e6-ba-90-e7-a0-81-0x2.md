---
title: EbookCoin 源码 0x2
url: 538.html
id: 538
date: 2018-05-03 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#%E5%89%8D "前")前
---------------------------------------------------------------------

这次的一个period是基于《Nodejs区块链开发》这本书的的区块链项目–亿书，的源码学习笔记

> EbookCoin项目地址

[](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#P2P%E7%BD%91%E7%BB%9C "P2P网络")P2P网络
-----------------------------------------------------------------------------------------

在书中的标题是一个精巧的P2P网络的实现.

### [](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#%E6%A8%A1%E5%9D%97%E5%8C%85%E5%90%AB "模块包含")模块包含

这部分主要的包含文件包括

*   ./modules/peer.js // 用于实现作为节点的功能
*   ./modules/transport.js // 实现传输？
*   ./helper/router.js // 如其名，路由

transport 和 router 作为 **peer** 的两个辅助模块，一起实现了一个p2p网络上的独立节点。

### [](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#router-js-%E8%B7%AF%E7%94%B1%E6%8B%93%E5%B1%95 "router.js 路由拓展")router.js 路由拓展

这个文件内容不多，**42 line**

*   **27** 路由定义
    
        var Router = function () {    var router = require('express').Router();    router.use(function (req, res, next) {        res.header("Access-Control-Allow-Origin", "*");        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");        next();    });    router.map = map;    return router;}
    
    这段代码，实现了一个 Router 的辅助模块。定义 router 是一个 基于Express 的拓展。实现两个功能：
    
    *   `"Access-Control-Allow-Origin", "*"` `"Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept"`说明允许跨域请求，任何的ip和端口的节点都可以被访问。
    *   `router.map` 设置(指定)了地址的映射方法。

    > [什么是跨域请求](https://blog.csdn.net/github_37360787/article/details/54834789)    

*   **3** 地址映射 这里的map有两个参数，root，config。
    
        function map(root, config) {
    
    *   root：定义开放API的逻辑函数
    *   config：定义了路由和root所定义的函数之间的对应关系。 等同于  
        router.get(‘/peers’, function(req,res,next){
        
            root.getPeers(...);
        
        }); **在Js中对象是散列的，所以`root.getPeer()` 和 root\[‘getPeer’\]相同** router[route\[0\]](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/route%5B1%5D,%20function%20(req,%20res,%20next) { 像是这里，实际上分割参数后，第一个就是目标(/peer)，第二个就是方法。 在 peer.js 的文件中，可以看到对map函数的调用  
        router.map(shared, {
        
            "get /": "getPeers","get /version": "version","get /get": "getPeer"
        
        }); 通过阅读源码我们可以知道，router 是对 **get /version**的分割。所以route\[0\] 是 **get**，route\[1\] 就是我们的请求的url。 那么这样的话我们可以把上面的代码进行转换 router[get](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/'/&%2339;,%20function%20(req,%20res,%20next) { `router.get` 又是什么？ 这样我们转到前面的路由定义 var router = require(‘express’).Router(); 所以说是 **express框架** 所提供的路由对象的方法。那么到网上检索之
        
        > ExpressRouter  
        > 用官方的话讲:对象的一个实例， METHOD 是一个 HTTP 请求方法， path 是服务器上的路径， callback 是当路由匹配时要执行的函数
        
        所以说，这里的代码，实际上是实现了路由地址和内容的绑定
        

* * *

    这里有个point     var router = this;这里的函数式编程就厉害了，凭空的一个 this ，实际上了解了之后，知道，这个this 就是指的**调用当前函数的对象**。        var router = new Router();        router.map(shared, {            "get /": "getPeers",            "get /version": "version",            "get /get": "getPeer"        });

### [](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#peer-js-%E8%8A%82%E7%82%B9 "peer.js 节点")peer.js 节点

#### [](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#%E8%8A%82%E7%82%B9%E8%B7%AF%E7%94%B1 "节点路由")节点路由

*   **16** 构造器 这里是JS中 Peer 的构造器。
    
        // Constructorfunction Peer(cb, scope) {    library = scope;                // 这里的scope 就是从app.js 传来的    self = this;    self.__private = privated;    privated.attachApi();    setImmediate(cb, null, self);    // 定时执行}
    

* * *

*   **25** 功能绑定 可以使用这样的方法实现保护函数。这个函数的功能从字面意义上就是绑定API。实际上就是使得我们的 http 请求对应的API，绑定到具体的返回操作。
    
        // private methodsprivated.attachApi = function () {    var router = new Router();    // 作为中间件    // 没有挂载路径的中间件，应用的每个请求都会执行该中间件    // 所以这段代码可以理解为，模块是否加载，如果没有，就返回错误    router.use(function (req, res, next) {        if (modules) return next();        res.status(500).send({success: false, error: "Blockchain is loading"});    });
    

            // 绑定关系        router.map(shared, {            "get /": "getPeers",            "get /version": "version",            "get /get": "getPeer"        });        // 怎么绕开了这个报错的中间件？        router.use(function (req, res) {            res.status(500).send({success: false, error: "API endpoint not found"});        });这里的 **express.use** 是作为中间件，这个前面就应该看看。>中间件（Middleware） 是一个函数，它可以访问请求对象（request object (req)）, 响应对象（response object (res)）, 和 web 应用中处于请求-响应循环流程中的中间件，一般被命名为 next 的变量。这里还有很重要的一点:**如果当前中间件没有终结请求-响应循环，则必须调用 next() 方法将控制权交给下一个中间件，否则请求就会挂起。**> [Express_中间件](http://www.expressjs.com.cn/guide/using-middleware.html)

* * *

*   **44** 拓展Express
    
            library.network.app.use('/api/peers', router);    library.network.app.use(function (err, req, res, next) {            if (!err) return next();            library.logger.error(req.url, err.toString());            res.status(500).send({success: false, error: err.toString()});    });};
    
    这一部分，一样的是以 use打头，前面是路径，后面是 Obj。所以这里是对上面的我们定义的 `var router = require('express').Router()` 的一个拓展。这样，以下请求，将会使用 **router这部分的功能**。
    
    *   http://ip:port/api/peers/
    *   http://ip:port/api/peers/version/
    *   http://ip:port/api/peers/get/ 后继的中间件，就是对错误情况进行处理了
        

* * *

*   **455** 合法性检测  
    这个是 z_scheme 模块中的功能，意在实现合法性检测。
    
        library.scheme.validate(query, {    type: "object",    properties: {        ip_str: {            type: "string",            minLength: 1        },        port: {            type: "integer",            minimum: 0,            maximum: 65535        }    },    required: ['ip_str', 'port']}, function (err) {
    
    可以看到，**validate** 的字面意思就是证实的意思。所以这样，可以用此，保证查询地址的合法性。 之后通过 `privated.getByFilter({` 查询路由表，这里涉及到 dblite，使用的是SQLite 数据库
    

#### [](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#%E8%8A%82%E7%82%B9%E5%AD%98%E5%82%A8 "节点存储")节点存储

    在上面的部分实现了，对于单个节点的查询功能，使得可以返回路由信息等。这在这里就会有，关于节点信息的存储。

* * *

*   **节点初始化** 由于P2P,没有中心服务器，所以各个节点之间，只能靠自己的网络发现，来寻找彼此，所以，使用互联网节点进行初始化，是很重要的事情。可以很大的提高组网速度。 在Config.js 文件中，也提供了初始化的节点列表
    
        "peers": {    "list": [        {            ip:0.0.0.0            port:7000        }    ],    "blackList": [],    "options": {        "timeout": 4000    }},
    

* * *

*   **347 写入节点** 根据函数的命名。可以看出这个是一个服务函数。是在区块链准备完成之后进行的。
    
    > **async简介**
    
        Peer.prototype.onBlockchainReady = function () {async.eachSeries(library.config.peers.list, function (peer, cb) {    library.dbLite.query("INSERT OR IGNORE INTO peers(ip, port, state, sharePort) VALUES($ip, $port, $state, $sharePort)", {        ip: ip.toLong(peer.ip),        port: peer.port,        state: 2,        sharePort: Number(true)    }, cb);}, function (err) {
    
    这里是对列表中的每个项目，进行顺序执行。执行数据库查询语句，把已知的数据插入到数据库中
    
        INSERT OR IGNORE INTO peers(ip, port, state, sharePort) VALUES($ip, $port, $state, $sharePort)
    
    这里就是插入语句，IGNORE 如果主键重复，就对其进行忽略，对于相同的列名进行插入。后面的指定是 插入内容的合法格式。这里的 **state:2** 是默认值，说明是**正常节点**。
    

    **364** 这里是使用的 bus 辅助模块，相当于模块之间的通信总线，当节点准备完毕之后，发送 **peerReady** 消息。触发了 **peerReady事件**。    library.bus.message('peerReady');

* * *

*   **374 节点更新** 这部分实现的节点数据的更新。
    
        Peer.prototype.onPeerReady = function () {    setImmediate(function nextUpdatePeerList() {    // 这里是定时执行的函数，前面见过        privated.updatePeerList(function (err) {            err && library.logger.error('updatePeerList timer', err);            setTimeout(nextUpdatePeerList, 60 * 1000);        })    });    setImmediate(function nextBanManager() {        privated.banManager(function (err) {            err && library.logger.error('banManager timer', err);            setTimeout(nextBanManager, 65 * 1000)        });    });};
    
    `setImmediate()` 立即执行预定的Callback.在I/O 实践回调之后立即触发。这里有领个，我们可以得知，第一个是循环(60s)更新节点列表，第二个是更新节点状态。 **52** 这里是上面定时执行的节点更新函数。
    
        privated.updatePeerList = function (cb) {    modules.transport.getFromRandomPeer({        api: '/list',        method: 'GET'    }, function (err, data) {
    
    这里是对`modules.transport.getFromRandomPeer()`的一次封装。翻译过来就是随机节点获取。**474** 这里是随机节点传输的实现函数：
    

        Transport.prototype.getFromRandomPeer = function (config, options, cb) {        if (typeof options == 'function') {            cb = options;            options = config;            config = {};        }        config.limit = 1;        async.retry(20, function (cb) {            modules.peer.list(config, function (err, peers) {    // 这里的函数就是list的callback                if (!err && peers.length) {                    var peer = peers[0];                    self.getFromPeer(peer, options, cb);    // 这里使用自身函数实现对其他节点的 Get请求                } else {                    return cb(err || "No peers in db");                }            });        }, function (err, results) {            cb(err, results);        });    };这里的 `async.retry` 是指对后面的函数重复 20 次。这里重复的 List 方法在Peer的定义里如下    **232**    Peer.prototype.list = function (options, cb) {        options.limit = options.limit || 100;        library.dbLite.query("select p.ip, p.port, p.state, p.os, p.sharePort, p.version from peers p " + (options.dappid ? " inner join peers_dapp pd on p.id = pd.peerId and pd.dappid = $dappid " : "") + " where p.state > 0 and p.sharePort = 1 ORDER BY RANDOM() LIMIT $limit", options, {            "ip": String,            "port": Number,            "state": Number,            "os": String,            "sharePort": Number,            "version": String        }, function (err, rows) {            cb(err, rows);        });    };这里实现了，对已知节点的数据查询，而且最大的查询数量是100个，其查询结果传入回调函数 cb。在上面的list调用中，rows，作为实参传给了 peers。如果没错误，且节点内容合法，读取 `peer[0]` (即IP地址)，对其发送 **getFromPeer** 请求！获取其他API。**518** 重点函数 getFromPeer实现了对其他节点的请求。下面是对请求结构的构造。    var req = {        url: 'http://' + ip.fromLong(peer.ip) + ':' + peer.port + url,        method: options.method,        json: true,        headers: _.extend({}, privated.headers, options.headers),        timeout: library.config.peers.options.timeout    };get函数会直接返回请求结果    return request(req, function (err, response, body) {    if (err || response.statusCode != 200) {    // 这里是对请求异常，分无法连接，和返回错误两种        library.logger.debug('Request', {            url: req.url,            statusCode: response ? response.statusCode : 'unknown',            err: err        });        if (peer) {            if (err && (err.code == "ETIMEDOUT" || err.code == "ESOCKETTIMEDOUT" || err.code == "ECONNREFUSED")) {                // 这里对于，异常节点，故障。进行删除                modules.peer.remove(peer.ip, peer.port, function (err) {                    if (!err) {                        library.logger.info('Removing peer ' + req.method + ' ' + req.url)                    }                });            } else {                // 这里是返回值异常的节点，对其状态更改ban掉                if (!options.not_ban) {                        modules.peer.state(peer.ip, peer.port, 0, 600, function (err) {                        if (!err) {                            library.logger.info('Ban 10 min ' + req.method + ' ' + req.url);                        }                    });                }            }        }        cb && cb(err || ('request status code' + response.statusCode));        return;    }这里是**核心函数**的前面的错误处理的部分，主要分两种情况，对无法连接和返回值异常的节点进行处理。前者直接进行删除，后者先ban十分钟。---**564** 在这个部分，对于请求的返回信息，进行解析，一样的使用的是`scheme.validate()`方法，其判断其格式是否如给定一样，如果解析失败，那么返回空数据。    var report = library.scheme.validate(response.headers, {        type: "object",        properties: {            os: {                type: "string",                maxLength: 64            },            port: {                type: "integer",                minimum: 1,                maximum: 65535            },            'share-port': {                type: "integer",                minimum: 0,                maximum: 1            },            version: {                type: "string",                maxLength: 11            }        },        required: ['port', 'share-port', 'version']    });    if (!report) {        return cb && cb(null, {body: body, peer: peer});    }**593** 这里剩下的就是正常的，可以被解析的数据了。先对其端口合法化进行检测，之后对比自身版本号是否相同，一切一切都OK了，那么我们就使用update进行更新        var port = response.headers.port;        if (port > 0 && port <= 65535 && response.headers['version'] == library.config.version) {            modules.peer.update({                ip: peer.ip,                port: port,                state: 2,                os: response.headers['os'],                sharePort: Number(!!response.headers['share-port']),                version: response.headers['version']            });        }        cb && cb(null, {body: body, peer: peer});至此，节点列表更新循环完毕。**382** 节点状态刷新循环。这个循环时循环的 BanManager    rivated.banManager = function (cb) {        library.dbLite.query("UPDATE peers SET state = 1, clock = null where (state = 0 and clock - $now < 0)", {now: Date.now()}, cb);    };这里就比较简单，对于超时的节点，对其状态进行刷新。

* * *

至此，一个P2P的网络构建完成。

[](https://www.diglp.xyz/2018/05/03/BC_EbookCoin_0x2/#%E5%90%8E "后")后
---------------------------------------------------------------------

虽说，通过这部分的源码理解，和对其源码设计的思考。基本上是了解了，一个基于http的P2P网络的构成。总结讲，就是对节点其他节点的列表请求来拓展自己目前的节点列表，从而一步步的构成一个P2P网络。

不过，实际上，还是有很多值得思考，和未知的地方，

1.  网络发现，因为我们不可能一开始就有多数的节点，所以网络发现感觉挺重要
2.  关于节点间通信，如果作为对等节点，基本的功能可以通过 api 实现，不过如果我是想，进行一个 点对点的通信，而不是广播请求。那么又将如何实现？