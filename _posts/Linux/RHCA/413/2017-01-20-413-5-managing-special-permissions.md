---
layout: post
title:  "Managing Special Permissions(413-5)"
categories: Linux
tags: 413
---

### Special Permissions Concepts

###### Special Permissions on Directories

*    sgid    - 针对目录
*    stick   - 针对目录

###### Special Permissions on Files

*    suid    - 针对普通文件

### Manipulating Special Permissions

###### Displaying Special Permissions

```
# ll /usr/bin/passwd     - 有suid权限位
-rwsr-xr-x. 1 root root 30768 Nov  2  2015 /usr/bin/passwd

# ll /usr/bin/wall       - 有sgid权限位
-r-xr-sr-x. 1 root tty 15224 Feb 26  2015 /usr/bin/wall

# ll -d /tmp                - 有stick权限位
drwxrwxrwt. 3 root root 4096 Jan 20 04:16 /tmp

```

###### Changing Special Permissions

```
chmod
```

###### Performance Checklist: Setting Special Permissions

```
# mount -o remount,exec /mnt/myencryptdisk/     -> 恢复exec挂载选项，上一章实验中给取消了
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

1. 设置uid
# cd /mnt/myencryptdisk/ 
# cp /usr/bin/vim ./newvim
# ./newvim /etc/passwd          -> root能编辑
# ./newvim /etc/shadow          -> root能编辑
# ll /etc/shadow
----------. 1 root root 933 Jan 20 00:36 /etc/shadow
# ls -ld /etc/
drwxr-xr-x. 97 root root 12288 Jan 20 05:13 /etc/
# ll /mnt/myencryptdisk/newvim 
-rwxr-xr-x. 1 root root 2324712 Jan 20 04:35 /mnt/myencryptdisk/newvim

# su - student
student$ /mnt/myencryptdisk/newvim /etc/shadow      -> 普通用户这时候不能编辑
"/etc/shadow" [Permission Denied]

# ll /mnt/myencryptdisk/newvim 
-rwxr-xr-x. 1 root root 2324712 Jan 20 04:35 /mnt/myencryptdisk/newvim
# chmod u+s /mnt/myencryptdisk/newvim               -> 使用root用户给newvim加上s权限
# ll /mnt/myencryptdisk/newvim 
-rwsr-xr-x. 1 root root 2324712 Jan 20 07:32 /mnt/myencryptdisk/newvim

student$ /mnt/myencryptdisk/newvim /etc/shadow      -> 普通用户这时候可以编辑了， 当用户执行某一个可执行文件时，bash会先判断这个文件对于这个用户有没有x权限（取出当前用户的uid，根据uid来判断他能不能执行这个文件，如果能执行则执行，当他执行这个执行文件newvim时，会打开另外一个文本文件，那个文本文件能不能打开/修改/读，取决于此uid对于那个文件能不能打开/修改/读。比方说，上面打开shadow时不能打开/修改/读。如果现在给newvim设置了uid权限位，此时，当用户执行newvim时，计算权限用的uid不是用户当前uid，而是设置为一个新的uid，这个新的uid就是newvim这个程序的所有者的uid，即root，uid为0）
（给一个可执行文件添加了suid权限，那么当任何一个用户执行这个可执行文件时，计算权限的uid会变成这个执行文件所有者的uid，如果所有者是root，权限会提升为超级管理员权限）


2. 设置gid
student$ mkdir /tmp/studentdir
student$ ls -ld /tmp/studentdir
drwxrwxr-x. 2 student student 4096 Jan 20 07:56 /tmp/studentdir

# mkdir /mnt/rootdir
# ls -ld /mnt/rootdir
drwxr-xr-x. 2 root root 4096 Jan 20 07:57 /mnt/rootdir

# ll -d /tmp
drwxrwxrwt. 4 root root 4096 Jan 20 07:56 /tmp
# chmod g+s /tmp 
# ls -ld /tmp
drwxrwsrwt. 4 root root 4096 Jan 20 07:56 /tmp

这时候在设置了sgid的目录下再创建一个目录时，这个文件夹的所属组将会是什么
student$ mkdir /tmp/studentdir2
student$ ls -ld /tmp/studentdir2
drwxrwsr-x. 2 student root 4096 Jan 20 08:02 /tmp/studentdir2       -> 所属组是 root, 因为给目录/tmp/添加了sgid后，再往目录(/tmp)下面创建东西时，它(/tmp)的所属组(root)会继承下去。这样子，方便，只要是组里面的成员往里面添加了新的文件夹或者文件，同组的用户对它（新文件夹和文件）都会有组权限

3. stick权限位
$ ll -d /tmp/studentdir*
drwxrwxr-x. 2 student student 4096 Jan 20 07:56 /tmp/studentdir
drwxrwsr-x. 2 student root    4096 Jan 20 08:02 /tmp/studentdir2

# useradd user2
# su - user2
user2$ mkdir /tmp/ccc
user2$ ls -ld /tmp/ccc
drwxrwsr-x. 2 user2 root 4096 Jan 20 08:10 /tmp/ccc         -> 所属组是root，是上面的sgid生效导致


# ll -d /tmp
drwxrwsrwt. 6 root root 4096 Jan 20 08:10 /tmp
# chmod o-t /tmp                    -> 先用超级用户将stick权限位取消
# ll -d /tmp
drwxrwsrwx. 6 root root 4096 Jan 20 08:10 /tmp
# mkdir /tmp/rootdir

user2$ ll -d /tmp/rootdir/
drwxr-sr-x. 2 root root 4096 Jan 20 08:14 /tmp/rootdir/
user2$ rm -rf /tmp/rootdir/         -> 可以将root创建的rootdir删除
user2$ ll -d /tmp/rootdir/
ls: cannot access /tmp/rootdir/: No such file or directory

user2$ ls -ld /tmp/studentdir
drwxrwxr-x. 2 student student 4096 Jan 20 07:56 /tmp/studentdir
user2$ rm -rf /tmp/studentdir       -> 可以将student创建的studentdir删除
user2$ ls -ld /tmp/studentdir
ls: cannot access /tmp/studentdir: No such file or directory

# chmod o+t /tmp                    -> 用超级用户将stick权限位恢复

user2$ ls -ld /tmp/studentdir2/
drwxrwsr-x. 2 student root 4096 Jan 20 08:02 /tmp/studentdir2/
user2$ rm -rf /tmp/studentdir2/     -> 不能删除其他用户创建的东西
rm: cannot remove `/tmp/studentdir2': Operation not permitted
```

```
user2$ ls -ld /mnt/myencryptdisk/newvim 
-rwsr-xr-x. 1 root root 2324712 Jan 20 07:32 /mnt/myencryptdisk/newvim          -> 有suid权限位
user2$ /mnt/myencryptdisk/newvim /etc/shadow        -> 可以编辑

# mount -o remount,nosuid /mnt/myencryptdisk
# mount
/dev/mapper/vg_cloudqe16vm02-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm02-lv_home on /home type ext4 (rw)
/dev/mapper/disk1 on /mnt/myencryptdisk type ext4 (rw,nosuid)                 <--------------
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

user2$ /mnt/myencryptdisk/newvim /etc/shadow        -> 刚才可以进行编辑，现在不可以了
"/etc/shadow" [Permission Denied] 

```

### Auditing Files with Special Permissions

```
# find /var -type d -perm -1000
# find /var -type d -perm  4000 
# find /var -type d -perm -4000
# find /var -type d -perm /4000
# find /var -type d -perm /6000 ?
# find /var -type d -perm -6000 ?
```

###### Demonstration: Matching Special Permissions with find

```
# find /usr/bin/ -type f -perm 4000     -> (精确匹配)有suid，没有任何普通权限， 这样子的文件一般不存在
# touch /usr/bin/a
# chmod 4000 /usr/bin/a
# ls -l /usr/bin/a
---S------. 1 root root 0 Jan 20 08:33 /usr/bin/a
# find /usr/bin/ -type f -perm 4000
/usr/bin/a

# find /usr/bin/ -type f -perm -4000 -> （模糊匹配 - 其中至少有一位匹配）有suid，普通权限不限制
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/crontab
/usr/bin/chage
/usr/bin/chfn
/usr/bin/a
/usr/bin/at
/usr/bin/chsh
/usr/bin/staprun
# find /usr/bin/ -type f -perm -4000 -ls
2500724   32 -rwsr-xr-x   1 root     root        30768 Nov  2  2015 /usr/bin/passwd
2502579   40 -rwsr-xr-x   1 root     root        40240 Feb  9  2016 /usr/bin/newgrp
2502577   76 -rwsr-xr-x   1 root     root        75640 Feb  9  2016 /usr/bin/gpasswd
2511514  124 ---s--x--x   1 root     root       123832 Mar  1  2016 /usr/bin/sudo
2502816   24 -rwsr-xr-x   1 root     root        22544 Mar  6  2015 /usr/bin/pkexec
2505415   52 -rwsr-xr-x   1 root     root        51784 Sep 22  2015 /usr/bin/crontab
2502576   72 -rwsr-xr-x   1 root     root        70480 Feb  9  2016 /usr/bin/chage
2503513   20 -rws--x--x   1 root     root        20184 Mar 15  2016 /usr/bin/chfn
2497242    0 ---S------   1 root     root            0 Jan 20 08:33 /usr/bin/a
2505421   56 -rwsr-xr-x   1 root     root        54496 Feb 16  2015 /usr/bin/at
2503515   20 -rws--x--x   1 root     root        20056 Mar 15  2016 /usr/bin/chsh
2502228  180 ---s--x---   1 root     stapusr    183072 Mar 24  2016 /usr/bin/staprun

# ls -l /usr/bin/a
---S------. 1 root root 0 Jan 20 08:33 /usr/bin/a
# chmod 6644 /usr/bin/a
# ls -l /usr/bin/a
-rwSr-Sr--. 1 root root 0 Jan 20 08:33 /usr/bin/a
# find /usr/bin/ -type f -perm -6000 -ls
2497242    0 -rwSr-Sr--   1 root     root            0 Jan 20 08:33 /usr/bin/a

# touch /usr/bin/b
# ls -l /usr/bin/b
-rw-r--r--. 1 root root 0 Jan 20 08:40 /usr/bin/b
# chmod g+s /usr/bin/b
# ls -l /usr/bin/b
-rw-r-Sr--. 1 root root 0 Jan 20 08:40 /usr/bin/b       -> 2644
# find /usr/bin/ -type f -perm -2000 -ls
2516440    0 -rw-r-Sr--   1 root     root            0 Jan 20 08:40 /usr/bin/b
2504000  140 -rwxr-sr-x   1 root     nobody     141384 Mar 15  2016 /usr/bin/ssh-agent
2492364   16 -r-xr-sr-x   1 root     tty         15224 Feb 26  2015 /usr/bin/wall
2503555   12 -rwxr-sr-x   1 root     tty         12016 Mar 15  2016 /usr/bin/write
2492399   40 -rwx--s--x   1 root     slocate     38464 Jan 26  2015 /usr/bin/locate
2497242    0 -rwSr-Sr--   1 root     root            0 Jan 20 08:33 /usr/bin/a
2500186   20 -rwxr-sr-x   1 root     mail        20376 Sep  4  2014 /usr/bin/lockfile


# find /usr/bin -type f -perm /6000 -ls         -> 特殊权限位相加不超过6
2516440    0 -rw-r-Sr--   1 root     root            0 Jan 20 08:40 /usr/bin/b
2500724   32 -rwsr-xr-x   1 root     root        30768 Nov  2  2015 /usr/bin/passwd
2502579   40 -rwsr-xr-x   1 root     root        40240 Feb  9  2016 /usr/bin/newgrp
2502577   76 -rwsr-xr-x   1 root     root        75640 Feb  9  2016 /usr/bin/gpasswd
2511514  124 ---s--x--x   1 root     root       123832 Mar  1  2016 /usr/bin/sudo
2502816   24 -rwsr-xr-x   1 root     root        22544 Mar  6  2015 /usr/bin/pkexec
2504000  140 -rwxr-sr-x   1 root     nobody     141384 Mar 15  2016 /usr/bin/ssh-agent
2505415   52 -rwsr-xr-x   1 root     root        51784 Sep 22  2015 /usr/bin/crontab
2492364   16 -r-xr-sr-x   1 root     tty         15224 Feb 26  2015 /usr/bin/wall
2502576   72 -rwsr-xr-x   1 root     root        70480 Feb  9  2016 /usr/bin/chage
2503513   20 -rws--x--x   1 root     root        20184 Mar 15  2016 /usr/bin/chfn
2503555   12 -rwxr-sr-x   1 root     tty         12016 Mar 15  2016 /usr/bin/write
2492399   40 -rwx--s--x   1 root     slocate     38464 Jan 26  2015 /usr/bin/locate
2497242    0 -rwSr-Sr--   1 root     root            0 Jan 20 08:33 /usr/bin/a
2505421   56 -rwsr-xr-x   1 root     root        54496 Feb 16  2015 /usr/bin/at
2500186   20 -rwxr-sr-x   1 root     mail        20376 Sep  4  2014 /usr/bin/lockfile
2503515   20 -rws--x--x   1 root     root        20056 Mar 15  2016 /usr/bin/chsh
2502228  180 ---s--x---   1 root     stapusr    183072 Mar 24  2016 /usr/bin/staprun

# find /usr/bin -type f -perm /7000 -ls         -> 特殊权限位相加不超过7, 即所有包含特殊权限位的文件都会搜索出来
```

### Unit Test: The Risks of SetUID Programs

```
# cp /usr/bin/vim /home/student/supervim
# ll supervim
-rwsr-xr-x. 1 root root 2324712 Jan 20 07:32 supervim
# su - student
```
