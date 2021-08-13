---
title: Ubuntu18.04安装
tags: [ubuntu]
categories: Ubuntu18.04
description: 实训平台适配凝思18.04.5，软件安装，第三方服务，Agent脚本等。
date: 2021/8/13 11:50:00
---

## 版本介绍

### ISO镜像
版本信息：ubuntu-18.04.5-live-server-amd64.iso

### 虚拟机资源配置
VCPU：1个 内存：1GB 磁盘：20GB

## 安装过程

### 安装

#### 选择系统语言： English
![](/images/ubuntu18.04/install-0.png)

#### 配置系统更新： 不更新
选择 -> Contiune without updating
![](/images/ubuntu18.04/install-1.png)

#### 键盘keyboard配置： 英文
选择 -> English(US) -> Done
![](/images/ubuntu18.04/install-2.png)

#### 网络连接配置： 默认DHCP
选择 -> Done
![](/images/ubuntu18.04/install-3.png)

## 配置代理： 不配置
Proxy address： '' 选择 -> Done
![](/images/ubuntu18.04/install-4.png)

#### 配置Ubuntu Mirror源地址：
默认：Ubuntu源 http://cn.archive.ubuntu.com/ubuntu/
Mirror address -> 修改为 清华大学源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
![](/images/ubuntu18.04/install-5.png)

#### 配置磁盘存储：
选择： (×) Use an entire disk [×] Set up this disk as an LVM group 选择 -> Done
![](/images/ubuntu18.04/install-disk1.png)
![](/images/ubuntu18.04/install-disk2.png)
![](/images/ubuntu18.04/install-disk3.png)

#### 配置文件设置：
用户名：ubuntu 密码：ubuntu
![](/images/ubuntu18.04/install-disk6.png)

#### SSH配置：
安装OpenSSH：[×]Install OpenSSH server
![](/images/ubuntu18.04/install-disk7.png)

#### 服务器特殊配置： 默认不选择，跳过
![](/images/ubuntu18.04/install-disk8.png)

#### 开始安装，安装结束重启系统
![](/images/ubuntu18.04/reboot.png)

### 配置

#### 设置root密码
```bash
ubuntu@ubuntu:~$ sudo passwd
[sudo] password for ubuntu: ubuntu
Enter new UNIX password: PASSWORD
Retype new UNIX password: PASSWORD
passwd: password updated successfully
```

#### 系统配置网卡名称为eth
GRUB配置文件路径: /etc/default/grub

授权GRUB配置文件:
```bash
root@ubuntu:~# chmod u+x /etc/default/grub
```

修改GRUB配置文件GRUB_CMDLINE_LINUX, vim /etc/default/grub
```bash
修改 GRUB_CMDLINE_LINUX="" ->
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

执行以下命令
```bash
root@ubuntu:~# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file /etc/default/grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.15.0-153-generic
Found initrd image: /boot/initrd.img-4.15.0-153-generic
done
root@ubuntu:~# reboot
```

#### 配置SSH服务
vim /etc/ssh/sshd_config
```bash
修改 #PermitRootLogin prohibit-password
->
PermitRootLogin yes
```
重启sshd服务
```bash
systemctl restart sshd
```

#### 禁用Ubuntu18系统cloud-init相关服务
查看cloud-init相关服务
```bash
root@ubuntu:/# systemctl status cloud-init-local cloud-init cloud-config cloud-final
● cloud-init-local.service - Initial cloud-init job (pre-networking)
   Loaded: loaded (/lib/systemd/system/cloud-init-local.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2021-08-13 06:46:52 UTC; 9min ago
  Process: 787 ExecStart=/bin/touch /run/cloud-init/network-config-ready (code=exited, status=0/SUCCESS)
  Process: 725 ExecStart=/usr/bin/cloud-init init --local (code=exited, status=0/SUCCESS)
 Main PID: 787 (code=exited, status=0/SUCCESS)

Aug 13 06:46:51 ubuntu systemd[1]: Starting Initial cloud-init job (pre-networking)...
Aug 13 06:46:52 ubuntu cloud-init[725]: Cloud-init v. 20.2-45-g5f7825e2-0ubuntu1~18.04.1 running 'init-local' at Fri, 13 Aug 2021 06:46
Aug 13 06:46:52 ubuntu systemd[1]: Started Initial cloud-init job (pre-networking).

● cloud-init.service - Initial cloud-init job (metadata service crawler)
   Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2021-08-13 06:46:52 UTC; 9min ago
  Process: 801 ExecStart=/usr/bin/cloud-init init (code=exited, status=0/SUCCESS)
 Main PID: 801 (code=exited, status=0/SUCCESS)

Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: +--------+-------+-----------+-----------+-------+-------------------+
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: |  eth0  | False |     .     |     .     |   .   | 00:0c:29:f1:72:2b |
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: |   lo   |  True | 127.0.0.1 | 255.0.0.0 |  host |         .         |
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: |   lo   |  True |  ::1/128  |     .     |  host |         .         |
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: +--------+-------+-----------+-----------+-------+-------------------+
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: +++++++++++++++++++Route IPv6 info+++++++++++++++++++
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: +-------+-------------+---------+-----------+-------+
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: | Route | Destination | Gateway | Interface | Flags |
Aug 13 06:46:52 ubuntu cloud-init[801]: ci-info: +-------+-------------+---------+-----------+-------+
Aug 13 06:46:52 ubuntu systemd[1]: Started Initial cloud-init job (metadata service crawler).

● cloud-config.service - Apply the settings specified in cloud-config
   Loaded: loaded (/lib/systemd/system/cloud-config.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2021-08-13 06:46:54 UTC; 9min ago
  Process: 968 ExecStart=/usr/bin/cloud-init modules --mode=config (code=exited, status=0/SUCCESS)
 Main PID: 968 (code=exited, status=0/SUCCESS)

Aug 13 06:46:54 ubuntu systemd[1]: Starting Apply the settings specified in cloud-config...
Aug 13 06:46:54 ubuntu cloud-init[968]: Cloud-init v. 20.2-45-g5f7825e2-0ubuntu1~18.04.1 running 'modules:config' at Fri, 13 Aug 2021 0
Aug 13 06:46:54 ubuntu systemd[1]: Started Apply the settings specified in cloud-config.

● cloud-final.service - Execute cloud user/final scripts
   Loaded: loaded (/lib/systemd/system/cloud-final.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2021-08-13 06:46:55 UTC; 9min ago
  Process: 1008 ExecStart=/usr/bin/cloud-init modules --mode=final (code=exited, status=0/SUCCESS)
 Main PID: 1008 (code=exited, status=0/SUCCESS)

Aug 13 06:46:54 ubuntu systemd[1]: Starting Execute cloud user/final scripts...
Aug 13 06:46:55 ubuntu cloud-init[1008]: Cloud-init v. 20.2-45-g5f7825e2-0ubuntu1~18.04.1 running 'modules:final' at Fri, 13 Aug 2021 0
Aug 13 06:46:55 ubuntu cloud-init[1008]: Cloud-init v. 20.2-45-g5f7825e2-0ubuntu1~18.04.1 finished at Fri, 13 Aug 2021 06:46:55 +0000.
Aug 13 06:46:55 ubuntu cloud-init[1008]: 2021-08-13 06:46:55,311 - cc_final_message.py[WARNING]: Used fallback datasource
Aug 13 06:46:55 ubuntu systemd[1]: Started Execute cloud user/final scripts.
```
停止并禁用cloud-init相关服务
```bash
root@ubuntu:/# systemctl stop cloud-init-local cloud-init cloud-config cloud-final
root@ubuntu:/# systemctl disable cloud-init-local cloud-init cloud-config cloud-final
Removed /etc/systemd/system/cloud-init.target.wants/cloud-final.service.
Removed /etc/systemd/system/cloud-init.target.wants/cloud-init-local.service.
Removed /etc/systemd/system/cloud-init.target.wants/cloud-init.service.
Removed /etc/systemd/system/cloud-init.target.wants/cloud-config.service.
```

查看cloud-init.target服务
```bash
root@ubuntu:/# systemctl status cloud-init.target
● cloud-init.target - Cloud-init target
   Loaded: loaded (/lib/systemd/system/cloud-init.target; enabled-runtime; vendor preset: enabled)
   Active: active since Fri 2021-08-13 06:46:55 UTC; 11min ago

Aug 13 06:46:55 ubuntu systemd[1]: Reached target Cloud-init target.
```

停止并禁用cloud-init.target相关服务
```bash
root@ubuntu:/# systemctl stop cloud-init.target
root@ubuntu:/# systemctl disable cloud-init.target
```

#### 安装ifupdown服务
安装包* ifupdown_0.8.17ubuntu1.1_amd64.deb
```bash
root@ubuntu:~# dpkg -i ifupdown_0.8.17ubuntu1.1_amd64.deb
Selecting previously unselected package ifupdown.
(Reading database ... 67145 files and directories currently installed.)
Preparing to unpack ifupdown_0.8.17ubuntu1.1_amd64.deb ...
Unpacking ifupdown (0.8.17ubuntu1.1) ...
Setting up ifupdown (0.8.17ubuntu1.1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/networking.service → /lib/systemd/system/networking.service.
Created symlink /etc/systemd/system/network-online.target.wants/networking.service → /lib/systemd/system/networking.service.
Processing triggers for systemd (237-3ubuntu10.50) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```
启动并查看networking服务
```bash
root@ubuntu:~# systemctl start networking
root@ubuntu:~# systemctl status networking
● networking.service - Raise network interfaces
   Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2021-08-13 07:26:52 UTC; 1s ago
     Docs: man:interfaces(5)
  Process: 1653 ExecStart=/sbin/ifup -a --read-environment (code=exited, status=0/SUCCESS)
  Process: 1651 ExecStartPre=/bin/sh -c [ "$CONFIGURE_INTERFACES" != "no" ] && [ -n "$(ifquery --read-environment --list --exclude=lo)"
 Main PID: 1653 (code=exited, status=0/SUCCESS)

Aug 13 07:26:52 ubuntu systemd[1]: Starting Raise network interfaces...
Aug 13 07:26:52 ubuntu systemd[1]: Started Raise network interfaces.
```

#### 禁用系统netplan及systemd-networkd服务
查看systemd-networkd相关服务状态
```bash
root@ubuntu:~# systemctl status systemd-networkd networkd-dispatcher.service systemd-networkd-wait-online systemd-resolved
● systemd-networkd.service - Network Service
   Loaded: loaded (/lib/systemd/system/systemd-networkd.service; enabled; vendor preset: enab
   Active: active (running) since Fri 2021-08-13 07:10:02 UTC; 18min ago
     Docs: man:systemd-networkd.service(8)
 Main PID: 670 (systemd-network)
   Status: "Processing requests..."
    Tasks: 1 (limit: 1077)
   CGroup: /system.slice/systemd-networkd.service
           └─670 /lib/systemd/systemd-networkd

Aug 13 07:10:02 ubuntu systemd[1]: Starting Network Service...
Aug 13 07:10:02 ubuntu systemd-networkd[670]: Enumeration completed
Aug 13 07:10:02 ubuntu systemd[1]: Started Network Service.
Aug 13 07:11:11 ubuntu systemd-networkd[670]: eth0: Link UP
Aug 13 07:11:11 ubuntu systemd-networkd[670]: eth0: Gained carrier
Aug 13 07:11:12 ubuntu systemd-networkd[670]: eth0: Gained IPv6LL

● networkd-dispatcher.service - Dispatcher daemon for systemd-networkd
   Loaded: loaded (/lib/systemd/system/networkd-dispatcher.service; enabled; vendor preset: e
   Active: active (running) since Fri 2021-08-13 07:10:05 UTC; 18min ago
 Main PID: 750 (networkd-dispat)
    Tasks: 2 (limit: 1077)
   CGroup: /system.slice/networkd-dispatcher.service
           └─750 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers

Aug 13 07:10:03 ubuntu systemd[1]: Starting Dispatcher daemon for systemd-networkd...
Aug 13 07:10:05 ubuntu networkd-dispatcher[750]: No valid path found for iwconfig
Aug 13 07:10:05 ubuntu systemd[1]: Started Dispatcher daemon for systemd-networkd.

● systemd-networkd-wait-online.service - Wait for Network to be Configured
   Loaded: loaded (/lib/systemd/system/systemd-networkd-wait-online.service; enabled; vendor
   Active: active (exited) since Fri 2021-08-13 07:10:02 UTC; 18min ago
     Docs: man:systemd-networkd-wait-online.service(8)
 Main PID: 691 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 1077)
   CGroup: /system.slice/systemd-networkd-wait-online.service

Aug 13 07:10:02 ubuntu systemd[1]: Starting Wait for Network to be Configured...
Aug 13 07:10:02 ubuntu systemd-networkd-wait-online[691]: ignoring: lo
Aug 13 07:10:02 ubuntu systemd[1]: Started Wait for Network to be Configured.

● systemd-resolved.service - Network Name Resolution
   Loaded: loaded (/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enab
   Active: active (running) since Fri 2021-08-13 07:11:11 UTC; 17min ago
     Docs: man:systemd-resolved.service(8)
           https://www.freedesktop.org/wiki/Software/systemd/resolved
           https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-ma
           https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
 Main PID: 1147 (systemd-resolve)
   Status: "Processing requests..."
    Tasks: 1 (limit: 1077)
   CGroup: /system.slice/systemd-resolved.service
           └─1147 /lib/systemd/systemd-resolved

Aug 13 07:11:11 ubuntu systemd[1]: Starting Network Name Resolution...
Aug 13 07:11:11 ubuntu systemd-resolved[1147]: Positive Trust Anchors:
Aug 13 07:11:11 ubuntu systemd-resolved[1147]: . IN DS 19036 8 2 49aac11d7b6f6446702e54a16073
Aug 13 07:11:11 ubuntu systemd-resolved[1147]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65
Aug 13 07:11:11 ubuntu systemd-resolved[1147]: Negative trust anchors: 10.in-addr.arpa 16.172
Aug 13 07:11:11 ubuntu systemd-resolved[1147]: Using system hostname 'ubuntu'.
Aug 13 07:11:11 ubuntu systemd[1]: Started Network Name Resolution.
```

停止并关闭systemd-networkd相关服务
```bash
root@ubuntu:~# systemctl stop systemd-networkd networkd-dispatcher.service systemd-networkd-wait-online systemd-resolved
Warning: Stopping systemd-networkd.service, but it can still be activated by:
  systemd-networkd.socket
root@ubuntu:~# systemctl disable systemd-networkd networkd-dispatcher.service systemd-networkd-wait-online systemd-resolved
Removed /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service.
Removed /etc/systemd/system/sockets.target.wants/systemd-networkd.socket.
Removed /etc/systemd/system/multi-user.target.wants/systemd-resolved.service.
Removed /etc/systemd/system/multi-user.target.wants/systemd-networkd.service.
Removed /etc/systemd/system/multi-user.target.wants/networkd-dispatcher.service.
Removed /etc/systemd/system/dbus-org.freedesktop.resolve1.service.
```

#### 安装vsftpd服务
安装包 * vsftpd_3.0.3-9build1_amd64.deb, ssl-cert_1.0.39_all.deb
```bash
root@ubuntu:~# dpkg -i vsftpd_3.0.3-9build1_amd64.deb ssl-cert_1.0.39_all.deb
Selecting previously unselected package vsftpd.
(Reading database ... 67182 files and directories currently installed.)
Preparing to unpack vsftpd_3.0.3-9build1_amd64.deb ...
Unpacking vsftpd (3.0.3-9build1) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack ssl-cert_1.0.39_all.deb ...
Unpacking ssl-cert (1.0.39) ...
Setting up vsftpd (3.0.3-9build1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /lib/systemd/system/vsftpd.service.
Setting up ssl-cert (1.0.39) ...
Processing triggers for systemd (237-3ubuntu10.50) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

修改vsftpd服务配置: vim /etc/vsftpd.conf
```bash
31  write_enable=YES
122 chroot_local_user=YES
123 chroot_list_enable=YES
125 chroot_list_file=/etc/vsftpd/chroot_list
126 allow_writeable_chroot=YES
158 pasv_enable=YES
159 pasv_min_port=64000
160 pasv_max_port=64321
161 port_enable=YES
162 pasv_address=127.0.0.1
163 pasv_addr_resolve=NO
```

创建chroot_list文件
```bash
root@ubuntu:~# cd /etc
root@ubuntu:/etc# mkdir vsftpd
root@ubuntu:/etc# cd vsftpd/
root@ubuntu:/etc/vsftpd# touch chroot_list
```

重启并查看vsftpd服务
```bash
root@ubuntu:/etc/vsftpd# systemctl restart vsftpd
root@ubuntu:/etc/vsftpd# systemctl status vsftpd
● vsftpd.service - vsftpd FTP server
   Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-08-13 08:11:34 UTC; 5s ago
  Process: 2224 ExecStartPre=/bin/mkdir -p /var/run/vsftpd/empty (code=exited, status=0/SUCCESS)
 Main PID: 2225 (vsftpd)
    Tasks: 1 (limit: 1077)
   CGroup: /system.slice/vsftpd.service
           └─2225 /usr/sbin/vsftpd /etc/vsftpd.conf

Aug 13 08:11:34 ubuntu systemd[1]: Starting vsftpd FTP server...
Aug 13 08:11:34 ubuntu systemd[1]: Started vsftpd FTP server.
```

创建ftpuser用户
```bash
root@ubuntu:/etc/vsftpd# useradd ftpuser
root@ubuntu:/etc/vsftpd# passwd ftpuser
Enter new UNIX password: ftpuser123
Retype new UNIX password: ftpuser123
passwd: password updated successfully
```

创建ftpuser目录并授权
```bash
root@ubuntu:/# cd /home
root@ubuntu:/home# mkdir ftpuser
root@ubuntu:/home# chmod 777 ftpuser/
root@ubuntu:/home# chown ftpuser:ftpuser ftpuser/
```

#### 安装python2.7并设置为系统默认python环境
Ubuntu18.04由于默认使用Python3.6, 这里安装Python2.7环境并设置为系统默认python环境
安装包:
python2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb
python2.7_2.7.17-1~18.04ubuntu1.6_amd64.deb
libpython2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb
libpython2.7-stdlib_2.7.17-1~18.04ubuntu1.6_amd64.deb

```bash
root@ubuntu:~/Python2.7# dpkg -i python2.7_2.7.17-1~18.04ubuntu1.6_amd64.deb python2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb libpython2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb libpython2.7-stdlib_2.7.17-1~18.04ubuntu1.6_amd64.deb
Selecting previously unselected package python2.7.
(Reading database ... 67248 files and directories currently installed.)
Preparing to unpack python2.7_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking python2.7 (2.7.17-1~18.04ubuntu1.6) ...
Selecting previously unselected package python2.7-minimal.
Preparing to unpack python2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking python2.7-minimal (2.7.17-1~18.04ubuntu1.6) ...
Selecting previously unselected package libpython2.7-minimal:amd64.
Preparing to unpack libpython2.7-minimal_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking libpython2.7-minimal:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Selecting previously unselected package libpython2.7-stdlib:amd64.
Preparing to unpack libpython2.7-stdlib_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking libpython2.7-stdlib:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Setting up libpython2.7-minimal:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Setting up libpython2.7-stdlib:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Setting up python2.7-minimal (2.7.17-1~18.04ubuntu1.6) ...
Linking and byte-compiling packages for runtime python2.7...
Setting up python2.7 (2.7.17-1~18.04ubuntu1.6) ...
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

设置为系统默认python环境
```bash
root@ubuntu:~# ln -sf /usr/bin/python2.7 /usr/bin/python
```

#### 安装python-dev
安装包:
libc6_2.27-3ubuntu1.4_amd64.deb
libc6-dev_2.27-3ubuntu1.4_amd64.deb
libc-dev-bin_2.27-3ubuntu1.4_amd64.deb
libexpat1-dev_2.2.5-3ubuntu0.2_amd64.deb
libpython2.7_2.7.17-1~18.04ubuntu1.6_amd64.deb
libpython2.7-dev_2.7.17-1~18.04ubuntu1.6_amd64.deb
libpython-dev_2.7.15~rc1-1_amd64.deb
libpython-stdlib_2.7.15~rc1-1_amd64.deb
linux-libc-dev_4.15.0-154.161_amd64.deb
manpages-dev_4.15-1_all.deb
python_2.7.15~rc1-1_amd64.deb
python2.7-dev_2.7.17-1~18.04ubuntu1.6_amd64.deb
python-dev_2.7.15~rc1-1_amd64.deb
python-minimal_2.7.15~rc1-1_amd64.deb

第一次安装遇到问题, 反复安装一次即可解决
```bash
root@ubuntu:~/python-dev# dpkg -i *.deb
(Reading database ... 71747 files and directories currently installed.)
Preparing to unpack libc6_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc6:amd64 (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Preparing to unpack libc6-dev_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc6-dev:amd64 (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Preparing to unpack libc-dev-bin_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc-dev-bin (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Preparing to unpack libexpat1-dev_2.2.5-3ubuntu0.2_amd64.deb ...
Unpacking libexpat1-dev:amd64 (2.2.5-3ubuntu0.2) over (2.2.5-3ubuntu0.2) ...
Preparing to unpack libpython2.7_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking libpython2.7:amd64 (2.7.17-1~18.04ubuntu1.6) over (2.7.17-1~18.04ubuntu1.6) ...
Preparing to unpack libpython2.7-dev_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking libpython2.7-dev:amd64 (2.7.17-1~18.04ubuntu1.6) over (2.7.17-1~18.04ubuntu1.6) ...
Preparing to unpack libpython-dev_2.7.15~rc1-1_amd64.deb ...
Unpacking libpython-dev:amd64 (2.7.15~rc1-1) over (2.7.15~rc1-1) ...
Preparing to unpack libpython-stdlib_2.7.15~rc1-1_amd64.deb ...
Unpacking libpython-stdlib:amd64 (2.7.15~rc1-1) over (2.7.15~rc1-1) ...
Preparing to unpack linux-libc-dev_4.15.0-154.161_amd64.deb ...
Unpacking linux-libc-dev:amd64 (4.15.0-154.161) over (4.15.0-154.161) ...
Preparing to unpack manpages-dev_4.15-1_all.deb ...
Unpacking manpages-dev (4.15-1) over (4.15-1) ...
Preparing to unpack python_2.7.15~rc1-1_amd64.deb ...
Unpacking python (2.7.15~rc1-1) ...
Preparing to unpack python2.7-dev_2.7.17-1~18.04ubuntu1.6_amd64.deb ...
Unpacking python2.7-dev (2.7.17-1~18.04ubuntu1.6) over (2.7.17-1~18.04ubuntu1.6) ...
Preparing to unpack python-dev_2.7.15~rc1-1_amd64.deb ...
Unpacking python-dev (2.7.15~rc1-1) over (2.7.15~rc1-1) ...
Preparing to unpack python-minimal_2.7.15~rc1-1_amd64.deb ...
Unpacking python-minimal (2.7.15~rc1-1) over (2.7.15~rc1-1) ...
Setting up libc6:amd64 (2.27-3ubuntu1.4) ...
Setting up libc-dev-bin (2.27-3ubuntu1.4) ...
Setting up libpython2.7:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Setting up libpython-stdlib:amd64 (2.7.15~rc1-1) ...
Setting up linux-libc-dev:amd64 (4.15.0-154.161) ...
Setting up manpages-dev (4.15-1) ...
Setting up python-minimal (2.7.15~rc1-1) ...
Setting up libc6-dev:amd64 (2.27-3ubuntu1.4) ...
Setting up libexpat1-dev:amd64 (2.2.5-3ubuntu0.2) ...
Setting up libpython2.7-dev:amd64 (2.7.17-1~18.04ubuntu1.6) ...
Setting up libpython-dev:amd64 (2.7.15~rc1-1) ...
Setting up python (2.7.15~rc1-1) ...
Setting up python2.7-dev (2.7.17-1~18.04ubuntu1.6) ...
Setting up python-dev (2.7.15~rc1-1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.2) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

#### 安装gcc
psutil安装需要gcc支持, 否则会报错:
```bash
x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPSUTIL_POSIX=1 -DPSUTIL_VERSION=522 -DPSUTIL_LINUX=1 -DPSUTIL_ETHTOOL_MISSING_TYPES=1 -I/usr/include/python2.7 -c psutil/_psutil_linux.c -o build/temp.linux-x86_64-2.7/psutil/_psutil_linux.o
unable to execute 'x86_64-linux-gnu-gcc': No such file or directory
error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
```
gcc安装包:
```bash
-rw-r--r-- 1 root root    3388 Aug 13 08:34 binutils_2.30-21ubuntu1~18.04.5_amd64.deb
-rw-r--r-- 1 root root  196672 Aug 13 08:34 binutils-common_2.30-21ubuntu1~18.04.5_amd64.deb
-rw-r--r-- 1 root root 1838912 Aug 13 08:34 binutils-x86-64-linux-gnu_2.30-21ubuntu1~18.04.5_amd64.deb
-rw-r--r-- 1 root root   27656 Aug 13 08:34 cpp_4%3a7.4.0-1ubuntu2.3_amd64.deb
-rw-r--r-- 1 root root 8591092 Aug 13 08:34 cpp-7_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root    5184 Aug 13 08:34 gcc_4%3a7.4.0-1ubuntu2.3_amd64.deb
-rw-r--r-- 1 root root 9381188 Aug 13 08:34 gcc-7_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root   18268 Aug 13 08:34 gcc-7-base_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  358376 Aug 13 08:34 libasan4_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root    9192 Aug 13 08:34 libatomic1_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  488584 Aug 13 08:34 libbinutils_2.30-21ubuntu1~18.04.5_amd64.deb
-rw-r--r-- 1 root root 2831804 Aug 13 08:34 libc6_2.27-3ubuntu1.4_amd64.deb
-rw-r--r-- 1 root root 2584868 Aug 13 08:34 libc6-dev_2.27-3ubuntu1.4_amd64.deb
-rw-r--r-- 1 root root   39384 Aug 13 08:34 libcc1-0_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root   71848 Aug 13 08:34 libc-dev-bin_2.27-3ubuntu1.4_amd64.deb
-rw-r--r-- 1 root root   42464 Aug 13 08:34 libcilkrts5_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root 2378308 Aug 13 08:34 libgcc-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root   76452 Aug 13 08:34 libgomp1_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  551452 Aug 13 08:34 libisl19_0.19-1_amd64.deb
-rw-r--r-- 1 root root   27904 Aug 13 08:34 libitm1_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  132688 Aug 13 08:34 liblsan0_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root   40784 Aug 13 08:34 libmpc3_1.1.0-1_amd64.deb
-rw-r--r-- 1 root root   11636 Aug 13 08:34 libmpx2_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  133600 Aug 13 08:34 libquadmath0_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  288108 Aug 13 08:34 libtsan0_8.4.0-1ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  126228 Aug 13 08:34 libubsan0_7.5.0-3ubuntu1~18.04_amd64.deb
-rw-r--r-- 1 root root  993704 Aug 13 08:34 linux-libc-dev_4.15.0-153.160_amd64.deb
-rw-r--r-- 1 root root 2216684 Aug 13 08:34 manpages-dev_4.15-1_all.deb
```

```bash
root@ubuntu:~/gcc# dpkg -i *.deb
Selecting previously unselected package binutils.
(Reading database ... 71802 files and directories currently installed.)
Preparing to unpack binutils_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package binutils-common:amd64.
Preparing to unpack binutils-common_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils-common:amd64 (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package binutils-x86-64-linux-gnu.
Preparing to unpack binutils-x86-64-linux-gnu_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.5) ...
Selecting previously unselected package cpp.
Preparing to unpack cpp_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking cpp (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package cpp-7.
Preparing to unpack cpp-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking cpp-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc.
Preparing to unpack gcc_4%3a7.4.0-1ubuntu2.3_amd64.deb ...
Unpacking gcc (4:7.4.0-1ubuntu2.3) ...
Selecting previously unselected package gcc-7.
Preparing to unpack gcc-7_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package gcc-7-base:amd64.
Preparing to unpack gcc-7-base_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libasan4:amd64.
Preparing to unpack libasan4_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libatomic1:amd64.
Preparing to unpack libatomic1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libbinutils:amd64.
Preparing to unpack libbinutils_2.30-21ubuntu1~18.04.5_amd64.deb ...
Unpacking libbinutils:amd64 (2.30-21ubuntu1~18.04.5) ...
Preparing to unpack libc6_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc6:amd64 (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Preparing to unpack libc6-dev_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc6-dev:amd64 (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Selecting previously unselected package libcc1-0:amd64.
Preparing to unpack libcc1-0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Preparing to unpack libc-dev-bin_2.27-3ubuntu1.4_amd64.deb ...
Unpacking libc-dev-bin (2.27-3ubuntu1.4) over (2.27-3ubuntu1.4) ...
Selecting previously unselected package libcilkrts5:amd64.
Preparing to unpack libcilkrts5_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libgcc-7-dev:amd64.
Preparing to unpack libgcc-7-dev_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Selecting previously unselected package libgomp1:amd64.
Preparing to unpack libgomp1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libisl19:amd64.
Preparing to unpack libisl19_0.19-1_amd64.deb ...
Unpacking libisl19:amd64 (0.19-1) ...
Selecting previously unselected package libitm1:amd64.
Preparing to unpack libitm1_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package liblsan0:amd64.
Preparing to unpack liblsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libmpc3:amd64.
Preparing to unpack libmpc3_1.1.0-1_amd64.deb ...
Unpacking libmpc3:amd64 (1.1.0-1) ...
Selecting previously unselected package libmpx2:amd64.
Preparing to unpack libmpx2_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libquadmath0:amd64.
Preparing to unpack libquadmath0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libtsan0:amd64.
Preparing to unpack libtsan0_8.4.0-1ubuntu1~18.04_amd64.deb ...
Unpacking libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Selecting previously unselected package libubsan0:amd64.
Preparing to unpack libubsan0_7.5.0-3ubuntu1~18.04_amd64.deb ...
Unpacking libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
dpkg: warning: downgrading linux-libc-dev:amd64 from 4.15.0-154.161 to 4.15.0-153.160
Preparing to unpack linux-libc-dev_4.15.0-153.160_amd64.deb ...
Unpacking linux-libc-dev:amd64 (4.15.0-153.160) over (4.15.0-154.161) ...
Preparing to unpack manpages-dev_4.15-1_all.deb ...
Unpacking manpages-dev (4.15-1) over (4.15-1) ...
Setting up binutils-common:amd64 (2.30-21ubuntu1~18.04.5) ...
Setting up gcc-7-base:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libc6:amd64 (2.27-3ubuntu1.4) ...
Setting up libcc1-0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libc-dev-bin (2.27-3ubuntu1.4) ...
Setting up libcilkrts5:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libgomp1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libisl19:amd64 (0.19-1) ...
Setting up libitm1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up liblsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libmpc3:amd64 (1.1.0-1) ...
Setting up libmpx2:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libquadmath0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libtsan0:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libubsan0:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up linux-libc-dev:amd64 (4.15.0-153.160) ...
Setting up manpages-dev (4.15-1) ...
Setting up cpp-7 (7.5.0-3ubuntu1~18.04) ...
Setting up libasan4:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up libatomic1:amd64 (8.4.0-1ubuntu1~18.04) ...
Setting up libbinutils:amd64 (2.30-21ubuntu1~18.04.5) ...
Setting up libc6-dev:amd64 (2.27-3ubuntu1.4) ...
Setting up libgcc-7-dev:amd64 (7.5.0-3ubuntu1~18.04) ...
Setting up binutils-x86-64-linux-gnu (2.30-21ubuntu1~18.04.5) ...
Setting up cpp (4:7.4.0-1ubuntu2.3) ...
Setting up binutils (2.30-21ubuntu1~18.04.5) ...
Setting up gcc-7 (7.5.0-3ubuntu1~18.04) ...
Setting up gcc (4:7.4.0-1ubuntu2.3) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.2) ...
```

#### 安装psutil
安装包: psutil-5.2.2.tar.gz

```bash
root@ubuntu:~# tar -zxvf psutil-5.2.2.tar.gz
psutil-5.2.2/
psutil-5.2.2/IDEAS
psutil-5.2.2/psutil/
psutil-5.2.2/psutil/_psutil_common.c
psutil-5.2.2/psutil/_psutil_bsd.c
psutil-5.2.2/psutil/arch/
psutil-5.2.2/psutil/arch/osx/
psutil-5.2.2/psutil/arch/osx/process_info.c
psutil-5.2.2/psutil/arch/osx/process_info.h
psutil-5.2.2/psutil/arch/windows/
psutil-5.2.2/psutil/arch/windows/process_handles.h
psutil-5.2.2/psutil/arch/windows/inet_ntop.c
psutil-5.2.2/psutil/arch/windows/security.c
psutil-5.2.2/psutil/arch/windows/security.h
psutil-5.2.2/psutil/arch/windows/process_info.c
psutil-5.2.2/psutil/arch/windows/glpi.h
psutil-5.2.2/psutil/arch/windows/services.c
psutil-5.2.2/psutil/arch/windows/process_handles.c
psutil-5.2.2/psutil/arch/windows/ntextapi.h
psutil-5.2.2/psutil/arch/windows/process_info.h
psutil-5.2.2/psutil/arch/windows/inet_ntop.h
psutil-5.2.2/psutil/arch/windows/services.h
psutil-5.2.2/psutil/arch/bsd/
psutil-5.2.2/psutil/arch/bsd/netbsd_socks.c
psutil-5.2.2/psutil/arch/bsd/netbsd.h
psutil-5.2.2/psutil/arch/bsd/netbsd_socks.h
psutil-5.2.2/psutil/arch/bsd/freebsd.h
psutil-5.2.2/psutil/arch/bsd/openbsd.h
psutil-5.2.2/psutil/arch/bsd/freebsd.c
psutil-5.2.2/psutil/arch/bsd/freebsd_socks.c
psutil-5.2.2/psutil/arch/bsd/freebsd_socks.h
psutil-5.2.2/psutil/arch/bsd/netbsd.c
psutil-5.2.2/psutil/arch/bsd/openbsd.c
psutil-5.2.2/psutil/arch/solaris/
psutil-5.2.2/psutil/arch/solaris/v10/
psutil-5.2.2/psutil/arch/solaris/v10/ifaddrs.h
psutil-5.2.2/psutil/arch/solaris/v10/ifaddrs.c
psutil-5.2.2/psutil/_psutil_posix.h
psutil-5.2.2/psutil/_psutil_sunos.c
psutil-5.2.2/psutil/_psutil_common.h
psutil-5.2.2/psutil/_psutil_windows.c
psutil-5.2.2/psutil/_psposix.py
psutil-5.2.2/psutil/_psbsd.py
psutil-5.2.2/psutil/_pssunos.py
psutil-5.2.2/psutil/_pslinux.py
psutil-5.2.2/psutil/_psutil_linux.c
psutil-5.2.2/psutil/_common.py
psutil-5.2.2/psutil/__init__.py
psutil-5.2.2/psutil/tests/
psutil-5.2.2/psutil/tests/test_bsd.py
psutil-5.2.2/psutil/tests/test_memory_leaks.py
psutil-5.2.2/psutil/tests/test_system.py
psutil-5.2.2/psutil/tests/test_linux.py
psutil-5.2.2/psutil/tests/test_misc.py
psutil-5.2.2/psutil/tests/__init__.py
psutil-5.2.2/psutil/tests/test_osx.py
psutil-5.2.2/psutil/tests/test_process.py
psutil-5.2.2/psutil/tests/test_windows.py
psutil-5.2.2/psutil/tests/runner.py
psutil-5.2.2/psutil/tests/test_posix.py
psutil-5.2.2/psutil/tests/test_sunos.py
psutil-5.2.2/psutil/tests/README.rst
psutil-5.2.2/psutil/_psutil_osx.c
psutil-5.2.2/psutil/_psosx.py
psutil-5.2.2/psutil/_compat.py
psutil-5.2.2/psutil/_psutil_posix.c
psutil-5.2.2/psutil/_pswindows.py
psutil-5.2.2/setup.py
psutil-5.2.2/docs/
psutil-5.2.2/docs/Makefile
psutil-5.2.2/docs/_template/
psutil-5.2.2/docs/_template/indexsidebar.html
psutil-5.2.2/docs/_template/page.html
psutil-5.2.2/docs/_template/indexcontent.html
psutil-5.2.2/docs/_template/globaltoc.html
psutil-5.2.2/docs/_static/
psutil-5.2.2/docs/_static/copybutton.js
psutil-5.2.2/docs/_static/sidebar.js
psutil-5.2.2/docs/index.rst
psutil-5.2.2/docs/make.bat
psutil-5.2.2/docs/conf.py
psutil-5.2.2/docs/README
psutil-5.2.2/docs/_themes/
psutil-5.2.2/docs/_themes/pydoctheme/
psutil-5.2.2/docs/_themes/pydoctheme/theme.conf
psutil-5.2.2/docs/_themes/pydoctheme/static/
psutil-5.2.2/docs/_themes/pydoctheme/static/pydoctheme.css
psutil-5.2.2/Makefile
psutil-5.2.2/MANIFEST.in
psutil-5.2.2/tox.ini
psutil-5.2.2/.coveragerc
psutil-5.2.2/psutil.egg-info/
psutil-5.2.2/psutil.egg-info/SOURCES.txt
psutil-5.2.2/psutil.egg-info/top_level.txt
psutil-5.2.2/psutil.egg-info/not-zip-safe
psutil-5.2.2/psutil.egg-info/dependency_links.txt
psutil-5.2.2/psutil.egg-info/PKG-INFO
psutil-5.2.2/LICENSE
psutil-5.2.2/make.bat
psutil-5.2.2/scripts/
psutil-5.2.2/scripts/procsmem.py
psutil-5.2.2/scripts/fans.py
psutil-5.2.2/scripts/temperatures.py
psutil-5.2.2/scripts/iotop.py
psutil-5.2.2/scripts/ifconfig.py
psutil-5.2.2/scripts/meminfo.py
psutil-5.2.2/scripts/procinfo.py
psutil-5.2.2/scripts/ps.py
psutil-5.2.2/scripts/sensors.py
psutil-5.2.2/scripts/killall.py
psutil-5.2.2/scripts/pmap.py
psutil-5.2.2/scripts/free.py
psutil-5.2.2/scripts/internal/
psutil-5.2.2/scripts/internal/bench_oneshot.py
psutil-5.2.2/scripts/internal/bench_oneshot_2.py
psutil-5.2.2/scripts/internal/print_announce.py
psutil-5.2.2/scripts/internal/download_exes.py
psutil-5.2.2/scripts/internal/winmake.py
psutil-5.2.2/scripts/who.py
psutil-5.2.2/scripts/disk_usage.py
psutil-5.2.2/scripts/battery.py
psutil-5.2.2/scripts/pstree.py
psutil-5.2.2/scripts/cpu_distribution.py
psutil-5.2.2/scripts/top.py
psutil-5.2.2/scripts/pidof.py
psutil-5.2.2/scripts/netstat.py
psutil-5.2.2/scripts/winservices.py
psutil-5.2.2/scripts/nettop.py
psutil-5.2.2/setup.cfg
psutil-5.2.2/INSTALL.rst
psutil-5.2.2/CREDITS
psutil-5.2.2/HISTORY.rst
psutil-5.2.2/DEVGUIDE.rst
psutil-5.2.2/PKG-INFO
psutil-5.2.2/README.rst
root@ubuntu:~# cd psutil-5.2.2/
```

```bash
root@ubuntu:~/psutil-5.2.2# python setup.py install
/usr/lib/python2.7/distutils/dist.py:267: UserWarning: Unknown distribution option: 'zip_safe'
  warnings.warn(msg)
/usr/lib/python2.7/distutils/dist.py:267: UserWarning: Unknown distribution option: 'test_suite'
  warnings.warn(msg)
/usr/lib/python2.7/distutils/dist.py:267: UserWarning: Unknown distribution option: 'tests_require'
  warnings.warn(msg)
running install
running build
running build_py
running build_ext
building 'psutil._psutil_linux' extension
x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPSUTIL_POSIX=1 -DPSUTIL_VERSION=522 -DPSUTIL_LINUX=1 -I/usr/include/python2.7 -c psutil/_psutil_linux.c -o build/temp.linux-x86_64-2.7/psutil/_psutil_linux.o
x86_64-linux-gnu-gcc -pthread -shared -Wl,-O1 -Wl,-Bsymbolic-functions -Wl,-Bsymbolic-functions -Wl,-z,relro -fno-strict-aliasing -DNDEBUG -g -fwrapv -O2 -Wall -Wstrict-prototypes -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -Wl,-Bsymbolic-functions -Wl,-z,relro -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC build/temp.linux-x86_64-2.7/psutil/_psutil_linux.o -o build/lib.linux-x86_64-2.7/psutil/_psutil_linux.so
building 'psutil._psutil_posix' extension
x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPSUTIL_POSIX=1 -DPSUTIL_VERSION=522 -DPSUTIL_LINUX=1 -I/usr/include/python2.7 -c psutil/_psutil_posix.c -o build/temp.linux-x86_64-2.7/psutil/_psutil_posix.o
x86_64-linux-gnu-gcc -pthread -shared -Wl,-O1 -Wl,-Bsymbolic-functions -Wl,-Bsymbolic-functions -Wl,-z,relro -fno-strict-aliasing -DNDEBUG -g -fwrapv -O2 -Wall -Wstrict-prototypes -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -Wl,-Bsymbolic-functions -Wl,-z,relro -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC build/temp.linux-x86_64-2.7/psutil/_psutil_posix.o -o build/lib.linux-x86_64-2.7/psutil/_psutil_posix.so
running install_lib
creating /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_pslinux.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_compat.py -> /usr/local/lib/python2.7/dist-packages/psutil
creating /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_osx.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_sunos.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_system.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/runner.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_memory_leaks.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_process.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_posix.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_misc.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_linux.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_bsd.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/test_windows.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/tests/__init__.py -> /usr/local/lib/python2.7/dist-packages/psutil/tests
copying build/lib.linux-x86_64-2.7/psutil/_pssunos.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_common.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_psosx.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_psutil_posix.so -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_pswindows.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_psutil_linux.so -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_psbsd.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/__init__.py -> /usr/local/lib/python2.7/dist-packages/psutil
copying build/lib.linux-x86_64-2.7/psutil/_psposix.py -> /usr/local/lib/python2.7/dist-packages/psutil
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_pslinux.py to _pslinux.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_compat.py to _compat.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_osx.py to test_osx.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_sunos.py to test_sunos.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_system.py to test_system.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/runner.py to runner.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_memory_leaks.py to test_memory_leaks.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_process.py to test_process.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_posix.py to test_posix.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_misc.py to test_misc.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_linux.py to test_linux.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_bsd.py to test_bsd.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/test_windows.py to test_windows.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/tests/__init__.py to __init__.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_pssunos.py to _pssunos.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_common.py to _common.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_psosx.py to _psosx.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_pswindows.py to _pswindows.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_psbsd.py to _psbsd.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/__init__.py to __init__.pyc
byte-compiling /usr/local/lib/python2.7/dist-packages/psutil/_psposix.py to _psposix.pyc
running install_egg_info
Writing /usr/local/lib/python2.7/dist-packages/psutil-5.2.2.egg-info
```
#### netifaces
安装包:
netifaces-0.11.0.tar.gz
python-setuptools_39.0.1-2_all.deb
python-pkg-resources_39.0.1-2_all.deb

```bash
root@ubuntu:~# dpkg -i python-setuptools_39.0.1-2_all.deb  python-pkg-resources_39.0.1-2_all.deb
(Reading database ... 72501 files and directories currently installed.)
Preparing to unpack python-setuptools_39.0.1-2_all.deb ...
Unpacking python-setuptools (39.0.1-2) over (39.0.1-2) ...
Selecting previously unselected package python-pkg-resources.
Preparing to unpack python-pkg-resources_39.0.1-2_all.deb ...
Unpacking python-pkg-resources (39.0.1-2) ...
Setting up python-pkg-resources (39.0.1-2) ...
Setting up python-setuptools (39.0.1-2) ...

root@ubuntu:~# tar -zxvf netifaces-0.11.0.tar.gz
netifaces-0.11.0/
netifaces-0.11.0/LICENSE
netifaces-0.11.0/MANIFEST.in
netifaces-0.11.0/PKG-INFO
netifaces-0.11.0/README.rst
netifaces-0.11.0/netifaces.c
netifaces-0.11.0/netifaces.egg-info/
netifaces-0.11.0/netifaces.egg-info/PKG-INFO
netifaces-0.11.0/netifaces.egg-info/SOURCES.txt
netifaces-0.11.0/netifaces.egg-info/dependency_links.txt
netifaces-0.11.0/netifaces.egg-info/top_level.txt
netifaces-0.11.0/netifaces.egg-info/zip-safe
netifaces-0.11.0/setup.cfg
netifaces-0.11.0/setup.py
netifaces-0.11.0/test.py
root@ubuntu:~# cd netifaces-0.11.0/

root@ubuntu:~/netifaces-0.11.0# python setup.py install
running install
running bdist_egg
running egg_info
writing netifaces.egg-info/PKG-INFO
writing top-level names to netifaces.egg-info/top_level.txt
writing dependency_links to netifaces.egg-info/dependency_links.txt
reading manifest file 'netifaces.egg-info/SOURCES.txt'
reading manifest template 'MANIFEST.in'
writing manifest file 'netifaces.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_ext
checking for getifaddrs...found.
checking for getnameinfo...found.
checking for IPv6 socket IOCTLs...not found.
checking for optional header files...netash/ash.h netatalk/at.h netax25/ax25.h neteconet/ec.h netipx/ipx.h netpacket/packet.h netrose/rose.h linux/irda.h linux/atm.h linux/llc.h linux/tipc.h linux/dn.h.
checking whether struct sockaddr has a length field...no.
checking which sockaddr_xxx structs are defined...at ax25 in in6 ipx un rose ash ec ll atmpvc atmsvc dn irda llc.
checking for routing socket support...no.
checking for sysctl(CTL_NET...) support...no.
checking for netlink support...yes.
will use netlink to read routing table
building 'netifaces' extension
x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DNETIFACES_VERSION=0.11.0 -DHAVE_GETIFADDRS=1 -DHAVE_GETNAMEINFO=1 -DHAVE_NETASH_ASH_H=1 -DHAVE_NETATALK_AT_H=1 -DHAVE_NETAX25_AX25_H=1 -DHAVE_NETECONET_EC_H=1 -DHAVE_NETIPX_IPX_H=1 -DHAVE_NETPACKET_PACKET_H=1 -DHAVE_NETROSE_ROSE_H=1 -DHAVE_LINUX_IRDA_H=1 -DHAVE_LINUX_ATM_H=1 -DHAVE_LINUX_LLC_H=1 -DHAVE_LINUX_TIPC_H=1 -DHAVE_LINUX_DN_H=1 -DHAVE_SOCKADDR_AT=1 -DHAVE_SOCKADDR_AX25=1 -DHAVE_SOCKADDR_IN=1 -DHAVE_SOCKADDR_IN6=1 -DHAVE_SOCKADDR_IPX=1 -DHAVE_SOCKADDR_UN=1 -DHAVE_SOCKADDR_ROSE=1 -DHAVE_SOCKADDR_ASH=1 -DHAVE_SOCKADDR_EC=1 -DHAVE_SOCKADDR_LL=1 -DHAVE_SOCKADDR_ATMPVC=1 -DHAVE_SOCKADDR_ATMSVC=1 -DHAVE_SOCKADDR_DN=1 -DHAVE_SOCKADDR_IRDA=1 -DHAVE_SOCKADDR_LLC=1 -DHAVE_PF_NETLINK=1 -I/usr/include/python2.7 -c netifaces.c -o build/temp.linux-x86_64-2.7/netifaces.o
creating build/lib.linux-x86_64-2.7
x86_64-linux-gnu-gcc -pthread -shared -Wl,-O1 -Wl,-Bsymbolic-functions -Wl,-Bsymbolic-functions -Wl,-z,relro -fno-strict-aliasing -DNDEBUG -g -fwrapv -O2 -Wall -Wstrict-prototypes -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -Wl,-Bsymbolic-functions -Wl,-z,relro -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-gnDdqE/python2.7-2.7.17=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC build/temp.linux-x86_64-2.7/netifaces.o -o build/lib.linux-x86_64-2.7/netifaces.so
creating build/bdist.linux-x86_64
creating build/bdist.linux-x86_64/egg
copying build/lib.linux-x86_64-2.7/netifaces.so -> build/bdist.linux-x86_64/egg
creating stub loader for netifaces.so
byte-compiling build/bdist.linux-x86_64/egg/netifaces.py to netifaces.pyc
creating build/bdist.linux-x86_64/egg/EGG-INFO
copying netifaces.egg-info/PKG-INFO -> build/bdist.linux-x86_64/egg/EGG-INFO
copying netifaces.egg-info/SOURCES.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying netifaces.egg-info/dependency_links.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying netifaces.egg-info/top_level.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying netifaces.egg-info/zip-safe -> build/bdist.linux-x86_64/egg/EGG-INFO
writing build/bdist.linux-x86_64/egg/EGG-INFO/native_libs.txt
creating dist
creating 'dist/netifaces-0.11.0-py2.7-linux-x86_64.egg' and adding 'build/bdist.linux-x86_64/egg' to it
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing netifaces-0.11.0-py2.7-linux-x86_64.egg
Copying netifaces-0.11.0-py2.7-linux-x86_64.egg to /usr/local/lib/python2.7/dist-packages
Adding netifaces 0.11.0 to easy-install.pth file

Installed /usr/local/lib/python2.7/dist-packages/netifaces-0.11.0-py2.7-linux-x86_64.egg
Processing dependencies for netifaces==0.11.0
Finished processing dependencies for netifaces==0.11.0
```

#### 安装qemu-guest-agent服务
安装包: qemu-guest-agent_1%3a2.11+dfsg-1ubuntu7.37_amd64.deb
```bash
root@ubuntu:~# dpkg -i qemu-guest-agent_1%3a2.11+dfsg-1ubuntu7.37_amd64.deb
Selecting previously unselected package qemu-guest-agent.
(Reading database ... 72525 files and directories currently installed.)
Preparing to unpack qemu-guest-agent_1%3a2.11+dfsg-1ubuntu7.37_amd64.deb ...
Unpacking qemu-guest-agent (1:2.11+dfsg-1ubuntu7.37) ...
Setting up qemu-guest-agent (1:2.11+dfsg-1ubuntu7.37) ...
Processing triggers for systemd (237-3ubuntu10.50) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

查看qemu-guest-agent服务状态
```bash
root@ubuntu:~# systemctl status qemu-guest-agent
● qemu-guest-agent.service - LSB: QEMU Guest Agent startup script
   Loaded: loaded (/etc/init.d/qemu-guest-agent; generated)
   Active: active (exited) since Fri 2021-08-13 08:43:22 UTC; 1min 7s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 1077)
   CGroup: /system.slice/qemu-guest-agent.service

Aug 13 08:43:22 ubuntu systemd[1]: Starting LSB: QEMU Guest Agent startup script...
Aug 13 08:43:22 ubuntu qemu-guest-agent[8752]:  * qemu-ga: transport endpoint not found, not starting
Aug 13 08:43:22 ubuntu systemd[1]: Started LSB: QEMU Guest Agent startup script.
```

#### 网卡热加载系统服务
```bash
drwxr-xr-x  2 root root  4096 Aug 13 08:47 ./
drwx------ 11 root root  4096 Aug 13 08:47 ../
-rw-r--r--  1 root root  4060 Aug 13 08:47 hotplug.c
-rw-r--r--  1 root root   187 Aug 13 08:47 hotplug.conf
-rw-r--r--  1 root root 12120 Aug 13 08:47 hotplugd
-rw-r--r--  1 root root   671 Aug 13 08:47 kdpa-hotplug
-rw-r--r--  1 root root   262 Aug 13 08:47 kdpa-hotplug.service
-rw-r--r--  1 root root  1231 Aug 13 08:47 nic_hotplug.py
```
以上程序拷贝到/var/www/kdpa/project/daemon/hotplug/目录下,
编译nic_hotplug.py
```bash
root@ubuntu:~/hotplug_rocky# mkdir -p /var/www/kdpa/project/daemon/hotplug
root@ubuntu:~/hotplug_rocky# cd /var/www/kdpa/project/daemon/hotplug
root@ubuntu:/var/www/kdpa/project/daemon/hotplug# cp /root/hotplug_rocky/* .
root@ubuntu:/var/www/kdpa/project/daemon/hotplug# python -m py_compile nic_hotplug.py
root@ubuntu:/var/www/kdpa/project/daemon/hotplug# chmod 755 *
root@ubuntu:/var/www/kdpa/project/daemon/hotplug# cp kdpa-hotplug.service /etc/systemd/system/.
```
启动kdpa-hotplug.service服务并设置开机启动
```bash
root@ubuntu:~# systemctl start kdpa-hotplug.service
root@ubuntu:~# systemctl status kdpa-hotplug.service
● kdpa-hotplug.service - kdpa-hotplug
   Loaded: loaded (/etc/systemd/system/kdpa-hotplug.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-08-13 08:52:25 UTC; 4s ago
  Process: 9011 ExecStart=/var/www/kdpa/project/daemon/hotplug/kdpa-hotplug start (code=exited, status=0/SUCCESS)
 Main PID: 9013 (hotplugd)
    Tasks: 1 (limit: 1077)
   CGroup: /system.slice/kdpa-hotplug.service
           └─9013 /var/www/kdpa/project/daemon/hotplug/hotplugd

Aug 13 08:52:25 ubuntu systemd[1]: Starting kdpa-hotplug...
Aug 13 08:52:25 ubuntu systemd[1]: Started kdpa-hotplug.
```

#### qga服务脚本qga_cmd.py
放到/root/qga_cmd.py

#### 网卡初始化脚本kd_nic_control.py
放到/root/kd_nic_control.py

#### 网卡eth0设置DHCP模式
允许网卡热插拔, 开机自启动, IP地址0.0.0.0, 掩码255.255.255.0
```bash
vim /etc/network/interfaces.d/eth0

auto eth0
iface eth0 inet static
address 0.0.0.0
netmask 255.255.255.0
```

#### 开机自启动脚本rc.local配置
```bash
root@ubuntu:~# touch /etc/rc.local
root@ubuntu:~# chmod 755 /etc/rc.local
root@ubuntu:~# vim /etc/rc.local

#!/bin/bash

/usr/bin/python /root/kd_nic_control.py init_conf
/usr/bin/python /root/kd_nic_control.py checksum_off
```
注:/usr/bin/python /root/kd_nic_control.py init_conf脚本只运行一次, 因此需制作镜像时添加。
