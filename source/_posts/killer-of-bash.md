---
title: Killer of Bash
tags:
  - Bash
  - Linux
url: 1013.html
id: 1013
categories:
  - 未分类
date: 2019-06-01 22:28:39
---

这里，在一下午的时间，来熟练掌握BASH下的大杀器，AWK，GREP，SED，VI，XARGS，等等的使用方法。来极大的提高在bash下的生产力，之前说是开专题，开到现在算是有了个头了。

sed
---

stream editor ，在linux极为突出的文字处理器。通过管道对数据进行处理。sed的经典应用：

*   自动编辑一个或多个文件
*   简化对多个文件的反复编辑
*   编写转换程序

sed 的工作原理，是对输入的行进行缓冲，之后使用模式匹配，来进行操作，完成之后再把数据送往屏幕。其基本的语法如下：

    sed [option] &#039;command&#039; file(s)

在sed里面，操作被抽象为一个操作符比如进行文本替换则使用的是`s`，那么得到的command是`s/aaa/bbb` 这里的`/`是定界符，可以为**任意字符**。'd'来删除指定字符串`sed '/sk/d'`，

在sed的表达式组合`sed '表达式' | sed '表达式`,'等价于：`sed '表达式; 表达式'`

AWK
---

用于，在shell中进行文本和字符串的处理。十分的方便，特别使在一些格式化提取的情境下。应用场景如下:

*   格式化的文本提取
*   执行编程
*   ...

这里里，先学习比较低阶的操作了。基本语法：

    awk [options] &#039;script&#039; var=value file(s)

可以使用 `-F` 来指定分隔符，示例如下：

    awk -F: &#039;print $1;print $NR&#039; /etc/passwd

* * *

简单的模式-过程实例

*   {print $1} 打印第一个参数
*   /pattern/ 打印匹配正则的行
*   {print NF} 当前的域数目Colum
*   $1 ~ /xxxx/ {print $3, $2} 在第一个域匹配的时候，交换 2，3域
*   /pattern/ {++x} END {print X} 打印出所有匹配行数的总和
*   {print(length($1))} 打印一列参数长度

* * *

awk的常用函数，`length`，`print`，`tolower`，`toupper`...etc.

GREP
----

grep 作为常用的字符串的查找指令，除了直接的 `ps -ef | grep xxx` 还是有很多妙用的，

*   `-c` 显示内容的计数
*   `-v` 反向匹配，即不显示匹配行
*   `-n` 显示行号
*   '^u' 正则匹配，显示u开头的项
*   '^\[^u\]' 显示非u显示的项
*   'u$' 同样的以u结尾

XARGS
-----

由于很多的命令实际上是不支持的，所以就需要使用xargs把管道的输入数据作为执行的命令的参数。使用命令，这个是错误的：

    find / --name &quot;124&quot; | rm

所以需要使用`xargs` 来实现

    find / --name &quot;123&quot; | xargs rm

下面是一个综合的使用案例

    ps -ef| grep frp | awk &#039;NR==1{print $2}&#039; | xargs kill