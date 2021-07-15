---
title: kafka集群管理工具kafka-manager的安装使用
tags: [kafka]
categories: Kafka
description: kafka-manager是目前最受欢迎的kafka集群管理工具，最早由雅虎开源，用户可以在Web界面执行一些简单的集群管理操作。
date: 2019/11/21 15:00:00
---

#### kafka-manager简介

kafka-manager是目前最受欢迎的kafka集群管理工具，最早由雅虎开源，用户可以在Web界面执行一些简单的集群管理操作。具体支持以下内容：

管理多个集群
轻松检查集群状态（主题，消费者，偏移，代理，副本分发，分区分发）
运行首选副本选举
使用选项生成分区分配以选择要使用的代理
运行分区重新分配（基于生成的分配）
使用可选主题配置创建主题（0.8.1.1具有与0.8.2+不同的配置）
删除主题（仅支持0.8.2+并记住在代理配​​置中设置delete.topic.enable = true）
主题列表现在指示标记为删除的主题（仅支持0.8.2+）
批量生成多个主题的分区分配，并可选择要使用的代理
批量运行重新分配多个主题的分区
将分区添加到现有主题
更新现有主题的配置
kafka-manager 项目地址：https://github.com/yahoo/kafka-manager

#### kafka-manager安装

下载安装包
使用Git或者直接从Releases中下载，这里我们下载 2.0.0.2
版本：https://github.com/yahoo/kafka-manager/releases

```php
wget https://github.com/yahoo/kafka-manager/archive/2.0.0.2.tar.gz
```

解压安装包
```php
tar zxvf 2.0.0.2.tar.gz
cd kafka-manager-2.0.0.2
```

sbt编译
yum安装sbt(因为kafka-manager需要sbt编译)
```php
curl https://bintray.com/sbt/rpm/rpm > bintray-sbt-rpm.repo
mv bintray-sbt-rpm.repo /etc/yum.repos.d/
yum install sbt
```

验证：检查sbt是否安装成功
```php
sbt
[info] [launcher] getting org.scala-sbt sbt 1.3.3  (this may take some time)...
```

编译kafka-manager
```php
./sbt clean dist
```

安装
将编译好的/root/kafka-manager-2.0.0.2/target/universal/kafka-manager-2.0.0.zip文件解压
```php
unzip /root/kafka-manager-2.0.0.2/target/universal/kafka-manager-2.0.0.zip
```

启动服务
启动zk集群，kafka集群，再启动kafka-manager服务。
bin/kafka-manager 默认的端口是9000，可通过 -Dhttp.port，指定端口;
-Dconfig.file=conf/application.conf指定配置文件:

```php
nohup bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=9000 &
```

WebUI查看：http://KAFKA_IP:9000/ 出现如下界面则启动成功。

