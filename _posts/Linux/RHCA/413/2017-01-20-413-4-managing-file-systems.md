---
layout: post
title:  "Managing File Systems(413-4)"
categories: Linux
tags: 413
---

### Secure Filesystems Using Security-Related Mount Options

| Filesystem               | Mount Options  |  Purpose                                              |
|--------------------------|----------------|-------------------------------------------------------|
|Non-Root Local Partitions |nodev           |Only /dev directory                                    |
|Removable Media Partitions|nodev,noexec    |Removable media should not contain special device files|
|/tmp and /dev/shm         |nodev,noexec    |World-writable directory                               |


###### Superblock Mount Options

tune2fs是调整和查看ext2/ext3文件系统的文件系统参数，Windows下面如果出现意外断电死机情况，下次开机一般都会出现系统自检。Linux系统下面也有文件系统自检，而且是可以通过tune2fs命令，自行定义自检周期及方式

*    -l 查看文件系统信息
*    -c max-mount-counts 设置强制自检的挂载次数，如果开启，每挂载一次mount conut就会加1，超过次数就会强制自检
*    -i interval-between-checks[d|m|w] 设置强制自检的时间间隔[d天m月w周]
*    -m reserved-blocks-percentage 保留块的百分比
*    -j 将ext2文件系统转换为ext3类型的文件系统
*    -L volume-label 类似e2label的功能，可以修改文件系统的标签
*    -r reserved-blocks-count 调整系统保留空间
*    -o [^]mount-option[,...] Set or clear the indicated default mount options in the filesystem. 设置或清除默认挂载的文件系统选项

```
# tune2fs -l /dev/vda1 | head -n 10
# tune2fs -o acl /dev/vda1
# tune2fs -o ^acl /dev/vda1
# tune2fs -l /dev/vda1 | grep 'mount options'
```

###### Performance Checklist: The nosuid and noexec Mount Options

```
# df
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/vg_cloudqe16vm02-lv_root
                      51475068 2227348  46626280   5% /
tmpfs                   961044       0    961044   0% /dev/shm
/dev/vda1               487652   39605    422447   9% /boot
/dev/mapper/vg_cloudqe16vm02-lv_home
                      10190136   36872   9628976   1% /home
/dev/mapper/disk1        93071    1552     86502   2% /mnt/myencryptdisk        <--------------

# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw)                          <--------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

# tune2fs -l /dev/mapper/disk1
tune2fs 1.41.12 (17-May-2010)
Filesystem volume name:   <none>
Last mounted on:          <not available>
Filesystem UUID:          fbd09d7d-92f3-435c-b7c6-1fee5682e646
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent flex_bg sparse_super huge_file uninit_bg dir_nlink extra_isize
Filesystem flags:         signed_directory_hash 
Default mount options:    (none)
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              25168
Block count:              100352
Reserved block count:     5017
Free blocks:              91519
Free inodes:              25156
First block:              1
Block size:               1024
Fragment size:            1024
Reserved GDT blocks:      256
Blocks per group:         8192
Fragments per group:      8192
Inodes per group:         1936
Inode blocks per group:   242
Flex block group size:    16
Filesystem created:       Fri Jan 20 01:46:16 2017
Last mount time:          Fri Jan 20 02:49:50 2017
Last write time:          Fri Jan 20 02:49:50 2017
Mount count:              2
Maximum mount count:      35
Last checked:             Fri Jan 20 01:46:16 2017
Check interval:           15552000 (6 months)
Next check after:         Wed Jul 19 02:46:16 2017
Lifetime writes:          7639 kB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:           128
Journal inode:            8
Default directory hash:   half_md4
Directory Hash Seed:      f2c7b9be-61f3-4f89-9a19-bd375d482d01
Journal backup:           inode blocks

# tune2fs -l /dev/mapper/disk1 | grep "mount options"
Default mount options:    (none)

# cp /usr/bin/vim /mnt/myencryptdisk/newvim
# ll /mnt/myencryptdisk/newvim
-rwxr-xr-x. 1 root root 2324712 Jan 20 04:35 /mnt/myencryptdisk/newvim
# cd /mnt/myencryptdisk
# ls -l
total 2286
drwxr-xr-x. 2 root root    1024 Jan 20 01:48 abc
drwx------. 2 root root   12288 Jan 20 01:46 lost+found
-rwxr-xr-x. 1 root root 2324712 Jan 20 04:35 newvim
# ./newvim /etc/issue
# mount -o remount,noexec /mnt/myencryptdisk/
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw,noexec)               <---------------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
# ./newvim /etc/issue
-bash: ./newvim: Permission denied
```

### Secure Individual Files with Filesystem Attributes

###### File Attributes

|File Attributes|                                    Description                                                             |
|---------------|------------------------------------------------------------------------------------------------------------|
|a              |Append only - prevents files from being overwritten                                                         |
|d              |Do not backup with dump command                                                                             |
|i              |Immutable - prevents file from being overwritten, deleted or renamed                                        |
|S              |Synchronous update - file updates will be immediately synced to disk                                        |
|j              |Journal - file data being written to ext3 or ext4 filesystems will be written with complete data journaling |
|e              |Indicates extents are being used to map file blocks on disk, is set by the system and cannto be manually set|

###### Extended File Attributes

```
# lsattr /etc/passwd
-------------e- /etc/passwd

# lsattr /etc/                
```

###### Performance Checklist: Manipulating Filesystem Attributes

```
# touch /etc/aa
# lsattr /etc/aa
-------------e- /etc/aa
# chattr +i /etc/aa
# lsattr /etc/aa
----i--------e- /etc/aa
# mv /etc/aa /etc/nn
mv: cannot move `/etc/aa' to `/etc/nn': Operation not permitted
# rm -rf /etc/aa
rm: cannot remove `/etc/aa': Operation not permitted
# echo "aaa" >> /etc/aa
-bash: /etc/aa: Permission denied

# chattr -i /etc/aa 
# chattr +a /etc/aa 
# lsattr /etc/aa
-----a-------e- /etc/aa
# vim /etc/aa
# echo "aaa" >> /etc/aa
# mv /etc/aa /etc/bb
mv: cannot move `/etc/aa' to `/etc/bb': Operation not permitted
# rm -rf /etc/aa
rm: cannot remove `/etc/aa': Operation not permitted
# cat /etc/aa
aaa

```
