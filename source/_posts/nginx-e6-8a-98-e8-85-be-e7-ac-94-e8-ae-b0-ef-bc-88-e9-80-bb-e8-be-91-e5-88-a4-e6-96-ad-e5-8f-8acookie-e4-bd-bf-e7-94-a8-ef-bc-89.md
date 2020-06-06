---
title: Nginx 折腾笔记（逻辑判断及Cookie使用）
tags:
  - Nginx
  - OPS
url: 1036.html
id: 1036
categories:
  - 未分类
date: 2019-06-03 14:48:33
---

在博客里面自己加上了wiki，需要进行一定程度上的访问控制。预想的效果，是使用`referer`来进行来源的控制，但是出了些问题。所以就在这里记录一下。

前面遇到的问题
-------

在进行访问控制的时候，本来打算是参考防盗链的方式进行配置。配置文件如下：

    valid_referers blocked www.diglp.xyz blog.diglp.xyz ray.i124u.cf;
    if ($invalid_referer) {
        return 403;
    }

这里先熟悉一下语法，这里的是使用的blocked 的类型，当referer不属于列出的域名的时候就直接置位`$invaild_referer`。

可以很如愿的得到首页的访问，觉得完事了，但是在后面进行尝试的时候，打开二级网页的时候就直接403了。想了想，这个referer是无粘滞的。所以在第二级的时候就判断为false了。

使用Cookie
--------

在nginx里，可以直接添加响应头的内容使用`add_header`进行内容的添加。来进行cookie 的设置，在nginx里进行设置，可以对cookie来进行逻辑判断

    add_header Set-Cookie &#039;referer=blog&#039;;
    if ($http_cookie !~* &quot;referer=blog&quot;) {
         return 403;
     }

* * *

这里是使用通配符来进行匹配判断的，在nginx里面对语法的要求比较严格，没有支持`!` 的取非操作，这里使用的是 `!~*` 来进行任意个的`~`忽略大小写的不匹配。

那么怎么实现访问控制呢？
------------

**这部分的内容应该是比较私密的**，也算是设计到了后端的逻辑，不过既然看到文章了，那基本上打开也没问题了。

这里再原来的referer的控制的前提下，使用cookie进行粘滞。当满足了设置了cookie且队开始是通过blog进行referer来进行访问的用户才允许访问。

    set $flag 0;
    if ($arg_referer != &quot;blog&quot;) {
        set $flag &quot;${flag}1&quot;;
    }
    if ($http_cookie !~* &quot;referer=blog&quot;) {
        set $flag &quot;${flag}1&quot;;
    }
    #逻辑与实现方式
    if ($flag = &quot;011&quot;){
        return 403;
    }
    if ($arg_referer = &quot;blog&quot;) {
        add_header Set-Cookie &#039;referer=blog&#039;;
    }

Nginx 的GET参数
------------

在Nginx里面可以很容易的提取出Get的参数，从而可以实现代理分流。`$arg_referer`这里的全局变量，就是对应的 `?referer=xxx`的xxx

Nginx 的逻辑判断
-----------

由于在nginx里面不能很好的支持逻辑判断，所以需要进行组合才能进行，这里使用的是字符串的拼接进行。这里的`${flag}`从最开始的0开始进行拼接，如果满足条件，那么就重新赋值为`${flag}1` 这样的话，变量内容变成**01**，周而复始，就可以得出，如果两个调节都满足，那么得到的就是 **011**于是就是实现了与的关系，那么或就是两者其一，即为**01**

后
-

梦中想到的事情，起床来就可以把它实现，快乐