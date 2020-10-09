---
title: CTF常见的题型
tags: [ctf]
categories: CTF
description: CTF常见的题型
date: 2020/5/19 09:00:00
---

### CTF常见的题型

CTF比赛通常包含的题目类型有七种，包括MISC、PPC、CRYPTO、PWN、REVERSE、WEB、STEGA。

1.**MISC**(Miscellaneous)√ 类型，即**安全杂项**，题目或涉及流量分析、电子取证、人肉搜索、数据分析等等。
杂项介绍
Miscellaneous简称MISC，意思是杂项，混杂的意思。
杂项大致有几种类型：
- 1.隐写，包括图片/音频/视频/其他等
- 2.压缩包处理，包括zip/rar等
- 3.流量分析，包括协议/文件/usb/wifi等
- 4.攻击取证，包括日志分析/内存分析/文件镜像等
- 5.基础
- 6.其它
处理方法：
1.查看文件类型：
- file、 010Editor
2.文件分离：
- Binwalk、 foremost、 dd、 010Editor
3.文件合并：
- linux环境、 windows环境、 python脚本


2.PPC(Professionally Program Coder)类型，即编程类题目，题目涉及到编程算法，相比ACM较为容易。

3.**CRYPTO**(Cryptography)√ 类型，即**密码学**，题目考察各种加解密技术，包括古典加密技术、现代加密技术甚至出题者自创加密技术。
- Morse编码
- ASCII编码
- Tap Code敲击码
- Base编码
- URL编码
- Unicode编码
- 凯撒密码
- RSA 加解密

4.**PWN**√ 类型，PWN在黑客俚语中代表着**攻破、取得权限**，多为溢出类题目。
- Pwn 是其中的一类重要题型，主要考查选手二进制逆向分析、漏洞挖掘以及 Exploit 编写的能力。
- PWN是一个黑客语法的俚语词，自"own"这个字引申出来的，这个词的含意在于，玩家在整个游戏对战中处在胜利的优势，或是说明竞争对手处在完全惨败的 情形下，这个词习惯上在网络游戏文化主要用于嘲笑竞争对手在整个游戏对战中已经完全被击败（例如："You just got pwned!"）。有一个非常著名的国际赛事叫做Pwn2Own，相信你现在已经能够理解这个名字的含义了，即通过打败对手来达到拥有的目的。
- CTF中PWN题型通常会直接给定一个已经编译好的二进制程序（Windows下的EXE或者Linux下的ELF文件等），然后参赛选手通过对二进制程 序进行逆向分析和调试来找到利用漏洞，并编写利用代码，通过远程代码执行来达到溢出攻击的效果，最终拿到目标机器的shell夺取flag

5.**REVERSE**√ 类型，即**逆向**工程，题目涉及到软件逆向、破解技术。
- 通过Ollydbg/IDA Pro/PEiD等工具在逆向分析中的基本使用方法，并通过动态调试和静态分析两种方法来找到程序密码。

6.STEGA(Steganography)类型，即隐写术，题目的Flag会隐藏到图片、音频、视频等各类数据载体中供参赛者获取。

7.**WEB**√ 类型，即题目会涉及到常见的Web漏洞，诸如注入、XSS、文件包含、代码执行等漏洞。
工具集
- 基础工具：Burpsuite，python，firefox(hackbar，foxyproxy，user-agent，swither等)
- 扫描工具：nmap，nessus，openvas
- sql注入工具：sqlmap等
- xss平台：xssplatfrom，beef
- 文件上传工具：cknife
- 文件包含工具：LFlsuite
- 暴力破解工具：burp暴力破解模块，md5Crack，hydra

常用套路总结
- 直接查看网页源码，即可找到flag
- 查看http请求/响应，使用burp查看http头部信息，修改或添加http请求头（referer--来源伪造，x-forwarded-for--ip伪造，user-agent--用户浏览器，cookie--维持登陆状态，用户身份识别）
- web源码泄漏：vim源码泄漏(线上CTF常见)，如果发现页面上有提示vi或vim，说明存在swp文件泄漏，地址：/.index.php.swp或index.php~
- 恢复文件 vim -r index.php。备份文件泄漏，地址：index.php.bak，www.zip，htdocs.zip，可以是zip，rar，tar.gz，7z等
- .git源码泄漏，地址：http://www.xxx.com/.git/config，工具：GitHack，dvcs-ripper
- svn导致文件泄漏，地址：http://www/xxx/com/.svn/entries，工具：dvcs-ripper，seay-svn
- 编码和加解密，各类编码和加密，可以使用在线工具解密，解码
- windows特性，短文件名，利用～字符猜解暴露短文件/文件夹名，如backup-81231sadasdasasfa.sql的长文件，
其短文件是backup1.sql，iis解析漏洞，绕过文件上传检测
- php弱类型
- 绕waf，大小写混合，使用编码，使用注释，使用空字节


当前我们环境中的题型为：

1.CRYPTO：
字符串(可能很长) 或 上传一个文件

2.PWN：
每一道题Docker container对外开放一个端口映射

3.MISC:
全部为 上传一个文件

4.Reserve：
全部为 上传一个文件

5.Web：
每一道题Docker container对外开放一个端口映射
