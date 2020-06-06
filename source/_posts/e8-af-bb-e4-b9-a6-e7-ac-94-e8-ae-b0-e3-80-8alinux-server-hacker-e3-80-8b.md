---
title: 读书笔记《Linux Server Hacker》
tags:
  - Book
  - Linux
  - OPS
url: 817.html
id: 817
categories:
  - 读本好书
date: 2019-04-24 00:49:27
---

简介
--

*   书名:《Linux Server Hacker》
*   作者: Rob·Flickenger
*   ISBN: 9787302071693

书里面的Hack，那时候是一个很cool 的词，再现在更应该被理解为 hackable。可能是风格问题吧，这本书拿起来随手翻翻，内容很是精彩。

* * *

文章以一篇 [《如何成为一名黑客》](https://link.zhihu.com/?target=http%3A//translations.readthedocs.org/en/latest/hacker_howto.html)，这篇文章也是相当的经典。

1.  世界上有很多待解决的迷人的人问题
2.  任何事情没有必要重复解决
3.  厌倦和苦干都是大忌
4.  自由至上
5.  态度并不能代表能力

保持激情，善假于物，消息共享，自动化，自由至上，敏锐的思维。

文章来自于 《what is a hacker》，在前言的部分有提到《New Hacker's Dictionary》以及著名的《Cathedral and the Bazaar》，这两本书要看看了。

* * *

书里有很多的，很Hack 的方法，看，看到之后，受益匪浅。

服务器基础
-----

### 绕过控制台登陆

在忘了密码的这个情况下，是相当有用的方法。**传统的方法是重启进入单用户模式**，在启动的时候使用组合键`Ctrl+Alt+Delete` 之后，进入 **LILO（或者GRUB）** 的命令行中，使用命令 `linux single` 来传递给内核参数，进入 root shell。

* * *

但是在一般安全性较高的系统里面，如 商用 Redhat 里面是需要 root密码。这里就可以通过修改 **LILO（GRUB）** 的启动参数。修改内容如下 `linux init=/bin/bash`。指定不启动 init 而直接启动 bash。这样就可以进入 bash 而不需要密码了。

不过问题在于，由于没有 init 的运行，所以文件系统没有经过检查，默认的都会被设置为只读的类型。所以需要对文件系统就进行重新的挂载，使用命令 `mount -o remount,rw /`来进行操作。

此外，在关机的时候，是无法使用常规方式的，因为没有 init 进程的存在，所以是无法使用 `init 0` 来进行正常关机的。所以这里需要把文件系统挂载只读，之后放心的关机。`mount -o remount,ro /`

![file](https://i.loli.net/2019/04/23/5cbea32dda514.png)

### 常用的引导参数

在GRUB的启动参数中，有很多可以可用参数，进行设置。

参数

说明

single

单用户模式启动

root=

设置被挂载为/的设备

console=

可以设置内核使用串口控制台。

ro

使/分区为只读

init=/bin/bash rw

指定启动的

### 使用 init 来持续创建可执行后台

由于书里面的东西比较老了已经，现在常常使用的是 注册服务的形式了， 在书中写道，`/etc/inittab` 里面来对一个进程继续注册。使用 `kill -HUP 1` 来向init进程发信号，使得其重载配置任务。

### 交换标准输入输出

这个小技巧是经常用到的，比如 `2>&1` 就是将标准错误重定向到 标准输出。那么这里如何去进行交互呢，一样的这里需要一个中间变量，

    command 3&gt;&amp;2 2&gt;&amp;1 1&gt;&amp;3 | ...

### 更机智的命令行操作

    grep "file" error_log | awk '{print $13}' | sort | uniq | less
    
    for x in <code>seq 10</code>; do
        echo $x
        echo $x|tr -d '1'
        echo $x | xargs echo
    done

### ext2/ext3 系统保护文件

设置 root 对无法删除的文件，实现系统的文件保护。比如：

    cp /dev/null tmp.txt

这样创建的的文件无法被删除。

* * *

通过 `lsattr` 的命令来查看文件属性，通过 `chattr` 来了改变文件附加属性，`chmod` 是改变文件标准属性。使用 `chattr +i <file>` 来实现写文件的系统保护位。

### 属性Shell的环境变量使得bash更加舒适

*   PS1 是默认的系统提示符
*   PROMPT_COMMAND 是在bash开始交互之前显示的内容
*   export IGNOREEOF=2 忽略连续的两个^D
*   export PATH=$PATH:~/bin 实现PATH的拓展

### 使用 sudo 使工作更高效

编辑 `/usr/sbin/visudo` 来管理sodo 的用户列表，使用 使用 `sudo -u` 来执行其他用户的操作，比如启动 web 服务器。

### 查看每个文件的空间占用

    alias ducks=&#039;du -cks * | sort -rn| head -11&#039;

### 好玩的 /proc

在前面看内核的时候指定，proc 是linux 的给出了来的系统的调试接口，里面有着各种各样的好玩的东西，不过谨慎操作，很容易可以 down 掉系统。

* * *

在 proc 里面的数字对应着PID，，文件和其他部分是内核运行的驱动程序，计数器等。eg:`cat version` 就是查看内核的版本。`ls -l kcore` 的大小就是我们的内核的

* * *

在这里 面探索每个进程的数据，是比较有有趣的。

    ➜  ~ sudo ls /proc/11636 -l
    total 0
    dr-xr-xr-x  2 root root 0 Apr 23 09:45 attr
    -r--------  1 root root 0 Apr 23 09:45 auxv
    -r--r--r--  1 root root 0 Mar 26 13:46 cgroup
    --w-------  1 root root 0 Apr 23 09:45 clear_refs
    -r--r--r--  1 root root 0 Mar 26 13:46 cmdline
    -rw-r--r--  1 root root 0 Mar 26 13:46 comm
    -rw-r--r--  1 root root 0 Apr 23 09:45 coredump_filter
    -r--r--r--  1 root root 0 Apr 23 09:45 cpuset
    lrwxrwxrwx  1 root root 0 Apr 23 09:45 cwd -> /
    -r--------  1 root root 0 Apr 23 09:45 environ
    lrwxrwxrwx  1 root root 0 Mar 26 13:46 exe -> /usr/sbin/NetworkManager
    ...

这里面有三个比较有趣的软链，`cwd`，`exe`，`root` 分别是 ：

*   进程的当前目录
*   当前进程的可执行文件
*   指的是该进程用户的 root目录

`cmdline`，`environ` 指的是：

*   最开始调用的时候的命令行
*   和进程的环境变量，

### procps 符号化的进程控制

使用名称而不是 PID 来向进程发送信号。如果经常使用 `ps awux | grep` 这种命令来查找杀死的PID，那么就需要考虑其他的方法。

最著名的工具包是 **procps**，有

*   `skill`，名称，用户，PID，向进程发送信号
*   `pkill`，
*   `pgrep`，输出名称相同的进程PID
*   `vmstat` 显示虚拟内存的信息，和CPU信息。

### 清理门户

在用户离开之后来清理门户。对旧用户锁定账户。

    passwd -l luser
    
    chsh -s /bin/true luser

上面的两种方式的锁定账户的方法，一个是锁定，一个是是其立即返回登出。

版本控制
----

这里解释了 RCS 和 CVS 着两种工具，不过现在有更好的 git 来实现版本控制了，这里主要看看内容。

备份
--

### 使用tar进行打包备份

使用 tar 和 ssh 来进行备份传输，使用bash 的流命令来实现。

    tar zcvf - /var/home | ssh admin &quot;cat &gt; backup&quot;
    #甚至直接可以写到磁盘中
    tar zcvf - /var/named/data | ssh admin &quot;cat &gt; /dev/sda&quot;

### rsync 实现备份

    rsync -ave ssh bcnu:/home/ftp/pub/ /home/ftp/pub/

这样实现源端到本地的文件同步。**当两个系统之间的差异很小的时候，rsync将快很多**

网络
--

### 十秒钟使用 NAT 来伪装 IP

在网关上使用 IP 伪装技术，来实现一级NAT 的保护，使用 IPtable 来实现。

    echo 1 &gt; /proc/sys/net/ipv4/ip_forward
    iptables -t nat -A POSTROUTING -o $EXT_IFACE -j MASQUERADE

`$EXT_IFACE` 这里是自己定义的外部网络接口

### iptable 的提示和技巧

### rinetd

这里是一个常用的端口转发的工具，实现端口的转发

> 使非本地服务看上去是来自本地端口

### IP隧道

和 VPN 类似，但是不同。通过 IP隧道协议有 ipip 和 GRE。

监控
--

### 监控分级

使用fifo来实现，配置如下：

    mkfifo -m 0664 /var/log/debug
    # 修改 syslog.conf
    *.debug | /var/log/debug

### watch 监控信息

使用 ps 来看运行了哪些进程，如果需要重复的操作的话，这里就要使用 watch 来进行了，使用方法如下：

    ➜  ~ watch 'ps'
    
    Every 2.0s: ps                                                                                     linaro-alip: Tue Apr 23 14:42:06 2019
    
      PID TTY          TIME CMD
    20816 pts/0    00:00:00 sudo
    20820 pts/0    00:00:01 zsh
    23065 pts/0    00:00:00 watch
    23066 pts/0    00:00:00 watch
    23067 pts/0    00:00:00 sh
    23068 pts/0    00:00:00 ps

执行之后每两秒对内容进行刷新。

### 查看端口对应进程

使用命令 `netstat -lnp`来查看每个开端口的进程。

### lsof 查看打开的文件和套接字

linux 一切皆文件，所以我们可以这样来实现losf来查看文件和网络等。

*   当我们尝试 umount 一个设备的时候得到了busy。就可以之间使用 `losf <file>` 来查看占用该文件或者该文件夹的进程。
*   使用 `lsof -p <PID>` 查看此进程打开的所有的文件。 或者使用 `lsof -c <process_name>`
*   `lsof -i:80` 查看80端口的进程

后面的话
----

> 刻苦的工作与奉献精神会导演一场狂热的演出，而不是一键苦差事。