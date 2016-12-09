---
layout: post
title:  "磁盘管理"
categories: Linux
tags: RHCSA vgcreate lvcreate vgs lvs lvresize fdisk mkswap swapon mdadmin mount
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看


### superblock 超级区块

*    superblock是记录整个filesystem相关信息的地方，没有superblock就没有filesystem，它记录的信息主要有  
*    block与inode的总量
*    未使用与已使用的inode/block数量
*    block与inode大小(block为1，2，3k，inode为128bytes)
*    filesystem的挂载时间，最近一次写入数据的时间，最近一次检验磁盘(fsck)的时间等文件系统的相关信息
*    一个valid bit数值，若此文件系统已被挂载，则valid bit为0，若未被挂载，则valid bit为1

### 硬盘中的分区

*    fdisk       不可以创建大于2t的分区，最多创建4个主分区，扩展分区从5开始。 MBR(Master Boot Record)
*    parted      可以创建大于2t的分区，但是这个命令是实时生效的，要小心使用，不像fdisk和gdisk，需要最后写入w才能生效
*    gdisk       可以创建大于2t的分区，可以创建最多128个分区。 GPT(GUID Partition Table) (globally unique identifier，GUID)
*    partprobe   重读分区表, 使用fdisk工具只是将分区信息写到磁盘，如果需要mkfs磁盘分区则需要重启系统，而使用partprobe则可以使kernel重新读取分区信息，从而避免重启系统。
*    partx       类似partprobe
*    swapon      -s / -a / <设备名>
*    swapoff     


### 创建文件系统

*    mkswap     使用fdisk来创建交换分区（例如 /dev/sda3 是创建的交换分区），使用 mkswap 命令来设置交换分区
*    mkfs       mkfs.ext3 mkfs.ext4 mkfs.xfs


### 挂载 卸载文件系统

###### mount/umount命令

*    mount -o loop <iso镜像文件> <挂载点目录>
*    mount -o ro
*    mount -o rw
*    mount -o remount
*    mount -a 挂载/etc/fstab中未挂载的设备
*    umount <挂载点目录>

```
# mount /dev/sda3 /mnt/sda3
# mount -o loop test.iso /mnt/iso
# mount -o remount,ro /mnt/sda3
```

###### fstab配置文件

*   包含了需要开机后自动挂载的文件系统记录
*   UUID，可以通过blkid <设备名>得到


```
# 添加一块大于2T的硬盘，分区也大于2T
# fdisk -l
# parted -l 
# gdisk -l /dev/sdb
# gdisk /dev/sdb
```

```
# swap分区
# fdisk /dev/sda
# partprobe             -> 重读分区表
# mkswap /dev/sda3      -> 使用fdisk来创建交换分区（例如 /dev/sda3 是创建的交换分区），使用 mkswap 命令来设置交换分区
# swapon -s             -> 显示交换区的使用状况，相当于cat /proc/swaps
# swapon /dev/sda3      -> 启用交换分区
# swapon -s
# swapoff /dev/sda3     -> 关闭指定的交换空间/dev/sda3，即禁用交换分区
# swapon -s 
# blkid
# vim /etc/fstab
UUID="xxxxxxxxxxxxxx"   swap    swap    defaults    0   0
# swapon -a             -> 将/etc/fstab文件中所有设置为swap的设备，启动为交换分区
# swapon -s
```

### 逻辑卷管理组成部分

*    物理卷(PV - Physical Volume)    物理卷在逻辑卷管理中处于最底层，它可以是实际物理硬盘上的分区，也可以时整个物理硬盘
*    卷组(VG - Volume Group)         卷组建立在物理卷之上，一个卷组中至少要包括一个物理卷，在卷组建立后可以动态添加物理卷到卷组中。一个逻辑卷管理系统工程中可以只有一个卷组，也可以有多个
*    逻辑卷(LV - Logical Volume)     逻辑卷建立在卷组之上，卷中的未分配空间可以用于建立新的逻辑卷，逻辑卷建立后可以动态扩展和缩小空间，多个逻辑卷可以属于同一个卷组，也可以属于不同的卷组

```
# lv创建      -> 创建顺序: pv -> vg -> lv
# fdisk /dev/sda        -> 创建sda5和sda6
# partprobe
# pvs
# pvcreate /dev/sda5 /dev/sda6
# pvs
# vgcreate qinvg /dev/sda5 /dev/sda6
# vgs
# vgdisplay
# vgremove qinvg
# vgcreate qinvg /dev/sda5 /dev/sda6
# vgs
# lvcreate -L 2.5G qinvg -n qinlv       -> 创建一个大小为2.5G的lv， -L后面跟实际的大小
# lvs
# lvdisplay
# lvcreate -l 14 qinvg -n qinlv2        -> 创建一个大小为14个PE大小的lv，-l后面跟PE的个数
# mkfs.xfs /dev/qinvg/qinlv
# blkid

# lv 删除     -> 删除顺序: lv -> vg -> pv
# lvremove /dev/qinvg/qinlv
# lvs
# lvremove /dev/qinvg/qinlv
# lvs
# vgs
# vgremove qinvg
# vgs
# pvs
# pvremove /dev/sda5 /dev/sda6
# pvs

# PE默认大小为4M，可以通过-s参数在创建vg时指定PE的大小
# vgcreate -s 16M vgname /dev/sdc1

# lv在线扩容
# mount /dev/qinvg/qinlv /mnt/qinlv/
# df -Th | grep qinlv
# lvresize -L 2G /dev/qinvg/qinlv
# df -th | grep qinlv
# xfs_growfs    /mnt/qinlv/
# df -Th | grep qinlv
```


```
# 针对ext4文件系统的quota
# setenforce 0      -> 暂时关掉selinux
# fdisk /dev/sda
# umount /mnt/qinlv/
# mkfs.ext4 /dev/qinvg/qinlv
# mount | grep qinlv
# vim /etc/fstab
UUID=xxxxxx /mnt/qinlv  ext4    defaults,usrquota,grpquota  0   0
# mount -a
# mount | grep qinlv
# chmod o+rwx /mnt/qinlv
# quotacheck -cvug /mnt/qinlv/  -> -c create， -u 创建aquota.user， -g 创建aquota.group
# ll /mnt/qinlv/                -> 出现aquota.user aquota.group文件
# quotaon -ugv /mnt/qinlv/
# setquota -u qin 10240 20480 5 6 /mnt/qinlv/   -> 为用户qin设置quota，可拷入文件大小最大不超过20480， 文件个数最多不超过6个
# su - qin
# touch /mnt/qinlv/file{1..00}
# ll /mnt/qinlv/
# rm -f /mnt/qinlv/*
# dd if=/dev/zero of=/mnt/qinlv/1 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/2 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/3 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/4 bs=1M count=9
# ll /mnt/qinlv/

# 针对xfs文件系统的quota
# setenforce 0      -> 暂时关掉selinux
# umount /mnt/qinlv/
# mount | grep qinlv
# mkfs.xfs /dev/qinvg/qinlv
# vim /etc/fstab
UUID=xxxxxx /mnt/qinlv  xfs    defaults,usrquota,grpquota  0   0
# mount -a
# mount | grep qinlv
# chmod o+rwx /mnt/qinlv
# setquota -u qin 10240 20480 5 6 /mnt/qinlv/   -> 为用户qin设置quota，可拷入文件大小最大不超过20480， 文件个数最多不超过6个
# su - qin
# touch /mnt/qinlv/file{1..00}
# ll /mnt/qinlv/
# rm -f /mnt/qinlv/*
# dd if=/dev/zero of=/mnt/qinlv/1 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/2 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/3 bs=1M count=9
# dd if=/dev/zero of=/mnt/qinlv/4 bs=1M count=9
# ll /mnt/qinlv/
```


### RAID

*    磁盘阵列(Redundant Arrays of Independent Disks，RAID)，独立冗余磁盘阵列，诞生于1987年，由美国加州大学伯克利分校提出
*    简单的说，就是将N块硬盘通过RAID Controller(分Hardware Software)结合成虚拟单块大容量的硬盘使用，其特色是N台硬盘同时读取速度加快及提供容错性(Fault Tolerant)，所以RAID是当成平时主要访问数据的Storage而不是Backup Solution


###### RAID0

RAID0又称为Stripe或Striping，中文译为集带工作方式，有时也可以理解为“拼凑”
它是将要存取的数据以条带形式尽量平均分配到多个硬盘上，读写时多个硬盘同时进行读写，从而提高数据的读写速度。RAID0另一目的是获得更大的"单个"磁盘容量

###### RAID1

*    又称为Mirror或Mirroring，中文译为镜像方式
*    这种工作方式的出现完全是为了数据安全考虑的，它是把用户写入硬盘的数据百分之百的自动复制到另外一块硬盘上或硬盘的不同地方（镜像）。当读取数据时，系统先从RAID1的源盘读取数据，如果数据读取成功，则系统不去管备份盘上的数据，如果读取源盘数据失败，则系统自动转而读取备份盘上的数据，不会造成用户工作任务的中断
*    由于对存储的数据进行百分百的备份，在所有RAID级别中，RAID1提供最高的数据安全保障。同样，由于数据的百分之百备份，备份数据占了总存储空间的一半，因而，Mirror的磁盘空间利用率低，存储成本高。


###### RAID5

*    RAID5是一种存储性能，数据安全和存储成本兼顾的存储解决方案，也是目前应用最广泛的RIAD技术
*    各块独立硬盘进行条带化分割，相同的条带区进行奇偶校验（异或运算），校验数据平均分配在每块硬盘上
*    以N块硬盘构建的RAID5阵列可以有2/3硬盘的容量，存储空间利用率非常高
*    RAID5不对存储的数据进行备份，而是把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上。当RAID5的任何一块磁盘上的数据丢失，均可以通过校验数据推算出来


###### Software RAID

一般的中高档服务器多使用硬件RAID控制器来实现Hardware RAID，但是由于硬件RAID控制器的价格昂贵，导致系统成本大大增加。而随着处理器的性能快速发展，使得软件RAID的解决方法得到人们的重视
Software RAID即软件磁盘阵列，软件RAID使您可以将两个或多个块设备（通常是磁盘区）组合为单个RAID设备（/dev/mdX）
例如，假定有三个空分区hda3 hdb3和hdc3，是哟该软件RAID管理工具mdadm就能将这些分区组合起来


###### mdadmin管理工具

mdadm工具是一个管理软件RAID的独立程序，它能完成所有的软RAID管理功能

mdadm常用选项：

*    -A <阵列设备名>, --assemble： 加入一个以前定义的阵列
*    -C <阵列设备名>, --create： 创建一个新的阵列
*    -D <阵列设备名>, --detail： 显示md device的详细信息
*    -a yes： 自动创建md阵列文件
*    -l，--level=： 设定raid level
*    -s，--scan：扫描配置文件或/proc/mdstat/以搜寻丢失的信息
*    -n，--raid-devices=： 指定阵列中可用device数目，这个数目只能由--grow修改
*    -x，--spare-devices： 指定初始阵列的富余device数目
*    -r,--remove： 删除指定磁盘
*    -f,--fail： same as --set-faulty，标记坏磁盘
*    -x,--spare-devices=： 指定热备磁盘数目


```
# RAID0
添加2块硬盘，并分区
# fdisk -l | grep "Disk /dev/sd"
# mdadm -C /dev/md0 -a yes -l 0 -n 2 /dev/sdb1 /dev/sdc1
# mkfs.xfs /dev/md0
# mkdir /mnt/md0
# mount /dev/md0 /mnt/md0
# dd if=/dev/zero of=/mnt/md0/md0test bs=1M count=500
# df -Th
# mdadm -D /dev/md0

# RAID1
# mdadm -C /dev/md1 -a yes -l 1 -n 2 /dev/sdb2 /dev/sdc2
# mkdir /mnt/md1
# mount /dev/md1 /mnt/md1
# dd if=/dev/zero of=/mnt/md0/md1test bs=1M count=500
# df -Th
# mdadm -D /dev/md1

# RAID5
# mdadm -C /dev/md5 -a yes -l 5 -n 3 /dev/sdb3 /dev/sdc3 /dev/sdd3
# mkdir /mnt/md5
# mount /dev/md5 /mnt/md5
# dd if=/dev/zero of=/mnt/md0/md5test bs=1M count=500
# df -Th
# mdadm -D /dev/md5

# RAID5坏一块硬盘
# dd if=/dev/zero of=/mnt/md5/md5file bs=1M count=500
# df -Th
# umount /mnt/md5
# mdadm /dev/md5 -f /dev/sdd3	-> 将/dev/sdd3标记为坏盘
# mdadm -D /dev/md5
# mount /dev/md5 /mnt/md5
# ll /mnt/md5
# df -Th
# mdadm /dev/md5 -r /dev/sdd3	-> 热拔/dev/sdd3
# mdadm -D /dev/md5		-> 显示只剩两块硬盘
# mdadm /dev/md5 -a /dev/sdd3	-> 热插/dev/sdd3
# mdadm -D /dev/md5		-> 快速查看，可以看到同步数据的百分比变化
# ll /mnt/md5

# RAID5热备
# mdadm -C /dev/md5x -a yes -l 5 -n 3 -x 1 /dev/sdb4 /dev/sdc4 /dev/sdd4 /dev/sde4
# mkfs.xfs /dev/md5x
# mkdir /mnt/md5x
# mount /dev/md5x /mnt/md5x
# dd if=/dev/zero of=/mnt/md5x/md5test bs=1M count=500
# df -Th
# mdadm -D /dev/md5x
```

