---
title: 《Linux就该这么学》笔记01
tags:
  - OPS
url: 1540.html
id: 1540
categories:
  - 读本好书
date: 2019-12-15 18:59:26
---

这个是朋友送的一本书，内容是比较的基础的，一下午时间把它读完～ 把自己觉得需要记录的东西写下来。

SHELL & 管道 p60
--------------

*   STDIN 0
*   STDOUT 1
*   STDERR 2

输入重定向符号的区别：

1.  1 < 2 文件 2 作为标准输入到 1
2.  1 << 2 从标准输入中读取内容遇到 2 之后结束，标准输入到 1
3.  1 < 2 > 3 文件2 作为标准输入输入到 1 其返回结果输出到文件 3

输入重定向符号的区别：

1.  `1 > 2` 输出创定向到文件2
2.  `1 2> 3` 错误输出重定向到文件3
3.  `1 >> 2` 标准输出追加到 文件2
4.  `1 >> 2 2>&1` 标准输出重定向到文件2，且错误输出定向到标准输出
5.  `1 &>> 2` 重定向所有输出追加到文件2

常用四个转义符号：

*   `\` 特殊符号转义
*   `''` 所有变量为普通字符串
*   `""` 保留变量属性
*   `` 返回执行结果

常用环境变量

*   HOME
*   SHELL
*   LANG
*   RANDOM 返回随机数字
*   PS1 当前的Cli提示符
*   PATH
*   EDITOR

VIM p71
-------

补充 VIM 的盲区知识，

*   `%d` 清空整个的内容
*   `set nu` 显示行号
*   `s/aaa/bbb` 行内替换第一个
*   `s/aaa/bbb/g` 行内替换所有
*   `%s/aaa/bbb/g` 全文所有替换
*   `? (/) str` 从上至下搜索/从下向上搜索

SHELL 逻辑
--------

条件表达式，基础的格式 `[ exp ]` 两边需要空格，分为4类

1.  文件测试
2.  逻辑测试
3.  整数值比较
4.  字符串比较

### 文件测试符

*   `-d` 是否为目录
*   `-e` 文件是否存在
*   `-f` 是否一般文件
*   `-r` 是否有权限读
*   `-w` 是否有权限写
*   `-x` 是否有权限执行

`$?` 为上一条命令的返回值，eg：

    ➜  ~ [ -d bak ]
    ➜  ~ echo $?
    0

### 逻辑运算

*   `&&` 成功之后执行
*   `||` 失败之后执行
*   `!` 在测试表达式内表示 非

    ➜  ~ [ -d bak ] || echo 123
    ➜  ~ [ -d bak ] && echo 123
    123
    ➜  ~ [ -d ba ] && echo 123
    ➜  ~ [ -d ba ] || echo 123
    123
    ➜  ~ [ ! -d ba ] && echo 123
    123
    
    # 条件可以并列
    ➜  ~ [ ! -d ba ] && echo 123 || echo 121
    123
    ➜  ~ [ -d ba ] && echo 123 || echo 121
    121

### 数值判断

*   `-eq` ==
*   `-ne` !=
*   `-gt` >
*   `-lt` <
*   `-le` <=
*   `-ge` >=

### 字符串判断

*   `=` 两个字符串相同
*   `!=` 两字符串不同
*   `-z` 是否空字符串

SHELL 流程控制
----------

### if

    if [ 123 = 123 ]
        then echo OK
        else echo NOT OK
    fi
    
    if [ $1 = 123 ]
        then echo OK
        elif [ $1 = 222 ]
        then OK
        else echo Not OK
    fi

### for/while/case

    for LINE in cat tmp; do
    echo $LINE
    
    # 查询所有的 用户 id
    for LINE in cat /etc/shadow | cut -d: -f1; do echo id $LINE;
    

    NUM=0
    while $NUM < 100; do let  $NUM++ echo $NUM; done;
    

    case $1 in 
    [abc])
    echo abc
    ;;
    [0-9]
    echo $1
    ;;

AT
--

计划任务，感觉和 cron不同的是，可以设置单次计划

    # 晚上10点执行
    ➜  ~ echo "echo 123>/tmp/123" | at 22:00
    ➜  ~ at -l
    1   Sun Dec 15 22:00:00 2019
    2   Sun Dec 15 22:00:00 2019
    ➜  ~ atrm 1
    ➜  ~ at -l
    2   Sun Dec 15 22:00:00 2019

用户管理
----

    #  创建不可登录的普通用户账户
    useradd -u  [ -s /bin/nologin]  
    # 创建用户组
    groupadd 
    # usermod 修改用户信息
    user -G  
    userdel -r 

passwd 不仅仅是用来改密码

    ➜  ~ passwd --help
    Usage: passwd [OPTION...] 
      -k, --keep-tokens       keep non-expired authentication tokens
      -d, --delete            delete the password for the named account (root only)
      -l, --lock              lock the password for the named account (root only)
      -u, --unlock            unlock the password for the named account (root only)
      -e, --expire            expire the password for the named account (root only)
      -f, --force             force operation
      -x, --maximum=DAYS      maximum password lifetime (root only)
      -n, --minimum=DAYS      minimum password lifetime (root only)
      -w, --warning=DAYS      number of days warning users receives before password
                              expiration (root only)
      -i, --inactive=DAYS     number of days after password expiration when an account
                              becomes disabled (root only)
      -S, --status            report password status on the named account (root only)
      --stdin                 read new tokens from stdin (root only)

文件隐藏书属性
-------

`chattr` 设置一下文件的隐藏权限。

    # 查看文件的隐藏位的设置
    lsattr
    # 修改它
    chattr

磁盘
--

### RAID

书中这里写的主要是通过系统来组`软RAID`，使用`mdadm`工具来进行。书中的示例配置

    mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sdb /dev/sdc /dev/sde /dev/sdd

### LVM

...

firewalls
---------

### Iptables

    # 列出所有当前 Iptable 规则
    iptables -nvL
    
    # 列出规则链
    iptables -nvL INPUT
    
    # 不响应ICMP
    iptables -I INPUT -p icmp -j REJECT
    
    # 指定网段访问 22
    iptables -I INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
    iptables -I INPUT -p tcp --dport 22  -j DROP
    
    # 指定端口段
    iptables -I INPUT -p tcp --dport 3000:4000 -s 192.168.1.0/24 -j ACCEPT

ssh
---

### 密钥登录

直接进行证书生成以及分发

    [root@k8s-node0 ~]# ssh-keygen
    [root@k8s-node0 ~]# ssh-copy-id k8s-master

后
-

后面的服务部署篇后面再接着搞吧