---
title: HackWare BootLoader 原理
url: 516.html
id: 516
date: 2018-06-21 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E5%89%8D "前")前
-----------------------------------------------------------------------

最近, follow 一个开源项目, BusPirate ,总线海盗,一个很棒的超多功能集一身的一个 tool. 对其中的 bootloader 印象深刻, 虽然不是第一次知道这东西, 这次也是来了兴致. 学习一下.

* * *

*   BusPirate Github
*   BusPirate Offical Website

[](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#What "What")What
------------------------------------------------------------------------

先看看词条的定义:

> 在嵌入式操作系统中，BootLoader是在操作系统内核运行之前运行。可以初始化硬件设备、建立内存空间映射图，从而将系统的软硬件环境带到一个合适状态，以便为最终调用操作系统内核准备好正确的环境。在嵌入式系统中，通常并没有像BIOS那样的固件程序（注，有的嵌入式CPU也会内嵌一段短小的启动程序），因此整个系统的加载启动任务就完全由BootLoader来完成。在一个基于ARM7TDMI core的嵌入式系统中，系统在上电或复位时通常都从地址0x00000000处开始执行，而在这个地址处安排的通常就是系统的BootLoader程序。

实际上我们简单讲, 就是在嵌入式设备中, 给我们即将执行的程序(系统), 提供运行初始化环境的东西, 在初始化完成之后, 直接进行 **jmp** 使得**PC** 到目标程序空间, 永不返回(一般性). 这里,就是以这个项目提供的bootloader源码进行.

[](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#How "How")How
---------------------------------------------------------------------

文件定位: https://github.com 本想着可能是使用 C 写的, 发现除了配置文件, 只有 .s 汇编了,

* * *

文件前段,是以下内容注释:

> ds30 Loader is free software: you can redistribute it and/or modify  
> ;  
> it under the terms of the GNU General Public License as published by the Free Software Foundation.

这个 bootloader 应该是,第三方所开发的框架, 直接对其进行修改就好

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E9%9D%99%E6%80%81%E5%88%9D%E5%A7%8B%E5%8C%96 "静态初始化")静态初始化

    ;------------------------------------------------------------------------------; Register usage;------------------------------------------------------------------------------        ;.equ    MIXED,        W0        ;immediate        .equ    DOERASE,    W1        ;flag indicated erase should be done before next write        .equ    WBUFPTR,    W2        ;buffer pointer        .equ    WCNT,        W3        ;loop counter        .equ    WADDR2,        W4        ;memory pointer        .equ    WADDR,        W5        ;memory pointer                .equ    PPSTEMP1,    W6        ;used to restore pps register        .equ    PPSTEMP2,    W7        ;used to restore pps register        .equ    WFWJUMP,    W8        ;did we jump here from the firmware?        ;.equ    UNUSED,        W9        ;        .equ    WDEL1,        W10        ;delay outer        .equ    WDEL2,        W11        ;delay inner        ;.equ    UNUSED,        W12        ;        .equ    WCMD,        W13        ;command        .equ     WCRC,         W14        ;checksum        .equ    WSTPTR,     W15     ;stack pointer

这里的一段代码来自文件首,`,equ`的伪指令也说明了 这里是,寄存器的宏定义,给不同寄存器,起别名, 保证了程序易读性. 下面导入, 单片机设置. 和我们平时使用的 `#include <reg52.h>` 类似

    ;------------------------------------------------------------------------------; Includes;------------------------------------------------------------------------------.include "settings.inc" 

* * *

在这段代码的注释部分已经说明, 这里是 constants的 ,不需要进行修改. 主要内容主要是, 对于 将会使用的常量的宏. 内容有 字符, 延时, 串口波特率, 页大小, 和 STARTADDR . 其原文的注释,对这些 静态符号也有很好的说明

    ;------------------------------------------------------------------------------; Constants, don't change;------------------------------------------------------------------------------        .equ    VERMAJ,        1                                        /*firmware version major*/        .equ    VERMIN,        0                                        /*fimrware version minor*/        .equ    VERREV,        2                                        /*firmware version revision*/        .equ     HELLO,         0xC1                .equ     OK,         'K'                                        /*erase/write ok*/        .equ     CHECKSUMERR,'N'                                        /*checksum error*/        .equ    VERFAIL,    'V'                                        /*verification failed*/        .equ       BLPROT,     'P'                                      /*bl protection tripped*/

​  
.equ BLDELAY, ( BLTIME _(FCY / 1000) / (65536_ 7) ) /_delay berfore user application is loaded_/  
;.equ UARTBR, ( (((FCY / BAUDRATE) / 8) - 1) / 2 ) /_baudrate_/  
/ _issue 11 in errata for A3, optimal value causes reception to fail_ /  
/ _autocalculate: 0x21, <2.5% error_ /  
/ _working: 0x22, <3% error, same as main firmware_ /  
.equ UARTBR, 0x22;((FCY/(4_BAUDRATE))-1)  
.equ PAGESIZE, 512 /_words_/  
.equ ROWSIZE, 64 /_words_/  
; 这个 指令是注释掉了的,  
; .equ STARTADDR, ( FLASHSIZE - 2_(PAGESIZE _2) ) /_place bootloader in 2nd last program page_/  
.equ STARTADDR, ( FLASHSIZE - (2_ (PAGESIZE)) ) /_place bootloader in last program page_/  
.equ BLCHECKST, ( STARTADDR - (ROWSIZE) ) /_precalculate the first row write position that would overwrite the bootloader_/  
.equ BLVERSION, 0x0405 ;bootloader version for Bus Pirate firmware (located at last instruction before flash config words) 对 一些 宏的 定义值进行合法性检测.

    ;------------------------------------------------------------------------------; Validate user settings;------------------------------------------------------------------------------        ; Internal cycle clock        .if FCY > 16000000            .error "Fcy specified is out of range"        .endif        ; Baudrate error        .equ REALBR,    ( FCY / (4 * (UARTBR+1)) )        .equ BAUDERR,    ( (1000 * ( BAUDRATE - REALBR)) / BAUDRATE )        .if ( BAUDERR > 30) || (BAUDERR < -30 )            .error "Baudrate error is more than 3%. Remove this check or try another baudrate and/or clockspeed."        .endif         .if BLDELAY<1           .error "Bootloader delay is 0".endif ...

这里,在数据段分配空间 , 存放固件签名地址

    ;------------------------------------------------------------------------------; Uninitialized variables in data memory;------------------------------------------------------------------------------    .bssbuffer:    .space ( ROWSIZE * 3 + 1/*checksum*/ )     .equ FIRMWARE_SIGNATURE_LOW, 0x3141    .equ FIRMWARE_SIGNATURE_HIGH, 0x5926    .global skip_pgc_pgd_check    .global firmware_signature    .section *, bss, address(0x27FA)skip_pgc_pgd_check: .space 2firmware_signature: .space 4

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E7%AD%BE%E5%90%8D%E6%A0%A1%E9%AA%8C "签名校验")签名校验

在文件的前面, 有声明一个全局符号

1  
2  
3  
4  

;------------------------------------------------------------------------------  
; Global declarations  
;------------------------------------------------------------------------------  
 .global __reset          ;the label for the first line of code, needed by the linker script  

由符号可以直接看出, 是一个复位向量, 下面是标号内容, 也就是我们一上电会执行的东西

1  
2  
3  
4  
5  

;------------------------------------------------------------------------------  
; Reset vector  
;------------------------------------------------------------------------------  
.section *, code, address(STARTADDR); 指定段 及段偏移  
\_\_reset:mov #\_\_SP_init, WSTPTR; 初始化栈指针  

​  

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  
33  
34  
35  
36  
37  
38  

;------------------------------------------------------------------------------  
; User specific entry code go here, see also user exit code section at end of file  
;------------------------------------------------------------------------------  
bclr OSCCON, #SOSCEN  
bclr CLKDIV, #RCDIV0 ;set clock divider to 0  
  
waitPLL:btss OSCCON, #LOCK; 锁相环初始化  
bra waitPLL ;wait for the PLL to lock  
  
mov #0xFFFF, W0 ;all pins to digital  
mov W0, AD1PCFG; IO初始化  
  
; Make sure the firmware has been started at least once.  
;  
; If the firmware signature is found in memory then it is  
; extremely plausible that skip\_pgc\_pgd_check has been  
; initialised to the correct value.  
  
; 读取片内地址, 与硬编码签名进行比较  
mov #firmware_signature, W0  
mov \[W0++\], W1; 这里读取高字节  
  
mov #FIRMWARE\_SIGNATURE\_HIGH, W2; 加载 正确签名高字节  
cp W1, W2; 比较  
bra nz, jumper_test; 不等跳转  
  
mov \[W0\], W1  
mov #FIRMWARE\_SIGNATURE\_LOW, W2; 加载 正确签名低字节  
cp W1, W2  
bra nz, jumper_test; 不等跳转  
  
mov #skip\_pgc\_pgd_check, W0; 加载标志位地址  
cp0.b \[W0\]  
clr.b \[W0\]       ; should not change flags  
; 上面的 PIC 汇编,没看懂, 查了查没有 .b 这样的语法???  
; 理应是把这个地址的 内容复位吧.  
  
bra nz, setup; 开始启动配置  

上面的这段代码, 实际上应该是对,当前的单片机内是否有 **有效的固件** 进行检测. 固件包含签名, 有签名就是有固件.

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E8%B7%B3%E7%BA%BF%E6%A3%80%E6%B5%8B "跳线检测")跳线检测

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  
33  

jumper_test:  
    
mov #skip\_pgc\_pgd_check, W0; 老规矩, 加载标志位的地址  
clr.b \[W0\]; 清空内容  
    
;jumper check test  
;setup the jumper check  
;enable input on PGx  
  
; 这里是引脚配置部分,使得拥有电平检测的功能  
bclr LATB, #LATB1 ;rb1 low  
bset TRISB, #TRISB1 ;rb1 input  
bset CNPU1, #CN5PUE ;enable pullups on PGC/CN5/RB1  
;ground/output on PGx  
bclr LATB, #RB0 ;rb0 low  
bclr TRISB, #TRISB0 ;rb0 output  
  
;wait  
nop  
nop  
  
;check for jumper  
btsc PORTB,#RB1;跳过下条指令, 如果 RB1=0(即存在jmper), 这样就继续进行配置部分  
  
;;;;;;;;;;;重要;;;;;;;;;;;;;  
bra quit ; 如果是不存在 jumper 的那么,就退出 ,开始执行 用户程序  
  
clr WFWJUMP;we came from jumper and reset, not firmware jump  
  
;注意,后面紧接着就是 setup  
setup:  
.ifdef BUSPIRATEV2  
...  

* * *

结合实际上的具体操作, 上面的这部分就是比较容易理解了. 实际上, 在给板子使用 bootloader 进行固件烧写的时候,的确需要一个 jumper 连接 pgc 和 pgd 这两个脚. 否则的话, 显示如下err内容,

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  

+++++++++++++++++++++++++++++++++++++++++++  
 Pirate-Loader for BP with Bootloader v4+  
 Loader version: 1.0.2  OS: WINDOWS  
+++++++++++++++++++++++++++++++++++++++++++  
  
Parsing HEX file \[BPv3-firmware-v6.2-r1981.hex\]  
Found 21502 words (64506 bytes)  
Fixing bootloader/userprogram jumps  
Opening serial device COM13...OK  
Configuring serial port settings...OK  
Sending Hello to the Bootloader...ERROR  
No reply from the bootloader, or invalid reply received: 0  
Please make sure that PGND and PGC are connected, replug the devide and try again  

上述的代码, 就是对上电时候是否有进行 跳线进行检测, 从而执行不同的后续操作.

*   Firmware Upgrade - dangerousprototypes

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E7%94%A8%E6%88%B7%E7%A8%8B%E5%BA%8F-UserApp "用户程序 UserApp")用户程序 UserApp

在上面的bootloader , bra 到quit 标号的时候, 惊奇的发现,后面就开始执行我们的用户程序了.

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  

quit:;clean up from jumper test  
; 根据注释,这里应该是把 之前的跳线检测,进行复位  
bclr CNPU1, #CN5PUE ;disable pullups on PGC/CN5/RB1  
bset TRISB, #TRISB0 ;rb0 back to input  
mov #0x0000, W0 ;clear pins to analog default  
mov W0, AD1PCFG  
  
;------------------------------------------------------------------------------  
; Load user application  
;------------------------------------------------------------------------------  
 ; 没错,就是这里, 直接到了我们的用户程序了, 永不返回;  
bra usrapp  

可是, 上面的 setup 必须是要有个 jumper 多麻烦, 这里的 G1k精神所在, 所以, 会有以下的标号段.

1  
2  
3  
4  
5  
6  

;------------------------------------------------------------------------------  
; firmware jump entry point (kind of like a function because it's never reached from the above code  
;------------------------------------------------------------------------------  
firmwarejump:  
mov #0xffff, WFWJUMP;flag that we jumped from firmware  
bra setup;jump to just after jumper check  

这个符号被导出, 我们的用户程序中, 可以进行一次跳转. 回到我们的 bootloader. 这里 也找到在固件中存在的 跳转部分

*   源文件链接 ProcMenu.c:666

C 代码如下  
​  

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  

// ProcMenu.c  
case '$': //bpWline("-bootloader jump");  
 if (agree()) { //bpWline("BOOTLOADER");  
 BPMSG1094;  
 bpDelayMS(100);  
 bpInit(); // turn off nasty things, cleanup first needed?  
 while (0 == UART1TXRdy()); //wait untill TX finishes  
  
// 这里使用内联汇编, 加载地址,直接跳转, 妙啊  
 asm volatile ("mov #BLJUMPADDRESS, w1 \\n" //bootloader location  
 "goto w1 \\n");  
}  
  
// Base.h  
//sets the address in the bootloader to jump to on the bootloader command  
//must be defined in asm  
asm (".equ BLJUMPADDRESS, 0xABF8");  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E4%B8%B2%E5%8F%A3%E5%88%9D%E5%A7%8B%E5%8C%96-UART "串口初始化 UART")串口初始化 UART

在前面的检测中, 如果jumper是存在的, 这里就进行 setup操作, 这里的第一步就是配置 硬件串口

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  

setup:  
;----------------------------------------------------------------------  
; UART pps config  
;----------------------------------------------------------------------  
.ifdef BUSPIRATEV2  
; Backup, these are restored in exit code at end of file  
; Changes needs to be done in exit, search for xxx  
movRPINR18, PPSTEMP1;xxx  
movRPOR2, PPSTEMP2;xxx  
  
; Receive, map pin to uart (RP5 on 2/3, RP3 on v1a)  
; 初始化 串口接收  
bsetRPINR18, #U1RXR0;xxx  
bclrRPINR18, #U1RXR1;xxx  
bsetRPINR18, #U1RXR2;xxx  
bclrRPINR18, #U1RXR3;xxx  
bclrRPINR18, #U1RXR4;xxx  
  
; Transmit, map uart to pin (RPOR2bits.RP4R = 3 on 2/3, RPOR1bits.RP2R=3 on v1a)  
...  
; 配置 串口发送  
  
; MODE LED on during bootload  (A1 on 2/3, B4 on v1a)  
; 增加性能的 rgb   
bset LATA, #LATA1 ;on  
bclr TRISA, #TRISA1 ;output  
.endif  
  
; 不同版本  
.ifdef BUSPIRATEV1A  
...  
.endif  

* * *

完成了配置之后, 进行初始化

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  

;------------------------------------------------------------------------------  
; Init  
;------------------------------------------------------------------------------  
clrDOERASE  
  
;UART  
mov#UARTBR, W0 ;set  
mov W0, UBRG; baudrate  
bsetUMODE, #BRGH;enable BRGH  
bset UMODE, #UARTEN;enable UART  
bset USTA, #UTXEN;enable TX  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E8%AE%BE%E5%A4%87%E6%8F%A1%E6%89%8B "设备握手")设备握手

这里的设备握手, 是直接在 串口的后面执行的.

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  

;------------------------------------------------------------------------------  
; Receive hello  
;------------------------------------------------------------------------------  
 rcall Receive; 这里调用 函数符号 读取串口, 具体函数见后  
 sub #HELLO, W0;check  
; .equ HELLO, 0xC1 在前面静态定义了 HELLO 的值  
bra z, helloOK; prompt  
  
sub #'#', W0; check  
; Exit point, clean up and load user application  
bra z, exit; prompt  
  
; 如果这两个符号都不是, 说明过程出现错误.  
; 打印,当前BL的硬编码版本   
SendL   'B'; 发送宏  
SendL   'L'  
SendL   '4'  
SendL   '+'  
; 同上面的exit同样  
 bra    checkexit  
  
;------------------------------------------------------------------------------  
; Send device id and firmware version  
;------------------------------------------------------------------------------  
; 发送宏  
helloOK:SendL DEVICEID  
SendLVERMAJ  
SendL(VERMIN*16 + VERREV)  
; 这里通过串口, 把板子数据发送出去  

* * *

串口内容的单字节接收函数, 用于握手识别

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  

;------------------------------------------------------------------------------  
; Receive  
;------------------------------------------------------------------------------  
; Init delay  
Receive:mov #BLDELAY, WDEL1  
  
; Check for received byte  
rpt1:clrWDEL2  
rptc:clrwdt;clear watchdog  
btss USTA, #URXDA  
bra notrcv; not receive 在这个函数中, 循环等待,接收  
mov URXREG, W0; 接收的数据装载在 W0  
add WCRC, W0, WCRC;add to checksum 和 进行循环冗余校验  
return  

串口数据发送宏, 实现单字节数据发送.

1  
2  
3  
4  
5  
6  
7  

;------------------------------------------------------------------------------  
; Send macro  
;------------------------------------------------------------------------------  
.macro SendL char  
mov #\\char, W0; 装载内容  
mov W0, UTXREG;   
.endm  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E5%88%9D%E5%A7%8B%E5%8C%96 "初始化")初始化

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  

; Send ok  
Main:SendL OK  
; Init checksum  
main1:clr WCRC  
  
;----------------------------------------------------------------------  
; Receive address  
;----------------------------------------------------------------------  
  
...  
rcall Receive   
mov W0, TBLPAG;   
mov.b WREG, PR1+1  
mov.b WREG, PR1  
...   
  
; 这里的重复过程 , 从串口读取数据, 写到分配的静态空间里去.  
;----------------------------------------------------------------------  
; Receive command  
;----------------------------------------------------------------------  
  
;----------------------------------------------------------------------  
; Receive nr of data bytes that will follow  
;----------------------------------------------------------------------  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E6%95%B0%E6%8D%AE%E6%8E%A5%E6%94%B6 "数据接收")数据接收

这里就是开始接收 数据了

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  

;----------------------------------------------------------------------  
; Receive data  
;----------------------------------------------------------------------  
;; .bss  
; buffer: .space ( ROWSIZE * 3 + 1/\*checksum\*/ )  
 ; 在附加段, 定义了一个大的缓冲区  
mov #buffer, WBUFPTR; 加载缓冲区地址  
rcvdata:  
rcall Receive; 接收字节  
mov.b W0, \[WBUFPTR++\]; 循环接收  
decWCNT, WCNT; 数据计数  
bra nz, rcvdata ; 不是0 的话, 就跳转回去继续接收  
  
;last byte received is checksum  
  
;----------------------------------------------------------------------  
; Check checksum  
;----------------------------------------------------------------------  
cp0.b WCRC;   
bra z, bladdrchk; 这里是CheckSum 的一个校验 ,合法就继续  
SendL CHECKSUMERR  
bra main1 ; 不合法, 发送校验值, 重新接收.  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E7%83%A7%E5%86%99%E5%87%86%E5%A4%87 "烧写准备")烧写准备

这里的部分, 是把数据真正的烧入 flash 前的检验工作, 确保 我写入的数据是不会影响到我 Bl 的本身的空间的, 如果影响到自己, 把自己抹掉了不就是尴尬了.  
​  
检查是否数据容量是否超出, 导致覆盖, 这里保留官方的注释

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  

;----------------------------------------------------------------------  
; Check address  
;----------------------------------------------------------------------  
;check that write and erase range does not overlap the bootloader  
;this is pretty specific to the bootloader being in the last page  
;additional checks are needed if your bootloader is located elsewhere.  
;TBLPAG is always = to 0 on this PIC, no need to verify (check if you have bigger than 64K flash)  
  
;check the end address检查结束的地址  
;write row size is fixed, any writes at (bootloader start-63) are an error  
;if write end address (W0) is <= bl start address (WCNT) then OK  
;= is ok because we don't DEC after adding, write 10 bytes to 10 = end at 19  
  
bladdrchk:  
;; 在前面有定义 BL 的起始地址  
;;.equBLCHECKST,  ( STARTADDR - (ROWSIZE) )/\*precalculate the first row write position that would overwrite the bootloader\*/  
  
mov#BLCHECKST, WCNT;last row write postion that won't overwrite the bootloader  
;; 比较 当前内存指针, 和我们的 BL 的末地址.  
cpWADDR, WCNT;compare end address, does it overlap?  
bra GTU, bladdrerror ;if greater unsigned then error  
  
...  
  
;handle the address error 地址错误的处理, 发送错误信息, 跳转进行重新的读.  
bladdrerror:clrDOERASE ;clear, just in case  
 SendL   BLPROT;send bootloader protection error  
bra main1 ;  

* * *

**指针初始化**

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  

;----------------------------------------------------------------------  
; Init pointer  
;----------------------------------------------------------------------  
; 装载缓冲区地址指针  
ptrinit:mov #buffer, WBUFPTR  
  
;----------------------------------------------------------------------  
; Check command  
;----------------------------------------------------------------------  
; Write row0x00 02 00 - 0x02 AB FA   
btscWCMD,#1; 这里是对烧写命令的判断  
braerase; 不为 1 ,就不擦  
  
; Else erase page  
mov#0xffff, DOERASE  
bra Main  

### [](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E7%83%A7%E5%86%99-Flash "烧写 Flash")烧写 Flash

这一部分, 就开始由 BL 实现对 Flash 的烧写了, 首先对flash 进行擦出.

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  

;----------------------------------------------------------------------  
; Erase page  
;----------------------------------------------------------------------  
erase:btssDOERASE, #0;; DOERASE  标志位, 是否已经被擦出  
braprogram  
tblwtlWADDR, \[WADDR\];"Set base address of erase block", equivalent to setting nvmadr/u in dsPIC30F?  
; Erase  
mov #0x4042, W0; 这里的W0 是一个控制字, 绝对了,写函数是进行 写, 还是擦出.  
rcall Write  
; Erase finished  
clr DOERASE  

对Flash 进行擦出之后, 就可以开始我们的烧写过程了

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  

;----------------------------------------------------------------------  
; Write row  
;----------------------------------------------------------------------  
program:mov #ROWSIZE, WCNT; 寄存Row的空间  
  
; Load latches  
latlo:tblwth.b \[WBUFPTR++\], \[WADDR\] ;upper byte  
tblwtl.b\[WBUFPTR++\], \[WADDR++\] ;low byte  
tblwtl.b\[WBUFPTR++\], \[WADDR++\] ;high byte  
dec WCNT, WCNT  
bra nz, latlo  
  
; Write flash row  
mov #0x4001, W0; 这里的W0 是一个控制字  
rcall Write  

这里, 是写命令的实现部分, 突然发现,这里用的应该是一个硬件控制器,实现的对,flash的控制

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  

;------------------------------------------------------------------------------  
; Write  
;------------------------------------------------------------------------------  
Write:mov W0, NVMCON; 这里是 控制器的配置部分  
mov #0x55, W0  
mov W0, NVMKEY  
mov #0xAA, W0  
mov W0, NVMKEY  
  
bset NVMCON, #WR; 置位状态字  
nop  
nop  
; Wait for erase/write to finish  
compl:btscNVMCON, #WR; 循环等待,烧写完成  
bra compl  
return  

* * *

对于烧写部分的后续过程, 还有, 内容校验, 和错误处理的过程.

    ;----------------------------------------------------------------------        ; Verify row;----------------------------------------------------------------------;----------------------------------------------------------------------; Verify fail;----------------------------------------------------------------------    

[](https://www.diglp.xyz/2018/06/21/HackWare_Booloader/#%E6%80%BB%E7%BB%93 "总结")总结
----------------------------------------------------------------------------------

通过这个源码, 算是对简单的Bootloader 有了简单的了解, 在 Bp 的这个 bootloader里面, 显示通过检测 调试开关 , 决定是否进入调试状态. 之后进行串口初始化. 完成之后进行串口握手以确认设备, 即版本, 随后开始进行数据接收, 把其存放在 一个 缓冲区内, 之后通过 NVM 控制器, 对Flash进行烧写, 完成烧写过程.