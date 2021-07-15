---
title: Mysql 1040 Too many connections 错误与解决办法
tags: [mysql]
categories: Mysql
description: CentOS7 mariadb mysql max_connections=214 无法修改的问题
date: 2019/7/1 10:40:00
---

日志报如下错误:

```
Traceback (most recent call last):
  File "/var/www/kdpa/project/api/host.py", line 339, in showInstanceConsole
  File "/var/www/kdpa/project/api/hostImpl.py", line 500, in showInstanceConsoleImpl
    opsVmId = self.hostDao.getOpsVmIdByUuid(uuid)
  File "/var/www/kdpa/project/core/modifier.py", line 46, in _dao
    return daoFn(*args, **kwargs)
  File "/var/www/kdpa/project/dao/host/hostDao.py", line 168, in getOpsVmIdByUuid
    host = session.query(Host).filter(Host.uuid == str(uuid)).first()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3222, in first
    ret = list(self[0:1])
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3012, in __getitem__
    return list(res)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3324, in __iter__
    return self._execute_and_instances(context)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3346, in _execute_and_instances
    querycontext, self._connection_from_session, close_with_result=True
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3361, in _get_bind_args
    mapper=self._bind_mapper(), clause=querycontext.statement, **kw
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/query.py", line 3339, in _connection_from_session
    conn = self.session.connection(**kw)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/session.py", line 1124, in connection
    execution_options=execution_options,
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/session.py", line 1130, in _connection_for_bind
    engine, execution_options
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/orm/session.py", line 431, in _connection_for_bind
    conn = bind._contextual_connect()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2226, in _contextual_connect
    self._wrap_pool_connect(self.pool.connect, None),
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2266, in _wrap_pool_connect
    e, dialect, self
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 1536, in _handle_dbapi_exception_noconnection
    util.raise_from_cause(sqlalchemy_exception, exc_info)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/util/compat.py", line 399, in raise_from_cause
    reraise(type(exception), exception, tb=exc_tb, cause=cause)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/base.py", line 2262, in _wrap_pool_connect
    return fn()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 363, in connect
    return _ConnectionFairy._checkout(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 760, in _checkout
    fairy = _ConnectionRecord.checkout(pool)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 492, in checkout
    rec = pool._do_get()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/impl.py", line 139, in _do_get
    self._dec_overflow()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/util/langhelpers.py", line 68, in __exit__
    compat.reraise(exc_type, exc_value, exc_tb)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/impl.py", line 136, in _do_get
    return self._create_connection()
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 308, in _create_connection
    return _ConnectionRecord(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 437, in __init__
    self.__connect(first_connect_check=True)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/pool/base.py", line 639, in __connect
    connection = pool._invoke_creator(self)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/strategies.py", line 114, in connect
    return dialect.connect(*cargs, **cparams)
  File "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/default.py", line 451, in connect
    return self.dbapi.connect(*cargs, **cparams)
  File "build/bdist.linux-x86_64/egg/MySQLdb/__init__.py", line 81, in Connect
    return Connection(*args, **kwargs)
  File "build/bdist.linux-x86_64/egg/MySQLdb/connections.py", line 187, in __init__
    super(Connection, self).__init__(*args, **kwargs2)
OperationalError: (_mysql_exceptions.OperationalError) (1040, 'Too many connections')
(Background on this error at: http://sqlalche.me/e/e3q8)
```

解决方式：


查看当前数据库最大连接数：
```bash
MariaDB [(none)]> select VARIABLE_VALUE from information_schema.GLOBAL_VARIABLES where VARIABLE_NAME='MAX_CONNECTIONS';
+----------------+
| VARIABLE_VALUE |
+----------------+
| 214            |
+----------------+
1 row in set (0.00 sec)
```

``` bash
vi /etc/systemd/system/mariadb.service.d/limits.conf

[Service]
LimitNOFILE=65535
LimitNPROC=65535
```


保存，退出。

``` bash
systemctl daemon-reload
systemctl restart mariadb.service
```

重启后查看：
```bash
MariaDB [(none)]> select VARIABLE_VALUE from information_schema.GLOBAL_VARIABLES where VARIABLE_NAME='MAX_CONNECTIONS';
+----------------+
| VARIABLE_VALUE |
+----------------+
| 4096           |
+----------------+
1 row in set (0.00 sec)
```

4096配置在Openstack的mysql配置 /etc/my.cnf.d/openstack.cnf中max_connections = 4096。
