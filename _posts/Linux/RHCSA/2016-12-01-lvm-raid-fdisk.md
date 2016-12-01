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

###### RAID0

###### RAID1

###### RAID5

###### mdadmin管理工具

```
# RAID0
```































