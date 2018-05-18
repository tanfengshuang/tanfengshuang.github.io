---
layout: post
title:  "RHEL7单用户修改密码和系统恢复救援"
categories: Linux
tags: RHCSA rescue
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### 破密码

开机菜单栏第一行按e，在linux16行尾，加入rc.break console=tty0
ctrl+x继续启动

```
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch ./autolabel
exit
exit
```


### 防止破密码

在/etc/grub.d/00_header文件结尾加入

```
cat << EOF
set superuser="qin"
password qin bing
EOF
```

然后，用命令grub2-mkconfig重新生成grub.cfg文件

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

重启后需要输入账号qin和密码bing方可进入编辑模式


### fstab错误的修复

```
# vim /etc/fstab
/dev/sda6   /mnt    xfs     defaults    0   0   -> 输入一个错误的dev设备，让系统在启动时加载设备失败
```

重启后系统无法启动，等待一段时间后输入root的密码可以进入单用户模式，修改fstab后可以正常启动
如果不能写入，需要重新以读写的模式挂载根

```
mount -o remount,rw /
```


### 应急模式

按e，在linux16行尾加入emergency(应急模式)
和单用户模式相似，只不过不加载/boot分区，也需要root密码


### 光盘修复

```
rm -rf /boot/*
dd if=/dev/zero of=/dev/sda bs=446 count=1
systemctl reboot
```

Troubleshooting (排错模式)
选择Rescue a Red hat Enterprise Linux
continue

```
chroot /mnt/sysimage
mount /dev/cdrom mnt/cdrom
cd /mnt/Packages
rpm -ivh /mnt/cdrom/Packages/kernel-3.1...  -> 安装boot分区
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg      -> grub配置文件后缀和RHEL6的不同
exit
exit    -> 重启2次后完成修复进入系统
```
