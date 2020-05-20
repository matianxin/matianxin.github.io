---
title: CTF中Crypto题目的常见题型
tags: [ctf]
categories: CTF
description: CTF中Crypto题目的常见题型
date: 2020/5/20 11:00:00
---

### CTF中Crypto题目的常见题型

- BugKu 提供了很多加解密工具，链接：http://tool.bugku.com/

- SSL在线工具网址提供了很多工具，链接：http://www.ssleye.com/

- text bin hex base64 dec 转码工具：https://conv.darkbyte.ru/

#### 1. 摩斯密码 / Morse编码
摩尔斯电码(Morse code)是一种时通时断的信号代码，通过不同的排列 顺序来表达不同的英文字母、数字和标点符号。是由美国人艾尔菲德·维 尔(Alfred Lewis Vail)与萨缪尔·摩尔斯(Samuel Finley Breese Morse) 在1836年发明。由点（·）和划（-）组成。

特征：点和横的组合。

在线工具：
http://www.zhongguosou.com/zonghe/moErSiCodeConverter.aspx
http://tool.bugku.com/mosi/

例如：-... -.- -.-. - ..-. -- .. ... -.-.

解密结果：BKCTFMISC

--/---/.-./.../.
解密结果：MORSE

#### 2.ASCII编码
ASCII(American Standard Code for Information Interchange，美国信息交换标准代码)是基于拉丁字母的一套 电脑编码系统，主要用于显示现代英语和其他西欧语言。它是现今最通用的单字节编码系统，并等同于国际标准ISO/IEC 646。

例如：72 105 65 115 99 105 105
解密结果：H i A s c i i

特点：一般常用字符为0-9（48-57）、A-Z（65-90）、a-z（97-122）、空格（32）

在线工具：http://www.ab126.com/goju/1711.html

#### 3.Tap Code敲击码
敲击码(Tap code)是一种以非常简单的方式对文本信息进行 编码的方法。因该编码对信息通过使用一系列的点击声音来编 码而命名，敲击码是基于 5 ×5 方格波利比奥斯方阵来实现的， 不同点是是用 K 字母被整合到 C 中。
   1  2  3  4  5
1  A  B C/K D  E
2  F  G  H  I  J
3  L  M  N  O  P
4  Q  R  S  T  U
5  V  W  X  Y  Z

例如：2,3 1,5 3,1 3,1 3,4
.. .../. ...../... ./... ./... ..../
解密结果：H E L L O

ps. 当出现1,3时，对比C/K，选择更合适的结果

#### 4.Base编码
Base64是网络上最常见的用于传输8Bit字节码的编码方式之一，Base64 就是一种基于64(65)个可打印字符来表示二进制数据的方法。 a-z、A-Z、0-9、符号“+”、“/”（再加上作为补位的"="，实际上是 65个字符） 。

Base xx 中的 xx 表示的是采用多少个字符进行编码，比如说 base64 就是采用 64 个字符编码，由于 2 的 6 次方等于 64，所以每 6 个比特 为一个单元，对应某个可打印字符。

例如：d2VsY29tZQ==   
解密结果：welcome

特点：base64 结尾可能会有=号，但最多有2个 。 base32 结尾可能会有=号,最多有 3 个等号。
根据 base 的不同，字符集会有所限制 。 有可能需要自己加等号。

在线工具：http://tool.chinaz.com/tools/base64.aspx

#### 5.URL编码
URL编码,又叫百分号编码，是统一资源定位(URL)编码方式。URL地址（常说网址）规定了常用地数字，字母可以直接使用，另外一批作为特殊用户字符也可以直接用（/,:@等），剩下的其它所有字符必须通过%xx编码 处理。现在已经成为一种规范了，基本所有程序语言都有这种编码，如js： 有encodeURI、encodeURIComponent，PHP有urlencode、urldecode等。编 码方法很简单，在该字节ascii码的的16进制字符前面加%.
如空格字符， ascii码是32，对应16进制是‘20’，那么urlencode编码结果是:%20。

例如：URL%E7%BC%96%E7%A0%81
解密结果：URL编码

特点：存在大量的%
在线工具：https://tool.oschina.net/encode?type=4

#### 6.Unicode编码
Unicode（统一码、万国码、单一码）是计算机科学领域里的一项业界 标准,包括字符集、编码方案等。Unicode是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的二进制编码，以满足跨语言、跨平台进行文本转换、处理的要求。1990年开始研发，1994年正式公布。

例如：&#72;&#101;&#108;&#108;&#111;&#67;&#84;&#70;
解密结果：HelloCTF
在线工具：https://tool.oschina.net/encode

#### 7.jsfuck
JSFuck 可以让你只用6个字符[ ]( )!+来编写 JavaScript 程序
在线工具：https://www.bugku.com/tools/jsfuck/

#### 8.brainfuck
Brainfuck 是一种极小化的计算机语言，按照"Turing complete(完整图灵机) "思想设计的语言，它的主要设计思路是:用最小的概念实现一种“简单”的语 言，BrainFXXk语言只有八种符号，
所有的操作都由这八种符号  > < + - . , [ ] 的组合来完成。

例如：+++++ +++++ [->++ +++++ +++<] >++++ .---. +++++ ++..+ ++.<+ +++++ ++[->
----- ---<] >---. <++++ ++++[ ->+++ +++++ <]>++ +++++ ++++. ----- ---.+
++.-- ----. ----- ---.< 

解密结果：hello,world
在线工具：https://www.splitbrain.org/services/ook

#### 9.凯撒密码
凯撒密码（Caesar）加密时会将明文中的每个字母都按照其在字母表中的顺序向后（或向前）移动固定数目（循环移动）作为密文。例如，当偏移量是左移 3 的时候（解密时 的密钥就是 3）。
使用时，加密者查找明文字母表中需要加密的消息中的每一个字母所在位置，并且写下密文字母表中对应的字母。需要解密的人则根据事先已知的密钥反过来操作，得到原来的明文。

根据偏移量的不同，还存在若干特定的恺撒密码名称：
偏移量为 10：Avocat （A→K）
偏移量为 13：ROT13
偏移量为 -5：Cassis （K 6）
偏移量为 -6：Cassette （K 7）

例如：Uryyb,Jrypbzr
解密结果：Hello,Welcome
在线工具：http://www.mxcz.net/tools/rot13.aspx

#### 10.栅栏密码
特征：大小写和字符，其实就是分组替换加密

例如：一只小羊翻过了2个栅栏

KYsd3js2E{a2jda}

解密结果：KEY{sad23jjdsa2}
在线工具：http://tool.bugku.com/jiemi/

#### 11.Ook 密码
特征：Brainfuck 类型密码，密文由 Ook 和 三种标点 . ! ? 构成，不见得都得用上，有的是Ook，有的没有Ook只有标点。

在线工具：https://tool.bugku.com/brainfuck/

#### 12.类栅栏密码
特征：会给你一串乱序的密文 C，同时给你从1 到 n 的一串乱序的数，找下 C 对于 n 的最大公约数，然后将他们重新排序。
动手画一画就出来了

BugKu中的一道题：
lf5{ag024c483549d7fd@@1}     ， 一张纸条上凌乱的写着2 1 6 5 3 4

2   1   6   5   3   4
l   f   5   {   a   g
0   2   4   c   4   8
3   5   4   9   d   7
f   d   @   @   1   }
再按照123456的顺序数从第一行排到最后一行

flag{52048c453d794df1}@@

提交 flag发现不对，这两个@@有点奇怪，去掉@就成功了，那么一般这种字符都是迷惑人的。

#### 13.由0和1组成的摩斯密码

特征：密文由0和1构成，可能由空格、tab或则其他字符分割，去掉这些分割后，0和1的总数是5的倍数
将 0 和 1 替换为 . 或 - ，多试几次就可能有发现。

BugKu中的一道题：

0010 0100 01 110 1111011 11 11111 010 000 0 001101 1010 111 100 0 001101 01111 000 001101 00 10 1 0 010 0 000 1 01111 10 11110 101011 1111101

把0换成点 .    1换成 -    后：
..-. .-.. .- --. ----.-- -- ----- .-. ... . ..--.- -.-. --- -.. . ..--.- .---- ... ..--.- .. -. - . .-. . ... - .---- -. ----. -.-.-- -----.-
在线工具解密一下，得到：

FLAG%u7bM0RSE_CODE_1S_INTEREST1N9!%u7d

结果有点奇怪，但八九不离十了。把%u7b 换成 {  把%u7d 换成 }，然后大小写切换

flag{m0rse_code_1s_interest1n9!}

#### 14.键盘格子密码
特征：由三到四个英文字母或数字为一组。

在键盘上找对应的键位，中间围起来的就是密文（这能能想得出。。。）

例如：r5yG lp9I BjM tFhBT6uh y7iJ QsZ bhM
解密结果：t o n g y u a n

#### 15.托马斯杰斐逊 转轮密码
特征：给你一个密码表，n行的26个字母，key是 1 - n 的数列， 密文是 n 个英文字母
根据 key 找对应行的密码表，然后在密码表上找密文字母，以这个字母为开头，重新排序。
1： <ZWAXJGDLUBVIQHKYPNTCRMOSFE <
2： <KPBELNACZDTRXMJQOYHGVSFUWI <
3： <BDMAIZVRNSJUWFHTEQGYXPLOCK <
4： <RPLNDVHGFCUKTEBSXQYIZMJWAO <
5： <IHFRLABEUOTSGJVDKCPMNZQWXY <
6： <AMKGHIWPNYCJBFZDRUSLOQXVET <
7： <GWTHSPYBXIZULVKMRAFDCEONJQ <
8： <NOZUTWDCVRJLXKISEFAPMYGHBQ <
9： <QWATDSRFHENYVUBMCOIKZGJXPL <
10： <WABMCXPLTDSRJQZGOIKFHENYVU <
11： <XPLTDAOIKFZGHENYSRUBMCQWVJ <
12： <TDSWAYXPLVUBOIKZGJRFHENMCQ <
13： <BMCSRFHLTDENQWAOXPYVUIKZGJ <
14： <XPHKZGJTDSENYVUBMLAOIRFCQW <

密钥： 2,5,1,3,6,4,9,7,8,14,10,13,11,12
密文：HCBTSXWCRQGL ES

在第 2 行密码表中找H开头的字母，然后以H开头再到尾过一遍，以此类推，整理出另一个密码表：

HGVSFUWIKPBELNACZDTR X MJQOY
CPMNZQWXYIHFRLABEUOT S GJVDK
BVIQHKYPNTCRMOSFEZWA X JGDLU
TEQGYXPLOCKBDMAIZVRN S JUWFH
SLOQXVETAMKGHIWPNYCJ B FZDRU
XQYIZMJWAORPLNDVHGFC U KTEBS
WATDSRFHENYVUBMCOIKZ G JXPLQ
CEONJQGWTHSPYBXIZULV K MRAFD
RJLXKISEFAPMYGHBQNOZ U TWDCV
QWXPHKZGJTDSENYVUBML A OIRFC
GOIKFHENYVUWABMCXPLT D SRJQZ
LTDENQWAOXPYVUIKZGJB M CSRFH
ENYSRUBMCQWVJXPLTDAO I KFZGH
SWAYXPLVUBOIKZGJRFHE N MCQTD

然后在这里找一些比较明显的（语句通顺的）话，就是flag。

XSXSBUGKUADMIN
提交不对的话就大小写换一下：xsxsbugkuadmin

这个其实可以写个程序出来，遍历密码表即可。
```python
# Rotor cipher decoder
# parameter input
rotor = [
    "ZWAXJGDLUBVIQHKYPNTCRMOSFE", "KPBELNACZDTRXMJQOYHGVSFUWI",
    "BDMAIZVRNSJUWFHTEQGYXPLOCK", "RPLNDVHGFCUKTEBSXQYIZMJWAO",
    "IHFRLABEUOTSGJVDKCPMNZQWXY", "AMKGHIWPNYCJBFZDRUSLOQXVET",
    "GWTHSPYBXIZULVKMRAFDCEONJQ", "NOZUTWDCVRJLXKISEFAPMYGHBQ",
    "QWATDSRFHENYVUBMCOIKZGJXPL", "WABMCXPLTDSRJQZGOIKFHENYVU",
    "XPLTDAOIKFZGHENYSRUBMCQWVJ", "TDSWAYXPLVUBOIKZGJRFHENMCQ",
    "BMCSRFHLTDENQWAOXPYVUIKZGJ", "XPHKZGJTDSENYVUBMLAOIRFCQW"
]
cipher = "HCBTSXWCRQGLES"

key = [2, 5, 1, 3, 6, 4, 9, 7, 8, 14, 10, 13, 11, 12]

tmp_list=[]

for i in range(0, len(rotor)):
    tmp=""
    k = key[i] - 1
    for j in range(0, len(rotor[k])):
        if cipher[i] == rotor[k][j]:
            if j == 0:
                tmp=rotor[k]
                break
            else:
                tmp=rotor[k][j:] + rotor[k][0:j]
                break
    tmp_list.append(tmp)
# print(tmp_list)

message_list = []
for i in range(0, len(tmp_list[i])):
    tmp = ""
    for j in range(0, len(tmp_list)):
        tmp += tmp_list[j][i]
    message_list.append(tmp)
print(message_list)
```

#### 16.Base91编码
特征：基本上是键盘上所有可打印的 ASC II 字符
```php
（0x21-0x7E），A-Z、a-z、1 - 9、 !@#$%^&*()_+-={}[]|\:;"'<,>.?/~·
```
之类的。

参考： http://base91.sourceforge.net/ （里面有工具源码）
在线工具：http://ctf.ssleye.com/base91.html

#### 17.Linux系统的 shadow 文件格式
特征：就是Linux的shadow文件格式
工具：Kali Linux 中的 John
root:$6$HRMJoyGA$26FIgg6CU0bGUOfqFB0Qo9AE2LRZxG8N3H.3BK8t49wGlYbkFbxVFtGOZqVIq3q
Q6k0oetDbn2aVzdhuVQ6US.:17770:0:99999:7:::
Linux的 /etc/shadow 文件存储了该系统下所有用户口令相关信息，只有 root 权限可以查看，用户口令是以 Hash + Salt 的形式保护的。
每个字段都用 “$” 或“:”符号分割；
第一个字段是用户名，如root ；
第二个字段是哈希算法，比如 6 代表SHA-512，1 代表 MD5；
第三个字段是盐，比如上面的 HRMJoyGA
第四个字段是口令+盐加密后的哈希值
后面分别是密码最后一次修改日期、密码的两次修改间隔时间（和第三个字段相比）、密码的有效期(和第三个字段相比）、密码修改到期前的警告天数（和第五个字段相比）、密码过期后的宽限天数（和第五个字段相比）、账号失效时间，这里不太重要要；

直接跑 John 试试
john shadow
如果解开了，加 --show 查看解密口令
john --show shadow

#### 18.ZIP 伪加密
特征：一个ZIP压缩包，建议先读一下Zip文件解析与利用，里面提到：
一格zip文件有三个部分组成：压缩源文件数据区 + 压缩源文件目录区 + 压缩源文件目录结束标志；
每一部分都由明文 PK （50 4B）开始；
这是三个头标记，主要看第二个；
压缩源文件数据区：
50 4B 03 04：这是头文件标记
压缩源文件目录区：
50 4B 01 02：目录中文件文件头标记
后面两位（如 1F 00 或 3F 00）：压缩使用的 pkware 版本 
再后面两位（如 14 00）：解压文件所需 pkware 版本 
再后面两位：全局方式位标记（有无加密，这个更改这里进行伪加密，00 00 是无密码，改为09 00打开就会提示有密码了）
压缩源文件目录结束标志 ：
50 4B 05 06：目录结束标记 
工具：ZipCenOp.jar 或 WinHex
e.g. BugKu 上的一道题 https://ctf.bugku.com/challenges#zip%E4%BC%AA%E5%8A%A0%E5%AF%86
1. 使用 ZipCenOp.jar
链接：https://pan.baidu.com/s/1yDcWWhY0lSlBArEJ4S6qUw 
提取码：g3it
（需要java环境）windows下在cmd中输入：
java -jar ZipCenOp.jar r xxx.zip

直接破解ZIP包；
2. 使用 WinHex
我们用winhex打开压缩包，搜索504B，点击第二个504B，从后面找第七、八位，发现是 09 00，改为 00 00 即可。
这种方式只适用于ZIP的伪加密，真加密了此方法不适用。

#### 19.RSA 加解密
特征：给一些 RSA 算法的参数，然后加密\解密消息获取 flag。
说一下 RSA 算法模式：
分三部分，密钥生成、加密、解密：
a) 密钥生成
    1) 选取两个长度为 K 的素数 P 和 Q，计算 N = P * Q；
    2) 计算 phi(N) = (P-1) * (Q-1)， 其中 phi(N) 是 Z_(N^*) 的阶；
    3) 随机选取一个int整数 e ∈ [ 1, phi(N) - 1 ]，使得 gcd( e, phi(N)) = 1；
    4) 计算它的逆 d，使得 [ e * d mod phi(N) ] = 1；
    5) 输出私钥和公钥 sk = ( N, d ), pk = ( N, e )；
b) 加密
    c = m^e mod(N)
c) 解密
    m = c^d mod(N)
工具：RsaCtfTool （用于输出RSA参数） libnum （用于密文的计算）
具体参考Bugku-加密-rsa(WriteUp)

#### 20.仿射密码 affine cipher
特征：可能会提示你是放射密码 affine，公式： y = k * x + b mod 26 （跟一元一次函数似的）
后面的取模，如果都是英文字母的话是26，不排除有其他形式，比如ASC II 什么的，取模可能会换。

工具：python代码
```python

# Q: y = 17x-8 flag{szzyfimhyzd}
flag = "szzyfimhyzd"

flaglist = []

for i in flag:
    flaglist.append(ord(i)-97)

flags = ""
for i in flaglist:
    for j in range(0,26):
        c = (17 * j - 8) % 26
        if(c == i):
            flags += chr(j+97)
print('flag{'+ flags + '}')
```

#### 21.进制转换
特征：二进制 b开头，八进制 o开头，十进制 d开头，十六进制 x开头

BugKu里的一道题：
d87 x65 x6c x63 o157 d109 o145 b100000 d116 b1101111 o40 x6b b1100101 b1101100 o141 d105 x62 d101 b1101001 d46 o40 d71 x69 d118 x65 x20 b1111001 o157 b1110101 d32 o141 d32 d102 o154 x61 x67 b100000 o141 d115 b100000 b1100001 d32 x67 o151 x66 d116 b101110 b100000 d32 d102 d108 d97 o147 d123 x31 b1100101 b110100 d98 d102 b111000 d49 b1100001 d54 b110011 x39 o64 o144 o145 d53 x61 b1100010 b1100011 o60 d48 o65 b1100001 x63 b110110 d101 o63 b111001 d97 d51 o70 d55 b1100010 d125 x20 b101110 x20 b1001000 d97 d118 o145 x20 d97 o40 d103 d111 d111 x64 d32 o164 b1101001 x6d o145 x7e

```python
s = 'd87 x65 x6c x63 o157 d109 o145 b100000 d116 b1101111 o40 x6b b1100101 b1101100 o141 d105 x62 d101 b1101001 d46 o40 d71 x69 d118 x65 x20 b1111001 o157 b1110101 d32 o141 d32 d102 o154 x61 x67 b100000 o141 d115 b100000 b1100001 d32 x67 o151 x66 d116 b101110 b100000 d32 d102 d108 d97 o147 d123 x31 b1100101 b110100 d98 d102 b111000 d49 b1100001 d54 b110011 x39 o64 o144 o145 d53 x61 b1100010 b1100011 o60 d48 o65 b1100001 x63 b110110 d101 o63 b111001 d97 d51 o70 d55 b1100010 d125 x20 b101110 x20 b1001000 d97 d118 o145 x20 d97 o40 d103 d111 d111 x64 d32 o164 b1101001 x6d o145 x7e'
ss = s.split()
sss = []
print(ss)
for i in ss:
    if i[0] == 'd':
        i = i[1:]
        i = int(i,10)
        i = chr(i)
        sss.append(i)
    elif i[0] == 'x':
        i = i[1:]
        i = int(i,16)
        i = chr(i)
        sss.append(i)
    elif i[0] == 'o':
        i = i[1:]
        i = int(i,8)
        i = chr(i)
        sss.append(i)
    elif i[0] == 'b':
        i = i[1:]
        i = int(i,2)
        i = chr(i)
        sss.append(i)
print(sss)
flag = ''.join(sss)
print(flag)
```
