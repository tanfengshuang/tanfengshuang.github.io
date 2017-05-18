---
layout: post
title:  "使用命令行界面管理块存储(110-15)"
categories: Linux
tags: RHCA 110
---

### 使用命令行界面管理卷

Cinder 服务管理 Red Hat OpenStack Platform 中的块存储。它允许创建卷，然后连接到实例。这些新设备接着可以格式化为所需的任何文件系统。这为将永久存储连接到实例提供了途径。

有多种后端可用于 Cinder 管理的卷。默认的一个由 PackStack 配置，称为 LVM。它基于在安装时配置的卷组；默认为 cinder-volumes。通过 Cinder 创建的每一个新卷将关联为 cinder-volumes 卷组中的逻辑卷作为后端。该逻辑卷很容易识别，因为其名称包含与相关 Cinder 卷关联的 UUID。

> Cinder 目前支持基于 LVM、NFS、GlusterFS 和 Ceph 的存储后端。它也支持主要的存储供应商系统。

*    使用 openstack 命令管理卷
*    命令 	                    说明
*    openstack volume create 	创建一个新卷
*    openstack volume delete 	删除卷
*    openstack volume list 	    列出卷
*    openstack volume show 	    显示卷详细信息

LVM 后端适合概念验证部署，不适合生产就绪型 Red Hat OpenStack Platform 环境。Cinder 包含适用于红帽 Ceph 存储等存储系统的驱动程序，提供可扩展的存储系统来与 Red Hat OpenStack Platform 服务轻松集成。根据所用的存储系统，Cinder 卷映射到所选存储系统中不同的对象。

###### 使用命令行界面管理卷

1. 在 servera 上，提供 /root/keystonerc_demouser 文件，以获取 demouser 的权限。

    [root@servera ~]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

2. 列出当前可用的卷。应当有名为 demosecondvolume 的卷。本练习中稍后将操作该卷。

    [root@servera ~(keystone_demouser)]$ openstack volume list
    +--------------------------------------+------------------+-----------+------+-------------+
    | ID                                   | Display Name     | Status    | Size | Attached to |
    +--------------------------------------+------------------+-----------+------+-------------+
    | 44d74ac9-d44a-457c-804b-6f99e146cc80 | demosecondvolume | available |    3 |             |
    +--------------------------------------+------------------+-----------+------+-------------+

3. 创建一个 2 GB 的新卷，命名为 demovolume。

    [root@servera ~(keystone_demouser)]$ openstack volume create --size 2 demovolume
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    ...
    | display_name        | demovolume                           |
    ...
    | size                | 2                                    |
    ...
    +---------------------+--------------------------------------+

4. 列出当前可用的卷。现在应当有两个卷，即 demosecondvolume 和 demovolume。

    [root@servera ~(keystone_demouser)]$ openstack volume list
    +--------------------------------------+------------------+-----------+------+-------------+
    | ID                                   | Display Name     | Status    | Size | Attached to |
    +--------------------------------------+------------------+-----------+------+-------------+
    | 44d74ac9-d44a-457c-804b-6f99e146cc80 | demosecondvolume | available |    3 |             |
    | 4ac44d79-4ad4-7c45-4b80-e146cc86f990 | demovolume       | available |    2 |             |
    +--------------------------------------+------------------+-----------+------+-------------+

5. 显示 demosecondvolume 卷的详细信息。其大小为 3 GB，其状态为可用，所以它已做好使用准备。

    [root@servera ~(keystone_demouser)]$ openstack volume show demosecondvolume
    +---------------------------------------+--------------------------------------+
    | Field                                 | Value                                |
    +---------------------------------------+--------------------------------------+
    ...
    | display_name                          | demosecondvolume                     |
    ...
    | size                                  | 3                                    |
    ...
    | status                                | available                            |
    ...
    +---------------------------------------+--------------------------------------+

6. 删除 demosecondvolume 卷。

    [root@servera ~(keystone_demouser)]$ openstack volume delete demosecondvolume


### 使用命令行界面管理实例中的卷

由 Cinder 服务管理的卷可以连接到 Red Hat OpenStack Platform 环境中的实例，从而为它们提供永久存储。同一个实例可以连接多个卷，但一个卷一个时间上只能连接到一个实例。

卷作为裸设备映射为实例中的设备，所以不能立即可供使用。若要使用它们，需要根据要求对它们进行手动分区，并使用所需的文件系统格式化这些分区。完成之后，新格式化好的设备必须挂载到实例文件系统树中。

*    使用 openstack 命令管理实例中的卷
*    命令 	                            说明
*    openstack server add volume 	    将卷与实例连接
*    openstack server remove volume 	将卷与实例分离

###### 使用命令行界面管理实例中的卷

1. 在 servera 上，提供 /root/keystonerc_demouser 文件，以获取 demouser 的权限。

    [root@servera ~]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

2. 创建一个 1 GB 新卷，命名为 demovolume。

    [root@servera ~(keystone_demouser)]$ openstack volume create --size 1 demovolume
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    ...
    | display_name        | demovolume                           |
    ...
    | size                | 1                                    |
    ...
    +---------------------+--------------------------------------+

3. 部署一个名为 demovm 的新实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --image demoimage --wait demovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    ...
    | image                                | demoimage (08c49af8-445a-4dd8-9712-503cbe3cfc50) |
    | key_name                             | demokeypair                                      |
    | name                                 | demovm                                           |
    ...
    +--------------------------------------+--------------------------------------------------+

4. 从 ext 池分配一个新的浮动 IP。

    [root@servera ~(keystone_demouser)]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    ...
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

5. 将新的浮动 IP 关联到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack ip floating add 172.25.250.27 demovm
          
6. 将 demovolume 卷连接到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack server add volume demovm demovolume
          
7. 以 cloud-user 用户身份并使用密钥 /root/demoprivatekey.key 登录 demovm 实例。

    [root@servera ~(keystone_demouser)]$ ssh -i /root/demoprivatekey.key cloud-user@172.25.250.27
    [cloud-user@demovm ~]$ 

8. 列出 demovm 实例中可用的设备。应当有一个存放根文件系统的 vda 设备，以及与 demovolume 卷对应的 vdb 设备。设备 vdb 的大小为 1 GB，demovolume 卷的大小也一样。完成时，从 demovm 注销。

    [cloud-user@demovm ~]$ sudo fdisk -l
    Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
    ...
       Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *        2048    41942314    20970133+  83  Linux

    Disk /dev/vdb: 1073 MB, 1073741824 bytes, 2097152 sectors
    ...
    [cloud-user@demovm ~]$ logout
    Connection to 172.25.250.27 closed.
   
9. 将卷 demovolume 与实例 demovm 分离。

    [root@servera ~(keystone_demouser)]$ openstack server remove volume demovm demovolume


### 使用命令行界面管理快照

Cinder 允许创建卷快照，它们是某一时间点上卷内容的副本。快照创建之后，可用作单独的卷，该卷也可与实例连接。一个卷可以关联有多个快照。注意快照占用的空间将算入块存储配额中。

> 在为卷执行快照之前，建议先将卷从任何连接的实例分离。

*    使用 openstack 命令管理快照
*    命令 	                    说明
*    openstack snapshot create 	创建新快照
*    openstack snapshot delete 	删除快照
*    openstack snapshot list 	列出快照
*    openstack snapshot show 	显示快照详细信息

###### 使用命令行界面管理快照

1. 在 servera 上，提供 /root/keystonerc_demouser 文件，以获取管理员的权限。

    [root@servera ~]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

2. 创建一个 1 GB 新卷，命名为 demovolume。

    [root@servera ~(keystone_demouser)]$ openstack volume create --size 1 demovolume
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    ...
    | display_name        | demovolume                           |
    ...
    | size                | 1                                    |
    ...
    +---------------------+--------------------------------------+

3. 部署一个名为 demovm 的新实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --image demoimage --wait demovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    ...
    | image                                | demoimage ...                                    |
    | key_name                             | demokeypair                                      |
    | name                                 | demovm                                           |
    ...
    +--------------------------------------+--------------------------------------------------+

4. 从 ext 池分配一个新的浮动 IP。

    [root@servera ~(keystone_demouser)]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    ...
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

5. 将新的浮动 IP 关联到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack ip floating add 172.25.250.27 demovm
          
6. 为 demovolume 卷创建一个新快照。将它命名为 demosnapshot。检查其大小是否为 1 GB，与卷 demovolume 相同。记下它的 UUID，以便在本演示的后续部分中使用。

    [root@servera ~(keystone_demouser)]$ openstack snapshot create --name demosnapshot demovolume
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    ...
    | display_name        | demosnapshot                         |
    | id                  | c78a7ef5-9a4c-4bb7-8791-e14ddd3b957f |
    ...
    | size                | 1                                    |
    ...
    +---------------------+--------------------------------------+

7. 根据 demosnapshot 快照创建一个新卷。将它命名为 demosnapvol。其大小应当为 1 GB，您将需要使用关联的 UUID（之前获取的）标识来源快照 demosnapshot。

    [root@servera ~(keystone_demouser)]$ openstack volume create --size 1 --snapshot c78a7ef5-9a4c-4bb7-8791-e14ddd3b957f demosnapvol
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    ...
    | display_name        | demosnapvol                          |
    ...
    | size                | 1                                    |
    | snapshot_id         | c78a7ef5-9a4c-4bb7-8791-e14ddd3b957f |
    ...
    +---------------------+--------------------------------------+

8. 显示 demosnapvol 卷的详细信息。其大小为 1 GB，其状态为可用，所以它已做好使用准备。

    [root@servera ~(keystone_demouser)]$ openstack volume show demosnapvol
    +---------------------------------------+--------------------------------------+
    | Field                                 | Value                                |
    +---------------------------------------+--------------------------------------+
    ...
    | display_name                          | demosnapvol                          |
    ...
    | size                                  | 1                                    |
    ...
    | status                                | available                            |
    ...
    +---------------------------------------+--------------------------------------+

9. 将 demosnapvol 卷连接到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack server add volume demovm demosnapvol
          
10. 以 cloud-user 用户身份并使用密钥 /root/demoprivatekey.key 登录 demovm 实例。

    [root@servera ~(keystone_demouser)]$ ssh -i /root/demoprivatekey.key cloud-user@172.25.250.27
    [cloud-user@demovm ~]$ 

11. 列出 demovm 实例中可用的设备。应当有一个存放根文件系统的 vda 设备，以及与 demosnapvol 卷对应的 vdb 设备。设备 vdb 的大小为 1 GB，demosnapvol 卷的大小也一样。完成时，从 demovm 注销。

    [cloud-user@demovm ~]$ sudo fdisk -l
    Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
    ...
       Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *        2048    41942314    20970133+  83  Linux

    Disk /dev/vdb: 1073 MB, 1073741824 bytes, 2097152 sectors
    ...
    [cloud-user@demovm ~]$ logout
    Connection to 172.25.250.27 closed.
            
12. 删除由实验脚本创建的 demosecondsnapshot 快照。

    [root@servera ~(keystone_demouser)]$ openstack snapshot delete demosecondsnapshot

### 总结

在本章中，您可以学到：

*    Red Hat OpenStack Platform 中的块存储服务由 Cinder 提供，利用的是其中一种可用的存储后端，如 LVM 或红帽 Ceph 存储。
*    Cinder 中创建的卷可以连接到 Red Hat OpenStack Platform 中部署的实例。
*    Cinder 卷以裸设备的形式呈现给实例，所以需要使用某一文件系统将它们格式化并进行挂载，然后才能使用它们。
