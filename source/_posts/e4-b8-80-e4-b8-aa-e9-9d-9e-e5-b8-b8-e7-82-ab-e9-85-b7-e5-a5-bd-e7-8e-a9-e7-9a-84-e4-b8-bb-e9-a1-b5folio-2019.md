---
title: 一个非常炫酷好玩的主页folio-2019
tags:
  - nodejs
  - Web
url: 1336.html
id: 1336
categories:
  - 玩点什么
date: 2019-10-27 23:06:03
---

偶遇了一个很好玩的东西，宇哥非常炫酷的个人主页。 和一般的点点点不一样，这个是能玩的。 示例页面：

> [https://bruno-simon.com](https://bruno-simon.com)

> 页面源码是开源的，可以直接在Ghub上找到； [https://github.com/brunosimon/folio-2019](https://github.com/brunosimon/folio-2019)

在pick这个小玩具的同事，这里也简要的记录一下部署记录

部署
--

首先安装 nodejs 的环境，由于使用的是centos的系统，默认的node源的版本比较低，所以先需要升级

    curl -sL https://rpm.nodesource.com/setup_10.x | bash -

来安装 node10 的源。

    git clone https://github.com/brunosimon/folio-2019.git npm install -g parcel-bundler npm i # dev 运行 npm run dev  # 编译在dist目录，可使用 pm2 等运行 npm run build

他山之石
----

本站也部署了一个，欢迎来玩

> [www.12ms.xyz/folio/](https://www.12ms.xyz/folio/)

nodejs的web开发
------------

这里首先推荐一篇文章：

> [Node](http://ourjs.com/detail/529ca5950cb6498814000005)[初学者入门，一本全面的](http://ourjs.com/detail/529ca5950cb6498814000005)[NodeJS](http://ourjs.com/detail/529ca5950cb6498814000005)[教程](http://ourjs.com/detail/529ca5950cb6498814000005) 不过，这些毕竟都是前端技术，尽管当想要增强页面的时候，使用jQuery总让你觉得很爽，但到最后，你顶多是个JavaScript用户，而非JavaScript开发者。

这里在文章中选取一句话，可能是自己对这个 JS用户执迷了太深，导致一致没有反应过来 JS的web开发到底是怎么一回事。 也是巧，遇见这个项目一下子明白不少。这个不就是和PHP一个道理吗，笑。

* * *

一样吗？不一样，事实上在folio这个项目里面，node的作用和php可没有一点关系。 项目使用的是最终的 build出来的dist包，这个 包里面全都是静态文件，和动态 没有一点关系。 node的web开发的思想是吧各个可以使用的组件作为一个包，安装即用。使用 nodejs 来进行编写，实现最终的效果，完成开发过程之后，直接使用build进行构建，就得到了静态的源码。比裸写html 不知道效率提高了多少个等级。