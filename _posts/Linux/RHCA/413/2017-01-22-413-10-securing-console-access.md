---
layout: post
title:  "Securing Console Access(413-10)"
categories: Linux
tags: 413 
---

### Securing Grub Bootloader

###### Adding a Global Password

```
# grub-crypt            -> 123456
Password:
Retype password:
$6$0OJhXAx3KFRv0dq4$HOdOV8o6StQ2HYt4LGfVBStC5a.NmsGFrHYqKzbUfZgIHfzeJ49FfbF7HkOBHgzL1NYf/vC3V1EFlzMFQpUAA.

# vim /etc/grub.conf                    -> password 加在 title 行前面, 以后想进入单用户模式，需要输入密码
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
password --encrypted $6$0OJhXAx3KFRv0dq4$HOdOV8o6StQ2HYt4LGfVBStC5a.NmsGFrHYqKzbUfZgIHfzeJ49FfbF7HkOBHgzL1NYf/vC3V1EFlzMFQpUAA.
title Red Hat Enterprise Linux 6 (2.6.32-642.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-642.el6.x86_64 ro root=/dev/mapper/vg_cloudqe16vm02-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=vg_cloudqe16vm02/lv_root SYSFONT=latarcyrheb-sun16 rd_LVM_LV=vg_cloudqe16vm02/lv_swap crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.32-642.el6.x86_64.img

# reboot

```

###### Adding a Grub Bootloader Password

```
# grub-crypt            -> 123456
Password: 
Retype password: 
$6$0OJhXAx3KFRv0dq4$HOdOV8o6StQ2HYt4LGfVBStC5a.NmsGFrHYqKzbUfZgIHfzeJ49FfbF7HkOBHgzL1NYf/vC3V1EFlzMFQpUAA.

# vim /etc/grub.conf                    -> password 加在 title 内部，每次启动系统都需要输入这个密码
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title Red Hat Enterprise Linux 6 (2.6.32-642.el6.x86_64)
        password --encrypted $6$0OJhXAx3KFRv0dq4$HOdOV8o6StQ2HYt4LGfVBStC5a.NmsGFrHYqKzbUfZgIHfzeJ49FfbF7HkOBHgzL1NYf/vC3V1EFlzMFQpUAA.
        root (hd0,0)
        kernel /vmlinuz-2.6.32-642.el6.x86_64 ro root=/dev/mapper/vg_cloudqe16vm02-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=vg_cloudqe16vm02/lv_root SYSFONT=latarcyrheb-sun16 rd_LVM_LV=vg_cloudqe16vm02/lv_swap crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.32-642.el6.x86_64.img

# reboot
```

###### Open Lab: Setting a Grub Bootloader Password

### Modifying Text Console Settings

###### Disable Control-Alt-Delete Key combination

```
1.
# cp /etc/init/control-alt-delete.conf /etc/init/control-alt-delete.override

2. 
# vim /etc/init/control-alt-delete.override
start on control-alt-delete
#exec /sbin/shutdown -r now "Control-Alt-Delete pressed"
exec /bin/true

3. Switch to display tty3, send key Ctrl+Alt+Delete
```

###### Displaying Acceptable Use Notifications(Text Console)

/etc/issue 当我们在终端接口登录的时候，会有几行提示字符串。在tty1-tty6没有登录的情况下显示登录前提示信息

*    \d  本地端时间的日期
*    \l  显示第几个终端接口
*    \m  显示硬件的等级
*    \n  显示主机的网络名称
*    \o  显示域名
*    \r  操作系统的版本
*    \t  显示本地端的时间
*    \s  操作系统的名称
*    \v  操作系统的版本 

/etc/motd 如果想让用户登录后获取一些消息，比如想让大家都知道的消息，就可以加入这个文件，当登录后，告诉登录者，系统将会在某个时间进行维护

```
# cat /etc/issue
Red Hat Enterprise Linux Server release 6.8 (Santiago)
Kernel \r on an \m

# vim /etc/issue
Red Hat Enterprise Linux Server release 6.8 (Santiago)
Kernel \r on an \m

RH413
GPGPGPGPGPG
GPGPGPGPGPG
```

###### Performance Checklist: Securing the Text Console

1. Modify the init subsystem to disable the ctr-alt-delet key combination's ability to reboot the system

```
# cp /etc/init/control-alt-delete.conf /etc/init/control-alt-delete.override
# vim /etc/init/control-alt-delete.override
start on control-alt-delete
#exec /sbin/shutdown -r now "Control-Alt-Delete pressed"
exec /bin/true
```

2. Download the standardized login banner content from: http://...

```
# wget http://... -O /etc/issue
```

3. Download the standardized acceptable use message from: http://...

```
# wget http://... -O /etc/motd
```

### Modifying Graphical Console Settings

###### Disable Poweroff and Rebbot on the Login Window

###### Disable User Display

###### Setting an Acceptable Use Message on the Graphical Login

###### Workshop: Applying Changes to the Gnome Display Manager(GDM)
