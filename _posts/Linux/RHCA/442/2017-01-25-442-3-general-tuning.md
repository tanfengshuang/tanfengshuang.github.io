---
layout: post
title:  "General Tuning(442-3)"
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

###  Installing, Enabling, and Configuring Tuned

RHEL7

|Tuned     | Profile Purpose|
|balanced  |This profile is ideal for systems that require a compromise between power saving and performance.|
|desktop   |This profile is derived from the balanced profile. It provides faster response of interactive applications.|
|latency-performance| This profile is for server systems that require low latency at the expense of power consumption.|
|network-latency    | This profile is derived from the latency-performance profile. It enables additional network tuning parameters to provide low network latency.|
|network-throughput | This profile is derived from the throughput-performance profile. Additional network tuning parameters are applied for maximum network throughput.|
|powersave          |This profile tunes the system for maximum power saving.|
|sap                |This profile is designed to tune the system for maximum performance of SAM software.|
|throughput-performance|This profile tunes the system for maximum throughput.|
|virtual-guest|This profile tunes the system for maximum performance if it runs on a virtual machine.|
|virtual-host|This profile tunes the system for maximum performance if it acts as a host for virtual machines.|


```
# yum install tuned
# chkconfig tuned --list
tuned          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
# chkconfig ktune --list
ktune          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
# service tuned status
tuned is stopped
# service ktune status
ktune settings are not applied.
# /etc/init.d/ktune status
ktune settings are not applied.
# service ktune start       # /etc/init.d/ktune start
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
# service tuned start       # /etc/init.d/tuned start
Starting tuned: [  OK  ]
# chkconfig ktune on
# chkconfig tuned on
# chkconfig tuned --list
tuned          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# chkconfig ktune --list
ktune          	0:off	1:off	2:on	3:on	4:on	5:on	6:off

# tuned-adm list            -> RHEL6.8 result
Available profiles:
- server-powersave
- laptop-ac-powersave
- laptop-battery-powersave
- enterprise-storage
- virtual-host
- throughput-performance
- spindown-disk
- latency-performance
- desktop-powersave
- default
- virtual-guest
Current active profile: default

# tuned-adm profile throughput-performance
Stopping tuned: [  OK  ]
Switching to profile 'throughput-performance'
Applying deadline elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Calling '/etc/ktune.d/tunedadm.sh start': [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
Starting tuned: [  OK  ]
# tuned-adm active
Current active profile: throughput-performance
Service tuned: enabled, running
Service ktune: enabled, running

# cd /etc/tune-profiles/
# ls
active-profile  desktop-powersave   functions            laptop-battery-powersave  server-powersave  throughput-performance  virtual-host
default         enterprise-storage  laptop-ac-powersave  latency-performance       spindown-disk     virtual-guest
# cd throughput-performance/
# ls
ktune.sh  ktune.sysconfig  sysctl.ktune  sysctl.s390x.ktune  tuned.conf

# cat ktune.sysconfig
...
# scheduler settings.
ELEVATOR="deadline"
# These are the devices, that should be tuned with the ELEVATOR
ELEVATOR_TUNE_DEVS="/sys/block/{sd,cciss,dm-,vd,dasd,xvd}*/queue/scheduler"

# cd ..
# cp -r throughput-performance my-performance
# tuned-adm list
Available profiles:
- server-powersave
- laptop-ac-powersave
- laptop-battery-powersave
- enterprise-storage
- virtual-host
- throughput-performance
- spindown-disk
- latency-performance
- desktop-powersave
- default
- virtual-guest
- my-performance
Current active profile: throughput-performance

# tuned-adm profile my-performance
Calling '/etc/ktune.d/tunedadm.sh stop': [  OK  ]
Reverting to cfq elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Stopping tuned: [  OK  ]
Switching to profile 'my-performance'
Applying deadline elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Calling '/etc/ktune.d/tunedadm.sh start': [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
Starting tuned: [  OK  ]
# tuned-adm active
Current active profile: my-performance
Service tuned: enabled, running
Service ktune: enabled, running
```

```
RHEL7:
# systemctl enable tuned
# systemctl start tuned

# tuned-adm active
Current active profile: virtual-guest

# tuned-adm list            -> RHEL7.3 result
Available profiles:
- balanced                    - General non-specialized tuned profile
- desktop                     - Optmize for the desktop use-case
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput.  Generally only necessary on older CPUs or 40G+ networks.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads.  This is the default profile for RHEL7.
- virtual-guest               - Optimize for running inside a virtual guest.
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest

# tuned-adm profile throughput-performance
# tuned-adm active
Current active profile: throughput-performance
# tuned-adm recommend
virtual-guest

Any tuning activity of the tuned service can be turned off instantly with tuned-adm off.
# tuned-adm off
# tuned-adm active
No current active profile.
# tuned-adm profile throughput-performance
# tuned-adm active
Current active profile: throughput-performance
```

### File Systems: Fragmentation and RAID layouts

###### Fragmentation

```
# df -H
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_cloudqe16vm01-lv_root
                       53G  2.3G   48G   5% /
tmpfs                 985M     0  985M   0% /dev/shm
/dev/vda1             500M   41M  433M   9% /boot
/dev/mapper/vg_cloudqe16vm01-lv_home
                       27G   47M   26G   1% /home
# e2freefrag /dev/vda       -> report free space fragmentation information
vda   vda1  vda2  
# e2freefrag /dev/vda1
Device: /dev/vda1
Blocksize: 1024 bytes
Total blocks: 512000
Free blocks: 453100 (88.5%)

Min. free extent: 1 KB 
Max. free extent: 122373 KB
Avg. free extent: 23581 KB

HISTOGRAM OF FREE EXTENT SIZES:
Extent Size Range :  Free extents   Free Blocks  Percent
    1K...    2K-  :             2             2    0.00%
  512K... 1024K-  :             1           965    0.21%
    1M...    2M-  :             4          7498    1.65%
    2M...    4M-  :             3         11052    2.44%
    4M...    8M-  :             2          8446    1.86%
    8M...   16M-  :             2         32250    7.12%
   32M...   64M-  :             3        155130   34.24%
   64M...  128M-  :             2        232705   51.36%

# filefrag /boot/grub/grub.conf             -> report on file fragmentation   
/boot/grub/grub.conf: 1 extent found
# ls -lh /boot/grub/grub.conf
-rw-------. 1 root root 871 Feb  4 04:09 /boot/grub/grub.conf
# dd if=/dev/zero of=/file bs=1M count=5
5+0 records in
5+0 records out
5242880 bytes (5.2 MB) copied, 0.00447326 s, 1.2 GB/s
# filefrag /file
/file: 1 extent found
```

###### Stripe & Stride sizes on RAID

RAID条带化技术就是一种自动的将 I/O 的负载均衡到多个物理磁盘上的技术，条带化技术就是将一块连续的数据分成很多小部分并把他们分别存储到不同磁盘上去。这就能使多个进程同时访问数据的多个不同部分而不会造成磁盘冲突，而且在需要对这种数据进行顺序访问的时候可以获得最大程度上的 I/O 并行能力，从而获得非常好的性能。由于条带化在 I/O 性能问题上的优越表现，以致于在应用系统所在的计算环境中的多个层次或平台都涉及到了条带化的技术，如操作系统和存储系统这两个层次中都可能使用条带化技术。    
条带化后，条带卷所能提供的速度比单个盘所能提供的速度要快很多，由于现在存储技术成熟，大多数系统都采用条带化来实现系统的I/O负载分担，如果OS有LVM软件或者硬件条带设备，决定因素是条带深度(stripe depth)和条带宽度(stripe width)。

```
# man mkfs.ext4
    -E extended-options
              Set  extended  options  for  the filesystem.  Extended options are comma separated, and may take an argument using the equals (’=’) sign.  The -E option
              used to be -R in earlier versions of mke2fs.  The -R option is still accepted for backwards compatibility.   The following  extended  options  are  sup-
              ported:

                   stride=stride-size
                          Configure  the  filesystem  for a RAID array with stride-size filesystem blocks. This is the number of blocks read or written to disk before
                          moving to the next disk, which is sometimes referred to as the chunk size.  This  mostly  affects  placement  of  filesystem  metadata  like
                          bitmaps at mke2fs time to avoid placing them on a single disk, which can hurt performance.  It may also be used by the block allocator.

                   stripe-width=stripe-width
                          Configure  the filesystem for a RAID array with stripe-width filesystem blocks per stripe. This is typically stride-size * N, where N is the
                          number of data-bearing disks in the RAID (e.g. for RAID 5 there is one parity disk, so N will be the number of disks in the array minus  1).
                          This allows the block allocator to prevent read-modify-write of the parity in a RAID stripe if possible when the data is written.

RAID 5： 2块数据盘，1块校验盘
 [A] [B] [C]
 1M --- chunk size(假设为64k)  -> 例如一个文件大小为1M，每次写完一个chunk size（64K）后才会移到下一个磁盘
                          
RAID 6： 3块数据盘，2块校验盘
 [A] [B] [C] [D] [E]
 1M --- chunk size
 
# mkfs -t ext4 -E stride=chunksize/blocksize,stripe-width=N(数据盘个数，例如RAID5为3-1，RAID6为5-2)*stride-size
# mkfs.ext4 -E stride=16(64/4),stripe-width=48(3*16) /dev/sda2

```
