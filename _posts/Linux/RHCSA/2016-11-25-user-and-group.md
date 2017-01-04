---
layout: post
title:  "用户和组"
categories: Linux
tags: RHCSA useradd usermod chage groupadd id finger
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### 添加用户

useradd

*   -c <备注>       加上备注文字
*   -d <目录        指定用户登入时的起始目录
*   -e <有效日期>   指定账户的有效期限
*   -g <群组>       指定用户所在的群组
*   -G <群组>       指定用户所在的附加群组
*   -m/-M           自动建立(-m)用户的登入目录或者不自动创建(-M)
*   -n              取消建立以用户名称为名的群组
*   -r              建立系统账号
*   -s <shell>      指定用户登入后所使用的shell
*   -u <uid>        指定用户ID

### 删除用户

userdel -r ...

```
# userdel linuxde       //删除用户linuxde，但不删除其家目录及文件
# userdel -r linuxde    //删除用户linuxde，其家目录及文件一并删除；
```

### 修改用户信息

usermod

*   -c <备注>       改变用户的描述信息
*   -d <目录        改变用户的主目录，如果加上-m则会将旧的家目录移动到新的目录中去（-m应该加在新目录之后）
*   -e <有效日期>   设置用户账户的过期时间(年-月-日)
*   -g <群组>       改变用户的主属组
*   -G <群组>       设置用户属于哪些组
*   -l              改变用户的登录用名
*   -L              锁住密码，使密码不可用。
*   -U              为用户密码解锁
*   -s <shell>      改变用户的默认shell
*   -u <uid>        改变用户的UID

### 修改用户密码

```
# echo 123.com | passwd --stdin root
```

### chage

快速清晰查看某个账户在/etc/shadow中描述的内容

```
# chage -l root
Last password change					: Jul 05, 2016
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

```

强制账户在第一次登录时修改密码

```
# chage -d 0 $new_user
```

### 组

*    groupadd
*    groupdel

组的配置信息文件
/etc/group
/etc/gshadow


### 检查用户身份

*   who     查询当前在线用户
*   w       查询当前在线用户的详细信息
*   groups  查询用户所属的组
*   id      显示用户id信息
*   finger  查询用户信息 登录时间 邮件

