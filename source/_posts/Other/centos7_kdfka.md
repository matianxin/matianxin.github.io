---
title: CentOS7安装kafka
tags: [kafka]
categories: Linux
description: Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群来提供实时的消息。
date: 2019/11/19 11:02:08
---

#### 安装准备：基于Centos7.5 1804 minimal

#### 安装JDK1.8

```php
yum -y install java-1.8.0-openjdk*
```

#### 安装kafka

下载kafka
```php
wget http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz
```

解压
```php
tar -zxvf kafka_2.12-2.3.0.tgz
```

修改配置，使kafka远程访问
```php
vi config/server.properties

# listeners=PLAINTEXT://:9092
->
listeners=PLAINTEXT://KAFKA_IP:9092
```

#### 启动kafka
进入kafka目录
```php
cd kafka_2.12-2.3.0
```

启动zookeeper
```php
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

检查zookeeper端口2181是否正常监听
```php
netstat -an|grep 2181
tcp6       0      0 :::2181                 :::*                    LISTEN
```

检查kafka默认的JVM参数配置是否需要修改
Kafka默认设置1G，即"-Xmx1G -Xms1G"。
如果你的测试机内存较低，需要修改才能成功启动。
参数配置位于：bin/kafka-server-start.sh

启动Kafka
```php
bin/kafka-server-start.sh config/server.properties
```

如果一切顺利，就会看到如下启动成功的日志：
```php
[2019-11-18 21:42:37,067] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

检查Kafka的端口9092监听是否正常
```php
[root@localhost kafka_2.12-2.3.0]# netstat -an|grep 9092
tcp6       0      0 :::9092                 :::*                    LISTEN
```

#### 测试kafka
创建一个测试主题
```php
cd kafka_2.12-2.3.0
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

Created topic "test".
```

查看刚刚创建的test主题
```php
bin/kafka-topics.sh --list --zookeeper localhost:2181

test
```

向刚刚创建的test主题中写入数据
```php
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>Hello, World!
>
```
"Hello World!"为写入的数据。

查看刚刚写入的数据
另外开一个ssh tab连接，然后执行如下命令
```php
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].

Hello, World!
```



