---
title: Windows禁用/重新启用笔记本电脑自带键盘
tags: [tool]
categories: 工具
description: 禁用/重新启用笔记本电脑自带键盘
date: 2020/10/9 10:00:00
---

#### 禁用：
```php
管理员身份运行cmd
输入 sc config i8042prt start= disabled 回车
重启电脑生效
```

#### 重新启用：
```php
管理员身份运行cmd
输入 sc config i8042prt start= disabled 回车
重启电脑生效
```