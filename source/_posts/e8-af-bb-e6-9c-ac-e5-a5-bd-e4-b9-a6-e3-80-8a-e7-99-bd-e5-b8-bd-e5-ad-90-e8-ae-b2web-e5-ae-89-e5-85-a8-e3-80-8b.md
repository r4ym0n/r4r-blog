---
title: 读本好书 《白帽子讲WEB安全》
url: 496.html
id: 496
categories:
  - 未分类
date: 2018-10-14 00:00:00
tags:
---

前
-

> 喜欢就去追寻
> 
> 互联网本来是安全的。自从有了研究安全的人之后，互联网就变得不安全了。

简介
--

*   书名：白帽子讲WEB安全
*   作者：吴翰清
*   ISBN：9787121160721

WEB 安全相关的入门书籍，范围很广，每个类型的威胁，都有相应的案例。

作为一本较为系统的拾遗书籍，加深自己对 WEB 安全的理解

浏览器（用户）安全
---------

### 浏览器的同源策略

如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的**源**。

浏览器自身用于保证安全的策略

> *   [浏览器的同源策略 \- MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
> *   [浏览器同源政策及其规避方法 \- ruanyf](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)

### CSRF

Cross-site request forgery,

### XSS

Cross-site scripting

### ClickJacking 点击劫持

点击劫持，拖拽劫持，经典的例子是 把球放在 海豹的头上。

作用机理是使用 看不见的 iframe

### AUTH and Session 会话劫持

Cookies被窃取，被盗取了登陆状态 ， SessionID 存在。

当把 Session 放在 URL 里面存在更大的问题，比如有个 邮件里的地址，用户点击之后，这个请求会在 referer 里面带上上一个站点的信息，很容易的泄露了sessionID。

Cookie 保持：使用一个慢速的固定请求保持 cookie 的过期时间。

> *   [常见WEB漏洞](https://zhuanlan.zhihu.com/p/26184730)

WEB服务器安全
--------

### SQL Injection

这个注入老问题了，后面补充把

### Upload 上传点

*   **00 截断**
    
    这部分是寻找上传点，并且绕过类型检测，以及正确解析。对于 php页面的上传点，如果文件名没有被修改，可以尝试 \\00 进行截断。
    
*   **MIME Sniff**
    
    如果通过对 文件头的魔数来识别类型的话，我们可以给 php 文件伪造一个 合法jpg的文件头，
    
*   **文件解析**
    
    当然，上传了一个文件之后，要对其进行解析，否则只是个图片
    
    又想到之前的 **一句话图马**
    
    *   Apache 文件解析（特性）
    
    apache 的 1.x 2.x 版本中，对文件后缀是 白名单设置 ，如果有文件文件名如下：
    
        a.php.qwe.qwe.qwe.qwe
    
    由于qwe 类型不在其白名单内，所以他不断遍历，直至第一个 php ，这样就把这个文件作为 php 进行解析了。所以 ，我可以伪造 jpg 后缀名绕过类型上传检测，之后使用该解析漏洞使得其进行执行。mine.type
    
    *   IIS6
    
    IIS 作为 win下的web 服务器，也是存在着解析漏洞，和 00 截断类似这里是 `;` 截断:
    
        a.asp;asdasd.jpg
    
    显然上面的是一个合法的 文件名，不过IIS 在进行进行的时候，就出现了截断为 `a.asp`
    
    且另外的一个漏洞是 如果可以新建文件夹， IIS 会把 `/*.asp/` 文件夹下面所有的 文件都作为 asp 文件进行执行。
    
    *   nginx
    
    Nginx 这个 官方称为特性的东西，存在于 `cgi.fix_pathinfo = 1` 这个选项中。实现这样的效果：
    
        http://www.example.com/test.jpg/test.php        #本身这个文件不存在
    
    在这里，test.jpg 将会作为 php 文件进行解析，问题存在于 nginx 的特性里面，由于是 fast-cgi 的特性导致，由于是 php 结尾的 url 所以是被分发到了 fast-cgi。其按路径进行解析，就会找到 这个jpg 文件执行了。
    

### 弱伪随机数/密码学安全

比较有意思，在后面对 加密算法 和加密模式做个总结

### Deny of Service

正如其名 拒绝服务攻击，使WEB 服务器失去原有的服务能力，比如选课 XD

#### 协议层

经典方法直接想到的 就是进行疯狂的请求这种方法就称之为 **泛洪** flood

*   SYN flood
*   UDP flood
*   UDP flood
*   ICMP flood # ping

可见 除了第一种，后面的都是完整的协议。使用最低的资源，得到最高的性能，这个当然是 DOS 所需要的。这里的 SYN 便是，SYN 很熟悉的出现在 网络中的 TCP 协议中，`syn syn/ack ack` 这样的一个三次握手的协议，SYN 不是一个完整的协议，所以其对发起攻击的要求更低了。

SYN flood 算的上是利用了 TPC 协议的漏洞。我们发起了 SYN 包之后，主机进行应答，我们不做任何回复，主机将进行重复的 3~5 次应答，直至 连接超时被施放。所以，可见 不应答的方法得到了更多的收益（资源占用）。所以SYN 泛洪是很常用的 DOS 的方法。对应方法，可以对 SYN 的发起地址 分配 Cookies 统计气质访问频率， 丢弃过多的请求。

#### 应用层 = CC 攻击

CC 攻击 起源 与对 绿盟的 反DOS 的设备（collaoasar黑洞）的挑战（challenge）所以简并 CC

其攻击原理不同于上面的网络层次。这里是对应用层次的请求进行泛洪，尽可能的消耗服务器的资源，比如 ：

数据库的增删查改。HTTP

在应用层产生的攻击其主要的解决方案，是配置服务器，对连接数进行限制，或者进行请求的分发

#### Slowloris 攻击

这个也是应用层次的攻击， 可以说是利用了 HTTP 协议的漏洞。原理也好理解，我们的 请求中设置 Keep-Alive。而且我们发送畸形的请求头。正常的请求头是 以 \\r\\n\\r\\n 结束的。我们使用只有一个 \\r\\n 的请求头，这样服务器认为 没有有结束或者接收完整，便保持连接，客户端，再以缓慢的速度发送不完整数据，来保持连接。这样的进行数个 连接的保持，就可以很快占用了所有的连接数 导致 DOS。

#### Server Limie DOS

这个比较有意思，实际上进行DOS 的对象是用户本身，不是服务器。利用了 HTTP 报头的长度限制这一属性。HTTP 中对 请求头的长度限制是 8192Bytes 如果我们使用恶意脚本，在用户的Cookies里面，添加大量的无用信息。导致 请求被 远程服务器 丢弃。

> 这个可以直接在 浏览器 的Cookies 进行修改，
> 
>     document.cookie = 'exp1=' + 'a' * 8192

PHP安全
-----

PHP 由于其天生的 特性，所以在WEB 开发上得到了广泛的应用，PHP 作为动态 ，弱类型的语言在方便的同时的确带来了 许多的隐患。

### 文件包含漏洞

这里的文件包含，在实际上指的是 代码注入， 使得用户代码在远程主机上非法执行。

#### 本地文件包含 LFI

Local File inclusion 正如 python 的 import 一样，PHP 中用于 包含文件使用的 函数是：

*   include() / include_once()
*   require() / require_once()

在使用上面的函数对文件本身进行引入的时候，解释器会自动的执行引入文件所包含代码。

    <? php
        include($_GET[test])
    ?>

上面是示例代码，使用

    curl "http://localhost:8000/test.php?test=../a.php“ -v 

这样会导致远程主机上的a.php 的非法执行，

**TIPS:** PHP 内河使用 C 艰辛编写所以在处理字符串的时候,会出现 00 截断,用于绕过不少的文件格式判断。

* * *

这里提到了一个很有意思的**远程文件包含**的方法，WEB 服务器的日志注入。基于 SessionID 的注入。我们本地进行的恶意 Session内容 的构建 （如果可以），之后提交请求，服务器会将我们的 Session的内容进行保存。所以我们可以远程的 include 包含我们注入代码的 session 文件，从而导致了远程的代码包含的复现。

> *   [Web安全实战系列：文件包含漏洞](http://www.freebuf.com/articles/web/182280.html)

* * *

    <? php
        <?php eval($_POST[test]);?>
    ?>

如果有拿过站的，就一定知道，功夫再高也怕菜刀这句话。中国菜刀这样的东西，带来了一句话木马的浪潮，这里的一句话木马，就是典型的代码注入，使用这一句话，我们可以实现在远程主机上进行的代码执行

    curl "http://localhost:8000/test.php?test=<?php phpinfo();?>“ -v 

> *   [那些强悍的PHP一句话后门](http://www.freebuf.com/articles/web/9396.html)

Web Server 安全
-------------

这部分的问题，出在Web 服务器的 漏洞 ， 或者说 ummmmm 特性。

Apache 的问题，多数出现在核心模块里面，都是由于其他的模块可能引起的漏洞，但是存在 root 运行的问题，一道被getshell 就是root。Nginx 在不断进化 漏洞还是不少。JBoss 有8080 的默认后台 同 Tomcat 有 8080 的默认管理

后
-

发现了同样的一篇 WEB 安全的，

> *   [读白帽子讲WEB安全](https://www.jianshu.com/p/12385b84c37b)

随手 PICK 一下