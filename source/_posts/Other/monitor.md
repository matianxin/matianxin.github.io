---
title: collectd + influxDB + Grafana 搭建监控平台
tags: [monitor]
categories: 监控
description: 采集数据（collectd）-> 存储数据（influxdb) -> 显示数据（grafana）
date: 2019/8/6 13:50:00
---

### 概述

常用配置：

influxdb + grafana安装在一台机器负责监控数据收集及展示
collectd安装在一台或多台被监控服务端,跟监控端的25826端口对接，上传本地监控的数据
influxdb监控25826端口以获得数据，自身处于8086端口，grafana从8086获得数据进行展示

- InfluxDB 是 Go 语言开发的一个开源分布式时序数据库，非常适合存储指标、事件、分析等数据
- collectd C 语言写的一个系统性能采集工具
- Grafana 是纯 Javascript 开发的前端工具，用于访问 InfluxDB，自定义报表、显示图表等

本次安装版本
- collectd 5.8.1
- influxDB 1.7.7
- Grafana 6.2.5

### Collectd
collectd是一个守护(daemon)进程，用来收集系统性能和提供各种存储方式来存储不同值的机制。比如以RRD 文件形式，当系统运行和存储信息的时候，Collectd会周期性统计系统的相关统计信息。那些信息可以用来找到当前系统性能瓶颈。（如作为性能分析 performance analysis）和预测系统未来的load（如能力部署capacity planning）

#### 安装

```php
# yum install -y collectd
```

#### 修改配置
```php
# vim /etc/collectd.conf
#更改以下内容，前面的#记得删除掉 没有就使用find / -name collectd.conf 查找在哪里

  FQDNLookup   true
  Hostname    "localhost" #直接使用hostname命令查看
  BaseDir     "/var/lib/collectd"
  PIDFile     "/var/run/collectd.pid"
  PluginDir   "/usr/lib64/collectd"
  TypesDB     "/usr/share/collectd/types.db"
  LoadPlugin  syslog
  LoadPlugin rrdtool
  LoadPlugin disk
  LoadPlugin interface
  LoadPlugin load
  LoadPlugin memory
  LoadPlugin network
  LoadPlugin processes
  LoadPlugin users
  <Plugin interface>
          Interface "eth0"
          IgnoreSelected false
  </Plugin>
  <Plugin network>
          Server "127.0.0.1" "25826" #这里填写的是influxDB安装的服务器ip
  <Plugin rrdtool>
          DataDir "/usr/var/lib/collectd/rrd" #如果你是使用下载安装前面应该会多出一个${prefix},未尝试是否有影响
  </Plugin>
```

#### 安装rrdtool插及依赖包
```php
# yum install collectd-rrdtool rrdtool rrdtool-devel
```

#### 启动collectd
```php
# systemctl start collectd.service #启动
# systemctl enable collectd.service #配置开机自启
```

#### 确认配置是否成功
```php
# cd /var/lib/collectd
# ls
rrd
```
如果/var/lib/collectd目录下生成rrd文件，说明有数据了，如果没有应该是配置问题


### Influxdb

#### 配置yum源
配置yum(此方法为最新版本InfluxDB)
```php
# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
>[influxdb]
>name = InfluxDB Repository - RHEL \$releasever
>baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
>enabled = 1
>gpgcheck = 1
>gpgkey = https://repos.influxdata.com/influxdb.key
>EOF
```

#### 安装
```php
# yum install -y influxdb
```

#### 启动InfluxDB
```php
# systemctl start influxdb.service#启动
# systemctl enable influxdb.service #配置influxdb开机自启
```
启动后TCP端口:8083 为InfluxDB 管理控制台
  TCP端口:8086 为客户端和InfluxDB通信时的HTTP API
启动后InfluxDB用户认证默认是关闭的，先创建用户:geekwolf geekwolf
命令行输入influx


#### 配置数据库
```mysql
# influx #进入InfluxDB
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> create database collectdb #创建数据库
> show databases #查看数据库
name: databases
\------
name
collectdb
> create user matianxin with password 'matianxin' #创建一个用户和密码
> show users #查看所有用户
user            admin
matianxin        false
> grant all on collectdb to matianxin #把上面创建的数据库的所有权限赋给geekwolf用户
> help show
Usage:
    connect <host:port>   connects to another node specified by host:port
    auth                  prompts for username and password
    pretty                toggles pretty print for the json format
    use <db_name>         sets current database
    format <format>       specifies the format of the server responses: json, csv, or column
    precision </format><format>    specifies the format of the timestamp: rfc3339, h, m, s, ms, u or ns
    consistency <level>   sets write consistency level: any, one, quorum, or all
    history               displays command history
    settings              outputs the current settings for the shell
    exit/quit/ctrl+d      quits the influx shell
    show databases        show database names
    show series           show series information
    show measurements     show measurement information
    show tag keys         show tag key information
    show field keys       show field key information
    A full list of influxql commands can be found at:
    https://docs.influxdata.com/influxdb/v0.10/query_language/spec
> quit #退出
```

#### 启用认证
修改配置文件启用认证
```php
# sed -i 's#auth-enabled = false#auth-enabled = true#g' /etc/influxdb/influxdb.conf
# systemctl restart influxdb.service #重启influxdb
```

#### 配置InfluxDB支持Collectd
```php
# vim /etc/influxdb/influxdb.conf
  [collectd]
  enabled = true
  bind-address = "127.0.0.1:25826"
  database = "collectdb"
  typesdb = "/usr/share/collectd/types.db" #查找一下types.db文件不一定在这个路径，如果路径配置错误就不能监听成功
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "10s"
  read-buffer = 0
```

```php
# systemctl restart influxdb.service #重启influxdb
```

查看25826这个端口是否已经监听，如果有，则代表启动正常

```php
# netstat -anp| grep 25826
udp    0   0 127.0.0.1:25826    0.0.0.0:*       2950/influxd
```


#### 确认数据
```mysql
[root@localhost ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> use collectdb
Using database collectdb
> show field keys
name: cpu_value
fieldKey fieldType
-------- ---------
value    float

name: disk_io_time
fieldKey fieldType
-------- ---------
value    float

name: disk_read
fieldKey fieldType
-------- ---------
value    float

name: disk_value
fieldKey fieldType
-------- ---------
value    float

name: disk_weighted_io_time
fieldKey fieldType
-------- ---------
value    float

name: disk_write
fieldKey fieldType
-------- ---------
value    float

name: interface_rx
fieldKey fieldType
-------- ---------
value    float

name: interface_tx
fieldKey fieldType
-------- ---------
value    float

name: load_longterm
fieldKey fieldType
-------- ---------
value    float

name: load_midterm
fieldKey fieldType
-------- ---------
value    float

name: load_shortterm
fieldKey fieldType
-------- ---------
value    float

name: memory_value
fieldKey fieldType
-------- ---------
value    float

name: processes_value
fieldKey fieldType
-------- ---------
value    float

name: users_value
fieldKey fieldType
-------- ---------
value    float
>  select * from cpu_value limit 10;
name: cpu_value
time                host      instance type type_instance value
----                ----      -------- ---- ------------- -----
1565060949576064259 localhost 0        cpu  user          2663
1565060949576080440 localhost 0        cpu  system        2952
1565060949576087737 localhost 0        cpu  wait          89
1565060949576094319 localhost 0        cpu  nice          0
1565060949576100786 localhost 0        cpu  interrupt     0
1565060949576106664 localhost 0        cpu  softirq       160
1565060949576108568 localhost 0        cpu  steal         0
1565060949576110126 localhost 0        cpu  idle          248067
1565060959576531164 localhost 0        cpu  user          2664
1565060959576547037 localhost 0        cpu  system        2955
>
```


### Grafana

####  安装
```php
# wget https://dl.grafana.com/oss/release/grafana-6.2.5-1.x86_64.rpm
# yum localinstall grafana-6.2.5-1.x86_64.rpm
```

#### 启动grafana
```php
# systemctl start grafana-server.service #启动
# systemctl enable grafana-server.service #配置开机自启
```

#### 配置Grafana
访问地址:http://127.0.0.1:3000 默认账号为admin 密码 admin

配置Data Source
- Name: InfluxDB -> default
- URL: http://localhost:8086
- Access: Server(Default)
- Basic Auth: √
 - Basic Auth Details
 - User: matianxin
 - Password: matianxin
 - InfluxDB Details
 - Database: collectdb
 - ...


