---
layout: post
title:  "Managing Special Permissions(413-5)"
categories: Linux
tags: 413
---

### Special Permissions Concepts

###### Special Permissions on Directories


###### Special Permissions on Files


### Manipulating Special Permissions

###### Displaying Special Permissions

```
/usr/bin/passwd
/usr/bin/wall
/tmp
```

###### Changing Special Permissions

```
chmod
```

###### Performance Checklist: Setting Special Permissions

### Auditing Files with Special Permissions

###### Demonstration: Matching Special Permissions with find

```
# find /var -type d -perm -1000
# find /var -type d -perm  4000 
# find /var -type d -perm -4000
# find /var -type d -perm /4000
# find /var -type d -perm /6000 ?
# find /var -type d -perm -6000 ?
```


```
# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 30768 Nov  2  2015 /usr/bin/passwd

# ls -ld /tmp/
drwxrwxrwt. 3 root root 4096 Jan 20 04:16 /tmp/

# mount -o remount,exec /mnt/myencryptdisk/
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

# cd /mnt/myencryptdisk/ 
# ./newvim /etc/passwd
# ./newvim /etc/shadow
# ll /etc/shadow
----------. 1 root root 933 Jan 20 00:36 /etc/shadow
# ls -ld /etc/
drwxr-xr-x. 97 root root 12288 Jan 20 05:13 /etc/
# ll /mnt/myencryptdisk/newvim 
-rwxr-xr-x. 1 root root 2324712 Jan 20 04:35 /mnt/myencryptdisk/newvim

# su - student
student$ mkdir /tmp/studentdir
student$ ls -ld /tmp/studentdir
student$ chmod g+s /tmp/studentdir
```
