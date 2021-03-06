---
title: 2FA 双因素认证
url: 594.html
id: 594
date: 2018-02-01 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/02/01/2FA/#%E5%89%8D "前")前
--------------------------------------------------------

登陆在现在的网络生活中是少不了, 登陆的本质就是认证(authentication) 通过认证, 来证明你是就你, 不是别人冒充,顶替的(知道密码的委托人除外) 昨天注册了一个网站, 和正常一样, 账号密码, 可是最后, 有了一项 2FA 绑定, 需要谷歌验证器. 打开后, 一种似曾相识的感觉飘来, 没错,就是很久很久以前的qq令牌,依稀记得这个是S40的jar了. 小时候就像过, 这个怎么这么神奇, 又不联网,怎么能都知道密码是多少呢.所以,当日常百科,来个所以然

[](https://www.diglp.xyz/2018/02/01/2FA/#2FA%E7%9A%84%E6%A6%82%E5%BF%B5 "2FA的概念")2FA的概念
---------------------------------------------------------------------------------------

2FA 是(two-fact authentication)的缩写, 下面是百科的定义

> 2FA是基于时间、历史长度、实物（信用卡、SMS手机、令牌、指纹）等自然变量结合一定的加密算法组合出一组动态密码，一般每60秒刷新一次。不容易被获取和破解，相对安全。

其实2FA 广义上说是一种认证方法, 比如去银行取钱, 银行卡, 和它的密码都需要,你才能取出钱来, 这个就是一种广义上的2FA.再者也有网上银行的U盾(虽然真的很鸡肋), 也是实现2FA, 它里面有着证书.

[](https://www.diglp.xyz/2018/02/01/2FA/#OTP%E6%A6%82%E5%BF%B5 "OTP概念")OTP概念
----------------------------------------------------------------------------

OTP(one-time password), 一次一密, 如果有点密码学的基础, 就知道一次一密, 理论上是绝对安全的. 其实在现实生活中, OTP也是存在的比如最常用的短信验证码, 邮箱验证码. 这些都是算的上是一次一密, 只不过,显得不是那么专业. (现在我国的短信,还是使用的2G频段的明文SMS, GSM嗅探很容易就做到)

### [](https://www.diglp.xyz/2018/02/01/2FA/#%E8%AE%A4%E8%AF%81%E5%99%A8%E7%9A%84%E5%8E%9F%E7%90%86 "认证器的原理")认证器的原理

使用认证器的时候, 有以下几个步骤

*   会从网站上获取一个秘钥. (这样服务器和手机都会有这个串)
*   成功导入之后, 会有一个6位的数字, 而且是动态的
*   在服务器输入当前的数字,以便验证

其中基本的原理, 就是这个`hash(pri-key, time)`  
服务器进行验证

    if(input() == hash(pri-key, time))    ...else    ...

[](https://www.diglp.xyz/2018/02/01/2FA/#OTP%E5%8E%9F%E7%90%86 "OTP原理")OTP原理
----------------------------------------------------------------------------

上面的伪代码看上去是那么回事, 可是时间虽说是统一变量, 不同设备之间当然会有不同的时间差, 那么如何保证这个问题. 的确, 这个是时间, 是服务器和客户端直接获取的本地量, 如果直接进行hash, 那么肯定会因为时间细小差别导致认证失败. 我们一般见到, 验证码是有一定的有效时间的, 这个也就是精妙之处的存在 假定有一个函数`gettime()`, 可以获得时间戳(unix-timestamp)(1970-1-1)到现在的秒数

    hash_para = gettime() / 30    //这样进行除运算,得到的是到现在有多少个三十秒

这样,就完美的解决了这个问题, 保证了密码的动态, 和准确 (设备时间, 不能喝网络时间错的太远…还记得那时候 的qq令牌, 时间不对, 密码不对, 当时差点炸掉)

[](https://www.diglp.xyz/2018/02/01/2FA/#%E7%BC%96%E7%A8%8B%E5%AE%9E%E7%8E%B0 "编程实现")编程实现
-----------------------------------------------------------------------------------------

[](https://www.diglp.xyz/2018/02/01/2FA/#%E5%90%8E%E8%AE%B0 "后记")后记
-------------------------------------------------------------------

这种基于时间的2FA, 算是十分安全的了, 比那一成不变的密码(admin),好上太多了.  
不过,如果在传输过程中泄漏了pri-key, 那也是完了…