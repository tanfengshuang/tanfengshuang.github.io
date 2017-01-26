layout: post
title:  "修改hostname"
categories: Linux
tags: hostnamectl
---

在Centos和RHEL中，有三种定义的主机名:a、静态的（static），b、瞬态的（transient），以及 c、灵活的（pretty）。“静态”主机名也称为内核主机名，是系统在启动时从/etc/hostname自动初始化的主机名。“瞬态”主机名是在系统运行时临时分配的主机名，例如，通过DHCP或mDNS服务器分配。静态主机名和瞬态主机名都遵从作为互联网域名同样的字符限制规则。而另一方面，“灵活”主机名则允许使用自由形式（包括特殊/空白字符）的主机名，以展示给终端用户（如Tan's Computer）。

### RHEL7

在CentOS/RHEL 7中，有个叫hostnamectl的命令行工具，它允许你查看或修改与主机名相关的配置。

```
要查看主机名相关的设置：
# hostnamectl status 
   Static hostname: dhcp-129-221.nay.redhat.com
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: 8ba2086c4fa74979bfb11c719ac6c8c6
           Boot ID: 7dc94b9c76284ff5be0ea6e7ce9021b0
  Operating System: Fedora 24 (Workstation Edition)
       CPE OS Name: cpe:/o:fedoraproject:fedora:24
            Kernel: Linux 4.5.5-300.fc24.x86_64
      Architecture: x86-64

只查看静态、瞬态或灵活主机名，分别使用“--static”，“--transient”或“--pretty”选项, 在修改静态/瞬态主机名时，任何特殊字符或空白字符会被移除，而提供的参数中的任何大写字母会自动转化为小写。一旦修改了静态主机名，/etc/hostname 将被自动更新。然而，/etc/hosts 不会更新以保存所做的修改，所以你需要手动更新/etc/hosts。
# hostnamectl status --static
dhcp-129-221.nay.redhat.com
# hostnamectl status --transient
dhcp-129-221.nay.redhat.com
# hostnamectl status --pretty       -> 结果为空

要同时修改所有三个主机名：静态、瞬态和灵活主机名：
# hostnamectl set-hostname dhcp-129-74.nay.redhat.com
# hostnamectl status --transient
dhcp-129-74.nay.redhat.com
# hostnamectl status --static
dhcp-129-74.nay.redhat.com
# hostnamectl status --pretty       -> 结果为空

# cat /etc/hostname
dhcp-129-74.nay.redhat.com

# hostnamectl set-hostname --pretty "Tan's Computer"
# hostnamectl status --pretty
Tan's Computer
```

### RHEL6

> hostname                 #查看当前主机的主机名

> hostname NEWHOSTNAME     #临时修改当前主机名

> vim /etc/sysconfig/network    #通过配置文件永久修改主机名
                  
```
# hostname
dhcp-129-74.nay.redhat.com
# hostname dhcp-129-221.nay.redhat.com
# hostname
dhcp-129-221.nay.redhat.com

# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=NEWHOSTNAME         #修改该值作为主机名，如：NEWPC

# vim /etc/hosts                                //设置本地DNS解析文件, 必须有三个字段：IP、FQDN、HOSTNAME
127.0.0.1   localhost.localdomain localhost     //该行强烈建议保留
192.168.0.1  rhel.lpwr.net  rhel
```
