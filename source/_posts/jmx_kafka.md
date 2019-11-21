---
title: JMX开启kafka
tags: [kafka]
categories: Kafka
description: JMX开启kafka
date: 2019/11/21 15:00:00
---

#### 开启JMX

kafka开启JMX的2种方式：
1.启动kafka时增加JMX_PORT=9988，即
```php
JMX_PORT=9988 bin/kafka-server-start.sh -daemon config/server.properties
```
2.修改kafka-run-class.sh脚本，第一行增加JMX_PORT=9988即可。

事实上这两种配置方式背后的原理是一样的，
我们看一下kafka的启动脚本kafka-server-start.sh的最后一行
```php
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka"$@"
```
实际上就是调用kafka-run-class.sh脚本，其中有一段这样的内容：
```php
# JMX port to use
if [  $JMX_PORT ]; then
  KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sun.management.jmxremote.port=$JMX_PORT "
fi
```
所以，本质是给参数JMX_PORT赋值，第二种方式在脚本的第一行增加JMX_PORT=9988，$JMX_PORT就能取到值；而第一种方式有点逼格，本质是设置环境变量然后执行启动脚本，类似下面这种方式给JMX_PORT赋值：
```php
[root@kafka]$ export JMX_PORT=9988
[root@kafka]$ bin/kafka-server-start.sh -daemon config/server.properties
```
jmx所有相关参数都在脚本kafka-run-class.sh中，如下所示：
```php
# JMX settings
if [ -z "$KAFKA_JMX_OPTS" ]; then
  KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=10.0.55.229 -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false "
fi

# JMX port to use
if [  $JMX_PORT ]; then
  KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sun.management.jmxremote.port=$JMX_PORT "
fi
```
某些服务器可能无法正确绑定ip，这时候我们需要显示指定绑定的host：-

```php
Djava.rmi.server.hostname=192.100.200.46
```

#### jconsole连接
配置好jmx并启动kafka后，可以启动jconsole验证jmx配置是否正确（连接远程进程的host就是参数java.rmi.server.hostname指定的值，port就是参数JMX_PORT指定的值）：

JMX开启远程访问（包括kafka启动）：

```php
sudo  KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.100.200.46 -Djava.net.preferIPv4Stack=true -Dcom.sun.management.jmxremote.port=9988" bin/kafka-server-start.sh config/server.properties
```
