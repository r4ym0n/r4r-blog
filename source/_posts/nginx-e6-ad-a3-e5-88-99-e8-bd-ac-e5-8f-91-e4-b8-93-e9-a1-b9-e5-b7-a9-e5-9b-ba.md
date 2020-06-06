---
title: Nginx 正则转发专项巩固
tags:
  - Nginx
url: 1277.html
id: 1277
categories:
  - OP之路
date: 2019-10-07 22:17:19
---

nginx 作为路由转发的服务器，那么关于路由的匹配规则和匹配顺序当然是是十分重要的一个部分了。这一篇帖子就针对着location的正则进行梳理。

几种正则方式
------

nginx 中的 location 的基本语法如下：

    location [=|~|~*|^~] /uri/ { … }

匹配优先级如下：

*   精确匹配 = /xxx
*   前缀匹配 ^～ /xxx
*   正则匹配 ～ /xxx
*   正则忽略大小写匹配 ～\* /xxx
*   普通正则匹配 /xxx
*   最终匹配 /

在存在上的不同类型的同时出现的话，会按照优先等级来进行匹配。比如：

    location = / {
      empty_gif;
    }
    location ~/xxx {
      empty_gif;
    }

规则顺序
----

Nginx 的匹配的规则有顺序吗？如果是存在着优先级的情况下，是遵循优先级的匹配顺序的。

**但是**如果有在同等优先级的情况下是根据规则的顺序来进行匹配的。比如

    location ～ /xxx {
      empty_gif;
    }
    location ~/xxx/123 {
    
    }

这样会导致 123 的块就不会被匹配到。

* * *

另外，还有一点，location 里面的表达式是否需要 / 来结尾，在nginx 的规则里`/aaa`是文件 `/aaa/`是目录。

> 根据约定，URL尾部的/表示目录，没有/表示文件。所以访问/some-dir/时，服务器会自动去该目录下找对应的默认文件。如果访问/some-dir的话，服务器会先去找some-dir文件，找不到的话会将some-dir当成目录，重定向到/some-dir/

所以在没有 `/` 的时候，是作为一个文件进行匹配的，所以就进行文件的搜索，匹配`/aaa` 在没有文件匹配的时候，会自动的被重定向到目录的匹配即 `/aaa/`

当在匹配的规则里，使用了`/aaa/` 这种形式，我们请求的 `/aaa` 将无法被匹配。

URL重写
-----

首先 先回顾一下前面一看就懂的文章：

> [Nginx 折腾笔记（SSL配置以及路由重写）](https://blog.12ms.xyz/2019/02/08/nginx-%e6%8a%98%e8%85%be%e7%ac%94%e8%ae%b0%ef%bc%88ssl%e9%85%8d%e7%bd%ae%e4%bb%a5%e5%8f%8a%e8%b7%af%e7%94%b1%e9%87%8d%e5%86%99%ef%bc%89/)

rewrite 后面的参数：

*   last ：表示完成rewrite，浏览器地址栏URL地址不变
*   break：本条规则匹配完成后，终止匹配，不再匹配后面的规则，浏览器地址栏URL地址不变

这两句怎么理解呢？看下面的的例子： 在nginx配置 proxypass的有一个很重要的配置：

    rewrite /xxx/(.*) /$1 break;

很多时候还会被写成这种形式：

    rewrite ^/(.*) /$1 break;

作用是改变proxy的url 的地址，起到了设置 basepath的作用。比如我有哥URL是 `aaa:5000/asd` 但是我希望使用nginx的反向代理来访问他`aaa/vvv/asd`。那么配置怎么写？简单？

    location ~ /vvv/ {
        proxy_pass http://aaa:5000;
    }

看似比较合理，但是问题来了，这样的话`/vvv/asd` 这一串东西都被传给了后端，于是乎后端返回了404的问题。这里就需要用到`rewrite`了。而且是`break` 这种rewrite。正如上面的介绍的，不在进行其他匹配。 所以，这个write就不会在内部或者外部产生重定向，其作用就是在内部改变 url。就加上这一句`rewrite /vvv/(.*) /$1 break;` 把我们的URL的前一级给拆掉。得到后面的部分在进行proxy，这样就得到了我们的真实内容。