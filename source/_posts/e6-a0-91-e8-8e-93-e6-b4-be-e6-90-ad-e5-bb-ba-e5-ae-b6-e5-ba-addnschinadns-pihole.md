---
title: 树莓派搭建家庭DNS(ChinaDNS & PiHole)
tags:
  - DNS
  - rpi
  - 树莓派
url: 1284.html
id: 1284
categories:
  - 玩点什么
date: 2019-10-03 01:32:45
---

手头有了个闲下来的树梅派，于是乎就想着，能不能玩点什么。再于是乎就有了遮盖新的分类，记点简单易玩的折腾的笔记。

这里就用树莓派来做一个家里的DNS来去广告啊，防止无良运营商的DNS污染之类的。

简介
--

Pi-hole:

> The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

ChinaDNS:

> ChinaDNS automatically queries local DNS servers to resolve Chinese domains and queries foreign DNS servers to resolve foreign domains. It is smart enough to work only with a Chinese IP range file, which doesn't change often.

* * *

这里的两个都出DNS服务，一个是根据解析规则对DNS进行过过滤，另一个是根据最新的解析列表来对被污染的域名进行正确的解析。

所以这里两个都被使用到，来作为家庭的DNS服务的提供方案，实现去广告以及防止DNS污染。

部署
--

由于是开源的组件，所以其安装过程都有很好的 guide，pihole 官方也是提供了 一键安装的方式。

    curl -sSL https://install.pi-hole.net | bash

安装过程一路绿灯即可，可能会出现的问题是其FTL组件的在github上的拉去出现github的网络问题的问题，通过改配置的方式来处理。

默认监听在本机的 53 端口，通过对路由器的配置定内网的 DNS 服务器为搭建的服务的主机的IP即可。过程中没什么大坑。服务的本身提供了dashborad，作为基础的功能的展示。

* * *

[ChinaDNS](https://github.com/shadowsocks/ChinaDNS) 也是github上的一个项目，配置过程也十分简单，编译之后使用 supervisor进行管理即可：

      ./configure && make
      src/chinadns -m -c chnroute.txt
      /home/qspace/chinadns-1.3.2/chinadns -m -c chnroute.txt -p 54

在54端口跑起来服务，使用 ns来指定不同端口来进行解析测试。

    rms@rm-TP:~$ nslookup google.com 192.168.66.188 -port=54
    rms@rm-TP:~$ nslookup google.com 192.168.66.188 -port=53

由于需要对封锁的列表进行更新配置cron任务：

    * 0 * * cd /home/qspace/chinadns-1.3.2 &&  curl 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | grep ipv4 | grep CN | awk -F\| '{ printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > chnroute.txt

后
-

至此，部署完毕，可能心理上的页面访问速度快上了那么一点吧。