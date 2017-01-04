---
layout: post
title:  "centos7/rhel7重置root密码(rd.break和init方法)"
categories: Linux
tags: RHCE RHEL7
---

* content
{:toc}

Reference Link: http://www.2cto.com/os/201412/365394.html

centos7/rhel7进入单用户方式和重置密码方式发生了较大变化，GRUB由b引导变成了ctrl+x引导。
重置密码主要有rd.break和init两种方法。

## rd.break方法
1. 启动的时候，在启动界面，相应启动项，内核名称上按“e”
2. 进入后，找到linux16开头的地方，按“end”键到最后，输入rd.break，按ctrl+x进入
3. 进去后输入命令mount，发现根为/sysroot/，并且不能写，只有ro=readonly权限
4. 重新挂载, 赋予r,w权限
```
mount -o remount,rw /sysroot/
```
5. 改变根
```
chroot /sysroot/
```
  方法一). 修改root密码为redhat，或者输入passwd，交互修改
```
echo redhat|passwd -stdin root
```
  方法二). 还有就是先cp一份，然后修改/etc/shadow文件
6. 使selinux生效
```
touch /.autorelabel
```
7. ctrl+d 退出
8. reboot
至此，密码修改完成


## init方法
1. 启动系统，并在GRUB2启动屏显时，按下e键进入编辑模式。
2. 在linux16/linux/linuxefi所在参数行尾添加以下内容：init=/bin/sh
3. 按Ctrl+x启动到shell。
4. 挂载文件系统为可写模式
```
mount -o remount,rw /
```
5. 运行passwd, 并按提示修改root密码。
6. 如何之前系统启用了selinux，必须运行以下命令，否则将无法正常启动系统
```
touch /.autorelabel
```
7. 运行下面命令正常启动
```
exec /sbin/init
```
或者用下面命令重新启动
```
exec /sbin/reboot
```


## 附
[Why touch /.autorelabel](https://bugzilla.redhat.com/show_bug.cgi?id=449420)

