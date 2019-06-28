---
title: 压力测试工具ab使用方法
tags: [tool]
categories: 工具
description: 压力测试工具ab使用及安装方法
date: 2019/6/28 11:00:00
---

### 安装

在CentOS7-mini环境下，ab运行需要依赖apr-util，httpd-tools包，安装命令为：
```bash
yum install apr-util httpd-tools -y
```

### 执行测试
```php
#vi post.txt
uuid=n-vp0eo8lm0c&
```

```java
ab -c 10 -n 10 -p 'post.txt' -T 'application/x-www-form-urlencoded'  http://192.100.200.140:8000/api/instance/show
```

```php
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.100.200.140 (be patient).....done


Server Software:        nginx/1.9.9
Server Hostname:        192.100.200.140
Server Port:            8000

Document Path:          /api/instance/show
Document Length:        23 bytes

Concurrency Level:      10
Time taken for tests:   1.378 seconds
Complete requests:      10
Failed requests:        6
   (Connect: 0, Receive: 0, Length: 6, Exceptions: 0)
Write errors:           0
Non-2xx responses:      3
Total transferred:      3398 bytes
Total body sent:        1940
HTML transferred:       1787 bytes
Requests per second:    7.26 [#/sec] (mean)
Time per request:       1377.853 [ms] (mean)
Time per request:       137.785 [ms] (mean, across all concurrent requests)
Transfer rate:          2.41 [Kbytes/sec] received
                        1.37 kb/s sent
                        3.78 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       1
Processing:    68  543 548.0    246    1256
Waiting:       68  543 548.0    246    1256
Total:         69  544 548.0    247    1257

Percentage of the requests served within a certain time (ms)
  50%    247
  66%   1095
  75%   1157
  80%   1196
  90%   1257
  95%   1257
  98%   1257
  99%   1257
 100%   1257 (longest request)
```

### 参数说明：
-n 10 表示请求总数为10

-c 10 表示并发用户数为10

http://192.100.200.140:8000/api/instance/show 表示这写请求的目标URL

测试结果也一目了然，测试出的吞吐率为：
**Requests per second**: 2015.93 [#/sec] (mean)  初次之外还有其他一些信息。

**Server Software** 表示被测试的Web服务器软件名称

**Server Hostname** 表示请求的URL主机名

**Server Port** 表示被测试的Web服务器软件的监听端口

**Document Path** 表示请求的URL中的根绝对路径，通过该文件的后缀名，我们一般可以了解该请求的类型

**Document Length** 表示HTTP响应数据的正文长度

**Concurrency Level** 表示并发用户数，这是我们设置的参数之一

**Time taken for tests** 表示所有这些请求被处理完成所花费的总时间

**Complete requests** 表示总请求数量，这是我们设置的参数之一

**Failed requests** 表示失败的请求数量，这里的失败是指请求在连接服务器、发送数据等环节发生异常，以及无响应后超时的情况。如果接收到的HTTP响应数据的头信息中含有2XX以外的状态码，则会在测试结果中显示另一个名为       “Non-2xx responses”的统计项，用于统计这部分请求数，这些请求并不算在失败的请求中。

**Total transferred** 表示所有请求的响应数据长度总和，包括每个HTTP响应数据的头信息和正文数据的长度。注意这里不包括HTTP请求数据的长度，仅仅为web服务器流向用户PC的应用层数据总长度。

**HTML transferred** 表示所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度。

**Requests per second** 吞吐率，计算公式：Complete requests / Time taken for tests

**Time per request** 用户平均请求等待时间，计算公式：Time token for tests/（Complete requests/Concurrency Level）

**Time per requet(across all concurrent request)** 服务器平均请求等待时间，计算公式：Time taken for tests/Complete requests，正好是吞吐率的倒数。也可以这么统计：Time per request/Concurrency Level

**Transfer rate** 表示这些请求在单位时间内从服务器获取的数据长度，计算公式：Total trnasferred/ Time taken for tests，这个统计很好的说明服务器的处理能力达到极限时，其出口宽带的需求量。

**Percentage of requests served within a certain time（ms）** 这部分数据用于描述每个请求处理时间的分布情况，比如以上测试，80%的请求处理时间都不超过6ms，这个处理时间是指前面的Time per request，即对于单个用户而言，平均每个请求的处理时间。
