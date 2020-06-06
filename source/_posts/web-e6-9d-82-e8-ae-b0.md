---
title: Web 杂记
url: 510.html
id: 510
categories:
  - 未分类
date: 2018-08-20 00:00:00
tags:
---

Foreword
--------

这个计划叫做 TC 计划， 没错，叫做 Tab Closing。

每周的的现象是什么？ 浏览器的标签爆炸。

总是有些好的内容，舍不得关闭。可是有没有再去看，所以，借机整理一下了~

### Context

### 关于 BootStrap

这个东西算是贯串了整个的开发周期的东西。什么是 BootStrap ？简单来讲就是一个前端的 UI 框架。

> Bootstrap是一组用于网站和网络应用程序开发的开源前端框架，包括HTML、CSS及JavaScript的框架，提供字体排印、窗体、按钮、导航及其他各种组件及Javascript扩展，旨在使动态网页和Web应用的开发更加容易。

简单来讲，其实就是可以很容易的是的我们的页面变得更加 的好看。我们的 html 的基本组件，显得十分单一啦。通过bootstrap的框架，使得我们可以使用其中预设的一些组件和样式，这样使得我们的ui的设计显得更加的容易和美观。使用期预设的组件，显得都是那么美观了。

* * *

这里 对资料进行一些整理。

*   [bootstrap - runboo](http://www.runoob.com/bootstrap/bootstrap-tutorial.html)
*   [bootstrap-slider](https://www.npmjs.com/package/bootstrap-slider)
*   [bootstrap-table](https://www.npmjs.com/package/bootstrap-table)
*   [bootstrap-switch](https://www.npmjs.com/package/bootstrap-switch)

### 关于 JQuery

很久之前， 有在图书馆 见到一本书 《锋利的jquery》，当时因为不清楚，也没有被 好奇心驱使 。所以，最近在知道这个 是什么东西。(笑， 看到 query 之前一直以为和数据库有关系)

这里搬出 百科内容。

> jQuery是一套跨浏览器的JavaScript库，简化HTML与JavaScript之间的操作。

*   [JavaScript - W3School](http://www.w3school.com.cn/js/index.asp)
*   [Jquery - W3School](http://www.w3school.com.cn/jquery/index.asp)
*   [AJAX - W3School](http://www.w3school.com.cn/ajax/index.asp)
*   [世界上最大的WEB开发者网站 - W3schools](https://www.w3schools.com/)

### Js 里的 (异步)回调

感觉 js 这种函数式 编程的精妙的结构，在下面的简简单单的三行代码，就体现出来。

    $("#profileTab").append(function (index, html) {
        return $('#tab_profile').html().replaceAll('tmpvar', id);
    });

* * *

JS 里面实现一个回调的结构：

    function add(arg1, arg2, callback){
        var num = arg1 + arg2;
        callback(num);　　//传递结果
    }
    
    add(10, 20, function(num){
       console.log("Callback called! Num: " + num); 
    });　　

之前，对这些东西，并不是十分理解，深圳觉得这样的结构是不是有些多余。这样的东西，完全可以使用一个顺序的正常的结构实现啊

    function add(arg1, arg2, callback){
        var num = arg1 + arg2;
        print(num);　　//传递结果
    }

这样就可以实现上面的功能了，而且简单很多，然而实际上，并不是这样的了。

回想起来，如果在之前的应用程序开发的过程中。如果把一个冗长的过程放在主线程里，会怎样？

比如在主线程里面进行网络请求，常常这个过程，会有一段响应时间，这样就直接导致了我们的主线程的阻塞。所以，Android 的 sdk 里面，已经禁止了在主线程里面进行网络请求。否则直接导致了 app 的卡顿了。所以这时候，要是传统的应用，理应会打开了一个 线程，来进行网络操作。

不过这个问题，在JS 里面有了另外一种的优雅的解决方案，就是这里括号里面的字，异步。在我们使用了回调结构之后。会发现，add 函数并不会阻塞原运行的过程。而是继续的向下运行。

> [彻底理解javascript的回调函数](https://www.cnblogs.com/moltboy/archive/2013/04/24/3040213.html)

### JS 的事件绑定

关于 JS 里面的页面事件 绑定，用其来，的确是很直观

    $("p").click(function(){
      $(this).hide();
    });

这样的很简单的语句，就是实现了在文中的所有的 `<p>` 元素绑定了点击事件。

可心中有了个疑问，这个事件是怎么进行绑定的呢？直到遇到了这样的问题：

    function tab_trOnclick(){
        var trs=$("#block_set_table tbody tr");
        for(var i=1;i<trs.length;i++){
            $(trs[i]).on("dblclick",function(){
                console.log(i);
            });
        }
    }

这里的代码，之前的我是这样理解的：

首先这段代码**得到运行**，得到每一行的标签`<tr>` ， 之后对每个元素的对于的 点击事件函数进行初始化。然后，对每一行的点击，之后控制台里会打印出对应的行的行号。第一行**里面的 i 是 1**，那么点击第一行就输出 1。以此类推。

不过事实上运行的 结果是：点击每一行都只是输出一个相同的数字。而且其数字等于 tr 的个数，也就是我们的行数。

why? 这里就出现问题了。前面的内容讲到：回调？异步？所以事实上的过程就清楚了。是这样的：

for 循环执行了 $(x).on() , 这个函数是个回调函数。事实上是一个有实践进行触发的函数。当没有时间的时候，这个回调，不会继续的执行下去。然而在主线程里面，这个 for 还在不断的给其他的 tr 来绑定事件。这个 **i** 的值还在不断**加1** 直到所有绑定全部完成。 (注意！这里的回单函数没有返回，所以这里的函数的作用域中的变量值一样的没有被释放)。 当我们进行点击的时候，**那时的 i 就是 tr 的个数** 没错了。

所以，这种情况下，需要在回调中获得原元素中的 id 信息，从而得到序号。最终得到的代码如下：

      function tab_trOnclick(){
            var trs=$("#block_set_table tbody tr");
            for(var i=1;i<trs.length;i++){
                $(trs[i]).on("dblclick",function(){
                    var tr=$(this);
                    var reg = new RegExp("^[0-9]*$");
                    rowNum = (tr.context.id).match("[0-9]*$")[0]
                    console.log(rowNum)
                });
            }
        }

### 元素复用列表追加

有个需要用到多次的元素， 如果直接使用 html

进行硬编码，得到 n*xx 的代码体验的确是不怎么优雅。 这里使用 JS 对空标签进行 append ：

    <div id="test"> </div>
    
    <script>
     for(var i=1;i<trs.length;i++){
        $('#test').append('<h4>hello</h4>');
     };
    </script>

### 遇到的问题

*   动态添加元素的事件绑定
    
    > [jquery动态添加元素无法触发绑定事件的解决方案](https://blog.csdn.net/u011277123/article/details/53330887)
    
*   元素重复初始化

后
-

慢慢的对 web 开发的整体结构有了了解，现在有了基础，就要开始高速适用了。