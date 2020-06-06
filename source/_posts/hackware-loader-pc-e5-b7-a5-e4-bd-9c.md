---
title: HackWare Loader PC工作
url: 514.html
id: 514
date: 2018-06-25 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E5%89%8D "前")前
-----------------------------------------------------------------------

前面的博文，差不多对 BP 所使用的 BL 做了个十分粗浅的理解了， 这一篇， 是对于其PC端上的和BootLoader 共同作用的一个软件（Pirate-loader）的一个源码的阅读笔记。对其原理进行学习。

* * *

文件定位 ：Bootloaders/pirate-loader/pirate-loader.c

[](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E6%BA%90%E7%A0%81-Src "源码 Src")源码 Src
----------------------------------------------------------------------------------------------

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E7%B3%BB%E7%BB%9F%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91%E5%AE%8F "系统条件编译宏")系统条件编译宏

这个标题可能是不大准确，实现的是我们很长用的功能，我们使用 GCC 编译器的时候， 在不同的平台编译，使用平台的特定系统API，实现功能相同的底层函数。（linux下的就是linuxC，Win下的就是Winapi）。 下面就是实现的预编译语句，在win下的编译过程中，编译器会自动的帮我们定义了 WIN32 这个宏。

    #ifdef WIN32    #include <windows.h>    #include <time.h>    #define O_NOCTTY 0    #define O_NDELAY 0    #define B115200 115200    #define OS WINDOWS    ...#else    // unix/linux    #include <unistd.h>    #include <termios.h>    #include <sys/select.h>    #include <sys/types.h>    #include <sys/time.h>#endif#if !defined OS    #define OS UNKNOWN#endif

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#Win%E7%9A%84%E5%87%BD%E6%95%B0%E5%B0%81%E8%A3%85 "Win的函数封装")Win的函数封装

在这种多平台编译的情况下，统一接口就是比较重要的过程了，在这个工程里，原作者使用win函数进行进一步封装，实现和 Linux 环境下的统一的接口，在后面的功能代码里面直接进行调用即可，这是个很好的思想，学习了

* * *

例如这里是，一个写函数的实现，

    int write(int fd, const void* buf, int len)    {        HANDLE hCom = (HANDLE)fd;    // 这里的文件描述符实际上是句柄了        int res = 0;        unsigned long bwritten = 0;        res = WriteFile(hCom, buf, len, &bwritten, NULL);        if( res == FALSE ) {            return -1;        } else {            return bwritten;            // 已写入字节        }    }

我们直接使用 `man 2 write` # 查看系统接口,可以看到，这个write函数的定义原型prototype：

    size_t    write(int fildes, const void *buf, size_t nbyte);

显然上面的定义是进行了相同的封装。

    int read(int fd, void* buf, int len){    ...     // 和write类似}int close(int fd){    HANDLE hCom = (HANDLE)fd;    CloseHandle(hCom);        // 关闭句柄    return 0;}

再下面的，这个open就是比较重要的一个函数了。具体的实现过程：

    int open(const char* path, unsigned long flags){    static char full_path[32] = {0};    // buf溢出风险    HANDLE hCom = NULL;    // 这里很是眼熟    if( path[0] != '\\' ) {        _snprintf(full_path, sizeof(full_path) - 1, "\\\\.\\%s", path);        path = full_path;    }    // 这里是打开串口的操作，后面的参数，OPEN_EXISTING, 说明了存在就打开    // 打开之后，返回我们的串口句柄    hCom = CreateFileA(path, GENERIC_WRITE | GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);    if( !hCom || hCom == INVALID_HANDLE_VALUE ) {        return -1;    } else {        return (int)hCom;    }}

实际上查看了相关的文章发现， 在win里面进行串口的打开实际上只是需要一个 ‘COMX’ 的端口号就是可以直接试一下 CreateFile 对串口进行打开。 不过这样前面的 `_snprintf` 的用法就是显得很迷了？为什么。突然一看后面， 有些眼熟 `\\.\COMx` 这个格式十分像之前的 win里面进程间通信的有名管道的用法。没错的。

* * *

也找到了这个写法的真正原因： 如果我们使用过 SMB 的服务，我们会发现在，进行计算机链接的时候我们的键入内容是？

    \\192.168.x.x

这样就是表面了对远程主机是发起了连接。转回到这里  
\\.\\COMx  
说明了什么？ 连接到 `.` 主机（也就是本地主机）的COMx， 妙哉。

* * *

**轮询**

    int __stdcall select(int nfds, fd_set* readfds, fd_set* writefds, fd_set* exceptfs, const struct timeval* timeout){        // 当前时间加上轮询时间    time_t maxtc = time(0) + (timeout->tv_sec);    COMSTAT cs = {0};    unsigned long dwErrors = 0;    if( readfds->fd_count != 1 ) {        return -1;    }    // 条件判断，在轮询时间内进行串口轮询    while( time(0) <= maxtc )    { //only one file supported        if( ClearCommError( (HANDLE)readfds->fd_array[0], 0, &cs) != TRUE ){            // 失败就是直接返回            return -1;        }        if( cs.cbInQue > 0 ) {            // 成功这里就返回            return 1;        }        Sleep(10);    }    return 0;}

* * *

Select 函数，在linux 下实现的是一个 I/O的复用，准确说是时分复用，这样相当于是给了内核任务，去查看IO的状态。这样就实现了重要的一点， 就是非阻塞IO。所以在Select的精妙的作用下， 可以实现在单线程里面，实现多个IO。 具体的使用，是根据其返回值，判断当前IO的状态，然后我们可以遍历 描述符集合。

*   Linux 下的Select函数

* * *

关于 __stdcall 的调用规则修饰，这里再加强记忆一下。

*   函数的调用规则(**cdecl,**stdcall,**fastcall,**pascal)

这种调用规则，是从右到左的参数压入顺序，被调用者把参数弹出栈， 用在winapi 里面是比较多的。其函数输出符号是 `_func@12` 后面是参数的字节数。

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#IO%E5%87%BD%E6%95%B0-%E5%B0%81%E8%A3%85 "IO函数 封装")IO函数 封装

上面的部分已经实现了对 Win和l下的接口的统一封装。 这一步是对一些IO细节的封装，以便适应我们的应用。

* * *

**读函数**  
由其定义名称得知，是带有IO超时的读函数。

    int readWithTimeout(int fd, uint8* out, int length, int timeout){    fd_set fds;    // 这里是select使用的结构    struct timeval tv = {timeout, 0};    int res = -1;    int got = 0;    do {        // 这里是对 文件描述符的集合进行刷新            // 在select 里面会有关闭        FD_ZERO(&fds);        FD_SET(fd, &fds);        res = select(fd + 1, &fds, NULL, NULL, &tv);        if( res > 0 ) {            // 说明IO就绪            res = read(fd, out, length);            if( res > 0 ) {                length -= res;                got    += res;                out    += res;            } else {                break;            }        } else {             return res;        }    } while( length > 0);    // 这里是读到缓冲区长度    // 感觉有问题，这个可以超出out的长度    return got;}

该函数，实现了对串口状态的非阻塞轮询读，在串口可读时，就多次读取串口数据，进入缓冲区。不可读时，就进行返回。 这里的问题是，在缓冲区将满的时候，还是存在一个读操作，虽说只是读length个长度，这个长度也是在不断减少，这样讲，好像没什么问题了。

* * *

这个也算是个对底层IO的封装了吧。发送命令等待回应，这个函数调用了上面的读函数。

    int sendCommandAndWaitForResponse(int fd, uint8 *command){    // 读取返回的状态字    uint8  response[4] = {0};    int    res = 0;    // 很正常的写操作    // 这里的长度值得分享    res = write(fd, command, HEADER_LENGTH + command[LENGTH_OFFSET]);    /fail    if( res <= 0 ) {        puts("ERROR");        return -1;    }    // 完成写之后，对串口进行读    res = readWithTimeout(fd, response, 1, 5);    if( res != 1 ) {        puts("ERROR");        return -1;    } else if ( response[0] != BOOTLOADER_OK ) {        // 串口返回的状态字，可以由PC解析发现什么问题        printf("ERROR [%02x]\n", response[0]);        return -1;    } else {        return 0;    }}

这里的前面的写操作所对应的，第三个参数是写入长度，可是这里加上的便宜，还涉及到了Cmd的内容

    res = write(fd, command, HEADER_LENGTH + command[LENGTH_OFFSET]);

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96 "文件读取")文件读取

这个算是关键函数，实现了对我们的固件的hex文件的读取 函数原型如下：

    int readHEX(const char* file, uint8* bout, unsigned long max_length, uint8* pages_used)

篇幅很长这里对函数体的重点：

    //////////////////////////////////static const uint32 HEX_DATA_OFFSET = 4;uint8* data = (linebin + HEX_DATA_OFFSET);char  line[512] = {0}; // 行指针组char  *pc;    // 字符指针char  *pline = line + 1;    // 行指针///////////////////////////////////// 这里的feof是标准库哦，学习了// 加载一行的内容while( !feof(fp) && fgets(line, sizeof(line) - 1, fp) )    {        // 行号，看得出来        line_no++;        // hex文件格式，        if( line[0] != ':' ) {            break;        }        // pline 是行内容指针        res = strlen(pline);        // 当前的行地址，加上行长度，减一，        // 也就是行末。        pc  = pline + res - 1;        // 注意， 这里的 <= ' '         // 一开始没有理解，发现空格的ascii是32，所以这里是去掉特殊字符，把他们直接给置0        while( pc > pline && *pc <= ' ' ) {            *pc-- = 0;    // 这里写0干嘛呢            res--;        }        // res 是当前行的字符剩余数，一下的情况都是非法的。        if( res & 0x01 || res > 512 || res < 10) {            fprintf(stderr, "Incorrect number of characters on line %d:%d\n", line_no, res);            return -1;        }        // CRC 校验        hex_crc = 0;                    for( pc = pline, i = 0; i<res; i+=2, pc+=2 ) {            linebin[i >> 1] = hexdec(pc);            hex_crc += linebin[i >> 1];        }        binlen = res / 2;        if( hex_crc != 0 )            ... // checksum失败                        ...                    if( binlen - (1 + 2 + 1 + hex_len + 1) != 0 )                ... // 字节数失败        if( hex_type == 0x00 )        {            f_addr  = (hex_base_addr | (hex_addr)) / 2; //PCU            if( hex_len % 4 ) {                // 数据没对齐 4字节                fprintf(stderr, "Misaligned data, line %d\n", line_no);                return -1;            } else if( f_addr >= PIC_FLASHSIZE ) {                // 编程地址超出PIC的flash地址                fprintf(stderr, "Current record address is higher than maximum allowed, line %d\n", line_no);                return -1;            }            hex_words = hex_len  / 4;            o_addr  = (f_addr / 2) * PIC_WORD_SIZE; //BYTES            for( i=0; i<hex_words; i++)            {                bout[o_addr + 0] = data[(i*4) + 2];                bout[o_addr + 1] = data[(i*4) + 0];                bout[o_addr + 2] = data[(i*4) + 1];                pages_used[ (o_addr / PIC_PAGE_SIZE) ] = 1;                o_addr    += PIC_WORD_SIZE;                num_words ++;            }        } else if ( hex_type == 0x04 && hex_len == 2) {            hex_base_addr = (linebin[4] << 24) | (linebin[5] << 16);        } else if ( hex_type == 0x01 ) {            break; //EOF        } else {            fprintf(stderr, "Unsupported record type %02x, line %d\n", hex_type, line_no);            return -1;        }    }    fclose(fp);    return num_words;}

* * *

从文件的 **循环行读取** 到 **正则** ，校验， 缓冲区偏移存储，记录值类型。完成了一个hex的读取的过程。 先是在循环中按行读取，根据前面的 **hex_type** 判断当前的读取的数据类型，当前行的实际字节数 **hex_len** 和地址，`f_addr = (hex_base_addr | (hex_addr)) / 2` 这一步，确定了，在flash的真实的地址映射，

* * *

    !feof(FILE *stream)while( !feof(fp) && fgets(line, sizeof(line) - 1, fp) )

这个循环读取的结构， 很棒，学习了

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E5%8F%91%E9%80%81%E5%9B%BA%E4%BB%B6 "发送固件")发送固件

    int sendFirmware(int fd, uint8* data, uint8* pages_used){    uint32 u_addr;    uint32 page  = 0;    uint32 done  = 0;    uint32 row   = 0;    uint8  command[256] = {0};    for( page=0; page<PIC_NUM_PAGES; page++)    {        u_addr = page * ( PIC_NUM_WORDS_IN_ROW * 2 * PIC_NUM_ROWS_IN_PAGE );        if( pages_used[page] != 1 ) {            if( g_verbose && u_addr < PIC_FLASHSIZE) {                fprintf(stdout, "Skipping page %ld [ %06lx ], not used\n", page, u_addr);            }            continue;        }        if( u_addr >= PIC_FLASHSIZE ) {            fprintf(stderr, "Address out of flash\n");            return -1;        }        //erase page        command[0] = (u_addr & 0x00FF0000) >> 16;        command[1] = (u_addr & 0x0000FF00) >>  8;        command[2] = (u_addr & 0x000000FF) >>  0;        command[COMMAND_OFFSET] = 0x01; //erase command        command[LENGTH_OFFSET ] = 0x01; //1 byte, CRC        command[PAYLOAD_OFFSET] = makeCrc(command, 5);        if( g_verbose ) {            dumpHex(command, HEADER_LENGTH + command[LENGTH_OFFSET]);        }        printf("Erasing page %ld, %04lx...", page, u_addr);        if( g_simulate == 0 && sendCommandAndWaitForResponse(fd, command) < 0 ) {            return -1;        }        puts("OK");        //write 8 rows        for( row = 0; row < PIC_NUM_ROWS_IN_PAGE; row ++, u_addr += (PIC_NUM_WORDS_IN_ROW * 2))        {            command[0] = (u_addr & 0x00FF0000) >> 16;            command[1] = (u_addr & 0x0000FF00) >>  8;            command[2] = (u_addr & 0x000000FF) >>  0;            command[COMMAND_OFFSET] = 0x02; //write command            command[LENGTH_OFFSET ] = PIC_ROW_SIZE + 0x01; //DATA_LENGTH + CRC            memcpy(&command[PAYLOAD_OFFSET], &data[PIC_ROW_ADDR(page, row)], PIC_ROW_SIZE);            command[PAYLOAD_OFFSET + PIC_ROW_SIZE] = makeCrc(command, HEADER_LENGTH + PIC_ROW_SIZE);            printf("Writing page %ld row %ld, %04lx...", page, row + page*PIC_NUM_ROWS_IN_PAGE, u_addr);            if( g_simulate == 0 && sendCommandAndWaitForResponse(fd, command) < 0 ) {                return -1;            }            puts("OK");            sleep(0);            if( g_verbose ) {                dumpHex(command, HEADER_LENGTH + command[LENGTH_OFFSET]);            }            done += PIC_ROW_SIZE;        }    }    return done;}

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E4%B8%B2%E5%8F%A3%E9%85%8D%E7%BD%AE "串口配置")串口配置

打开串口之后，对串口的参数进行配置， 这部分代码可以收藏，重用机会是挺多的。

    int configurePort(int fd, unsigned long baudrate){#ifdef WIN32    // 系统宏    DCB dcb = {0};    HANDLE hCom = (HANDLE)fd;    dcb.DCBlength = sizeof(dcb);    dcb.BaudRate = baudrate;    dcb.ByteSize = 8;    dcb.Parity = NOPARITY;    dcb.StopBits = ONESTOPBIT;    if( !SetCommState(hCom, &dcb) ){        return -1;    }    return (int)hCom;#else    struct termios g_new_tio;    memset(&g_new_tio, 0x00 , sizeof(g_new_tio));    cfmakeraw(&g_new_tio);    g_new_tio.c_cflag |=  (CS8 | CLOCAL | CREAD);    g_new_tio.c_cflag &= ~(PARENB | CSTOPB | CSIZE);    g_new_tio.c_oflag = 0;    g_new_tio.c_lflag = 0;    g_new_tio.c_cc[VTIME] = 0;    g_new_tio.c_cc[VMIN] = 1;    cfsetispeed (&g_new_tio, baudrate);    cfsetospeed (&g_new_tio, baudrate);    tcflush(fd, TCIOFLUSH);    return tcsetattr(fd, TCSANOW, &g_new_tio);#endif}

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E5%91%BD%E4%BB%A4%E8%A1%8C%E8%A7%A3%E6%9E%90 "命令行解析")命令行解析

这里一样的是，一个挺实用的部分。也算是当做代码片收藏了

    int parseCommandLine(int argc, const char** argv){    int i = 0;    // 从 1 开始解析参数，后面疯狂进行对比    for(i=1; i<argc; i++)    {        if( !strncmp(argv[i], "--hex=", 6) ) {            g_hexfile_path = argv[i] + 6;        } else if ( !strncmp(argv[i], "--dev=", 6) ) {            g_device_path = argv[i] + 6;        } else if ( !strcmp(argv[i], "--verbose") ) {            g_verbose = 1;        } else if ( !strcmp(argv[i], "--hello") ) {            g_hello_only = 1;        } else if ( !strcmp(argv[i], "--simulate") ) {            g_simulate = 1;        } else if ( !strcmp(argv[i], "--help") ) {            argc = 1; //that's not pretty, but it works :)            break;        } else {        // 没有找到对应的参数            fprintf(stderr, "Unknown parameter %s, please use pirate-loader --help for usage\n", argv[i]);            return -1;        }    }    if( argc == 1 )    {        //print usage        puts("pirate-loader usage:\n");        puts(" ./pirate-loader --dev=/path/to/device --hello");        puts(" ./pirate-loader --dev=/path/to/device --hex=/path/to/hexfile.hex [ --verbose ]");        puts(" ./pirate-loader --simulate --hex=/path/to/hexfile.hex [ --verbose ]");        puts("");        return 0;    }    return 1;}

虽说这里是很实用的 代码，不过感觉蠢蠢的，通过代码的遍历比较，感觉有什么不对，进行一个全局的标志位的操作。 不过也是很巧妙：

    strncmp(argv[i], "--hex=", 6)        g_hexfile_path = argv[i] + 6;strncmp(argv[i], "--dev=", 6)g_device_path = argv[i] + 6;

突然一想，发现这个没有空格啊。 没错，是没有空格的，参数和这个输入的本身是没有空格的，使用的是 `=` 进行的连接，所以在后面使用 `= argv[i] + 6;` 这种形式，就可以直接偏移到我们的输入内容，妙哉

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E8%BE%85%E5%8A%A9%E5%87%BD%E6%95%B0 "辅助函数")辅助函数

这部分就是一些辅助函数，进行字符转换之类的东西，虽说简单，但是写的精妙

    // 这个函数，把十六进制字符串，转为整型值unsigned char hexdec(const char* pc){    unsigned char temp;    // 从ASCII从大到小，依次来    if(pc[0]>='a'){        temp=pc[0]-'a'+10;    }else if(pc[0] >= 'A'){        temp=pc[0]-'A'+10;            }else{        temp=pc[0] - '0';    }    // 第一个字符的整型值放在这个Char的高位    // 别忘了，Char可是8位的    temp=temp<<4;    // 这里统一使用 |= 直接位或，放在低位就好    if(pc[1]>='a'){        temp|=pc[1]-'a'+10;    }else if(pc[1] >= 'A'){        temp|=pc[1]-'A'+10;            }else{        temp|=pc[1] - '0';    }    // 这里再做一次位与，一眼看去没怎么搞懂这个的作用    // 这里就十分有趣了，后面讲嘻嘻    return(temp & 0x0FF);    // 这里的一句话就是很强了，直接使用条件表达式        //return (((pc[0] >= 'A') ? ( pc[0] - 'A' + 10 ) : ( pc[0] - '0' ) ) << 4 |     //        ((pc[1] >= 'A') ? ( pc[1] - 'A' + 10 ) : ( pc[1] - '0' ) )) & 0x0FF;}

* * *

这里比较好玩的一点，就是这个 &0xff 看上去的确是没啥作用呀。实际上，这里就有了符号位这样的一个东西. 记住，我们的输入的数据只是有两位十进制，对吧，所以分别在b0~b3，和b4~b7，所以说，我们使用了这个char的8位数据，不过事实上，这里的问题是什么？？？ 在PC上面呀，char是16位的。最高位的数据我们是没有用到的的。这里存在的符号位，当然会影响我们的值得真实大小，所以使用 0&ff 实际上应该写成 & 0x00ff。这样前面用不到的地方全部清零，就没有了符号位的影响

*   byte为什么要与上0xff
    
*   // 打印缓冲区内容，没啥好讲的  
    void dumpHex(uint8* buf, uint32 len)  
    {
    
        uint32 i=0;for(i=0; i<len; i++){    printf("%02X ", buf[i]);}putchar('\n');
    
    } // CRC 的实现过程  
    uint8 makeCrc(uint8* buf, uint32 len)  
    {
    
        uint8 crc = 0, i = 0;for(i=0; i<len; i++){    crc -= *buf++;}return crc;
    
    }  
    CRC 的在这里的实现过程，简单的讲一句话，把每字节的值逐字节进行运算。最后得到一个字节的值，这样只能使得一定可能的查错。要是两个值刚好一个加一，一个减一，没办法了
    

* * *

    int openPort(const char* dev, unsigned long flags){    return open(dev, O_RDWR | O_NOCTTY | O_NDELAY | flags);}

### [](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E4%B8%BB%E5%87%BD%E6%95%B0%E5%85%A5%E5%8F%A3 "主函数入口")主函数入口

`int main (int argc, const char** argv)` 都是逻辑代码，所以这里只是贴出部分的有趣的代码

    // 256k 的优雅的分配bin_buff = (uint8*)malloc(256 << 10); //256kBif( !bin_buff ) {    fprintf(stderr, "Could not allocate 256kB buffer\n");        goto Error;}memset(bin_buff, 0xFFFFFFFF, (256 << 10));

* * *

**设备握手** 这里的握手过程，典型的业务代码吧。发送握手，接收，之后判断 XD

    #define BOOTLOADER_HELLO_STR "\xC1"//send HELLOres = write(dev_fd, BOOTLOADER_HELLO_STR, 1);res = readWithTimeout(dev_fd, buffer, 4, 3);if( res != 4 || buffer[3] != BOOTLOADER_OK ) {    puts("ERROR");    fprintf(stderr, "No reply from the bootloader, or invalid reply received: %d\n", res);    fprintf(stderr, "Please make sure that PGND and PGC are connected, replug the device and try again\n");    goto Error;}puts("OK\n"); //extra LF for spacingprintf("Device ID: %s [%02x]\n", (buffer[0] == 0xD4) ? "PIC24FJ64GA002" : "UNKNOWN", buffer[0]);printf("Bootloader version: %d,%02d\n", buffer[1], buffer[2]);if( buffer[0] != 0xD4 ) {    fprintf(stderr, "Unsupported device (%02x:UNKNOWN), only 0xD4 PIC24FJ64GA002 is supported\n", buffer[0]);    goto Error;}

* * *

**错误处理** 很多地方到处宣扬着 goto 有害论.实际上，在C这个异常处理尚不健全的情况下。使用Goto 实现异常处理的方法，是十分OK的。 源程序的后面，实现了两个异常处理的标号：

    Finished:    if( bin_buff ) {         free( bin_buff );    }    close(dev_fd);    return 0;Error:    if( bin_buff ) {        free( bin_buff );    }    if( dev_fd >= 0 ) {        close(dev_fd);    }    return -1;

[](https://www.diglp.xyz/2018/06/25/HackWare_PC_loader/#%E5%90%8E "后")后
-----------------------------------------------------------------------

熟读代码三千行，不会编程也会背。2333