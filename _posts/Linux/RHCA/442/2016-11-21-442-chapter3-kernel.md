---
layout: post
title:  "442-chapter3-kernel"
categories: Linux
tags: RHCA 442
---


=============================================
 Displaying and Configuring Module Parameters
=============================================
# lsmod			-> 查看当前系统已经装载的模块
# modinfo usb_storage
# modinfo -p usb_storage
quirks:supplemental list of device IDs and their quirks
delay_use:seconds to delay before using a new device
swi_tru_install:TRU-Install mode (1=Full Logic (def), 2=Force CD-Rom, 3=Force Modem)
option_zero_cd:ZeroCD mode (1=Force Modem (default), 2=Allow CD-Rom
# lsmod | grep usb
usb_storage            49100  0

临时修改一个值
1).
# cat /sys/module/usb_storage/parameters/delay_use
1
# echo 5 >  /sys/module/usb_storage/parameters/delay_use
# cat /sys/module/usb_storage/parameters/delay_use
5

2).
# rmmod usb_storage					-> 卸载模块
# cat /sys/module/usb_storage/parameters/delay_use 
cat: /sys/module/usb_storage/parameters/delay_use: No such file or directory
# modprobe usb_storage delay_use=6			-> 加载模块的同时手动加载参数
# cat /sys/module/usb_storage/parameters/delay_use 
6


永久修改一个值
1).
# vim /etc/modprobe.d/usb_storage.conf
options usb_storage delay_use=4
# rmmod usb_storage
# cat /sys/module/sub_storage/parameters/delay_use
cat: /sys/module/usb_storage/parameters/delay_use: No such file or directory
# modprobe user_storage					-> 加载模块，执行此命令时，会读取/etc/modprobe.d/目录下的所有conf结尾的文件，看看有没有相关模块的选项
#  cat /sys/module/sub_storage/parameters/delay_use
4
# depmod -a						-> 下次开机还能加载，把当前的已经加载的所有模块的信息写到一个配置文件里面，下次开机如果还能读取这个配置文件，会加载文件里的所有模块

2).
# grep -n  modules /etc/rc.sysinit			->  rc.sysinit系统初始化脚本
# Load other user-defined modules
for file in /etc/sysconfig/modules/*.modules ; do	-> 找到/etc/sysconfig/modules/目录下的所有modules结尾的文件
  [ -x $file ] && $file					-> 检查是否有可执行权限，如果有，直接执行
done

# cd /etc/sysconfig/modules/
# ls 
bluez-uinput.modules  kvm.modules
# cat bluez-uinput.modules
#!/bin/sh
if [ ! -c /dev/input/uinput ] ; then
	exec /sbin/modprobe uinput >/dev/null 2>&1
fi

# vim usb_storage.modules
modprobe usb_storage delay_use=6		-> delay_use=6 也可以放在/etc/modprobe.d/usb_storage.conf中
# chmod +x usb_storage.modules
# ./usb_storage.modules
# cat /sys/module/usb_storage/parameters/delay_use
6


=================================================================
  Installing and Enabling tuned   -> rhel6新引进的系统优化程序，可以提供9种性能模式
=================================================================
# yum install tuned
# /etc/init.d/ktune start
# /etc/init.d/tuned start
# chkconfig ktune on
# chkconfig tune on
# tuned-adm list
Available profiles:
- laptop-battery-powersave
- laptop-ac-powersave
- spindown-disk
- latency-performance
- throughput-performance
- desktop-powersave
- server-powersave
- enterprise-storage
- default
Current active profile: default
# tuned-adm profile server-powersave
# cd /etc/tune-profile/
# 
