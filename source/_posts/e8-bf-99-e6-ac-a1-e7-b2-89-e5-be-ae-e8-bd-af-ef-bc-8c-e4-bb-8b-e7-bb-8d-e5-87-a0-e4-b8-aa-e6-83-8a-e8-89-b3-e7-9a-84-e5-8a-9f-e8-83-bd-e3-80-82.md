---
title: 这次粉微软，介绍几个惊艳的功能。
tags:
  - Bash
  - Linux
  - misc
url: 983.html
id: 983
categories:
  - 未分类
date: 2019-06-09 22:55:32
---

**这次我粉微软**，很久没有怎么关注资讯了，突然给了个推送,介绍了MS最新的terminal,终于可以一改前貌.是一个重大革新，这里先放一个图，可以看看。界面是相当的漂亮，支持多标签，ssh，etc

![file](https://i.loli.net/2019/06/09/5cfccf19bb98084403.png)

新的终端 terminal
-------------

这里给出 MS 的项目的地址，[`microsoft/terminal`](https://github.com/microsoft/terminal)。由于并没有给出预编译版本，所以自己使用需要自行编译，存在一定的使用难度，所以这里就在其他地方找了预编译版本直接给出来。下载安装即可。

> [下载地址](https://guetbucket-1252029944.cos.ap-shenzhen-fsi.myqcloud.com/WindowsTerminal_x86_x64_arm64.rar)

先安装证书，之后再进行应用的安装。目前项目蓬勃发展中，期待。

新的eage
------

eage说实话，看起来很符合自己的感觉，简洁简单。但是！自己的独创内核的性能真的不咋地，记得eage刚刚出的时候，简直不能被称为一款产品。就是一款bug。

* * *

但是，MS又发了大招，推出了一款全新的eage，一样的外观不一样的内核，叫做`eage insider`，这里给出[项目地址](https://www.microsoftedgeinsider.com/en-us/)，使用的是Chromium 内核，所以大多数的Chrome的插件是可以直接被使用的。目前的dev的版本更新频率在一周每次，总体的感觉还是很不错的。目前就在使用，后面打算作为主力的浏览器。（最总要的是有了多个平台，可以使用outlook的账号实现一站式的登陆，绕过了大陆的google无法登陆的问题）这里给一张预览图：

![file](https://i.loli.net/2019/06/09/5cfd1aba6bfb978991.png)

WSL Win 下的Liunx
---------------

在win下使用Linux必须要跑一个沉重的虚拟机吗？有时候得开个vbox之后开个SSHd，再由Xshell连上去，实属太麻烦了。所以这里就推荐又一大神器**WSL(Install Windows Subsystem for Linux)**，这样就可以再WIN上完美的跑起一个LINUX子系统了。 ![file](https://i.loli.net/2019/06/09/5cfd1cb499f9582659.png)

Linux的原生体验，真的很Nice，这样的话在WIN上也可以又一个很好的Linux环境，就不会因为环境问题，苦苦的挣扎在线上的服务器和虚拟机之间了。

### 给右键菜单添加BashHere

打开注册表编辑器，新建项`HKEY_CLASSES_ROOT\Directory\Background\shell\Bash Here`，再项目内再新建项`command`，其默认键内容为`"C:\Windows\System32\bash.exe"`，新建Icon来指定图标即可

![file](https://i.loli.net/2019/06/01/5cf256ac0c05468211.png)