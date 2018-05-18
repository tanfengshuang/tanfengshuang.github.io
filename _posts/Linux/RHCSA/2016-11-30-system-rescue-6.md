---
layout: post
title:  "RHEL6单用户修改密码和系统恢复救援"
categories: Linux
tags: RHCSA rescue
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看


### 密码修复

开机读秒按任意键
grub配置按e
选择kernel按3
在最后添加空格1，回车后按b启动，进入单用户模式

```
getenforce
setenforce 0
passwd root
reboot
```

### grub修复

```
vim /boot/grub/grub.conf
rm -f /boot/grub/grub.conf
reboot
grub> root (hd0,0)
# 如果系统不止一块硬盘，需执行grub> setup (hd0)
grub> kernel /vmlinuz-2.6.32-71.el6.x86_64 ro root=/dev/sda2
# lv系统改为root=/dev/mapper/vgqin1-lvroot 或者 root=LABEL=/
grub> initrd /initramfs-2.6.32-71.el6.x86_64.img 
grub> boot
```

### 系统修复和救援

###### 备份重要资料

```
# mkdir /backup
# dd if=/dev/sda of=/backup/mbr.bak bs=512 count=1
# cp /etc/fstab /backup/fstab.bak
# cp /etc/inittab /backup/inittab.bak
# cp /etc/rc.d/rc.sysinit /backup/rc.sysinit.bak
# cp /etc/rc.d/rc.local /backup/rc.local.bak
```

###### 破坏性操作

```
# rm -f /boot/*
# rm -f /etc/fstab
# rm -f /etc/inittab
# rm -f /etc/rc.d/rc.sysinit
# rm -f /etc/rc.d/rc.local
# dd if=/dev/zero of=/dev/sda bs=446 count=1
# reboot
```

###### 选择修复模式

光盘启动选择Rescue installed system
若需网络引导，选择URL模式，本地光盘选择Local cdrom，且无需网络支持
continue
shell -> start shell


###### 修复fstab

```
# fdisk -l
# mkdir /qin
# mount /dev/sda2 /qin
# #lv下需要执行 lvm vgscan 和 lvm vgchange -ay 激活vg才能挂载
# cp /qin/backup/fstab.bak /qin/etc/fstab
# reboot
```
再次进入修复模式，如果看到 chroot /mnt/sysimage，说明/etc/fstab恢复成功


###### 恢复内核

内核文件vmlinuz initramfs等可以通过安装包kernel的方式来生成

```
# mkdir qin
# mount /dev/cdrom /qin
# rpm -ivh /qin/Packages/kernel-2.6.32-71.el6.x86_64.rpm --root=/mnt/sysimage/ --force
```

###### 恢复引导程序

可以通过grub-install恢复/boot/grub下面的内容，但是grub.conf必须自己手动输入

```
# chroot /mnt/sysimage/
# grub-install /dev/sda
# ls /boot/grub
# vim /boot/grub/grub.conf
default=0
timeout=3
title linux for resue
    root (hd0,0)
    kernel /vmlinuz-2.6.32-71.el6.x86_64 ro root=/dev/sda2
    # lv系统改为root=/dev/mapper/vgqin1-lvroot 或者 root=LABEL=/a
    # r! ls /boot/vmlinuz-2.6.32-71.el6.x86_64 
    initrd /initramfs-2.6.32-71.el6.x86_64.img
    # r! ls /boot/initramfs-2.6.32-71.el6.x86_64.img
```

###### 恢复init

```
# rpm -qf /etc/inittab
# rpm -qf /etc/rc.d/rc.sysinit
# rpm -qf /etc/rc.d/rc.local
# mount /dev/cdrom /mnt/cdrom
# rpm -ivh /mnt/cdrom/Packages/initscripts-9.03.17-1.el6.x86_64.rpm --force
```

再次exit退出到图像界面，选择reboot
重启后系统自动执行selinux relabel，几分钟后自动重启，至此，系统恢复完成








