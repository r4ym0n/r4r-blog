---
title: 读本好书《智能硬件安全》
url: 574.html
id: 574
date: 2018-03-03 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E5%89%8D "前")前
----------------------------------------------------------------------------------------------------------------

我想未来对安全方面的需求会越来越大，随着这个体系的越发庞大，其潜在的安全威胁，可能愈发的丰富。

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E7%AE%80%E4%BB%8B "简介")简介
---------------------------------------------------------------------------------------------------------------------------

*   书名：揭秘家用路由器0Day挖掘技术
*   作者：吴少华
*   ISBN：9787121263927

硬件安全，将会随着物联网的兴起得到苏醒，而且变得更加的多元化 本书分为 三个部分

*   基础知识
*   原理与应用
*   分析与利用

同样这里对内容，所学做极简要总结。

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86 "基础知识")基础知识
-------------------------------------------------------------------------------------------------------------------------------------------------

路由器漏洞分类：

*   密码破解漏洞 WPA/WPS/WEP
*   WEB漏洞 SQL注入/远程命令执行/跨站脚本
*   特定后面 调试端口/admin
*   溢出漏洞 这个算是系统级别的

路由的web安全和服务器web安全类似。

* * *

常见的路由器其处理器结构基本是都是 **MIPS** 运行精简 **LINUX** ， 主要的基本 shell 功能由 **BusyBox** 实现

    busybox ls -lbusybox cd...

路由器中的`ls`等基本命令由busyBox的链接实现。

* * *

*   GNU 工具集

GCC 常用功能，不做展开。 **GDB** 作为主要调试器 命令常用需要掌握

> GDB十分钟教程

值得注意的是，在使用GDB调试之前，elf文件需要包含调试信息

    gcc -ggdb main.c

* * *

*   MIPS汇编及体系

（感觉除了X86汇编，其他的都十分奇怪。。。） 一个**32**个寄存器，特殊的是 $0 寄存器，的值总是零，提供需要用到0的地方。29 = sp，30 = fm，31 = ra （返回） MIPS是大端序(BIG_endian)，和网络字节序相同

> **高在高位是小端，高在低位是大端**

    {num[n],num[n+1],num[n+2],num[n+3]}    #MSB    (Most Significant Byte){num[n+3],num[n+2],num[n+1],num[n]}    #LSB    (Least Significant Byte)

> “大端”和“小端”可以追溯到1726年的Jonathan Swift的《格列佛游记》，其中一篇讲到有两个国家因为吃鸡蛋究竟是先打破较大的一端还是先打破较小的一端而争执不休，甚至爆发了战争。

* * *

*   HTTP协议

路由器的很多漏洞是存在于 Web服务器没有正确的仅需攻击者所发送的HTTP请求。 **HTTP请求行**

    [Method] [Request-URI] [HTTP-Version] [CRLF] eg: GET /from.html HTTP/1.1 (CRLF)

> (CRLF是Carriage-Return Line-Feed的缩写，意思是回车换行，就是回车(CR, ASCII 13, \\r) 换行(LF, ASCII 10, \\n))

这里规定，必须是以 **CRLF** 结尾，不允许出现单独 **CR(回车)/LF(换行)**,**“\\r\\n”**,**“\\x0D\\x0A”**。 （之前一个Qt用socket实现的http请求，不能得到正确GET的Respond 的原因） Method 有很多种 GET/POST/HEAD/PUT/DELETE/TRACE/CONNECT/OPTIONS. POST克服了GET方法的一些缺点。因为通过Post进行表单数据的提交的时候，数据本身不是URL请求的一部分，而是作为标准数据传送给服务器，这点克服了GET进行数据传递的时候信息无法加密和提交数据量太小的缺点。 **HTTP报头**

*   Accept: 表示希望接收的资源类型
*   Accept-Encoding: 表示内容编码
*   Cookies: 表示客户端向服务器进行Cookies认证的信息
*   Accept-Encoding: 指定一种自然语言
*   **Host:主机及其端口号，默认80，通常从URL中得到**
*   User-Agent： 用户代理，实际上包含着用户的部分信息，系统，浏览器内核等等

请求头示例：

    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8Accept-Encoding: gzip, deflate, brAccept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Cache-Control: max-age=0Connection: keep-aliveCookie: BAIDUID=5E1B56CB86750A3F365EDFC9FA1DA1A9:FG=1; BIDUPSID=E3A36946B81579878C6378864B750C4A; PSTM=1512745351; Hm_lvt_55b574651fcae74b0a9f1cf9c8d7c93a=1524673678,1524718817,1524727227,1524729701; Hm_lpvt_55b574651fcae74b0a9f1cf9c8d7c93a=1524729701; H_PS_PSSID=1420_21106; BDRCVFR[gltLrB7qNCt]=mk3SLVN4HKm; PSINO=7; pgv_pvi=3976877056; pgv_si=s5736833024DNT: 1Host: baike.baidu.com

* * *

*   软件工具

主要是虚拟机，IDA，BinWalk，QEMU。这些工具。 前两种不多做介绍了，虚拟机，和静态反编译工具。 BinWalk，主要用于对于固件包的自动化解包和分析，可以对目标架构，和目录结构，内核版本等等的信息进行自动分析。

> BinWalk的使用

QEMU 和 Bochs 类似是一个处理器模拟软件，在环境中，我们配合MIPS的交叉编译工具，可以实现一个本机模拟的MIPS的机器平台，方便我们进行各种测试。

* * *

*   路由0day基本挖掘方法

对路由0day挖掘的初步方法大致以下

*   固件分析 使用BinWalk对固件进行解包，提取其中的关键文件进行分析。
*   **动态运行库的劫持** 对于一些关键的so(Sharded Object) 的里面的函数进行重写，即保留同样的函数符号，使用我们自己的函数内容实现一个新的so文件，并且替换源文件。
    
        #include <stdio.h>#include <stdlib.h>int apmib_init(void){    //Fake it    return 1;}$ mips-linux-gcc -fPIC -shared apmib.c -o apmib-ld.so
    
    实际上的过程，涉及到对so文件的静态分析，通过IDA对器进行逆向分析，在保留器原有函数的基础上，对部分的符号代码进行修改。
    

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E8%B7%AF%E7%94%B1%E5%99%A8%E5%AE%89%E5%85%A8 "路由器安全")路由器安全
------------------------------------------------------------------------------------------------------------------------------------------------------------

*   web部分

该书的这部分很简略，篇幅在几页，主要讲了有 **XSS** 和 **CSRF**。 XSS(Cross Site Scipting)，防止和 CSS(Cascading Style Sheets) 起名为 XSS。具体就是恶意的JS脚本插入，分为反射型和存储型。前者是主动触发，我们可以找到XSS点，构造反射链接，发送给受害者，存储型，常见的就是留言板。233 CSRF(Cross-Site Request Forgery) 是一种对网页的恶意利用。和XSS有着十分大的差别。其实际上不是通过插入的JS实现功能，实际上是对网页原文进行劫持之后进行的修改，最终实现了自己的代码会在目标主机执行。

*   路由器后门  
    这种漏洞出现于官方的预留的端口，或者其他的超级密码。

* * *

*   路由器溢出漏洞

溢出漏洞是一个相当高危而且普遍的漏洞。 **栈溢出**

> 在计算机科学中，栈是一种先进后出得(FILO)队列的数据结构。调用栈(Call Stack)是值存放在一个正在运行的函数的信息栈。调用栈本身又是由栈帧(Stack Frame)构成，每个栈帧对应一个未完成函数。 函数的调用过程(栈帧)

在MIPS架构中，参数传递使用 $a1~$a4 着四个寄存器，（$a0是零值寄存器），所以我们的参数超过5个之后就会使用到了栈，进而进行第五个参数的传递。 MIPS中的溢出可行性:X86的架构不同， x86的调用过程，是发生函数调用时，把当前的函数地址压入栈中，在函数返回时直接进行弹栈，从而返回原函数的地址空间，但是在MIPS的架构下，函数调用时**不会把原函数地址压栈** 而是直接存入寄存器 **$ra(返回地址寄存器)**。下面时书中的溢出可行性分析： 在MIPS的架构中，有着 **叶子函数和非叶子函数** （这里竟然是查无此词，应该是作者自己的词）。

> 个人理解讲:叶子函数这个词的叶子可以取于树结构，叶子说明函数体内没有调用其它的函数，也就是没有后继节点。非叶反之.

**非叶子函数**的情况:由于**$ra**寄存器只存在一个,所以实际上,如果是非叶子函数了,子函数体内部再次发生 Call 这样的话,会发生,把上一个函数的地址压栈,把调用函数的地址存入 **$ra** 所以和经典的溢出思路相同,还是覆盖掉压入栈中的返回地址. **叶子函数**情况:作为叶子函数,其没有后继的函数Call 所以,返回地址是保存在 **$ra** 中的,所以我们无法通过经典的思路进行覆盖(这个是寄存器了),不过也是存在利用可能,我们使用足够大的数据,覆盖掉上层函数的返回地址.(**上层调用了我,上层一定时非叶子对吧**) **缓冲区溢出**  
在缓冲区分配,和使用过程中的问题,比如对所缓冲数据没有做检测,导致其对栈内数据发生了覆盖 一般实现功能:拒绝服务,获得用户级权限,获得系统级权限(提权).

    #include <stdio.h>  #define PASSWORD "1234567"  int verify_password (char *password)  {     int authenticated;     char buffer[8]; // add local buffto be overflowed     authenticated=strcmp(password,PASSWORD);     strcpy(buffer,password); // over flowed here!     return authenticated;  }  main()  {     int valid_flag=0;     char password[1024];     while(1)     {        printf("please input password: ");        scanf("%s", password);        valid_flag=verify_password(password);        if(valid_flag)        {           printf("incorrect password!\n\n");        }        else        {           printf("Congratulation! You have passed the verification!\n");           break;        }     }  } 

这里贴上一段简单代码,注释已经标明了溢出点,在;进行Cpy的时候,没有进行长度检测. 我们知道,局部变量是依次在栈中分配空间的,所以我们分配的8个字节的数组紧邻的就是 int . 这里我们可以实现的是 ![](https://upload-images.jianshu.io/upload_images/2897833-ec918cbd4442e4c4.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/453)

> 图片来自

1.  通过对 buffer的溢出,从而覆盖 那个int的值
2.  覆盖返回地址(这一点就是无限的空间了)**SHELLCODE**

* * *

*   ShellCode

可以使用覆盖返回地址之后.我们就可以让当前函数返回到我们希望的地方了. 这个地方,就可以是我们构造的ShelCode.

    char shellcode[] ="\x55\x8b\xec\x51\x51\x83\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89""\xf3\x8d\x4e\x08\x31\xd2\xcd\x80\xe8\xe4\xff\xff\xff\x2f\x62\x69\x6e""\x2f\x73\x68\x58";   //...VOID Sub_2(){    ((void(WINAPI*)(void))&ShellCode)();}

这里具体来又是一本书了…

* * *

*   文件系统提取

固件的提取思路主要是找到一个文件的签名头,这样才可以识别出到底是什么文件,比如我们常用的`file`

1.  **strings|grep** 全文检索文件系统的 magic 签名头
2.  **hexdump|grep** 检索 magic 签名偏移
3.  **dd|file** 确定migic签名偏移处的文件系统格式

eg: cramfs 的magic的签名是 0x28cd3d45,squashfs 有 sqsh,hsqs…

    string firmware.bin | grep `python -c 'print "\x28\xcd\x3d\x45"'`string firmware.bin | grep `python -c 'print "\x45\x3d\xcd\x28"'`

这里对整个文件的字符串进行检索找到有没有符合 cramfs 的签名,这里之所以会寻找两次,是为了保证,大端和小端的两种情况.找到特征签名之后,我们就开始定位文件偏移

    hexdump -C firmware.bin | grep -n 'hsqs'

这里找到特征字符串的偏移.之后我们可以使用 DD 对文件进行提取

    dd if=firmware.bin bs=1 count=100 skip=1441936 of=squash.bin

这里就是使用dd对文件进行偏移的提取了. **自动提取大法 BinWalk**

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E6%BC%8F%E6%B4%9E%E6%8E%A2%E7%B4%A2 "漏洞探索")漏洞探索
-------------------------------------------------------------------------------------------------------------------------------------------------

后面的部分针对漏洞的实际应用做了总结,慢慢的学习其中的过程。 实际上漏洞挖掘于应用流程如下

1.  劫持PC，确定缓冲区大小，并且定位确定控制偏移
2.  编写代码通过QEMU虚拟机进行验证，并调试
3.  确定攻击路径，并且构造ROP
4.  利用攻击数据，编写 exploit代码，对路由进行测试

如果知道了漏洞，其分析手法是可以直接通过 Strings 找到相关字符串的 应用位置，之后根据CallStack ，找到潜在的危险函数。 之后用脚本，构造测试用例。比如 `'A'*600` ，使其输入到对应的可能函数。在可疑部分下断，之后进行调试。 输入我们的用例，观测我们的栈中的 **saved_ra** （即非叶子函数的返回地址的压栈）是否被覆盖，从而可以得知此处是否有溢出漏洞，如果有那么 **saved_ra** 的值应该被覆盖为 **0x41414141**（即A的ascii）。由此我们可以确定缓冲区溢出存在。 之后就是确认我们的溢出的定位，使得我们可以精确的覆盖返回地址。这种方法叫做 **ROP (Return-oriented programming)** 一般的POC中使用的溢出Payload，使用的是 System/Exec 这个函数，使得我们可以得到一个系统权限的Cli的返回

*   硬件部分

**FLASH**的读取 这个直接拆下来进行数据的完全读取。 **串口探测** 通过电压的测量 ，和对PCB的目测。 **JTAG探测** （jointed test action group）

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E6%BC%8F%E6%B4%9E%E5%8F%91%E6%8E%98%E6%80%9D%E8%B7%AF "漏洞发掘思路")漏洞发掘思路
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

代码审计：静态审计，模糊测试 静态测试：通过IDA进行反编译，发现潜在的危险函数。 获取用户的数据函数

*   argv
*   read(),fscanf(),getc(),
*   stdin
*   read(),recv(),recvfrom()

数据操作函数

*   strcpy()
*   strncpy()
*   system(),execve()
*   sprintf(),snprintf()

[](https://www.diglp.xyz/2018/03/03/Book_%E6%99%BA%E8%83%BD%E7%A1%AC%E4%BB%B6%E5%AE%89%E5%85%A8/#%E5%90%8E "后")后
----------------------------------------------------------------------------------------------------------------

实际上,关于硬件安全的分析,在这本书从软件和硬件层面都去展现了一个漏洞发掘的过程,很难\- 得,也是巩固了不少相关的知识.