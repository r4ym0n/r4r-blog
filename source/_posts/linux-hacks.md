---
title: Linux Hacks
tags:
  - Linux
  - misc
  - OPS
url: 874.html
id: 874
categories:
  - OP之路
date: 2019-04-29 19:10:41
---

前
-

这里主要收集一些不是那么常用，但是一用起来就很麻烦的一些小trick

单IP限制每分钟连接数
-----------

使用 `iptable` 来进行单IP的每分钟最大的连接次数限制

    iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 5 --name SSH --rsource -j DROP 
    iptables -A INPUT -p tcp --dport 22 -m recent --set --name SSH --rsource -j ACCEPT 

Screen 会话共享
-----------

    # A会话
    screen -S test
    # B 会话
    scree -x

可以实现两个会话共享一个 screen

sysrq-trigger
-------------

一个“危险的”调试入口

    # 立即重新启动计算机
    echo "b" > /proc/sysrq-trigger
    # 立即关闭计算机
    echo "o" > /proc/sysrq-trigger
    # 导出内存分配的信息 （可以用/var/log/message 查看）
    echo "m" > /proc/sysrq-trigger
    # 导出当前CPU寄存器信息和标志位的信息
    echo "p" > /proc/sysrq-trigger
    # 导出线程状态信息
    echo "t" > /proc/sysrq-trigger
    # 故意让系统崩溃
    echo "c" > /proc/sysrq-trigger
    # 立即重新挂载所有的文件系统 
    echo "s" > /proc/sysrq-trigger
    # 立即重新挂载所有的文件系统为只读
    echo "u" > /proc/sysrq-trigger

修改上一条命令
-------

如果修改上一条命令，但是太难调整光标怎么办？试试`fc`吧

快速100%
------

    cat /dev/zero | md5sum

终端命令发送
------

    write root # 用户组
    wrtie root pts/1 # 指定会话

    wall "test" # 通知所有人

快速查看当前硬件架构
----------

    arch
    x86_64

进程优先级的设置
--------

top 的第三列`PR` 表示当前进程的优先级，越小，那么资源会被越优先进行调度。`priority = PR+NI`

进程优先级体现在 `nice值` 的设置上面，使用 top命令可以看到第四列就是的`NI`就是指得是我们的进程优先级调整参数，为负数提升优先级，反之降低优先级，使用`r`，输入PID来进行进程nice 的重新指定。

*   或者， `nice` eg：
    
        nice [OPTION] [COMMAND [ARG]...]
        nice -5 ls
    
*   或者使用 `renice`
    
        renice [-n] priority [-gpu] identifier...
        renice -5 -p 1234
    

网络路由指定
------

    route add -net 192.168.62.0 netmask 255.255.255.0 gw 192.168.1.1
    route del -net 192.168.122.0 netmask 255.255.255.0

指定网络号和其指定的路由网关

iptables 的备份和恢复
---------------

     iptables-save > /opt/iptables.txt  ###备份所有表的规则
     iptables-restore < /opt/iptables.txt   # 对iptables'的规则进行恢复

原来iptables重启，没有保存的规则会丢失。。。。

命令实现简单的发包
---------

Linux的天然的网络特性可以实现简单的IP发包

    # UDP
    echo "test" > /dev/udp/127.0.0.1/8888
    # TCP
    echo "test" > /dev/tcp/127.0.0.1/8080

简单的命令实现 基本的传输层的发包操作，简单易得

已删除单空间未释放
---------

文件已删除，但是空间没有释放

    losf| grep deleted

因为使用这个文件的服务还没有停止，文件句柄是无法释放的。

sar 查看性能历史统计
------------

查看历史的性能数据，CPU，IO，network

    sar -p  # CPU
    sar -d  # disk
    sar -b  # 所有块设备
    sar -n 

**TIPs** 使用`-d` 参数得到的是 `dev8-240`，怎么和 `sd*`来进行对应呢？ A：这里的 8 和 16 指的是主设备号和次设备号，一一对应即可

    [rms@123 /]$ ls -l /dev/sd*
    brw-rw---- 1 root disk 8,  0 Sep 22 14:34 /dev/sda
    brw-rw---- 1 root disk 8,  1 Sep 22 14:34 /dev/sda1
    brw-rw---- 1 root disk 8,  2 Sep 22 14:34 /dev/sda2
    brw-rw---- 1 root disk 8,  3 Sep 22 14:34 /dev/sda3
    brw-rw---- 1 root disk 8,  4 Sep 22 14:34 /dev/sda4
    brw-rw---- 1 root disk 8, 16 Jul 18 15:21 /dev/sdb
    brw-rw---- 1 root disk 8, 17 Jul 18 15:21 /dev/sdb1

GZ的流解压
------

在查看nginx的日志的时候，其默认的配置是对历史的日志文件进行 gzip 的压缩。查看日志的时候，需要指定流解压

    cat /var/log/nginx/error.*.gz | gzip -d

查看电池状态
------

    $ upower -i /org/freedesktop/UPower/devices/battery_BAT0

benchmark工具
-----------

     sudo curl -fsSL https://ilemonrain.com/download/shell/LemonBench.sh | sudo bash -s fast

很全面的benchmark的脚本，测试整机的性能。

journalctl 分级查看日志
-----------------

*   u 指定服务
*   p 指定等级（1-8） -f follow滚动

使用strace来排查问题
-------------

使用strace 来进行进程的性能排查

    strace -c  -T -tt -o out.txt -p pid

该命令会收集进程的系统调用，排查性能问题。

解决自动补全
------

有些时候用起来bash发现竟然不能自动补全，一想这功能应该随时带的有啊。实际上这个是bash 一个插件，有些时候是没有被安装的。所以这里给出一键安装的命令，安装 `bash—completion` 即可：

    yum install bash-completion

查看当前的动态库的链接情况
-------------

    ldconfig -p | grep pcap
    
    #ldconfig -p:打印当前缓存所保存的所有库的名字。
    #grep pcap:用管道符解析libpcap.so是否已加入缓存中。

使用上面的方法可以很好的查看所有的已经配置了的连接库

* * *

同样的使用`ldd`可以查看二进制文件所需要的链接库

    ldd a.out

SHELL中实现串口通信
------------

liunx 里面一切皆文件，所以之间对串口的操作是十分方便的。先需要对串口进行配置，使用`stty`

在LINUX下首先需要检测串行口驱动是否正常，而串行口设备一般为_/dev/ttyS_,如果是USB转串行口的，则为_/dev/ttyUSB_，

* * *

使用命令来进行串口的参数配置，

    sudo stty -F /dev/ttyUSB0 raw speed 9600 cs8 -parenb -cstopb cread clocal
    
    # -F /dev/ttyUSB0     使用-F可以指定设备名
    # raw                 这是一组设置的组合体,详情可man stty
    # speed 9600          指定波特率
    # cs8                 指定数据位
    # -parenb             无奇偶效检
    # -cstopb             1停止位(如果为2位，则为cstopb)
    # cread               允许输入能够接收
    # clocal              禁止调制解调器的控制信号(不明)

* * *

    sudo cat /dev/ttyUSB0
    sudo echo asd > /dev/ttyS0
    # 来发送十六进制的数据
    sudo echo  -en "\x80\x123\x8F" > /dev/ttyS0

双网卡时候添加路由
---------

在树莓派的双网卡配置的时候，很容易出现上不了网的情况，因为流量都到另一个接口了，所以添加正确的路由即可：

    sudo route add default gw 192.168.0.1  #192.168.0.1默认的网关的地址

使用 watch 命令
-----------

`watch` 这个工具可以帮助我们定时的执行一个命令并且返回结果，就像我们的top一样，watch 的命令是普适的，可以实现对与命令的执行并且进行内容的刷新。

screen 来保存会话以及 shell 的分屏
------------------------

*   `screen -S <>` 开始一个新的会话
*   `screen -r <>` 恢复一个会话

统计开机时的服务加载时间
------------

用于开机缓慢的时候进行的排除，是之前看书里遇到的。这里有用到了就再记录一下。用来分析启动时间的`systemd-analyze`

    systemd-analyze blame 列出所有的服务启动时间
    systemd-analyze plot > img.svg # 打印出矢量图
    journalctl -u -f [servicename] 查看服务的日志。

使用SSH的反向隧道
----------

很多时候需要内网穿透，用到Ngork 或者 frp ，殊不知使用SSH也可以很容易的实现端口的隧道。

    ssh -p 22 -qngfNT -R 6766:localhost:22 usera@a.site

使用dpkg来查看所有的包大小并排序
------------------

    dpkg-query -W --showformat='${Installed-Size} ${Package} ${Status}\n'|grep -v deinstall|sort -n|awk '{print $1" "$2}'

解决so依赖问题
--------

有些时候一些应用跑不起来，由于缺运行库的原因，一般使用 `ldd` 来查看二进制文件所需要的so依赖是否满足，如果显示有 not found，那么一般就是直接安装对于的lib即可。

但是在一些情况下，so的版本各异，所以有的时候需要自己来手动的添加对应的版本的链接。使用`sudo ldconfig -v`来刷新以及来列出目前的所有的so文件。来根据需要进行排查，并可以使用 `|less` 来找到so文件的所在路径。

递归计算目录的所有文件的哈希
--------------

对当前目录的所有文件得到一个统一的哈希，用于进行整体的文件校验：

    find path/to/folder -type f -print0 | sort -z | xargs -0 sha1sum | sha1sum