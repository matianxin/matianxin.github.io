---
title: zerorpc优化
tags: [tool]
categories: 工具
description: zerorpc并发测试及优化修改
date: 2021/2/18 9:15:00
---

### zerorpc官网：https://www.zerorpc.io/


Introduction：

An easy to use, intuitive, and cross-language RPC
zerorpc is a light-weight, reliable and language-agnostic library for distributed communication between server-side processes. It builds on top of ZeroMQ and MessagePack. Support for streamed responses - similar to python generators - makes zerorpc more than a typical RPC engine. Built-in heartbeats and timeouts detect and recover from failed requests. Introspective capabilities, first-class exceptions and the command-line utility make debugging easy.


简介：

一个易于使用、直观和跨语言的RPC

zerorpc是一个轻量级、可靠和语言无关的库，用于服务器端进程之间的分布式通信。它构建在ZeroMQ和MessagePack之上。支持流响应——类似于python生成器——使得zerorpc不仅仅是一个典型的RPC引擎。内置的心跳和超时检测并从失败的请求中恢复。内省功能、一级异常和命令行实用程序使调试变得容易。

官网示例：

```php
# 官网-Server
import zerorpc

class HelloRPC(object):
    def hello(self, name):
        return "Hello, %s" % name

s = zerorpc.Server(HelloRPC())
s.bind("tcp://0.0.0.0:4242")
s.run()
```

```php
# 官网-Client
import zerorpc

c = zerorpc.Client()
c.connect("tcp://127.0.0.1:4242")
print c.hello("RPC")
```

#### 优化1：使用gevent方式启动zerorpc服务端，提高并发连接数量。
```php
# 优化1-Server
import zerorpc

s = zerorpc.Server(HelloRPC())
s.bind("tcp://0.0.0.0:4242")
zerorpc.gevent.spawn(s.run)
while True:
    zerorpc.gevent.sleep(10)
```

#### 优化2：报错：报错zerorpc.exceptions.LostRemote: Lost remote after 10s heartbeat，这是因为zerorpc有一个心跳检测，如果客户端没及时得到服务端反馈就报错。

解决办法为：在客户端引用zerorpc.Client时加一个参数heartbeat=None， 即c = zerorpc.Client(heartbeat=None) 但是有时还是报错zerorpc.exceptions.LostRemote: Lost remote after 30s heartbeat， 这是相应延长大于30s，没有找到很好的彻底解决响应时间的办法，但是可以“治标”： 对应报错的地方会有提示报错文件：Python27\lib\site-packages\zerorpc\channel.py"或者Python27\lib\site-packages\zerorpc\core.py"，哪个文件报错就改哪个，将文件里面对timeout赋值的语句timeout=30改成timeout=300或者更长，视你需要的响应时间而定，亦可以更长，timeout=30的需要改，然后就可以了，如果不行别忘了前面加上heartbeat=None参数。

```php
# 优化2-Client
import zerorpc

c = zerorpc.Client(heartbeat=None)
c.connect("tcp://127.0.0.1:4242")
print c.hello("RPC")
```


#### zerorpc服务器端不并发处理，客户端连接阻塞。
```php
# 举例1-Server
import zerorpc
import time

class HelloRPC(object):
    def hello(self, name):
        time.sleep(10) # -> zerorpc.gevent.sleep(10)
        return "Hello, %s" % name

s = zerorpc.Server(HelloRPC())
s.bind("tcp://0.0.0.0:4242")
s.run()
```

这里由于time.sleep()导致所有该zerorpc的进程使用，导致连接的客户端协程全部被阻塞，故这里应该使用协程的sleep()方法，协程才能正常的并发执行。
