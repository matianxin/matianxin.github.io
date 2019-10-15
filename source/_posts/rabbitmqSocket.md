---
title: Openstack 扩大 RabbitMQ Socket Limit 方法
tags: [rabbitmq]
categories: rabbitmq
description: Openstack 扩大 RabbitMQ Socket Limit 方法
date: 2019/10/15 17:46:17
---

在RabbitMQ中，Socket descriptors 是 File descriptors 的子集，它们也是一对此消彼长的关系。
然而，它们的默认配额并不大，File descriptors 默认值为“1024”，而 Socket descriptors 的默认值也只有“829”，同时，File descriptors 所能打开的最大文件数也受限于操作系统的配额。
因此，如果要调整 File descriptors 文件句柄数，就需要同时调整操作系统和RabbitMQ参数。


```php
[root@controller ~]# rabbitmqctl status
Status of node rabbit@controller
[{pid,3407},
{running_applications,
     [{rabbit,"RabbitMQ","3.6.16"},
      {rabbit_common,
          "Modules shared by rabbitmq-server and rabbitmq-erlang-client",
          "3.6.16"},
      {xmerl,"XML parser","1.3.14"},
      {ranch,"Socket acceptor pool for TCP protocols.","1.3.2"},
      {mnesia,"MNESIA  CXC 138 12","4.14.3"},
      {syntax_tools,"Syntax tools","2.1.1"},
      {ssl,"Erlang/OTP SSL application","8.1.3.1"},
      {os_mon,"CPO  CXC 138 46","2.4.2"},
      {public_key,"Public key infrastructure","1.4"},
      {crypto,"CRYPTO","3.7.4"},
      {asn1,"The Erlang ASN1 compiler version 4.0.4","4.0.4"},
      {recon,"Diagnostic tools for production use","2.3.2"},
      {compiler,"ERTS  CXC 138 10","7.0.4.1"},
      {sasl,"SASL  CXC 138 11","3.0.3"},
      {stdlib,"ERTS  CXC 138 10","3.3"},
      {kernel,"ERTS  CXC 138 10","5.2"}]},
{os,{unix,linux}},
{erlang_version,
     "Erlang/OTP 19 [erts-8.3.5.3] [source] [64-bit] [smp:96:96] [async-threads:1024] [hipe] [kernel-poll:true]\n"},
{memory,
     [{connection_readers,20241544},
      {connection_writers,1200000},
      {connection_channels,4846312},
      {connection_other,53785896},
      {queue_procs,6683888},
      {queue_slave_procs,0},
      {plugins,0},
      {other_proc,23512680},
      {metrics,2466240},
      {mgmt_db,0},
      {mnesia,847160},
      {other_ets,3015688},
      {binary,1999169528},
      {msg_index,176072},
      {code,21467691},
      {atom,891849},
      {other_system,75976100},
      {allocated_unused,483082808},
      {reserved_unallocated,0},
      {total,766799872}]},
{alarms,[]},
{listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
{vm_memory_calculation_strategy,rss},
{vm_memory_high_watermark,0.4},
{vm_memory_limit,53742133248},
{disk_free_limit,50000000},
{disk_free,996377710592},
{file_descriptors,
     [{total_limit,878},
      {total_used,776},
      {sockets_limit,829},
      {sockets_used,774}]},
{processes,[{limit,1048576},{used,9483}]},
{run_queue,0},
{uptime,161034},
{kernel,{net_ticktime,60}}]
```

#### 修改系统内核参数

系统级别

```php
vim /etc/sysctl.conf

fs.file-max=655350
```

```php
sysctl -p | grep file-max
```

用户级别

```php
vim /etc/security/limits.conf

* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```

#### 修改rabbitmq配置

如果是以systemd方式管理rabbitmq服务，则需要修改rabbitmq的service文件。

```php
vim /usr/lib/systemd/system/rabbitmq-server.service
```

添加如下参数，其值请根据实际情况进行调整：

```php
[Service]
LimitNOFILE=16384
```

重启rabbitmq即可：

```php
systemctl daemon-reload
systemctl restart rabbitmq-server
```
