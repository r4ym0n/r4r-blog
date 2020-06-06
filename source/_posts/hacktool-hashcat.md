---
title: HackTool HashCat
url: 528.html
id: 528
date: 2018-05-28 00:00:00
tags:
---

[](https://www.diglp.xyz/2018/05/28/HackTool_HashCat/#%E5%89%8D "前")前
---------------------------------------------------------------------

这个 catalogue ,用来记录一些炫酷好玩的HackTool. 属于没有什么技术深度,又是比较好玩炫酷的东西. 作为第一篇,这里就介绍一下,这个 **HashCat** 的项目吧.

> HashCat 项目地址

[](https://www.diglp.xyz/2018/05/28/HackTool_HashCat/#What-is-HashCat "What is HashCat")What is HashCat
-------------------------------------------------------------------------------------------------------

哈希猫??? Cat 在美英里面,是工具集的意思了,所以　Hashcat是一款开源密码恢复工具，我们可以用它来攻击160多种哈希类型的密码, 通过的就是我们的 **爆破(brute force)** 工具. 其具体的爆破过程, 使用GPU算力,对所有的主流hash 都有很好的支持. 这里,做一个简单的基准测试:

    PS C:\Users\R4y\hashcat-4.1.0> .\hashcat64.exe -b -w 3hashcat (v4.1.0) starting in benchmark mode...OpenCL Platform #1: Advanced Micro Devices, Inc.================================================* Device #1: Tahiti, 2393/3072 MB allocatable, 28MCU* Device #2: AMD FX(tm)xxxx Processor, skipped.Benchmark relevant options:===========================* --workload-profile=3Hashmode: 0 - MD5Speed.Dev.#1.....:   955.9 MH/s (59.76ms) @ Accel:128 Loops:64 Thr:256 Vec:1Hashmode: 100 - SHA1Speed.Dev.#1.....:  1082.1 MH/s (53.82ms) @ Accel:128 Loops:64 Thr:256 Vec:1Hashmode: 1400 - SHA-256Speed.Dev.#1.....:   618.6 MH/s (94.45ms) @ Accel:128 Loops:64 Thr:256 Vec:1Hashmode: 1700 - SHA-512Speed.Dev.#1.....: 49115.4 kH/s (74.25ms) @ Accel:32 Loops:16 Thr:256 Vec:1Hashmode: 2500 - WPA/WPA2 (Iterations: 4096)Speed.Dev.#1.....:   120.3 kH/s (53.15ms) @ Accel:128 Loops:32 Thr:256 Vec:1Hashmode: 1000 - NTLMSpeed.Dev.#1.....:  2314.9 MH/s (101.20ms) @ Accel:256 Loops:128 Thr:256 Vec:1Hashmode: 3000 - LMSpeed.Dev.#1.....:  7699.6 MH/s (60.40ms) @ Accel:64 Loops:1024 Thr:256 Vec:1Hashmode: 5500 - NetNTLMv1 / NetNTLMv1+ESSSpeed.Dev.#1.....:  2272.6 MH/s (102.62ms) @ Accel:256 Loops:128 Thr:256 Vec:1Hashmode: 5600 - NetNTLMv2Speed.Dev.#1.....: 22629.1 kH/s (80.76ms) @ Accel:32 Loops:8 Thr:256 Vec:1Hashmode: 1500 - descrypt, DES (Unix), Traditional DESSpeed.Dev.#1.....:   272.7 MH/s (53.13ms) @ Accel:2 Loops:1024 Thr:256 Vec:1...

发现其综合的计算能力,还是很强的, 这次我们的 WPA 也可以到 120kH/s 也就是12W的级别. 而且根据:

    > hashcat help

里可见,其支持的种类还是相当之多的.特别是,这里,还有这几个选项.所以,密码一定要足够强~

    11300 | Bitcoin/Litecoin wallet.dat                      | Password Managers12700 | Blockchain, My Wallet                            | Password Managers15200 | Blockchain, My Wallet, V2                        | Password Managers16600 | Electrum Wallet (Salt-Type 1-3)                  | Password Managers13400 | KeePass 1 (AES/Twofish) and KeePass 2 (AES)      | Password Managers15500 | JKS Java Key Store Private Keys (SHA1)           | Password Managers15600 | Ethereum Wallet, PBKDF2-HMAC-SHA256              | Password Managers15700 | Ethereum Wallet, SCRYPT                          | Password Managers16300 | Ethereum Pre-Sale Wallet, PBKDF2-HMAC-SHA256     | Password Managers

[](https://www.diglp.xyz/2018/05/28/HackTool_HashCat/#Start-From-Now "Start From Now")Start From Now
----------------------------------------------------------------------------------------------------

这里, 就使用 HC 来破解一下,我们的 Wifi 密码吧. 这里的离线密码文件是 ***.cap** . 使用 AirCrack-ng 工具集拿到的,具体的过程,后面闲的话,可能是会有教程的.

    hashcat  –help                 #查看帮助文档General:-m   （–hash-type=NUM）         #hash种类，下面有列表，后面跟对应数字-D     –opencl-device-types     | Str| OpenCL device-types to use, separate with comma   #选择用CPU还是GPU来破解-a    （–attack-mode=NUM）          #破解模式，下面也有列表

attack-mode：

*   0 = Straight （字典破解）
*   1 = Combination （组合破解）
*   2 = Toggle-Case
*   3 = Brute-force （掩码暴力破解）
*   4 = Permutation （组合破解）
*   5 = Table-Lookup

根据 help 文档,我们的启动参数如下

    ./hashcat64.exe -a 3  -m 2500 "J:\h60.hccap" --session=all -o "C:\result\asd.found2500.txt" --outfile-format=2 -w 3  -i --increment-min=8 --increment-max=10 -1 ?d?d?d?d?d?d?d?d?d?d ?1?1?1?1?1?1?1?1?1?d

参数说明, 可以看出 ,

    -m [type] [in_file] -o [out_file] [optional]

这里使用.8位到十位的递增模式, 其掩码为 ?d?d?d… 这里的D就是指的数字的意思.一个回车执行破解过程:现在是破解中

    Session..........: allStatus...........: ExhaustedHash.Type........: WPA/WPA2Hash.Target......: Huawei_H60_9a79 (AP:88:28:b3:f6:2c:e6 STA:48:8a:d2:7b:8b:6e)Time.Started.....: Fri Jun 08 04:07:25 2018 (2 hours, 9 mins)Time.Estimated...: Fri Jun 08 06:17:11 2018 (0 secs)Guess.Mask.......: ?1?1?1?1?1?1?1?1?1 [9]Guess.Charset....: -1 ?d?d?d?d?d?d?d?d?d?d, -2 Undefined, -3 Undefined, -4 UndefinedGuess.Queue......: 1/2 (50.00%)Speed.Dev.#1.....:   128.4 kH/s (52.72ms) @ Accel:128 Loops:32 Thr:256 Vec:1Recovered........: 0/1 (0.00%) Digests, 0/1 (0.00%) SaltsProgress.........: 1000000000/1000000000 (100.00%)Rejected.........: 0/1000000000 (0.00%)Restore.Point....: 100000000/100000000 (100.00%)Candidates.#1....: 689009973 -> 676464973HWMon.Dev.#1.....: Temp: 65c Fan: 51% Util: 99% Core:1050MHz Mem:1250MHz Bus:16

找到Key之后: 这里的密码选取是 1234567890 所以跑数据的过程,还是比较久的.

    Session..........: allStatus...........: CrackedHash.Type........: WPA/WPA2Hash.Target......: Huawei_H60_9a79 (AP:88:28:b3:f6:2c:e6 STA:48:8a:d2:7b:8b:6e)Time.Started.....: Fri Jun 08 06:17:12 2018 (2 hours, 8 mins)Time.Estimated...: Fri Jun 08 08:25:48 2018 (0 secs)Guess.Mask.......: ?1?1?1?1?1?1?1?1?1?d [10]Guess.Charset....: -1 ?d?d?d?d?d?d?d?d?d?d, -2 Undefined, -3 Undefined, -4 UndefinedGuess.Queue......: 2/2 (100.00%)Speed.Dev.#1.....:   128.5 kH/s (53.14ms) @ Accel:128 Loops:32 Thr:256 Vec:1Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) SaltsProgress.........: 991821824/10000000000 (9.92%)Rejected.........: 0/991821824 (0.00%)Restore.Point....: 99090432/1000000000 (9.91%)Candidates.#1....: 1890099734 -> 1549567890HWMon.Dev.#1.....: Temp: 65c Fan: 52% Util: 99% Core:1050MHz Mem:1250MHz Bus:16

上面的过程 ,对于 10^10 的尝试在 2Hr 里有了结果. 结果保存在我们的目标文件夹中,密码是1234567890.

* * *

**注意**

我们通过, HashCat 使用的是 hccap 文件,我们一般 TCPDUMP 导出的是 Cap的内容, 如果直接使用的话,是会报错的.

    Hashfile 'J:\caca.cap': Invalid hccapx signature

这里需要对包的格式进行转换.

> 官方 Issue

使用其,附带的工具. **cap2hccapx** 对其进行转换

    cap2hccapx <out.cap> -J <out.hccap> 

* * *

参考文档

*   hashcat官网
*   Cracking WPA/WPA2 with oclHashcat
*   在线cap转为hccap格式
*   HashCat 项目地址