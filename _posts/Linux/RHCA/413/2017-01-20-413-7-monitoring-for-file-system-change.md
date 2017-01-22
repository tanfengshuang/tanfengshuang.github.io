---
layout: post
title:  "Monitoring for File System Change(413-7)"
categories: Linux
tags: 413 aide prelink
---

### Using Intrusion Detection Software to Monitor Changes

AIDE(Adevanced Intrusion Detection Environment,高级入侵检测环境)是个入侵检测工具，主要用途是检查文档的完整性。       
AIDE能够构造一个指定文档的数据库，他使用aide.conf作为其配置文档。AIDE数据库能够保存文档的各种属性，使用下列算法：sha1、md5、rmd160、tiger，以密文形式建立每个文档的校验码或散列号。系统管理员应该建立新系统的AIDE数据库。这第一个AIDE数据库是系统的一个快照和以后系统升级的准绳。这个数据库不应该保存那些经常变动的文档信息，例如：日志文档、邮件、/proc文档系统、用户起始目录连同临时目录。

aide是一个Tipwire的替代和扩展软件，它有一些Tripwire所不具备的特征。aide当前具备的特征包括：多种完整性检验算法、把数据库输出到标准输出设备/文件的能力、通过配置文件进行配置以及数据库压缩支持。将来aide会提高更多的特征。 

###### Install AIDE

```
# yum install -y aide
```

###### Configure AIDE

AIDE默认的组有以下这些：
*    p      权限
*    inode  索引节点
*    n      连接数量
*    u      用户
*    g      用户组
*    s      大小
*    m      最后一次修改时间
*    a      最后一次访问时间
*    c      创建时间
*    S      检查增加的大小
*    md5    md5校验
*    sha1   sha1校验
*    rmd160 校验
*    tiger  tiger校验
*    R      p+i+n+u+g+s+m+c+md5
*    L      p+i+n+u+g
*    E      空组

> 增长的日志文件 p+u+g+i+n+S


如果在编译时，加入了mhash库的支持，还可以使用如下的组：

*    crc32  crc32校验
*    haval  haval校验
*    gost   gost校验


*    /dir1   group      -> 忽略/dev目录 
*    =/dir2  group      -> 只把/dir2目录加入到数据库，不包括其子目录
*    !/dir3  group      -> 忽略/dir3目录

通常，进行配置时，要尽量忽略经常8变动的目录和文件，例如：临时目录、邮件spool目录、日志目录、porc文件系统、用户起始目录、WEB正文目录以及其它经常变动的目录和文件。此外，要尽量包含所有的系统二进制文件、库文件、包含文件、系统源文件。其它一些你不经常注意的文件和目录，例如：/dev、usr/man/*、/usr/包含在内。

注意:如果你只想包含单个文件，就应该在表达式的结尾加$。例如：=/tmp$，表示只把tmp目录加入，而不包括其子目录。 


```
# vim /etc/aide.conf
@@define DBDIR /var/lib/aide
database=file:@@{DBDIR}/aide.db.gz          -> 数据库生成路径

report_url=file:@@{LOGDIR}/aide.log

NORMAL = R+rmd160+sha256                    -> param = value(param is not a built-in AIDE setting)
DIR = p+i+n+u+g+acl+selinux+xattrs
PERMS = p+i+u+g+acl+selinux

/boot   NORMAL
/bin    NORMAL
/sbin   NORMAL

# man 5 aide.conf
```

###### Initialize the AIDE Database

```
# aide --init       -> 初始化一个数据库 - /var/lib/aide/aide.db.new.gz
# mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

###### Check for File System Changes

```
# aide --check
```

###### Quiz: Installing, Configuring and Using AIDE Software

```
# vim /etc/aide.conf
,$s/^/#/g           -> 注释掉当前行到文件结尾的所有行
88,$s/^/#/g         -> 注释掉88行到文件结尾的所有行
/bin        PERMS
/dir1       PERMS
/etc/shadow PERMS
/etc/passwd NORMAL

# mkdir /dir1

# aide --init
AIDE, version 0.14
### AIDE database at /var/lib/aide/aide.db.new.gz initialized.

# ls /var/lib/aide/aide.db.new.gz
/var/lib/aide/aide.db.new.gz
# mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

# aide --check
AIDE, version 0.14
### All files match AIDE database. Looks okay!

# echo file111111111111 > /dir1/file1
# echo file222222222222 > /dir1/file2

# aide --check
AIDE found differences between database and filesystem!!
Start timestamp: 2017-01-22 02:41:43
Summary:
  Total number of files:	130
  Added files:			2
  Removed files:		0
  Changed files:		0
---------------------------------------------------
Added files:
---------------------------------------------------
added: /dir1/file2
added: /dir1/file1

# aide --init
# mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

# touch /dir1/file3
# useradd test
useradd: user 'test' already exists
# useradd test_aide
# rm /dir1/file2 -rf

# aide --check
AIDE found differences between database and filesystem!!
Start timestamp: 2017-01-22 02:46:12
Summary:
  Total number of files:	131
  Added files:			1
  Removed files:		1
  Changed files:		3
---------------------------------------------------
Added files:
---------------------------------------------------
added: /dir1/file3

---------------------------------------------------
Removed files:
---------------------------------------------------
removed: /dir1/file2

---------------------------------------------------
Changed files:
---------------------------------------------------
changed: /etc/passwd
changed: /etc/passwd-
changed: /etc/shadow

--------------------------------------------------
Detailed information about changes:
---------------------------------------------------
File: /etc/passwd
  Size     : 1462                             , 1509
  Mtime    : 2017-01-20 08:09:57              , 2017-01-22 02:44:41
  Ctime    : 2017-01-20 08:09:57              , 2017-01-22 02:44:41
  Inode    : 1705108                          , 1705111
  MD5      : BlAOpdgic9TMd0K7L0+Gpw==         , OfpMTBXHeBknoxQtWryHCw==
  RMD160   : xHLRO2UeLM0F22P42EtEl8lIEUs=     , XYRyiepfUqjh5iF2EcPOgTWYts0=
  SHA256   : SrSWryjxiU8H5Cinj98GZYXLHPwdHONP , XuEBMrmkq6c8sWRrSSy0XHUXYHjXSsdB
File: /etc/passwd-
  Size     : 1423                             , 1462
  Mtime    : 2017-01-20 05:22:53              , 2017-01-20 08:09:57
  Ctime    : 2017-01-20 08:09:57              , 2017-01-22 02:44:41
  MD5      : lmuNy92b8nUpwuRG9ffMDw==         , BlAOpdgic9TMd0K7L0+Gpw==
  RMD160   : cALmtY8ggXXw1w/y6Ji0YDINPmA=     , xHLRO2UeLM0F22P42EtEl8lIEUs=
  SHA256   : 1bnF9gRY6rDGTp5+tL7P8aZVTIM42cYg , SrSWryjxiU8H5Cinj98GZYXLHPwdHONP
File: /etc/shadow
  Inode    : 1705079                          , 1705108
```

```
如果要监控二进制文件，需要将prelink禁用
# grep -r prelink /etc | grep cron
# vim /etc/cron.daily/prelink
# man prelink
       prelink  is  a program that modifies ELF shared libraries and ELF dynamically linked binaries in such a way that the time needed for the dynamic linker to per-
       form relocations at startup significantly decreases.  Due to fewer relocations, the run-time memory consumption decreases as well  (especially  the  number  of
       unshareable  pages).   The prelinking information is only used at startup time if none of the dependent libraries have changed since prelinking; otherwise pro-
       grams are relocated normally.

       prelink first collects ELF binaries to be prelinked and all the ELF shared libraries they depend on. Then it assigns a unique virtual  address  space  slot  to
       each  library  and relinks the shared library to that base address.  When the dynamic linker attempts to load such a library, unless that virtual address space
       slot is already occupied, it maps the library into the given slot.  After this is done, prelink, with the help of dynamic linker, resolves all  relocations  in
       the  binary  or  library  against  its  dependent  libraries  and stores the relocations into the ELF object.  It also stores a list of all dependent libraries
       together with their checksums into the binary or library.  For binaries, it also computes a list of conflicts (relocations  that  resolve  differently  in  the
       binary’s symbol search scope than in the smaller search scope in which the dependent library was resolved) and stores it into a special ELF section.

       At  runtime,  the  dynamic  linker first checks whether all dependent libraries were successfully mapped into their designated address space slots, and whether
       they have not changed since the prelinking was done.  If all checks are successful, the dynamic linker just replays the list of  conflicts  (which  is  usually
       significantly shorter than total number of relocations) instead of relocating each library.

```

[prelink](https://linux.die.net/man/8/prelink)

### Unit Test: Detecting Filesystem Changes with AIDE


