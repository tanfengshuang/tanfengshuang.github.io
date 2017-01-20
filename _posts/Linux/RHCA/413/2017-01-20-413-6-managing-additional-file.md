---
layout: post
title:  "Managing Additional File(413-6)"
categories: Linux
tags: 413
---

### Access Controls

### Setting Default File Permissions

```
# umask
0022
# touch rootfile
# mkdir rootdir
# ls -ld root*
drwxr-xr-x. 2 root root 4096 Jan 20 05:24 rootdir
-rw-r--r--. 1 root root    0 Jan 20 05:24 rootfile

dir     -> 777 - 022
rwxrwxrwx
----w--w-
rwxr-xr-x

file    -> 666 - 022
rw-rw-rw-
----w--w-
rw-r--r--
```

```
# su - student
$ umask
0002
$ touch studentfile
$ mkdir studentdir
$ ls -ld student*
drwxrwxr-x. 2 student student 4096 Jan 20 05:25 studentdir
-rw-rw-r--. 1 student student    0 Jan 20 05:25 studentfile

dir     -> 777 - 002
rwxrwxrwx
-------w-
rwxrwxr-x

file    -> 666 - 002
rw-rw-rw-
-------w-
rw-rw-r--
```

###### Performance Checklist: umask Experimentation

```
$ vim ~/.bashrc
umask 0033
$ exit
# su - student
$ umask
0033
$ rm -rf student*
$ touch studentfile
$ mkdir studentdir
$ ll -d student*
drwxr--r--. 2 student student 4096 Jan 20 05:37 studentdir
-rw-r--r--. 1 student student    0 Jan 20 05:33 studentfile

dir     -> 777 - 033
rwxrwxrwx
----wx-wx
rwxr--r--   

file    -> 666 - 033
rw-rw-rw-
----wx-wx
rw-r--r--
```

### Managing Access Control Lists

###### Managing ACLs

ACL主要目的是提供传统的权限之外的具体权限的设置。ACL可以针对单一用户，单一文件或目录来进行r，w，x的权限设置。对于需要特殊权限的使用状况非常有帮助。

ACL主要可以针对以下几个项目来进行设置

*    用户：可以针对用户来设置权限
*    用户组：可以针对用户组来设置其权限
*    默认属性：可以在该目录下在新建目录时设置新数据的默认权限

###### Performance Checklist: Granting privileges with ACLs

```
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw)          <----------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

# mount -o remount,acl /mnt/myencryptdisk/          ---> 重启后失效
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw,acl)      <----------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

# tune2fs -l /dev/mapper/disk1  | grep "mount options"
Default mount options:    (none)

# vim /etc/fstab        --> 不用这种方法设置acl挂载参数
# df /mnt/myencryptdisk/
Filesystem        1K-blocks  Used Available Use% Mounted on
/dev/mapper/disk1     93071  3823     84231   5% /mnt/myencryptdisk

# # mount -o remount,rw /mnt/myencryptdisk/         --> 取消acl的临时挂载
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw)      <------------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

# tune2fs -l /dev/mapper/disk1 | grep "mount options"
Default mount options:    (none)
# tune2fs -o acl /dev/mapper/disk1          -> 使用tune2fs设置acl挂载选项，重启后仍然生效
tune2fs 1.41.12 (17-May-2010)
# tune2fs -l /dev/mapper/disk1 | grep "mount options"
Default mount options:    acl
# tune2fs -o ^acl /dev/mapper/disk1         -> 使用tune2fs取消acl挂载选项
tune2fs 1.41.12 (17-May-2010)
# tune2fs -l /dev/mapper/disk1 | grep "mount options"
Default mount options:    (none)
```


```
# tune2fs -o acl /dev/mapper/disk1
tune2fs 1.41.12 (17-May-2010)
# tune2fs -l /dev/mapper/disk1 | grep "mount options"
Default mount options:    acl

# cd /mnt/myencryptdisk
# touch file1
# echo file1content > file1
# mkdir dir1
# ll
total 4
drwxr-xr-x. 2 root root 1024 Jan 20 06:10 dir1
-rw-r--r--. 1 root root   13 Jan 20 06:09 file1

# getfacl dir1
# file: dir1
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# getfacl file1 
# file: file1
# owner: root
# group: root
user::rw-
group::r--
other::r--

# setfacl -m u:student:rwx file1    -> 针对用户来设置其权限
# getfacl file1 
# file: file1
# owner: root
# group: root
user::rw-
user:student:rwx
group::r--
mask::rwx
other::r--

# setfacl -m g:group1:rwx file1     -> 针对用户组来设置其权限
# getfacl file1 
# file: file1
# owner: root
# group: root
user::rw-
user:student:rwx
group::r--
group:group1:rwx
mask::rwx
other::r--

# setfacl -x g:group1 file1     ->  删除单个acl权限
# setfacl -x u:student file1

# getfacl file1 
# file: file1
# owner: root
# group: root
user::rw-
group::r--
mask::r--
other::r--

# ll file1
-rw-r--r--+ 1 root root 13 Jan 20 06:09 file1       ->  去掉添加的acl权限后，+仍然存在

# setfacl -b file1          ->  去掉所有acl权限
# ll file1
-rw-r--r--. 1 root root 13 Jan 20 06:09 file1
# getfacl file1 
# file: file1
# owner: root
# group: root
user::rw-
group::r--
other::r--

```

```
普通用户修改ssh登录时的提示信息
# tune2fs -l /dev/mapper/vg_cloudqe16vm02-lv_root | grep "mount options"
Default mount options:    user_xattr acl
# getfacl /etc/motd
getfacl: Removing leading '/' from absolute path names
# file: etc/motd
# owner: root
# group: root
user::rw-
group::r--
other::r--

# setfacl -m u:student:rwx /etc/motd
# getfacl /etc/motd
getfacl: Removing leading '/' from absolute path names
# file: etc/motd
# owner: root
# group: root
user::rw-
user:student:rwx
group::r--
mask::rwx
other::r--
# ll /etc/motd
-rw-rwxr--+ 1 root root 1328 Jan 19 22:35 /etc/motd

# su - student
$ vim /etc/motd     -> 修改登录时显示内容，保存
Welcome from student!
$ exit
# ssh student@...
Welcome from student!
```

```
目录下在新建目录时设置新数据的默认权限
# mkdir /var/tmp/collab
# getfacl /var/tmp/collab
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/collab
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# ll /var/tmp/collab
total 0
# setfacl -m d:u:student:rw /var/tmp/collab
# getfacl /var/tmp/collab
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/collab
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:student:rw-
default:group::r-x
default:mask::rwx
default:other::r-x

# echo rootfile > /var/tmp/collab/rootfile
# cat /var/tmp/collab/rootfile
rootfile
# ll /var/tmp/collab/rootfile
-rw-rw-r--+ 1 root root 9 Jan 20 06:40 /var/tmp/collab/rootfile
# getfacl /var/tmp/collab/rootfile
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/collab/rootfile
# owner: root
# group: root
user::rw-
user:student:rw-
group::r-x          #effective:r--
mask::rw-
other::r--

# setfacl -x d:u:student /var/tmp/collab/

# setfacl -m d:g:group1:rw /var/tmp/collab/
# setfacl -m d:o:rw /var/tmp/collab/

```

### Unit Test: Tightening User Permissions

