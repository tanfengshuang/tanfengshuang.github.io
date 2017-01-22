---
layout: post
title:  "Manage User Accounts(413-8)"
categories: Linux
tags: 413 chage getent
---

### Managing Password Aging

###### /etc/shadow Fields

1. 登录名(username): 是与/etc/passwd文件中的登录名相一致的用户账号
2. 口令(password): 字段存放的是加密后的用户口令字，长度为13个字符。如果为空，则对应用户没有口令，登录时不需要口令；如果含有不属于集合{./0-9A-Za-z}中的字符，则对应的用户不能登录。
3. 最后一次修改时间(date of last password change): 表示的是从某个时刻起，到用户最后一次修改口令时的天数。时间起点对不同的系统可能不一样。例如在SCOLinux中，这个时间起点是1970年1月1日。 (0表示登录即要求修改密码)
4. 最小时间间隔(minimum password age): 指的是两次修改口令之间所需的最小天数。(0 表示当时就可以修改密码，7表示至少7天后才能修改密码)
5. 最大时间间隔(maxmum password age): 指的是口令保持有效的最大天数。
6. 警告时间(password warning period): 字段表示的是从系统开始警告用户到用户密码正式失效之间的天数。(0 = no warning given)
7. 不活动时间(password inactive period): 表示的是用户没有登录活动但账号仍能保持有效的最大天数。
8. 失效时间(expiredate): 字段给出的是一个绝对的天数(numbers of days since 1970.01.01)，如果使用了这个字段，那么就给出相应账号的生存期。期满后，该账号就不再是一个合法的账号，也就不能再用来登录了。

###### chage

chage命令是用来修改帐号和密码的有效期限     
[chage](http://man.linuxde.net/chage)

*    -m：密码可更改的最小天数。为零时代表任何时候都可以更改密码
*    -M：密码保持有效的最大天数
*    -w：用户密码到期前，提前收到警告信息的天数
*    -E：帐号到期的日期。过了这天，此帐号将不可用
*    -d：上一次更改的日期
*    -i：停滞时期。如果一个密码已过期这些天，那么此帐号将不可用
*    -l：例出当前的设置。由非特权用户来确定他们的密码或帐号何时过期。

可以编辑/etc/login.defs来设定几个参数，以后设置口令默认就按照参数设定为准： 

```
# vim /etc/login.defs
PASS_MAX_DAYS 99999 
PASS_MIN_DAYS 0 
PASS_MIN_LEN 5 
PASS_WARN_AGE 7 
```

当然在/etc/default/useradd可以找到如下2个参数进行设置：
 
```
# vim /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

通过修改配置文件，能对之后新建用户起作用，而目前系统已经存在的用户，则直接用chage来配置。

```
# chage -h
Usage: chage [options] LOGIN

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximim number of days before password change to MAX_DAYS
  -R, --root CHROOT_DIR         directory to chroot into
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
  
# chage -l test
Last password change					: Jan 20, 2017
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

# cat /etc/shadow | grep test
test:$6$oIW3o2Mr$XbWZKaM7nA.cQqudfDJScupXOia5h1u517t6Htx/Q/MgXm82Pc/OcytatTeI4ULNWOMJzvpCigWiL4xKP9PX4.:17186:0:99999:7:::
# chage -d 0 test
# cat /etc/shadow | grep test
test:$6$oIW3o2Mr$XbWZKaM7nA.cQqudfDJScupXOia5h1u517t6Htx/Q/MgXm82Pc/OcytatTeI4ULNWOMJzvpCigWiL4xKP9PX4.:0:0:99999:7:::
# chage -m 90 -W 5 -I 14 test
# cat /etc/shadow | grep test
test:$6$oIW3o2Mr$XbWZKaM7nA.cQqudfDJScupXOia5h1u517t6Htx/Q/MgXm82Pc/OcytatTeI4ULNWOMJzvpCigWiL4xKP9PX4.:0:90:99999:5:14::
# chage -E 2017-02-08 test
# cat /etc/shadow | grep test
test:$6$oIW3o2Mr$XbWZKaM7nA.cQqudfDJScupXOia5h1u517t6Htx/Q/MgXm82Pc/OcytatTeI4ULNWOMJzvpCigWiL4xKP9PX4.:0:90:99999:5:14:17205:
```

###### Performance Checklist: Tuning Default Password Expiration Setting

```
通过修改配置文件，能对之后新建用户起作用，而目前系统已经存在的用户，则直接用chage来配置。
# vim /etc/login.defs
PASS_MAX_DAYS   99999
PASS_MIN_DAYS   0
PASS_MIN_LEN    5
PASS_WARN_AGE   7

# useradd u1
# grep u1 /etc/shadow
u1:!!:17188:0:99999:7:::

# vim /etc/login.defs
PASS_MAX_DAYS   30
PASS_MIN_DAYS   5
PASS_MIN_LEN    3
PASS_WARN_AGE   10

# useradd u2
# grep u2 /etc/shadow
u2:!!:17188:5:30:10:::

```

### Auditing User Accounts

Username and User ID numbers should be unique on each Linux system.

getent: 用来察看系统的数据库中的相关记录，即使这写数据库不是在本地，比如ldap或者nis中的数据库，也可以使用getent察看(display both local and network-authenticated passwd entries)

The getent command displays entries from databases supported by the Name Service Switch libraries, which are configured in /etc/nsswitch.conf.  If one or more key arguments are provided, then only the entries that match the supplied keys will be displayed.  Otherwise, if no key is provided, all entries will  be  dis-played (unless the database does not support enumeration).


```
# cat /etc/nsswitch.conf | egrep -v '#|^$'
passwd:     files
shadow:     files
group:      files
hosts:      files dns
bootparams: nisplus [NOTFOUND=return] files
ethers:     files
netmasks:   files
networks:   files
protocols:  files
rpc:        files
services:   files
netgroup:   nisplus
publickey:  nisplus
automount:  files nisplus
aliases:    files nisplus

# getent hosts
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
127.0.0.1       localhost localhost.localdomain localhost6 localhost6.localdomain6

# getent networks
default               0.0.0.0
loopback              127.0.0.0
link-local            169.254.0.0

# getent rpc
# getent shadow
# getent services

# getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
...

# getent passwd | cut -d : -f 3 | sort -n
0
1
2
...
```

###### Performance Chceklist: Deleting Duplicate Users

```
# id u2
uid=504(u2) gid=505(u2) groups=505(u2)
# useradd -o -u 504 -g 504 u4
# man useradd
        -o, --non-unique
           Allow the creation of a user account with a duplicate (non-unique) UID.
           This option is only valid in combination with the -u option.
# id u4
uid=504(u2) gid=504(u1) groups=505(u2)

# getent passwd | cut -d : -f 3 | sort -n | uniq -d
504

# getent passwd | grep --color 504
u1:x:503:504::/home/u1:/bin/bash
u2:x:504:505::/home/u2:/bin/bash
u4:x:504:504::/home/u4:/bin/bash

# userdel -r u2
# userdel -r u4
# getent passwd | grep --color 504
u1:x:503:504::/home/u1:/bin/bash

# man userdel
        -r, --remove
           Files in the user´s home directory will be removed along with the home directory itself and the user´s mail spool. Files located in other file systems will have to be searched for and deleted manually.
           The mail spool is defined by the MAIL_DIR variable in the login.defs file.

# man uniq
        -d, --repeated
              only print duplicate lines
```

### Unit Test: Managing Password Expiration

```
# chage -E 2018-12-12 test
```


