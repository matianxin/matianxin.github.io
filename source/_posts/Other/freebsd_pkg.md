---
title: FreeBSD更新pkg
tags: [FreeBSD]
categories: Linux
description: FreeBSD12更新pkg源。
date: 2021/06/16 13:45:00
---

#### 1.PKG二进制仓储源文件存储目录 的使用方法

系统中常用的PKG二进制仓储源文件存储目录有两个，一个是系统级仓储文件目录，另一个是用户级仓储文件目录，系统级仓储文件目录为 /etc/pkg/， 而用户级别仓储文件目录为 /usr/local/etc/pkg/repos/，这个目录在默认情况下并不存在于系统中，需要用户手工建立，而这两个路径均由 /usr/local/etc/pkg.conf 中 REPOS_DIR 变量控制，原则上可以由用户自由定制。

#### 2. 建立通用户级PKG二进制仓储源文件存储目录

```php
# mkdir -p /usr/local/etc/pkg/repos
```

#### 3. 建立常用的用户级PKG二进制仓储源文件

```php
# cd /usr/local/etc/pkg/repos
```

建立仓储源文件的格式要求为：必须使用 .conf 为后缀名结尾的文件。最好使用 "0." 或者 "1-" 等数字为前缀的文件名，因为同时启用多个 PKG 源文件时，PKG源文件名的第一个数字前缀直接影响着源的使用优先级，其文件格式的建立如下：

定制第一个仓储文件为 0.bme.conf

```php
bme: {
  url: "pkg+http://pkg0.bme.freebsd.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "none",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

定制第二个仓储文件为 1.nyi.conf

```php
nyi: {
  url: "pkg+http://pkg0.nyi.freebsd.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "none",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

定制第三个仓储文件为 2.ydx.conf
```php
ydx: {
  url: "pkg+http://pkg0.ydx.freebsd.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "none",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

定制第四个仓储文件为 3.isc.conf
```php
isc: {
  url: "pkg+http://pkg0.isc.freebsd.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "none",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

定制第五个仓储文件为 4.chinafreebsd.conf
```php
chinafreebsd: {
  url: "pkg+http://pkg.reebsd.cn/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "none",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

**重要:**
**建议只选用一个最快的PKG源，而不是同时启用多个，如果同时启用了多个 PKG 源，那么在安装软件包或升级 PKG 源时候请使用 -r 选项指定要操作的 PKG 源。**
**比如：pkg update -r chinafreebsd 或者 pkg install -r chinafreebsd -y xxxx 。**


#### 4. 禁用默认 PKG 仓储源
可以直接禁用系统级源文件，换源的主要目的是增加 pkg install 命令下载速度，或者是不能忍受默认源的龟速，用户可以直接禁用系统默认的官方源。比如：
```php
# echo "FreeBSD: { enabled: no }" > /usr/local/etc/pkg/repos/FreeBSD.conf
```
注意:
此步骤不是必须，如果保留系统仓储源文件，则系统默认源将和用户源一同有效

#### 5. 更新源

所有源在使用前最好进行一次强制更新。比如：

```php
# pkg update -f Updating nyi repository catalogue...
Fetching meta.txz: 100%    944 B   0.9kB/s    00:01
Fetching packagesite.txz: 100%    6 MiB 268.1kB/s    00:22
Processing entries: 100%
nyi repository update completed. 25828 packages processed.
Updating ydx repository catalogue...
Fetching meta.txz: 100%    944 B   0.9kB/s    00:01
Fetching packagesite.txz: 100%    1 KiB   0.0kB/s    01:00
......
```

或者更新某个想要使用的源。比如：
```php
# pkg update -r bme Updating bme repository catalogue...
Fetching meta.txz: 100%    944 B   0.9kB/s    00:01
Fetching packagesite.txz: 100%    6 MiB 268.1kB/s    00:22
Processing entries: 100%
bme repository update completed. 25828 packages processed.
```

#### 6. 验证当前生效源
使用 pkg -vv 命令可以打印当前 PKG 的配置信息，以及所有生效源配置，例如：
```php
# pkg -vv Version                 : 1.8.8
PKG_DBDIR = "/var/db/pkg";
PKG_CACHEDIR = "/var/cache/pkg";
PORTSDIR = "/usr/ports";
INDEXDIR = "";
INDEXFILE = "INDEX-11";
HANDLE_RC_SCRIPTS = false;
DEFAULT_ALWAYS_YES = false;
ASSUME_ALWAYS_YES = false;
REPOS_DIR [
    "/etc/pkg/",
    "/usr/local/etc/pkg/repos/",
]
PLIST_KEYWORDS_DIR = "";
SYSLOG = true;
ABI = "FreeBSD:11:amd64";
ALTABI = "freebsd:11:x86:64";
DEVELOPER_MODE = false;
VULNXML_SITE = "http://vuxml.freebsd.org/freebsd/vuln.xml.bz2";
FETCH_RETRY = 3;
PKG_PLUGINS_DIR = "/usr/local/lib/pkg/";
PKG_ENABLE_PLUGINS = true;
PLUGINS [
]
DEBUG_SCRIPTS = false;
PLUGINS_CONF_DIR = "/usr/local/etc/pkg/";
PERMISSIVE = false;
REPO_AUTOUPDATE = true;
NAMESERVER = "";
HTTP_USER_AGENT = "pkg/1.8.8";
EVENT_PIPE = "";
FETCH_TIMEOUT = 30;
UNSET_TIMESTAMP = false;
SSH_RESTRICT_DIR = "";
PKG_ENV {
}
PKG_SSH_ARGS = "";
DEBUG_LEVEL = 0;
ALIAS {
    all-depends = "query %dn-%dv";
    annotations = "info -A";
    build-depends = "info -qd";
    cinfo = "info -Cx";
    comment = "query -i \"%c\"";
    csearch = "search -Cx";
    desc = "query -i \"%e\"";
    download = "fetch";
    iinfo = "info -ix";
    isearch = "search -ix";
    prime-list = "query -e '%a = 0' '%n'";
    leaf = "query -e '%#r == 0' '%n-%v'";
    list = "info -ql";
    noauto = "query -e '%a == 0' '%n-%v'";
    options = "query -i \"%n - %Ok: %Ov\"";
    origin = "info -qo";
    provided-depends = "info -qb";
    raw = "info -R";
    required-depends = "info -qr";
    roptions = "rquery -i \"%n - %Ok: %Ov\"";
    shared-depends = "info -qB";
    show = "info -f -k";
    size = "info -sq";
}
CUDF_SOLVER = "";
SAT_SOLVER = "";
RUN_SCRIPTS = true;
CASE_SENSITIVE_MATCH = false;
LOCK_WAIT = 1;
LOCK_RETRIES = 5;
SQLITE_PROFILE = false;
WORKERS_COUNT = 0;
READ_LOCK = false;
PLIST_ACCEPT_DIRECTORIES = false;
IP_VERSION = 0;
AUTOMERGE = true;
VERSION_SOURCE = "";
CONSERVATIVE_UPGRADE = true;
PKG_CREATE_VERBOSE = false;
AUTOCLEAN = false;
DOT_FILE = "";
REPOSITORIES {
}
VALID_URL_SCHEME [
    "pkg+http",
    "pkg+https",
    "https",
    "http",
    "file",
    "ssh",
    "ftp",
    "ftps",
    "pkg+ssh",
    "pkg+ftp",
    "pkg+ftps",
]
ALLOW_BASE_SHLIBS = false;
WARN_SIZE_LIMIT = 1048576;


Repositories:
  bme: {
    url             : "pkg+http://pkg0.bme.freebsd.org/FreeBSD:11:amd64/quarterly",
    enabled         : yes,
    priority        : 0,
    mirror_type     : "SRV",
    fingerprints    : "/usr/share/keys/pkg"
  }
  nyi: {
    url             : "pkg+http://pkg0.nyi.freebsd.org/FreeBSD:11:amd64/quarterly",
    enabled         : yes,
    priority        : 0,
    mirror_type     : "SRV",
    fingerprints    : "/usr/share/keys/pkg"
  }
  ydx: {
    url             : "pkg+http://pkg0.ydx.freebsd.org/FreeBSD:11:amd64/quarterly",
    enabled         : yes,
    priority        : 0,
    mirror_type     : "SRV",
    fingerprints    : "/usr/share/keys/pkg"
  }
  isc: {
    url             : "pkg+http://pkg0.isc.freebsd.org/FreeBSD:11:amd64/quarterly",
    enabled         : yes,
    priority        : 0,
    mirror_type     : "SRV",
    fingerprints    : "/usr/share/keys/pkg"
  }
  chinafreebsd: {
    url             : "pkg+http://pkg.freebsd.cn/FreeBSD:11:amd64/quarterly",
    enabled         : yes,
    priority        : 0,
    mirror_type     : "SRV",
    fingerprints    : "/usr/share/keys/pkg"
  }
```

#### 7. 用 -r 选项使用任意指定源安装软件
安装二进制安装包时可以使用 -r 选项选定要使用的源，源名称为源文件中第一行中冒号之前的名称 比如 bme
````php
# pkg install -y -r bme vim-lite
```

