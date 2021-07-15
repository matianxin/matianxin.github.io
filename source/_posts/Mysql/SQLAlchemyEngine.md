---
title: SQLAlchemy 中的 Engine 是什么
tags: [sqlalchemy]
categories: SQLAlchemy
description: Engine 翻译过来就是引擎的意思，汽车通过引擎来驱动，而 SQLAlchemy 是通过 Engine 来驱动，Engine 维护了一个连接池（Pool）对象和方言（Dialect）。方言简单而言就是你连的到底是 MySQL 还是 Oracle 或者 PostgreSQL 还是其它数据库。
date: 2019/7/17 9:56:17
---

连接池很重要，因为每次发送sql查询的时候都需要先建立连接，如果程序启动的时候事先就初始化一批连接放在连接池，每次用完后又放回连接池给其它请求使用，就能大大提高查询的效率。

#### Engine 初始化

Engine 的初始化非常简单，通过工厂函数 create_engine 就可以创建。

```python
from sqlalchemy import create_engine

engine = create_engine('mysql://user:password@localhost:3306/test?charset=utf8')
```

构建好 Engine 对象的同时，连接池和Dialect也创建好了，但是这时候并不会立马与数据库建立真正的连接，只有你调用 Engine.connect() 或者 Engine.execute(sql) 执行SQL请求的时候，才会建立真正的连接。
因此 Engine 和 Pool 的行为称之为延迟初始化，等真正要派上用场的时候才去建立连接。

需要注意的是，创建引擎时，如果数据库的密码含有特殊字符，需要先编码处理

```python
>>> import urllib.parse
>>> urllib.parse.quote_plus("kx%jj5/g")
'kx%25jj5%2Fg'
```

其它数据库方言初始化 engine 的方式可参考官方文档：
- https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls

create_engine 还有很多可选参数，这里介绍几个重要的参数。

```python
engine = create_engine('mysql://user:password@localhost:3306/test?charset=utf8',
                       echo=False
                       pool_size=100,
                       pool_recycle=3600,
                       pool_pre_ping=True)
```

**echo** ：为 True 时候会把sql语句打印出来，当然，你可以通过配置logger来控制输出，这里不做讨论。

**pool_size**： 是连接池的大小，默认为5个，0表示连接数无限制

**pool_recycle**： MySQL 默认情况下如果一个连接8小时内容没有任何动作（查询请求）就会自动断开链接，出现 MySQL has gone away的错误。设置了 pool_recycle 后 SQLAlchemy 就会在指定时间内回收连接。如果设置为3600 就表示 1小时后该连接会被自动回收。

**pool_pre_ping** ： 这是1.2新增的参数，如果值为True，那么每次从连接池中拿连接的时候，都会向数据库发送一个类似 select 1 的测试查询语句来判断服务器是否正常运行。当该连接出现 disconnect 的情况时，该连接连同pool中的其它连接都会被回收。

参考链接：

- https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls
- https://stackoverflow.com/questions/34322471/sqlalchemy-engine-connection-and-session-difference
- https://docs.sqlalchemy.org/en/13/core/pooling.html#dealing-with-disconnects
