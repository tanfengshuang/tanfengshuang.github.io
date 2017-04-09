---
layout: post
title:  "更新 RHEL Atomic(270-2)"
categories: Linux
tags: RHCA 270
---

### 注册 RHEL Atomic 

###### 注册 RHEL Atomic 主机

RHEL Atomic 主机服务器必须通过网络同时连接到 subscription.rhn.redhat.com:443 和 cdn.redhat.com:443 才能进行注册。还须提供有效的红帽网络帐户用户名和密码与可用的红帽企业 Linux Atomic 主机权限才能成功注册。

```
# subscription-manager register
Username: rhn-username
Password: rhn-password
The system has been registered with ID: af27f5ed-4f79-46ba-bf37-2e478b83e45b
```
 
###### 订阅 RHEL Atomic 产品

通常，管理员将使用 subscription-manager register --auto-attach 命令将注册和订阅步骤组合在一个命令中，实际上有两项操作正在进行。 

```
# subscription-manager list

+-------------------------------------------+
    Installed Product Status
+-------------------------------------------+
Product Name:   Red Hat Enterprise Linux Atomic Host
Product ID:     271
Version:        7
Arch:           x86_64
Status:         Not Subscribed
Status Details: Not supported by a valid subscription.
Starts:
Ends:

# subscription-manager list --available | less
...Output omitted...
Subscription Name: Red Hat Employee Subscription
Provides:          Red Hat Enterprise Linux Atomic Host
                   Red Hat Enterprise Linux Atomic Host Beta
                   Red Hat Enterprise Linux Atomic Host HTB
...Output omitted...
Pool ID:           1234f9843e3d687a013e3ddd3a66ffff

# subscription-manager attach --pool=1234f9843e3d687a013e3ddd3a66ffff
Successfully attached a subscription for: Red Hat Employee Subscription

# subscription-manager list

+-------------------------------------------+
    Installed Product Status
+-------------------------------------------+
Product Name:   Red Hat Enterprise Linux Atomic Host
Product ID:     271
Version:        7
Arch:           x86_64
Status:         Subscribed
Status Details:
Starts:         04/23/2013
Ends:           12/31/2021
```

### 更新 RHEL Atomic 软件

###### OSTree 概念

在红帽网络中安装、注册和订阅 RHEL Atomic 主机后，最好升级 RHEL Atomic 位以运行最新的可用 OSTree 软件。RHEL Atomic 使用 atomic host 管理从 cdn.redhat.com 下载的文件系统 OSTree。atomic host 命令是用于下载和管理这些文件系统 OSTree 的实用工具。

yum 和 rpm 命令不能用于升级 RHEL Atomic 主机上的软件。yum 未安装。rpm 应仅用于查询，因为 RHEL Atomic 主机中唯一的可写目录为 /etc 和 /var。 

```
# atomic host status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME               REFSPEC
* 2015-02-05 14:52:09     7.1.0     9d04d17969     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
```

标有星号的版本为当前正在使用的 OSTree。由于 atomic host 命令在场景后调用 rpm-ostree 命令，因此还可使用 rpm-ostree status 生成以上输出。事实上，相比 atomic host status，管理员可能更喜欢使用 rpm-ostree status，因为它提供了额外的 -p 选项（出于美观而缩写），该选项可提供更完整且易于读取的输出。

```
# rpm-ostree status -p
============================================================
  * DEFAULT ON BOOT
----------------------------------------
  version    7.1.0
  timestamp  2015-02-05 14:52:09
  id         9d04d179695a81a0764916360fc35f64f0de04ffee80bbf9bca66af038541cd4.0
  osname     rhel-atomic-host
  refspec    rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
============================================================
```

atomic host status 和 rpm-ostree status 输出中的 id 值是 rpm-ostree status -p 输出中提供的值的缩写版本。未缩写的 id 值可用于识别 OSTree 中包含实际文件的目录。

```
# ls /ostree/deploy/rhel-atomic-host/deploy/9d04d179695a81a0764916360fc35f64f0de04ffee80bbf9bca66af038541cd4.0
bin   dev  home  lib64  mnt  ostree  root  sbin  sys      tmp  var
boot  etc  lib   media  opt  proc    run   srv   sysroot  usr
```

###### 更新 RHEL Atomic 主机软件

假设在红帽网络中正确注册和订阅了 RHEL Atomic 主机，则通过运行 atomic host upgrade 命令完成升级。 

```
# atomic host upgrade
Updating from: rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard

613 metadata, 3173 content objects fetched; 122756 KiB transferred in 338 seconds
Copying /etc changes: 10 modified, 4 removed, 36 added
Transaction complete; bootconfig swap: yes deployment count change: 1
Changed:
  NetworkManager-1:0.9.9.1-29.git20140326.4dba720.el7_0.x86_64
  NetworkManager-glib-1:0.9.9.1-29.git20140326.4dba720.el7_0.x86_64
  docker-1.3.2-4.el7.x86_64
  dracut-033-161.el7_0.173.x86_64
  gnutls-3.1.18-10.el7_0.x86_64
  kernel-3.10.0-123.13.1.el7.x86_64
  kubernetes-0.6-4.0.git993ef88.el7.x86_64
...Output omitted...
Removed:
  btrfs-progs-3.12-4.el7.x86_64
  git-1.8.3.1-4.el7.x86_64
  libgnome-keyring-3.8.0-3.el7.x86_64
  perl-4:5.16.3-283.el7.x86_64
  perl-Carp-1.26-244.el7.noarch
  perl-Encode-2.51-7.el7.x86_64
  perl-Error-1:0.17020-2.el7.noarch
...Output omitted...
Updates prepared for next boot; run "systemctl reboot" to start a reboot
```

atomic host upgrade 命令成功运行后，新的 OSTree 应可供使用。使用 atomic host status 命令显示可用的 OSTree 新列表。

```
# atomic host status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME               REFSPEC
  2015-02-06 14:52:09     7.1.1     8d04d17868     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
* 2015-02-05 14:52:09     7.1.0     9d04d17969     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
```

之前输出中的星号表示，系统正在使用旧版本的 RHEL Atomic 主机软件（版本 7.1.0）运行。由于首先列出的是更新软件版本 7.1.1，该版本为系统重启时由 GRUB2 引导加载程序选择的默认版本。要完成升级，请使用 systemctl 命令重启系统。

```
# systemctl reboot
```

###### 退出升级

RHEL Atomic 主机中内置的一个重要特性是可将升级版回滚到之前的 OSTree。运行 atomic host rollback 命令以修改部署顺序。

```
# atomic host status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME               REFSPEC
* 2015-02-06 14:52:09     7.1.1     8d04d17868     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
  2015-02-05 14:52:09     7.1.0     dcf0c846ff     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
-bash-4.2# atomic host rollback
Moving 'dcf0c846ff87f251d48439f6c90948f1183654a9b9d46b28c3f5e0f42c1ddf8e.0' to be first deployment
Transaction complete; bootconfig swap: yes deployment count change: 0
...Output omitted...
Successfully reset deployment order; run "systemctl reboot" to start a reboot
-bash-4.2# atomic host status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME               REFSPEC
  2015-02-05 14:52:09     7.1.0     dcf0c846ff     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
* 2015-02-06 14:52:09     7.1.1     8d04d17868     rhel-atomic-host     rhel-atomic-host-ostree:rhel-atomic-host/7/x86_64/standard
```

因 OSTree 顺序已发生改变，首先列出旧版软件（r 版本 7.1.0）。该版本为系统重启时选择的版本。使用 systemctl 命令重启系统以完成回滚。 


### Summary

注册 RHEL Atomic .  在本节，您学到：

*    使用 subscription-manager register 在红帽网络中注册 RHEL Atomic 主机服务器。
*    使用 subscription-manager list 命令显示在服务器上安装的产品的状态。
*    使用 subscription-manager attach --pool=ID 命令为已安装产品附上有效的订阅。

更新 RHEL Atomic 软件 .  在本节，您学到：

*    使用 atomic status 命令显示已安装的 OSTree 列表。
*    使用 atomic upgrade 命令安装最新版的 RHEL Atomic 主机 OSTree。
*    使用 atomic rollback 命令恢复到较早版本的 OSTree。
     
