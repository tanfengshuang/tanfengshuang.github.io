---
layout: post
title:  "Configuring ACLs and Quotas(236-6)"
categories: Linux
tags: RHCA 236
---

### 设置 POSIX ACL

###### 启用 ACL

红帽存储卷支持使用 POSIX 访问控制列表(ACL)，实现对文件和目录的精细访问控制。

在使用原生客户端时，可通过挂载选项 acl 启用 POSIX ACL。在使用 NFSv3 客户端或 Samba 客户端时，POSIX ACL 默认为启用。

###### 使用 POSIX ACL

使用 getfacl 和 setfacl 命令，可分别查询和设置 POSIX ACL。ACL 具有 [d:]u:UID:PERMS 或 [d:]g:GID:PERMS 格式，但也提供其他格式来设置 mask和其他权限。

如果 ACL 的前缀为 d:，这表示默认 ACL。默认 ACL 本身不授予权限，但代表将对目录中任何新创建文件和目录设置的 ACL。默认 ACL 仅可放在目录上，不能放在普通文件上。

1. 设置 POSIX ACL

设置 POSIX ACL 通过 setfacl 命令执行。此命令可以递归方式作用（借助 -R 选项），修改或添加 ACL（-m 选项）或删除 ACL（-x 选项）。例如，若要递归授予用户 lisa 对 /springfield-library 目录中所有文件和目录的读/写访问权限：

      [root@demo ~]# setfacl -R -m u:lisa:rwX /springfield-library

> 注意: 上例中的大写 X 表示仅授予对目录和已设置有执行位的文件的执行权限。

若要确保 lisa 将自动获取对 /springfield-library 目录中任何新创建的文件和目录的读/写权限，可使用以下命令：

      [root@demo ~]# setfacl -m d:u:lisa:rwX /springfield-library
      
2. 查询 ACL

在运行正常的 ls -l 命令时，权限集末尾的加号 (+) 表示文件上已设置了 POSIX ACL。getfacl 命令可用于查看这些 ACL。 


### 引导式练习：设置 POSIX ACL

红帽 Gluster 存储部署从 servera 提供名为 groupdata 的卷。已要求您利用原生客户端将这个卷永久挂载到 workstation 上的 /mnt/groupdata 中，并且要符合以下要求：

*    其中应当有一个新目录 /mnt/groupdata/admindocs，所有者为 admins 组。
*    在 /mnt/groupdata/admindocs 中创建的任何新文件和目录也都应当归 admins 组所有。
*    admins 组中的用户应当对 /mnt/groupdata/admindocs 目录及其内容具有完整控制。
*    managers 组中的用户应当对 /mnt/groupdata/admindocs 及其内容具有只读访问权限。
*    不属于 admins 或 managers 组的用户应当对 /mnt/groupdata/admindocs 目录无访问权限。 

出于测试目的，workstation 上具有三个用户帐户，密码均为 redhat：

*    dave 是 admins 的成员
*    boss 是 managers 的成员
*    simon 同时是 admins 和 managers 的成员

1. 使用 acl 选项，将 servera:/groupdata 卷永久挂载到 workstation 上的 /mnt/groupdata。

    a. 在 workstation 上安装 glusterfs-fuse 软件包。

    [root@workstation ~]# yum -y install glusterfs-fuse

    b. 在 workstation 上，创建挂载点 /mnt/groupdata。

    [root@workstation ~]# mkdir /mnt/groupdata

    c. 将以下条目添加到 workstation 上的 /etc/fstab。

    servera:/groupdata /mnt/groupdata glusterfs _netdev,acl 0 0

    d. 挂载卷。

    [root@workstation ~]# mount /mnt/groupdata

2. 在 workstation 上创建目录 /mnt/groupdata/admindocs，将组所有权设置到 admins，然后确保该目录中的任何新文件或目录也都属于 admins 组。赋予 admins 组完整访问权限，并且限制其他组的访问权限。

    a. 在 workstation 上创建目录 /mnt/groupdata/admindocs。

    [root@workstation ~]# mkdir /mnt/groupdata/admindocs

    b. 将新目录的所有权设置到 admins。

    [root@workstation ~]# chgrp admins /mnt/groupdata/admindocs

    c. 设置 /mnt/groupdata/admindocs 的权限，使它成为 admins 组的共享目录。

    [root@workstation ~]# chmod 2770 /mnt/groupdata/admindocs

3. 在 /mnt/groupdata/admindocs 上配置 ACL，允许 admins 组的成员对任何现有文件和目录以及任何新创建文件具有完整访问权限。

    a. 在 /mnt/groupdata/admindocs 上配置 ACL，授予 admins 组对任何现有文件和目录的完整访问权限。

    [root@workstation ~]# setfacl -R -m g:admins:rwX /mnt/groupdata/admindocs

    b. 在 /mnt/groupdata/admindocs 上配置 ACL，授予 admins 组对任何新文件和目录的完整访问权限。

    [root@workstation ~]# setfacl -R -m d:g:admins:rwX /mnt/groupdata/admindocs

4. 在 /mnt/groupdata/admindocs 上配置 ACL，允许 managers 组的成员对任何现有文件和目录以及任何新创建文件具有只读访问权限。

    a. 在 /mnt/groupdata/admindocs 上配置 ACL，授予 managers 组对任何现有文件和目录的只读访问权限。

    [root@workstation ~]# setfacl -R -m g:managers:rX /mnt/groupdata/admindocs

    b. 在 /mnt/groupdata/admindocs 上配置 ACL，授予 managers 组对任何新文件和目录的只读访问权限。

    [root@workstation ~]# setfacl -R -m d:g:managers:rX /mnt/groupdata/admindocs


### 设置配额

###### 启用配额

红帽 Gluster 存储支持在卷中的各个目录上设置配额。这可帮助管理员控制所用存储的位置。与您或许已熟悉的文件系统配额不同，这是在目录基础上设置，而不是用户/组基础。这类似于 XFS 文件系统上的项目配额。使用以下命令，为卷启用配额：

    [root@demo ~]# gluster volume quota <VOLUME> enable
    
> 注意: 默认情况下，配额不是对整个卷设置，而是对卷上的各个目录进行设置。如果需要为整个卷设置限制，可以将限制作用于该卷上的 /。

> 重要: 在启用配额后，使用原生客户端的客户端必须先卸载，然后重新挂载该卷，以从该客户端启用配额执行。 

###### 设置目录限制

配额可以设置为 hard-limit，即某一目录下所有文件和目录的最大允许大小；也可设置为 soft-limit，即硬限制的百分比。超过软限制后，日志文件中将放入与组成该卷的 brick 对应的条目。达到硬限制后，写入将失败，直到已用空间大小重新下降到硬限制以下。

若要对目录设置配额 (hard-limit)，可使用以下命令。

    [root@demo ~]# gluster volume quota <VOLUME> limit-usage <PATH-ON-VOLUME> <SIZE> [<SOFTLIMIT-PERCENTAGE>]

soft-limit 默认配置为 hard-limit 的 80%，它可以通过 softlimitsize 字段以百分数形式进行配置。

注意 <SIZE> 可以使用 MB、GB 和 PB 等单位。虽然这使用 SI 单位表示法，但实际运用中也使用 MiB 和 GiB 等二进制单位。例如，设置 1GB 限制会将该限制设为 1024 MiB。

> 指定路径名称： 在指定卷上目录的路径名称时，请仅指定所处卷上顶级目录的目录，还要注意路径名称必须使用前导斜杠。例如，如果卷挂载到 /mnt/serenity 上，并且需要对 /mnt/serenity/alliance 设置目录限制，请将路径名称指定为 /alliance。 

###### 设置超时和警告

1. 设置默认软限制

可以使用以下命令，将默认 soft-limit 大小重新配置为所需的百分比：

    [root@demo ~]# gluster volume quota <VOLUME> default-soft-limit <SOFTLIMIT-PERCENTAGE>
    
这将为尚未手动重新配置 soft-limit 的任何目录重新配置其 soft-limit。

2. 设置配额更新超时

红帽 Gluster 存储不以原子方式更新配额用量表，而是以特定的间隔重新计算用量。

在达到 soft-limit 前，每隔 soft-timeout 秒重新计算总使用量。软超时的默认值为 60 秒。由于不同的卷可能有不同的访问模式，此超时值可以调整，以匹配卷上的预期数据增长。要更改此超时，可使用以下命令：

    [root@demo ~]# gluster volume quota <VOLUME> soft-timeout <TIMEOUT>
    
<TIMEOUT> 以秒为单位指定，范围是 1 秒到 1800 秒（30 分钟）。要指定更大的超时值，可以使用后缀 s（秒钟）或 m（分钟）。

hard-timeout 是当用量超过软限制后配额服务器端转换器检查卷用量的频率。当磁盘用量介于软限制和硬限制之间时，硬超时发挥作用。hard-timeout 的默认值为 5 秒。可以使用以下命令配置硬超时：

    [root@demo ~]# gluster volume quota <VOLUME> hard-timeout <TIMEOUT>

> 重要: 由于配额更新不是原子性的，可能会有某一目录超出配额的情况。例如，如果硬超时设为 5 秒钟，并且存储后端能够处理的总写入速度为 800MiB/s，则最糟糕时可能会超过配额 4000MiB；如果尚未达到软限制，并且软超时较长，这一数字可能会更大。

因此，设置的超时值务必要能体现出目录允许超过配额的空间量，以及存储后端的写入效率。 

###### 报告配额

以下命令显示卷的配额使用量：

    [root@demo ~]# gluster volume quota <VOLUME> list

这将显示当前的限制、卷上实施有限制的所有目录的使用量，以及是否超过了这些限制。

1. 使用 df 报告配额

由于大部分用户没有受信存储池中服务器的管理访问权限，因此可对卷进行配置，以使用 df 命令报告目录的配额信息。

无需额外的配置，只要以参数形式提供卷上的目录，df 命令就能显示卷上可用空间总量。如果设置卷选项 quota-deem-statfs on，则 df 命令会将 hard-limit 报告为目录的可用空间。

    [root@demo ~]# gluster volume set <VOLUME> quota-deem-statfs on

      
### 引导式练习：设置配额

要求根据下方的要求为 graphics 卷（挂载于 workstation 上的 /mnt/graphics）实施配额：

*    /raw 中文件的大小不应超过 1 GiB。
*    使用量达到限制的 50% 时，日志文件中应当开始出现各个 brick 的日志消息。
*    在达到软限制前，应当每五秒更新配额信息。
*    在达到硬限制后，应当每秒更新配额信息。
*    在 workstation 上运行 df -h /mnt/graphics/raw 时，它应当报告该目录（而非整个卷）所剩余的空间量。 

1. 为 graphics 卷启用配额，并且为 /raw 目录设置硬限制和软限制（1 GiB 和 50%）。

    a. 从 workstation 卸载 /mnt/graphics。

    [root@workstation ~]# umount /mnt/graphics

    b. 在 servera 上，为 graphics 卷启用配额。

    [root@servera ~]# gluster volume quota graphics enable

    c. 为 graphics 上的 /raw 设置限制，硬限制为 1 GiB，软限制为 50%。

    [root@servera ~]# gluster volume quota graphics limit-usage /raw 1GB 50%
    
2. 为 graphics 设置配额更新超时，达到软限制前为 5 秒，超过软限制后为 1 秒。

    a. 将 graphics 的软超时限制设为 5 秒。

    [root@servera ~]# gluster volume quota graphics soft-timeout 5s

    b. 将 graphics 的硬超时限制设为 1 秒。

    [root@servera ~]# gluster volume quota graphics hard-timeout 1s

3. 配置 graphics 卷，使得 df 命令报告配额使用情况的剩余空间量，而非整个物理可用空间量。

    a. 为 graphics 卷启用 quota-deem-statfs。

    [root@servera ~]# gluster volume set graphics quota-deem-statfs on

4. 重新将 graphics 卷挂载到 workstation 上，然后测试 /mnt/graphics/raw 的配额。

    a. 在 workstation 上重新挂载 graphics 卷。

    [root@workstation ~]# mount /mnt/graphics

    b. 测试 /mnt/graphics/raw 的配额。您可能会超过 1 GiB 硬限制，但不会很多。

    [root@workstation ~]# dd if=/dev/zero of=/mnt/graphics/raw/testfile bs=1M
    dd: error writing ’/mnt/graphics/raw/testfile’: Disk quota exceeded
    dd: closing output file ’/mnt/graphics/raw/testfile’: Disk quota exceeded

    c. 检查 /mnt/graphics/raw/testfile 的大小。

    [root@workstation ~]# ls -lh /mnt/graphics/raw/testfile
    -rw-r--r--. 1 root root 1.2G Apr 29 16:18 testfile

    d. 删除测试期间使用的文件。

    [root@workstation ~]# rm /mnt/graphics/raw/testfile


### 实验：配置 ACL 和配额

红帽 Gluster 存储部署从 servera 提供名为 finance 的卷。已要求您利用原生客户端将这个卷永久挂载到 workstation 上的 /mnt/finance 中，并且要符合以下要求：

*    其中应当有一个新目录 /mnt/finance/profits，所有者为 accountants 组。
*    在 /mnt/finance/profits 中创建的任何新文件和目录也都应当归 accountants 组所有。
*    accountants 组中的用户应当对 /mnt/finance/profits 目录及其内容具有完整控制。
*    directors 组中的用户应当对 /mnt/finance/profits 及其内容具有只读访问权限。
*    不属于 accountants 或 directors 组的用户应当对 /mnt/finance/profits 目录无访问权限。 

出于测试目的，workstation 上具有三个用户帐户，密码均为 redhat：

*    aramis 是 accountants 的成员
*    porthos 是 directors 的成员
*    athos 同时是 accountants 和 directors 的成员 

您也被要求根据如下规格，限制对 /mnt/finance/profits 的使用：

*    /mnt/finance/profits 中文件的大小不应超过 1 GiB。
*    使用了限制的 85% 时，日志文件中应当开始出现各个 brick 的日志消息。
*    在达到软限制前，应当每隔 30 秒更新配额信息。
*    在达到硬限制后，应当每隔 5 秒更新配额信息。 

1. 使用 acl 选项，将 servera:/finance 卷永久挂载到 workstation 上的 /mnt/finance。

    a. 在 workstation 上安装 glusterfs-fuse 软件包。

    [root@workstation ~]# yum -y install glusterfs-fuse

    b. 在 workstation 上，创建挂载点 /mnt/finance。

    [root@workstation ~]# mkdir /mnt/finance

    c. 将以下条目添加到 workstation 上的 /etc/fstab。

    servera:/finance /mnt/finance glusterfs _netdev,acl 0 0

    d. 挂载卷。

    [root@workstation ~]# mount /mnt/finance

2. 在 workstation 上创建目录 /mnt/finance/profits，将组所有权设置到 accountants，然后确保该目录中的任何新文件或目录也都属于 accountants 组。赋予 accountants 组完整访问权限，并且限制其他组的访问权限。

    a. 在 workstation 上创建目录 /mnt/finance/profits。

    [root@workstation ~]# mkdir /mnt/finance/profits

    b. 将新目录的所有权设置到 accountants。

    [root@workstation ~]# chgrp accountants /mnt/finance/profits

    c. 设置 /mnt/finance/profits 的权限，使它成为 accountants 组的共享目录。

    [root@workstation ~]# chmod 2770 /mnt/finance/profits

3. 在 /mnt/finance/profits 上配置 ACL，允许 accountants 组的成员对任何现有文件和目录以及任何新创建文件具有完整访问权限。

    a. 在 /mnt/finance/profits 上配置 ACL，授予 accountants 组对任何现有文件和目录的完整访问权限。

    [root@workstation ~]# setfacl -R -m g:accountants:rwX /mnt/finance/profits

    b. 在 /mnt/finance/profits 上配置 ACL，授予 accountants 组对任何新文件和目录的完整访问权限。

    [root@workstation ~]# setfacl -R -m d:g:accountants:rwX /mnt/finance/profits

4. 在 /mnt/finance/profits 上配置 ACL，允许 directors 组的成员对任何现有文件和目录以及任何新创建文件具有只读访问权限。

    a. 在 /mnt/finance/profits 上配置 ACL，授予 directors 组对任何现有文件和目录的只读访问权限。

    [root@workstation ~]# setfacl -R -m g:directors:rX /mnt/finance/profits

    b. 在 /mnt/finance/profits 上配置 ACL，授予 directors 组对任何新文件和目录的只读访问权限。

    [root@workstation ~]# setfacl -R -m d:g:directors:rX /mnt/finance/profits

5. 为 finance 卷启用配额，并且为 /profits 目录设置硬限制和软限制（1 GiB 和 85%）。

    a. 从 workstation 卸载 /mnt/finance。

    [root@workstation ~]# umount /mnt/finance

    b. 在 servera 上，为 finance 卷启用配额。

    [root@servera ~]# gluster volume quota finance enable

    c. 为 finance 上的 /profits 设置硬限制为 1GiB，软限制为 85%。

    [root@servera ~]# gluster volume quota finance limit-usage /profits 1GB 85%

6. 为 finance 设置配额更新超时，达到软限制前为 30 秒，超过软限制后为 5 秒。

    a. 将 finance 的软超时限制设为 30  秒。

    [root@servera ~]# gluster volume quota finance soft-timeout 30s

    b. 将 finance 的硬超时限制设为 5 秒。

    [root@servera ~]# gluster volume quota finance hard-timeout 5s

    c. 在 workstation 上重新挂载 /mnt/finance。

    [root@workstation ~]# mount /mnt/finance


### 总结

在本章中，您已学会如何：

*    使用以下命令设置 POSIX ACL：setfacl
*    使用以下命令查询 POSIX ACL：getfacl
*    为原生、NFS 和 Samba 客户端启用 ACL。
*    为卷启用目录配额。
*    利用 gluster volume quota VOLUME> limit-usage 限制卷上的目录使用量。
*    通过 soft-timeout 和 hard-timeout 配置配额更新间隔。
*    报告配额使用情况。
*    启用 quota-deem-statfs 选项，以利用 df 进行客户端配额报告。 
