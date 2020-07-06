---
title: 解决Docker pull镜像速度过慢的问题
tags: [docker]
categories: Docker
description: 解决Docker pull镜像速度过慢的问题
date: 2020/7/6 10:00:00
---

目前，Docker拥有中国的官方镜像，具体内容可访问https://www.docker-cn.com/registry-mirror

在使用时，Docker 中国官方镜像加速可通过 **registry.docker-cn.com**访问。该镜像库只包含流行的公有镜像。私有镜像仍需要从美国镜像库中拉取。

您可以使用以下命令直接从该镜像加速地址进行拉取：

$ docker pull registry.docker-cn.com/myname/myrepo:mytag
例如:

$ docker pull registry.docker-cn.com/library/ubuntu:16.04
注: 除非您修改了 Docker 守护进程的 `--registry-mirror` 参数 (见下文), 否则您将需要完整地指定官方镜像的名称。
例如，library/ubuntu、library/redis、library/nginx。

使用 --registry-mirror 配置 Docker 守护进程
您可以配置 Docker 守护进程默认使用 Docker 官方镜像加速。这样您可以默认通过官方镜像加速拉取镜像，而无需在每次拉取时指定 registry.docker-cn.com。

您可以在 Docker 守护进程启动时传入 --registry-mirror 参数：

$ docker --registry-mirror=https://registry.docker-cn.com daemon
为了永久性保留更改，您可以修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值。
```php
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

修改保存后重启 Docker 以使配置生效。

```php
systemctl docker restart
```

注: 您也可以使用适用于 Mac 的 Docker 和适用于 Windows 的 Docker 来进行设置。
