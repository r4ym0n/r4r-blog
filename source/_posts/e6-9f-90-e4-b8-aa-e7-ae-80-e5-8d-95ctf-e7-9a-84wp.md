---
title: 某个简单CTF的WP
url: 500.html
id: 500
categories:
  - 未分类
date: 2018-10-03 00:00:00
tags:
---

前
-

得找到自己的一点点爱好吧，不然闲暇真是 boring 。

* * *

晚上突然的心血来潮，找了个CTF 的网站，手头没什么工具，所以就怎么简单怎么来啦。

*   amigos2001在2018-10-01 01:51:08解出了misc-4
*   amigos2001在2018-10-01 01:45:53解出了misc-2
*   amigos2001在2018-10-01 01:36:43解出了misc-1
*   amigos2001在2018-10-01 01:29:45解出了code-1
*   amigos2001在2018-10-01 00:47:28解出了web-2
*   amigos2001在2018-10-01 00:37:38解出了web-1

WEB
---

### SHA1 碰撞？

    <?php
    include('flag.php');
    if(isset($_GET[a]) && isset($_GET[b]))
    {
        $c = $_GET[a];
        $d = $_GET[b];
        if($c != $d && sha1($c)===sha1($d))
        {
            echo 'you got it<br>';
            echo $flag;
        }
        else
        {
            echo 'try again<br>';
            show_source(__FILE__);
        }
    }
    else
    {
        show_source(__FILE__);
    }

* * *

TIPs:

*   isset() 是判断变量是否被赋值的函数
*   $_GET\[\] php的特性全局变量
*   sha1() 常用的哈希算法 输出 320 (40*8bit) 位
*   show_source(\_\_FILE\_\_);

* * *

上面的就是题目，这个代码逻辑，不难看出。最后是需要一个同时满足：

    $c != $d && sha1($c)===sha1($d)

这样的条件的一个输入。如果乍一看，发现要 两个输入的变量不同，然而这个变量对应的哈希又要是一样的，这个是碰撞 ？ Collusion？显然不会这么直接吧？

KEY：**PHP 中 哈希函数的问题** （PHP 作为动态弱类型的语言，导致安全性的问题）

> 如果 GET 参数中设置 `name[]=a`，那么 `$_GET['name'] = [a]`，php 会把 `[]=a` 当成数组传入， `$_GET` 会自动对参数调用 `urldecode`。
> 
> `$_POST` 同样存在此漏洞，提交的表单数据，`user[]=admin`，`$_POST['user']` 得到的是 `['admin']` 是一个数组。

这里可以控制，get的参数作为一个数组，专利由于 php 的hash 函数的性质，或者说是漏洞。对于 php中的hash 函数，输入一个 数组，得到的这时候，函数将会有一个警告，而且会返回 null，这里就有趣了，

    sha1($c)===sha1($d) => null===null => true

可见，我们很好的，就可以实现这里的条件啦。所以这里我们构造url

    www.example.com/web?a[]=1&b[]=2

这里可见，ab都为数组，且 \[1\] <> \[2\] , 后面有满足了 null === null 的条件，所以我们 got flag。

### SHA1 碰撞 !?

    <?php
    include('flag.php');
    if(isset($_GET[a]) && isset($_GET[b]))
    {
        $c = (string)$_GET[a];
        $d = (string)$_GET[b];
        if($c != $d && sha1($c)==sha1($d))
        {
            echo 'you got it<br>';
            echo $flag;
        }
        else
        {
            echo 'try again<br>';
            show_source(__FILE__);
        }
    }
    else
    {
        show_source(__FILE__);
    }

> PHP 和 JS 一样，作为一个弱类型的语言，虽然的确给开发者提供了不少的编码自由，淡出其中的安全问题，也是的确的不能忽视。

* * *

这次，职业敏感的，直接把目光集中在这个条件语句上，这次，和上次有了些差别。

    $c = (string)$_GET[a];
    $d = (string)$_GET[b];
    if($c != $d && sha1($c)==sha1($d))

这里，使用了强制的类型转换，显然，这里把我们的传入数组的路子是封死了。不过，有转机的地方在于这个等于号。

    $c != $d && sha1($c)==sha1($d)

前面的内容不同没有改变，后面的hash 相等，从 === 变成了 == 。

wiki 之

> [浅析JavaScript和PHP中三个等号（===）和两个等号（==）的区别](https://blog.csdn.net/ithomer/article/details/5855240)
> 
> *   == 两边值类型不同的时候，要先进行类型转换，再比较。
> *   === 不做类型转换，类型不同的一定不等。

* * *

简单来讲，== 只是要求值相等，而 === 要求类型和值。可见机会来了。有趣的是，这个要从科学计数法说起。

在科学计数法里面，`1e2 == 1*10^2`。 这样的语法在 C / 以及 PHP 等语言里面都是存在的。`1e2 == 10e1`那么

`0e123` 和 `0e456` 呢? 显然可见，在数值上讲，0 乘以的任意次幂，结果当然是 0 的。综上，== 会进行自动的类型转换。得到这样的推论。

    '0e12123341231231' == '0e612371973712937'

现在，回去看题啦，我们要解决的问题是

    sha1($c)==sha1($d)

一看便有了思路吧！对的，找到符号相同的结构的 hash 就好啦！是 `0e\d{38}` 类型的就好了！所以，我就天真的，天真的，天真了！！！

如下：

    import re
    import hashlib as h
    for i in range(1,100000):
        hs = h.sha1(str(i).encode()).hexdigest()
        a = re.compile('0e\d{38}').match(hs)
        if a:
            print(a,i)

这个概率当然是相当低的，不过自己还突然有了学习 OpenCL 的冲动。。。

* * *

最后，的方案当然是求助Google了，这里找到了 magic Hashs。里面列举了，多个满足 `0e\d{38}`的这个条件。

> [Magic Hashs](https://www.whitehatsec.com/blog/magic-hashes/)

这里找到了这样的两个符合要求的输入

    10932435112
    aaroZmOk
    
    >>> hashlib.sha1('10932435112').hexdigest()
    '0e07766915004133176347055865026311692244'
    >>> hashlib.sha1('aaroZmOk').hexdigest()
    '0e66507019969427134894567494305185566735'

### SHA1 · 真碰撞

    <?php
    include('flag.php');
    if(isset($_GET[a]) && isset($_GET[b]))
    {
        $c = (string)$_GET[a];
        $d = (string)$_GET[b];
        if($c != $d && sha1($c)===sha1($d))
        {
            echo 'you got it<br>';
            echo $flag;
        }
        else
        {
            echo 'try again<br>';
            show_source(__FILE__);
        }
    }
    else
    {
        show_source(__FILE__);
    }

到了这里了，就发现了一些问题了，首先前面的强制类型转阻止了使用数组返回 null 的思路。可是后面是hash 比较，这里使用的是 `===` ，严格等于， 需要类型和其值相等，且不会进行类型转换。

感觉这次，是真的真的要进行碰撞了，在网上进行搜索

> *   [Announcing the first SHA1 collision -- Google Security Blog](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html)

可见，Google 的确已经发现了第一个的 sha1 哈希碰撞

* * *

对，后面的内容进行继续的探索，也很容易的发现了类似的 ctf 题目，有说是站姿巨人的肩膀上，所以这里学习，和记录了，链接在此 。

> *   [关于SHA1碰撞——比较两个binary的不同之处](https://blog.csdn.net/caiqiiqi/article/details/68953730)

    a=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01sF%DC%91f%B6%7E%11%8F%02%9A%B6%21%B2V%0F%F9%CAg%CC%A8%C7%F8%5B%A8Ly%03%0C%2B%3D%E2%18%F8m%B3%A9%09%01%D5%DFE%C1O%26%FE%DF%B3%DC8%E9j%C2/%E7%BDr%8F%0EE%BC%E0F%D2%3CW%0F%EB%14%13%98%BBU.%F5%A0%A8%2B%E31%FE%A4%807%B8%B5%D7%1F%0E3.%DF%93%AC5%00%EBM%DC%0D%EC%C1%A8dy%0Cx%2Cv%21V%60%DD0%97%91%D0k%D0%AF%3F%98%CD%A4%BCF%29%B1&
    b=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01%7FF%DC%93%A6%B6%7E%01%3B%02%9A%AA%1D%B2V%0BE%CAg%D6%88%C7%F8K%8CLy%1F%E0%2B%3D%F6%14%F8m%B1i%09%01%C5kE%C1S%0A%FE%DF%B7%608%E9rr/%E7%ADr%8F%0EI%04%E0F%C20W%0F%E9%D4%13%98%AB%E1.%F5%BC%94%2B%E35B%A4%80-%98%B5%D7%0F%2A3.%C3%7F%AC5%14%E7M%DC%0F%2C%C1%A8t%CD%0Cx0Z%21Vda0%97%89%60k%D0%BF%3F%98%CD%A8%04F%29%A1

得到了类似以上面的参数，进行提交，也的确得到了 flag。于是这个倒是水水的过了，最重要的是，看到了sha1 的碰撞的这件事。MD5 在 03/2005 已经被发现碰撞了。

CODE
----

### CODE 1

这个也是比较简单的编码题目。题目如下

    ^q|l^7xm]\RpRnVj][9o\6Rlg6Z}jUAA
    
    听说是移位+base64

题目给出了明确的暗示，是位移加上 base64 , b64 这个加密的特点就是末尾的 ==

> 关于 == 的来源，这里 pick 一个好的文章
> 
> [Base64 笔记 -- 阮一峰](http://www.ruanyifeng.com/blog/2008/06/base64.html)

那么位移的思路，就是把 AA 移成 ==， 看一看 ascii 的顺序

    >>> ord('A')
    65
    >>> ord('=')
    61

所以我们的思路是向下位移， 接下来上脚本：

    a = "^q|l^7xm]\RpRnVj][9o\6Rlg6Z}jUAA"
    c = ""
    
    for i in range(10):     # 为了验证思路多试几次
        for x in a:
            try:
                d = chr(ord(x) - i)
            except:
                pass
            c += d
        print(c)
        c = ""

代码跑完之后 得到这样的东西：

    ^q|l^7xm]\RpRnVj][9o═Rlg6Z}jUAA
    ]p{k]6wl\[QoQmUi\Z8n║Qkf5Y|iT@@
    \ozj\5vk[ZPnPlTh[Y7m╝Pje4X{hS??
    [nyi[4ujZYOmOkSgZX6l╚Oid3WzgR>>
    ZmxhZ3tiYXNlNjRfYW5k╗Nhc2VyfQ==
    YlwgY2shXWMkMiQeXV4j╔Mgb1UxeP<<
    XkvfX1rgWVLjLhPdWU3i Lfa0TwdO;;
    WjueW0qfVUKiKgOcVT2hhKe`/SvcN::
    VitdV/peUTJhJfNbUS1ggJd_.RubM99
    UhscU.odTSIgIeMaTR0ffIc^-QtaL88
    

显然我们找到了 == 的条件， 解码：

    ### Then
    b64 = "ZmxhZ3tiYXNlNjRfYW5k╗Nhc2VyfQ=="
    print(base64.b64decode(b64))
    

解释器抛异常， 仔细一看混进去了一个奇怪的东西 `‘╗’`base64 出来都应该是 az09 才是。

尝试删除，替换无果后，回到原字符串， 找到导致这个符号的位置

    a = "^q|l^7xm]\RpRnVj][9o\6Rlg6Z}jUAA"
                             ^
    a = "^q|l^7xm]\RpRnVj][9o\\6Rlg6Z}jUAA"
    

随手拍脑袋，加个 `\` 再次解码 得到 flag；

MISC
----

### Misc 1

这个题，简单类型，看到了 flag 几个字母，找找规律。

    finnsl_kc_at_e0ghf_k{iei}
    

本想着一个个数的，数着数着就乱了，算了写脚本吧：

    a ="finnsl_kc_at_e0ghf_k{iei}"
    b=""
    for x in range(5):
        for i in range(5):
            if 5*i < len(a):
                b += a[5*i+x]
    print(b)
    

后
-

难得闲暇的时间，可以去玩一下，自己喜欢的东西，希望以后，坚持一下这个爱好吧，不然每天过的太空洞。

参考
--

> *   [https://www.itcodemonkey.com/article/2185.html](https://www.itcodemonkey.com/article/2185.html)
> *   [https://blog.csdn.net/u013943420/article/details/75733175](https://blog.csdn.net/u013943420/article/details/75733175)
> *   [https://www.ctftools.com/down/](https://www.ctftools.com/down/)
> *   [http://www.freebuf.com/articles/web/129607.html](http://www.freebuf.com/articles/web/129607.html)
> *   [https://ask.helplib.com/php/post_1218190](https://ask.helplib.com/php/post_1218190)
> *   [](https://getpocket.com/a/read/1904045592)[https://getpocket.com/a/read/1904045592](https://getpocket.com/a/read/1904045592)