---
layout: post
title:  "442-chapter3-kernel"
categories: Linux
tags: RHCA 442
---

### Queueing Theory

Many of resources in the computer can be moderated as queues.

###### Little's Law

> L = λ × W (Queue length = average arrival rate × average wait time)

###### Wait Time

> W = S + Q (Wait time = service time + queue time)

###### Queue Stablility

For a queue to maintain the same length, the rate at which new requests arrive must be the same as the rate at which requests are completed.

###### The Forced Flow Law

Xsystem = Xresource / Vresource

### Asymptotic Complexity 

*    Procedures or methods used by programs to perform tasks are called algorithms
*    One way to evaluate the efficiency of two algorithms is to look at how fast the running time, memory usage, or other resource usage of each algorithm grows as the number of inputs the algorithm needs to work on grows.
*    The asymptotic complexity of an algorithm is a measure of how fast its resource requirements grow as the size of its input grows toward infinity.


### Displaying and Configuring Module Parameters

```
# lsmod			-> 查看当前系统已经装载的模块
Module                  Size  Used by
ipv6                  336282  36 
ecb                     2209  0 
drbg                   23248  1 
ansi_cprng              4716  0 
ablk_helper             3215  0 
cryptd                 10040  1 ablk_helper
lrw                     4216  0 
gf128mul                7993  1 lrw
glue_helper             7474  0 
aes_x86_64              7837  2 
aes_generic            27609  1 aes_x86_64
cbc                     3083  1 
dm_crypt               15654  1 
microcode             112205  0 
joydev                 10480  0 
virtio_balloon          4798  0 
virtio_net             22002  0 
i2c_piix4              11232  0 
i2c_core               29132  1 i2c_piix4
ext4                  379655  4 
jbd2                   93252  1 ext4
mbcache                 8193  1 ext4
virtio_blk              7132  5 
virtio_pci              7416  0 
virtio_ring             8891  4 virtio_balloon,virtio_net,virtio_blk,virtio_pci
virtio                  5639  4 virtio_balloon,virtio_net,virtio_blk,virtio_pci
pata_acpi               3701  0 
ata_generic             3837  0 
ata_piix               24409  0 
dm_mirror              14864  0 
dm_region_hash         12085  1 dm_mirror
dm_log                  9930  2 dm_mirror,dm_region_hash
dm_mod                102467  17 dm_crypt,dm_mirror,dm_log

# modinfo usb_storage
filename:       /lib/modules/2.6.32-642.el6.x86_64/kernel/drivers/usb/storage/usb-storage.ko
license:        GPL
description:    USB Mass Storage driver for Linux
author:         Matthew Dharm <mdharm-usb@one-eyed-alien.net>
srcversion:     27463E0913E4C1933B5817F
alias:          usb:v03EBp2002d0100dc*dsc*dp*ic*isc*ip*
...
depends:        
vermagic:       2.6.32-642.el6.x86_64 SMP mod_unload modversions 
parm:           option_zero_cd:ZeroCD mode (1=Force Modem (default), 2=Allow CD-Rom (uint)
parm:           swi_tru_install:TRU-Install mode (1=Full Logic (def), 2=Force CD-Rom, 3=Force Modem) (uint)
parm:           delay_use:seconds to delay before using a new device (uint)
parm:           quirks:supplemental list of device IDs and their quirks (string)

# modinfo -p usb_storage
quirks:supplemental list of device IDs and their quirks
delay_use:seconds to delay before using a new device
swi_tru_install:TRU-Install mode (1=Full Logic (def), 2=Force CD-Rom, 3=Force Modem)
option_zero_cd:ZeroCD mode (1=Force Modem (default), 2=Allow CD-Rom

# modprobe usb_storage
# lsmod | grep usb
usb_storage            49329  0 
```

###### 临时修改一个值

```
方法一：
# ls /sys/module/usb_storage/parameters/
delay_use  option_zero_cd  quirks  swi_tru_install
# cat /sys/module/usb_storage/parameters/delay_use
1
# echo 5 >  /sys/module/usb_storage/parameters/delay_use
# cat /sys/module/usb_storage/parameters/delay_use
5
```

```
方法二：
# rmmod usb_storage					-> 卸载模块
# cat /sys/module/usb_storage/parameters/delay_use 
cat: /sys/module/usb_storage/parameters/delay_use: No such file or directory
# modprobe usb_storage delay_use=6			-> 加载模块的同时手动加载参数
# cat /sys/module/usb_storage/parameters/delay_use 
6
```

######  永久修改一个值

```
方法一： 这种方法适用于系统启动之后加载模块
# vim /etc/modprobe.d/usb_storage.conf
options usb_storage delay_use=4
# rmmod usb_storage
# cat /sys/module/sub_storage/parameters/delay_use
cat: /sys/module/usb_storage/parameters/delay_use: No such file or directory
# modprobe user_storage					-> 加载模块，执行此命令时，会读取/etc/modprobe.d/目录下的所有conf结尾的文件，看看有没有相关模块的选项
#  cat /sys/module/sub_storage/parameters/delay_use
4
# depmod -a						-> 下次开机还能加载，把当前的已经加载的所有模块的信息写到一个配置文件里面，下次开机如果还能读取这个配置文件，会加载文件里的所有模块
```

```
方法二： 这种方法在系统启动时加载模块
# grep -n  modules /etc/rc.sysinit			->  rc.sysinit系统初始化脚本
# vim /etc/rc.sysinit
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
```

###  Installing and Enabling tuned   

RHEL6新引进的系统优化程序，可以提供9种性能模式

```
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
```

### Configuring Tuned

### Limiting Resource Usage

### File Systems: Fragmentation and RAID layouts
