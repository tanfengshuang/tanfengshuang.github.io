---
layout: post
title:  "Creating File System(413-3)"
categories: Linux
tags: 413
---

### Allocate Filesystem for Secure Containment

| Mount Point  |                                Purpose                              |
|--------------|---------------------------------------------------------------------|
|/tmp          |World-writeable temporary storage area                               |
|/var          |System services and applications use this for frequently chaging data|
|/var/log      |Directory where system and service log files are kept                |
|/var/log/audit|Directory where system audit log files are kept                      |
|/home         |Storage area for local user accounts                                 |

### Implement Filesystem Encryption

Linux Unified Key Setup-on-disk-format technology - LUKS

###### Encryption at Installation

kickstart: --encrypted and --passphrase=

```
part /home --fstype=ext4 --size=10000 --onpart=vda2 --encrypted --passphrase=PASSPHRASE(plaintext)
```

###### Encryption Post-installation

```
# cryptsetup luksFormat /dev/vdb1
# cryptsetup luksOpen /dev/vdb1 name
# mkfs.ext4 /dev/mapper/name
# mkdir /secretstuff
# mount /dev/mapper/name /secretstuff
# umount /secretstuff
# cryptsetup luksClose name
```

###### Persistently Mount Encrypted Partitions

```
# dd if=/dev/urandom of=/path/to/password/file bs=4096 count=1
# chmod 600 /path/to/password/file
# cryptsetup luksAddkey /dev/vda2 /path/to/password/file
# vim /etc/crypttab
name    /dev/vda2   /path/to/password/file
# vim /etc/fstab
/dev/mapper/name    /secret ext4    defaults    1   2
```

###### Performance Checklist: Encrypt a New Filesystem

### Unit Test: Securing the Contents of /home

```
手动使用加密磁盘
# lvs
  LV      VG               Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_home vg_cloudqe16vm02 -wi-ao---- 10.00g
  lv_root vg_cloudqe16vm02 -wi-ao---- 50.00g
  lv_swap vg_cloudqe16vm02 -wi-ao----  3.94g 

# vgs
  VG               #PV #LV #SN Attr   VSize   VFree 
  vg_cloudqe16vm02   2   3   0 wz--n- 159.50g 95.57g

# lvcreate -n mylv1 -L 100m vg_cloudqe16vm02
  Logical volume "mylv1" created.

# pvs
  PV         VG               Fmt  Attr PSize  PFree 
  /dev/vda2  vg_cloudqe16vm02 lvm2 a--u 79.51g 25.47g
  /dev/vdb1  vg_cloudqe16vm02 lvm2 a--u 80.00g 70.00g

# vgs
  VG               #PV #LV #SN Attr   VSize   VFree 
  vg_cloudqe16vm02   2   4   0 wz--n- 159.50g 95.47g

# lvs
  LV      VG               Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_home vg_cloudqe16vm02 -wi-ao----  10.00g
  lv_root vg_cloudqe16vm02 -wi-ao----  50.00g
  lv_swap vg_cloudqe16vm02 -wi-ao----   3.94g
  mylv1   vg_cloudqe16vm02 -wi-a----- 100.00m 

# ls /dev/vg_cloudqe16vm02/mylv1 
/dev/vg_cloudqe16vm02/mylv1

# cryptsetup luksFormat /dev/vg_cloudqe16vm02/mylv1

WARNING!
========
This will overwrite data on /dev/vg_cloudqe16vm02/mylv1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter LUKS passphrase: 
Verify passphrase: 

# cryptsetup luksOpen /dev/vg_cloudqe16vm02/mylv1 myencryptdisk
Enter passphrase for /dev/vg_cloudqe16vm02/mylv1:

# ls /dev/mapper/myencryptdisk 
/dev/mapper/myencryptdisk

# mkfs.ext4 /dev/mapper/myencryptdisk
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
25168 inodes, 100352 blocks
5017 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67371008
13 block groups
8192 blocks per group, 8192 fragments per group
1936 inodes per group
Superblock backups stored on blocks: 
    8193, 24577, 40961, 57345, 73729

Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 35 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.

# mkdir /mnt/myencryptdisk
# mount /dev/mapper/myencryptdisk /mnt/myencryptdisk
# cd /mnt/myencryptdisk
# mkdir abc
# ls
abc  lost+found

# umount /mnt/myencryptdisk
# cryptsetup luksClose /dev/mapper/myencryptdisk

# ls /mnt/myencryptdisk
# mount /dev/mapper/
control                   vg_cloudqe16vm02-lv_home  vg_cloudqe16vm02-lv_root  vg_cloudqe16vm02-lv_swap  vg_cloudqe16vm02-mylv1
# mount /dev/mapper/vg_cloudqe16vm02-mylv1 /mnt/myencryptdisk
mount: unknown filesystem type 'crypto_LUKS'

```

```
开机永久挂载加密磁盘
1. 生成密码文件
# dd if=/dev/urandom of=/etc/password1 bs=4096 count=1
1+0 records in
1+0 records out
4096 bytes (4.1 kB) copied, 0.00178804 s, 2.3 MB/s
# ll /etc/password1
-rw-r--r--. 1 root root 4096 Jan 20 01:54 /etc/password1
# chmod 600 /etc/password1
# ll /etc/password1
-rw-------. 1 root root 4096 Jan 20 01:54 /etc/password1
# file /etc/password1
/etc/password1: data

2. 修改/etc/crypttab使其开机时使用密码文件，自动解密
# cryptsetup luksAddKey /dev/vg_cloudqe16vm02/mylv1 /etc/password1
Enter any passphrase: 
# vim /etc/crypttab
disk1   /dev/vg_cloudqe16vm02/mylv1     /etc/password1
# vim /etc/fstab
/dev/mapper/disk1       /mnt/myencryptdisk/     ext4    defaults        1 2

# ls /dev/mapper/disk1
ls: cannot access /dev/mapper/disk1: No such file or directory

# df /mnt/myencryptdisk/
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/vg_cloudqe16vm02-lv_root
                      51475068 2191376  46662252   5% /
# mount -a
mount: special device /dev/mapper/disk1 does not exist

# reboot
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
 
# ls /mnt/myencryptdisk
abc  lost+found
```


