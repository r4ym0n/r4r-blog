---
title: Django WEB 开发入门
url: 512.html
id: 512
categories:
  - 未分类
date: 2018-08-10 00:00:00
tags:
---

Foreword
--------

古语有言：勤能补拙。这个也算是自己的一周的总结吧。把一周的学到的，和自己的一些感悟做个记录吧。

虽然，是很基础的东西，不过一切都是从基础开始的对吧？

sigh...

Context
-------

### 理论

#### Django 是什么？

> Python下有许多款不同的 Web 框架。Django是重量级选手中最有代表性的一位。

Django 是 python 的一个 web的框架。在框架中整合了，web 服务中的所涉及的功能。

*   提供web服务
*   后台数据获取
*   请求响应

前端什么的真的是不怎么懂，自然会有些，很傻的问题。不过实际上也是困扰了自己很久的问题：

> [大（中）型网站的后端会用到C/C++吗](https://bbs.csdn.net/topics/390268641)
> 
> [Web 开发中，Python 和 PHP 哪个有优势？](https://bbs.csdn.net/topics/390268641)

* * *

*   PHP 本身是一种编程语言，是和 python 一个层级，.py 需要 python runtime 。.php 需要 php 的runtime。两个东西都可以很简单的使用 `python x.py` 和 `php x.php` 的运行起来
    
*   当 python 使用 flask/Django 开始web 开发的时候了。和 PHP 的web框架(ThinkPHP) 作用在一个层级了。**不准确的讲**每次访问，都会启动一个进程，对这个脚本内的内容进行运行， 这里的运行是在服务器端的。对我的的请求进行解析，比如url 的路由路径， 请求方式等进行响应。
    
*   这时候仍然是在服务器端， 进行数据库的增删查改。 把操作(得到的数据，传递到前端（这里是前后端的数据交互方式，后面写）) `index.php?category=x`
    
*   前端进行数据获取，由 js 脚本，实现静态html 的显示刷新

#### WEB 请求的流程

PHP开发Web应用时所以的请求需要指向具体的入口文件。WebServer是一个内容分发者，他接受用户的请求后，如果是请求的是css、js等静态文件，WebServer会找到这个文件，然后发送给浏览器；如果请求的是/index.php，根据配置文件，WebServer知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。

> 内容引用自 [SF - WEB请求的流程](https://segmentfault.com/a/1190000008838804)

当然，这里是的标题是 PHP 的web 请求流程，其实感觉删掉前面的几个字，也是差不多的。

这里 ，开始列举名词 。 nginx，Apache， php， python ，django ， thinkphp。清楚了不少吧。

慢慢的对这个过程还是有点清楚的了，有个东西 叫做 [**CGI** 通用网关接口](https://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E7%BD%91%E5%85%B3%E6%8E%A5%E5%8F%A3)

#### MVC 是什么？

到底什么才是标准的 MVC 这个问题，到现在作者也没有一个**确切的**答案；不过多个框架以及书籍对 MVC 的理解有一点是完全相同的，也就是它们都将整个应用分成 Model、View 和 Controller 三个部分，而这些组成部分其实也有着几乎相同的职责。

*   视图：管理作为位图展示到屏幕上的图形和文字输出；
*   控制器：翻译用户的输入并依照用户的输入操作模型和视图；
*   模型：管理应用的行为和数据，响应数据请求（经常来自视图）和更新状态的指令（经常来自控制器）；

> 上述内容出自 [Applications Programming in Smalltalk-80: How to use Model-View-Controller (MVC)](http://www.dgp.toronto.edu/~dwigdor/teaching/csc2524/2012_F/papers/mvc.pdf) 一文。

* * *

Django的MTV模式本质上和MVC是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同，Django的MTV分别是值：

*   M 代表模型（Model）：负责业务对象和数据库的关系映射(ORM)。 # 这里是和数据库交互的地方
*   T 代表模板 (Template)：负责如何把页面展示给用户(html)。# 这个就是我们的 html 页面
*   V 代表视图（View）：负责业务逻辑，并在适当时候调用Model和Template # 实现 前端请求响应的地方

（上面👆，的内容在 工程文件加里，都明确的使用各个文件分开了）

除了以上三层之外，还需要一个URL分发器，它的作用是将一个个URL的页面请求分发给不同的View处理，View再调用相应的Model和Template

\[![img](http://assets.tianmaying.com/md-image/f94775f85c94e999bc685f0b11d3c461)

> 以上引用自 [csdn - YangHeng816](https://blog.csdn.net/YangHeng816/article/details/52213983)

* * *

这里描述的，是 Django 这个web 框架的 结构。model， template，view。这几个文件的功能如上面所描述的一样， 数据库交互， 页面渲染， 请求响应。

#### Nginx 关系？

> **Nginx**（发音同engine x）是一个异步框架的 [Web服务器](https://zh.wikipedia.org/wiki/%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)，也可以用作[反向代理](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)，[负载平衡器](https://zh.wikipedia.org/wiki/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1) 和 [HTTP缓存](https://zh.wikipedia.org/wiki/HTTP%E7%BC%93%E5%AD%98)。该软件由 [Igor Sysoev](https://zh.wikipedia.org/wiki/%E4%BC%8A%E6%88%88%E7%88%BE%C2%B7%E8%B3%BD%E7%B4%A2%E8%80%B6%E5%A4%AB)创建，并于2004年首次公开发布。[\[6\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-Mobily-6) 同名公司成立于2011年，以提供支持。[\[7\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-D-7)
> 
> 一个web服务器面对的是外部世界。它能直接从文件系统提供文件 (HTML, 图像， CSS等等)。然而，它无法 **直接** 与Django应用通信；它需要借助**一些工具**的帮助，这些东西会运行运用，接收来自web客户端（例如浏览器）的请求，然后返回响应。
> 
> **一个Web服务器网关接口（Web Server Gateway Interface） - WSGI - 就是干这活的。 [WSGI](http://wsgi.org/) 是一种Python标准。**

WebServer 也就是 WEB 服务器。前面有说过，这个是是作为内容分发者，其自动的动态的数据请求和静态的资源请求，这些东西，分发到不同的部分进行 运行，静态内容由服务器自己获取。剩下的动态东西，就交给了WSGI 这东西，执行 Py 的内容，运行后得到结果。

我们完整的组件栈看起来将是这样的:

    the web client <-> the web server <-> the socket <-> uwsgi <-> Django

这里的 server 通过 socket 与 uwsgi 进行进程间的同学，是的，其可以很好的执行该执行的脚本。且返回结果

* * *

在这里使用 Django 进行 开发的时候，实际上只是个 `manager runserver` 命令，就可以跑起来一个页面了对吧？

实际上，这样滴的确已经成功的构建了web 服务。 而且可以实现基本所有的正常功能了。

那么 nginx 的作用？ 当然是更专业的提供一个更强的性能保证。

> 内容部分引用自 [uWSGI - manual - zh](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html#id3)

### 实践

#### 第一个工程

这里是使用的 PyCharm 这个IDE 新建的 工程了，工程以及模板，差不多是一键生成的状态。目录树的结构差不多如下：

    HelloWorld/
    |-- HelloWorld
    |   |-- __init__.py
    |   |-- settings.py
    |   |-- urls.py
    |   |-- view.py
    |   `-- wsgi.py
    |-- manage.py
    `-- templates
        `-- hello.html、

直接执行 `manage.py runserver 8000` 就可以很方便的在 本机的 8000 跑起了web 的服务了 。

#### 路由配置

原有的 实例工程已经给我们配置好了基本的所需， 这里的是这些文件的基本配置。

在 **url.py** 这个文件中的是，这个工程的路由配置

    from helloworld import views
    
    urlpatterns = [
        url(r'^index$', views.index),
        url(r'^get_data$', views.get_data),
        url(r'test$', views.test)
    ]

这里的 index ，get_data， test ，就是 代表着不同的路由。实际上，我们可以理解成：对我们访问的 URL 进行解析的方式。

前面的是 路由路径， 后面的是这个 路由路径所对应的视图 其函数定义：

    def url(regex, view, kwargs=None, name=None):

> 在 Django 中，网页和其他内容都是从视图派生而来。每一个视图表现为一个简单的 Python 函数（

* * *

这里用到了通配符，这个自己只是了解，具体的 有些还是记不清的，这里总结一下：

*   ^ $ 这里的两个符号，前者代表匹配开头， 后者代表匹配的结束
    
    . 匹配除了换行之外的 任何符号一次
    
    \* 多次匹配， 代表**多次**任意字符
    
    \[\] 这里的方括号内容，就是知己手动指定的匹配范围
    
        例如 [xyz]  \[c-f] \[0-9] 
    
    ? 单次匹配一个**或者0**个字符
    
    \+ 和\* 类似多次匹配任意字符， 但是 **不匹配0个**
    

> [shell中的正则表达式和通配符 - Fengya](https://www.jianshu.com/p/49d5ee46de47)

#### 视图配置

在 django里面 ，视图实际上就是一个函数

这里的就是 我们的第一个hello 所对应的视图

    def index(req):
        return render(req, './demo.html')

这里的一个 render 函数，也就是渲染器。把我们本地的 html 进行渲染之后，就可以返回到前端。

这样我们就可以看到了我们的页面。

看 render 函数的定义

    def render(request, template_name, context=None, content_type=None, status=None, using=None):

后面是有缺省参数的， 这里的 context ，我们可以在 **html文件** 里面加入变量。（当然这个不是 html 的标准语法，是django的）类似于这样：

    <h1>{ { hello }}</h1>

这样在我们的页面中就存在了一个参数。 这样我可以指定我们的 渲染参数。

    context = {
        "hello": 'hahaha'
    }
    return render(req, './demo.html', context)

然后进行网站访问之后， 可以 f12 看到：

    <h1>hahaha</h1>

* * *

当然，这里的视图函数不一定，不许是个html 的页面， 我们可以基础的发过去一个请求或者一段文字

    return http.HttpResponse("rayd")

或者 json 内容：

    return http.HttpResponse(json.dumps(data), content_type="application/json")

#### 目录配置

再回到之前的 目录结构。因为之前不了解这个问题，所以导致了页面的的很多元素是无法找到的问题

    HelloWorld/
    |-- HelloWorld
    |   |-- __init__.py
    |   |-- settings.py
    |   |-- urls.py
    |   |-- view.py
    |   `-- wsgi.py
    |-- manage.py
    |-- static
    |   |-- css
    |   `-- js
    `-- templates
        `-- hello.html、

在之前的目录结构的基础上这里发生了一些小小的变化。

添加了一个 static的文件夹,什么是静态文件：这里是标准的定义

> 静态文件是指 网站中的 js, css, 图片，视频等文件

* * *

所以，在先前的模板文件中的，静态文件的引用位置是需要发生改变 的。

这里是原文件中的引用位置:

      <!-- Bootstrap core CSS-->
      <link href="vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
      <!-- Custom fonts for this template-->
      <link href="vendor/font-awesome/css/font-awesome.min.css" rel="stylesheet" type="text/css">
      <!-- Page level plugin CSS-->
      <link href="vendor/datatables/dataTables.bootstrap4.css" rel="stylesheet">
      <!-- Custom styles for this template-->
      <link href="css/sb-admin.css" rel="stylesheet">

**注意：** /\* 指的是相对根 ， \* 指的是相对当前文件

在 Django 的框架中，其指定了静态文件夹

    STATIC_URL = '/static/'
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]

所以我们的 引用目录变成：

        <!-- Bootstrap Core CSS -->
        <link href="/static/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
        <!-- MetisMenu CSS -->
        <link href="/static/vendor/metisMenu/metisMenu.min.css" rel="stylesheet">
        <!-- Custom CSS -->
        <link href="/static/dist/css/sb-admin-2.css" rel="stylesheet">
        <!-- Morris Charts CSS -->
        <link href="/static/vendor/morrisjs/morris.css" rel="stylesheet">
        <!-- Custom Fonts -->
        <link href="/static/vendor/font-awesome/css/font-awesome.min.css" rel="stylesheet" type="text/css">

#### 请求响应

在视图函数中，可以很容易的 ，进行对于不同请求的响应

    def get_data(req):
        req.encoding = 'utf-8'
        if req.GET:
            pass
        if req.POST:
            pass
        # ...

#### 前后端的数据交互

既然是WEB 开发， 这里的东西就是重要到不能再重要的 地方了。

> **前端**（英语：**front-end**）和**后端**（英语：**back-end**）是描述进程开始和结束的通用词汇。前端作用于采集输入信息，后端进行处理。计算机程序的[界面](https://zh.wikipedia.org/wiki/%E7%95%8C%E9%9D%A2)样式，视觉呈现属于前端。

用户看见的，既是前端， 数据来的地方就是后端。

* * *

关于 前后端的数据交互这里有篇很好的文章，也是同学发来的，作为收藏。

> [前后端数据交互方法](https://blog.csdn.net/qfgg3/article/details/51780760)

这里的应用主要是应用了 AJAX ，在前端发起的异步请求，以获得后端的 json数据。

后端代码：

    def test(request):
        return http.HttpResponse("heko")
    

前端代码：

    <button id="btn_add" type="button" class="btn btn-default" onclick=hello()>
        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>新增
    </button>
    var hello = function () {
        alert("hello");
    
        var data = $.ajax({url:"/test", async:false});
        alert(data.responseText)
        // 这里后面就可以实现对html页面进行改变
        // 比如，指定 元素的ID
    };
    

这样就可以实现 前后端 的数据通信。当然，对于 ajax 的请求。是可以向后台传递参数的。后台对我们的参数进行响应和解析

#### WSGI

这里虽然目前还是没有用上，不过想看看，学习一下了，以后上线之后 向 nginx 搬移是必经 的。

*   \[WSGI是什么\](#Nginx 关系？)

这里先贴出教程的链接了

*   [Django Nginx+uwsgi 安装配置](http://www.runoob.com/django/django-nginx-uwsgi.html)
*   [uWSGI - manual - zh](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html#id3)

后
-

很高兴，能有人看到这段话。没有因为内容的索然无味，直接close 掉。

是作为自己的笔记了，因为的确，对于 web 的接触比较少。所以可以算是从零开始了。

记载着自己的一些 感想和思路，和比较重要的点。

**勤能补拙**