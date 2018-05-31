---
layout: post
title:  "管理类别(210-2)"
categories: Linux
tags: RHCA 210
---

*    管理类别
*    练习：管理类别
*    验证类别功能
*    练习：验证类别功能
*    自定义类别属性
*    练习：自定义类别属性
*    管理类别访问权限
*    练习：管理类别访问权限
*    实验：管理类别
*    总结

### 管理类别

###### 类别

类别是资源模板，决定实例的 RAM 和磁盘的大小，以及核心数量（即容量）。类别也可以指定辅助临时存储、交换磁盘、用于限制使用的元数据，或者特殊项目访问权限。OpenStack 默认安装中提供五种类别。不过，在一些用例中可能需要创建和管理专门的类别，例如更改默认内存和容量来满足底层硬件需求，或者添加元数据以强制实例使用特定的 I/O 速率。

默认类别

*    名称 	    VCPU 	    RAM 	    根磁盘大小
*    m1.tiny 	1 日关闭。 	512 MB 	    1 GB
*    m1.small 	1 日关闭。 	2048 MB 	20 GB
*    m1.medium 	2 	        4096 MB 	40 GB
*    m1.large 	4 	        8192 MB 	80 GB
*    m1.xlarge 	8 	        16384 MB 	160 GB

###### 创建类别所用的参数

类别允许用户定义参数，使得用户可以选择要运行的实例类型。除了与 RAM 大小、VCPU 和磁盘相关的灵活性外，元数据可以定义 NUMA 拓扑的自由特征；例如，允许 RAM 用于视频设备，以及设置磁盘读/写 IOPS 配额限制等。

类别参数

*    参数 	详情
*    名称 	类别的名称
*    ID 	类别的 ID。默认值为 auto，可生成 UUID 值，但也可以手动指定为整数或 UUID 值。
*    VCPU 	虚拟 CPU 的数量
*    RAM (MB) 	内存（以 MB 为单位）
*    根磁盘 (GB) 	临时磁盘的大小（以 GB 为单位）；若要使用原生映像大小，可指定 0。如果在启动实例时 Instance Boot Source=Boot from Volume，将忽略此参数。
*    临时磁盘 (GB) 	辅助临时磁盘大小（以 GB 为单位）
*    交换磁盘 (MB) 	交换磁盘大小（以 MB 为单位）

注意: 类别访问权限允许用户指定选定项目，或可以使用该类别的项目。如果不选择项目，则所有类别都具有公共访问权限。

###### 演示：管理类别

1. 在 workstation 上提供 overcloudrc 文件，以获取 admin 权限。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ 
```

2. 获取关于 openstack 命令提供哪些类别管理选项的其他信息。

```
[student@workstation ~]$ openstack help flavor
Command "flavor" matches:
  flavor create
  flavor delete
  flavor list
  flavor set
  flavor show
  flavor unset
```

3. 获取关于 openstack flavor create 命令的其他信息。

```
[student@workstation ~]$ openstack help flavor create
usage: openstack flavor create [-h]
                               [-f {html,json,json,shell,table,value,yaml,yaml}]
                               [-c COLUMN] [--max-width <integer>]
                               [--noindent] [--prefix PREFIX] [--id <id>]
                               [--ram <size-mb>] [--disk <size-gb>]
                               [--ephemeral <size-gb>] [--swap <size-gb>]
                               [--vcpus <vcpus>] [--rxtx-factor <factor>]
                               [--public | --private]
                               <flavor-name>

Create new flavor

positional arguments:
  <flavor-name>         New flavor name
...Output omitted...
```

4. 使用上述信息，创建一个名为 demoflavor 的新类别，它具有 1 个 VCPU、1024 MB RAM、10 GB root 文件系统磁盘、2 GB 临时磁盘，以及 512 MB 交换内存。

```
[student@workstation ~]$ openstack flavor create --vcpus 1 --ram 1024 --disk 10 --ephemeral 2 --swap 512 demoflavor
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 2                                    |
| disk                       | 10                                   |
| id                         | 2446b0f9-bad9-48e7-8ddc-5feb3e2c6ae2 |
| name                       | demoflavor                           |
| os-flavor-access:is_public | True                                 |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 512                                  |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

5. 列出可用的类别。应当会显示新创建的 demoflavor。

```
[student@workstation ~]$ openstack flavor list
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
| ID                                   | Name       |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
...Output omitted...
| 2446b0f9-bad9-48e7-8ddc-5feb3e2c6ae2 | demoflavor |  1024 |   10 |         2 |     1 | True      |
...Output omitted...
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
```

6. 获取类别 demoflavor 的详细信息。

```
[student@workstation ~]$ openstack flavor show demoflavor
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 2                                    |
| disk                       | 10                                   |
| id                         | 2446b0f9-bad9-48e7-8ddc-5feb3e2c6ae2 |
| name                       | demoflavor                           |
| os-flavor-access:is_public | True                                 |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 512                                  |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

7. 删除由实验脚本创建的类别 demosecondflavor。

```
[student@workstation ~]$ openstack flavor delete demosecondflavor
```

8. 运行 openstack flavor list 命令，确认已成功删除 demosecondflavor。

```
[student@workstation ~]$ openstack flavor list
+--------------------------------------+------------+-------+
| ID                                   | Name       |   RAM |
+--------------------------------------+------------+-------+
| 1                                    | m1.tiny    |   512 |
| 2                                    | m1.small   |  2048 |
| 3                                    | m1.medium  |  4096 |
| 4                                    | m1.large   |  8192 |
| 5                                    | m1.xlarge  | 16384 |
| f63b0b95-902b-40d4-bcae-c96b12a12838 | demoflavor |  1024 |
+--------------------------------------+------------+-------+
------+-----------+-------+-----------+
 Disk | Ephemeral | VCPUs | Is Public |
------+-----------+-------+-----------+
    1 |         0 |     1 | True      |
   20 |         0 |     1 | True      |
   40 |         0 |     2 | True      |
   80 |         0 |     4 | True      |
  160 |         0 |     8 | True      |
   10 |         2 |     1 | True      |
------+-----------+-------+-----------+
```


### 练习：管理类别

1. 在workstation上，获取 admin 凭据，以便创建类别。

    a. 提供文件overcloudrc，以获取 admin 凭据。

```
    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ 
```

2. 使用openstack flavor list命令，列出当前可用的类别。您应当会看到列出默认类别，以及由实验脚本所创建的名为 devsecondflavor 的类别。

```
[student@workstation ~]$ openstack flavor list
+--------------------------------------+-----------------+-------+------+-----------+-------+-----------+
| ID                                   | Name            |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------------+-------+------+-----------+-------+-----------+
| 09137934-f246-4798-92d0-e0023f0f52c9 | devsecondflavor |  2048 |   10 |         2 |     2 | True      |
| 1                                    | m1.tiny         |   512 |    1 |         0 |     1 | True      |
| 2                                    | m1.small        |  2048 |   20 |         0 |     1 | True      |
| 3                                    | m1.medium       |  4096 |   40 |         0 |     2 | True      |
| 4                                    | m1.large        |  8192 |   80 |         0 |     4 | True      |
| 5                                    | m1.xlarge       | 16384 |  160 |         0 |     8 | True      |
+--------------------------------------+-----------------+-------+------+-----------+-------+-----------+
```

3. 创建一个名为 devflavor 的新类别，它具有 1 个 VCPU、1024 MB RAM、10 GB root 文件系统磁盘、2 GB 临时磁盘，以及 512 MB 交换内存。

    a. 使用openstack flavor create命令，创建类别 devflavor，它具有 1 个 VCPU、1024 MB RAM、10 GB root 文件系统磁盘、2 GB 临时磁盘，以及 512 MB 交换内存。

```
    [student@workstation ~]$ openstack flavor create --vcpus 1 --ram 1024 --disk 10 --ephemeral 2 --swap 512 devflavor
    ...Output omitted...
```

    b. 使用openstack flavor show命令，列出新创建的类别 devflavor 的详细信息。

```
    [student@workstation ~]$ openstack flavor show devflavor
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 2                                    |
    | disk                       | 10                                   |
    | id                         | bbca1d0c-e86a-4580-8eb6-0c8bfc564f22 |
    | name                       | devflavor                            |
    | os-flavor-access:is_public | True                                 |
    | properties                 |                                      |
    | ram                        | 1024                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       | 512                                  |
    | vcpus                      | 1                                    |
    +----------------------------+--------------------------------------+
```

4. 删除由实验脚本创建的类别 devsecondflavor。

    a. 使用openstack flavor delete命令，删除类别 devsecondflavor。

```
    [student@workstation ~]$ openstack flavor delete devsecondflavor
```


### 验证类别功能

###### 验证类别

类别定义一组资源，它们分配到使用该类别的实例。下列命令可以帮助验证资源是否已正确分配，并且遵循类别要求：

> 注意: 请记住，临时磁盘挂载到实例中的 /mnt 下。

实例上的资源检查

*    命令 	            详情
*    lscpu 	            包含关于所配置的 VCPU 数量的详细信息
*    free -m 	        提供内存及交换磁盘信息
*    df -h mount_point 	提供关于根磁盘和临时磁盘的信息

### 练习：验证类别功能

1. 提供文件overcloudrc，以获取 admin 凭据。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ 
```

2. 使用openstack flavor list命令，列出可用的类别。其中应当列出由实验脚本创建的 devflavor 类别。

```
[student@workstation ~]$ openstack flavor list
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
| ID                                   | Name       |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
...Output omitted...
| ecbfcd56-d492-4fc5-8b59-463cd1374597 | devflavor |  1024 |   10 |         2 |     1 | True      |
+--------------------------------------+------------+-------+------+-----------+-------+-----------+
```

3. 使用 openstack flavor show 命令价差类别 devflavor 的要求是否包含 1024 MB RAM、10 GB 根磁盘、2 GB 临时磁盘、512 MB 交换内存以及 1 个 VCPU。

```
[student@workstation ~]$ openstack flavor show devflavor
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 2                                    |
| disk                       | 10                                   |
| id                         | ecbfcd56-d492-4fc5-8b59-463cd1374597 |
| name                       | devflavor                            |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 512                                  |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

4. 使用 openstack image list 命令，列出可用的映像。其中应当列出由实验脚本创建的 devimage 映像。

```
[student@workstation ~]$ openstack image list
+--------------------------------------+----------+
| ID                                   | Name     |
+--------------------------------------+----------+
| 0e6afa93-0a2d-4698-9c0b-3231a352ef47 | devimage |
+--------------------------------------+----------+
```

5. 使用 openstack image show 命令，显示 devimage 映像的详细信息。您需要使用其 UUID 来执行此操作，这已通过上一命令获取。检查映像要求；devflavor 需要的资源应当同时提供 min_disk 和 min_ram。

```
[student@workstation ~]$ openstack image show 0e6afa93-0a2d-4698-9c0b-3231a352ef47
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
...Output omitted...
| id               | 0e6afa93-0a2d-4698-9c0b-3231a352ef47                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | devimage                                             |
...Output omitted...
+------------------+------------------------------------------------------+
```

6. 使用 openstack keypair list 命令，列出可用的密钥对。其中应当列出由实验脚本创建的密钥对 devkeypair。

```
[student@workstation ~]$ openstack keypair list
+-------------+-------------------------------------------------+
| Name        | Fingerprint                                     |
+-------------+-------------------------------------------------+
| devkeypair | 0f:ec:e8:2f:c8:d0:95:85:ee:7b:f7:cd:6b:91:87:97 |
+-------------+-------------------------------------------------+
```

7. 使用 openstack network list 命令，列出可用的网络。其中应当列出由实验脚本创建的网络 devnet。

```
[student@workstation ~]$ openstack network list
+--------------------------------------+--------+--------------------------------------+
| ID                                   | Name   | Subnets                              |
+--------------------------------------+--------+--------------------------------------+
| 8ea87df6-4ddb-4bee-9024-68c6e4341c2e | ext    | 67be1902-60f6-49df-ae43-1dc6814a511b |
| ea806142-cd19-47e9-812b-9c2753602b51 | devnet | acecd647-2875-49ef-b10e-ba12a165115b |
+--------------------------------------+--------+--------------------------------------+
```

8. 使用 openstack server create 命令，创建名为 devvm 的新实例，它使用前面检查过的资源，包括 devimage 映像、devkeypair 密钥对、devnet 网络和 devflavor 类别。

```
[student@workstation ~]$ openstack server create --flavor devflavor --key-name devkeypair --image devimage --nic net-id=devnet devvm
+------------+---------------------------------------------------+
| Field      | Value                                             |
+------------+---------------------------------------------------+
...Output omitted...
| created    | 2016-01-20T12:50:09Z                              |
| flavor     | devflavor (...)                                   |
| hostId     |                                                   |
| image      | devimage (...)                                    |
| key_name   | devkeypair                                        |
| name       | devvm                                             |
...Output omitted...
+------------+---------------------------------------------------+
```

9. 使用 openstack server list 命令检查 devvm 状态。等待它变为 ACTIVE。

```
[student@workstation ~]$ openstack server list
+--------------------------------------+-----------+--------+--------------------------------------+
| ID                                   | Name      | Status | Networks                             |
+--------------------------------------+-----------+--------+--------------------------------------+
| 3840437e-f32f-45c5-aaf0-4d6d851f5407 | devvm     | ACTIVE | devnet=192.168.1.16                |
+--------------------------------------+-----------+--------+--------------------------------------+
```

10. 使用 openstack ip floating create 命令，从 ext 池分配一个新的浮动 IP。该浮动 IP 池已由实验脚本配置。

```
[student@workstation ~]$ openstack ip floating create ext
+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| fixed_ip    | None                                 |
| id          | 5dfda71b-c94b-4f31-93e3-7bc08bececfa |
| instance_id | None                                 |
| ip          | 172.25.250.27                        |
| pool        | ext                                  |
+-------------+--------------------------------------+
```

11. 使用 openstack ip floating add 命令，将之前分配的浮动 IP 关联到 devvm 实例。

```
[student@workstation ~]$ openstack ip floating add 172.25.250.27 devvm
```

12. 使用与 devkeypair 密钥对关联并由实验脚本创建的 /home/student/dev/devkeypair.pem 私钥文件，以 cloud-user 用户身份通过之前分配的浮动 IP 登录 devvm 实例。

```
[student@workstation ~]$ ssh -i /home/student/dev/devkeypair.pem cloud-user@172.25.250.27
[cloud-user@devvm ~]$ 
```

13. 检查 devvm 实例已部署了 1 个 VCPU，如 devflavor 类别中所配置。

```
[cloud-user@devvm ~]$ lscpu
...Output omitted...
CPU(s):                1
...Output omitted...
```

14. 检查 devvm实例已部署了 1024 MB RAM、512 MB 交换内存和 10 GB 磁盘，如 devflavor 类别中所配置。

```
[cloud-user@devvm ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           992          79        1619          16         141        1617
Swap:           511           0         511
[cloud-user@devvm ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    253:0    0   10G  0 disk
└─vda1 253:1    0   10G  0 part /
vdb    253:16   0    2G  0 disk /mnt
vdc    253:32   0  512M  0 disk [SWAP]
```

15. 检查 devvm 实例已部署了 2 GB 临时磁盘，如 devflavor 类别中所配置。完成时注销。

```
[cloud-user@devvm ~]$ df -h /mnt
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdb        2.0G  4.0K  2.0G   1% /mnt
[cloud-user@devvm ~]$ logout
Connection to 172.25.250.27 closed.
```

### 自定义类别属性

###### 自定义类别

红帽 OpenStack 平台为云管理员提供针对具体环境和用例自定义类别的功能。云管理员在使用网络连接不甚理想的环境时，可以限制特定实例的带宽。在存在设有套接字数上限的系统的环境中，云管理员可以在类别层面上配置套接字、核心和线程的限值和首选项。如果适用磁盘调优，则可利用磁盘 I/O 配额为用户设置每秒写入数上限。这种类别自定义可以通过 extra_specs 元素进行。

######### 使用 extra_specs 元素

extra_specs 类别元素可用于定义自由特征，在决定实例调配位置时具有很大的灵活性，而不仅限于 RAM、CPU 和磁盘的大小。该元素使用键值对来定义某一类别可以运行的计算节点。这些对必须与计算节点上对应的对匹配。例如，若要配置最大支持数量的 CPU 套接字，可使用 hw:cpu_max_sockets 键。下表列出了 extra_specs 元素提供的键：

*    hw:action             配置支持限值的操作。
*    hw:NUMA_def           实例的 NUMA 拓扑定义。
*    hw:watchdog_action    在实例出现某种失败（或挂起）时触发操作。
*    hw_rng:action         此操作项实例添加一个随机数生成器设备。
*    quota:option          对实例强制实施的一个限值。

######### 磁盘调优

extra_specs 元素提供可自定义性能的磁盘调优选项。下表列出了有效的选项：

类别参数

*    参数 	                详情
*    disk_read_bytes_sec 	磁盘读取上限（以字节/秒为单位）。
*    disk_read_iops_sec 	每秒读取 I/O 操作数上限。
*    disk_write_bytes_sec 	磁盘写入上限（以字节/秒为单位）。
*    disk_write_iops_sec 	每秒写入 I/O 操作数上限。
*    disk_total_bytes_sec 	磁盘总吞吐量上限（以字节/秒为单位）。
*    disk_total_iops_sec 	每秒总 I/O 操作数上限。
*    交换磁盘 (MB) 	        交换磁盘大小（以 MB 为单位）。 

######### 磁盘调优

若要实施磁盘 I/O 配额，可使用 openstack flavor set 命令。例如，要将 VM 用户的最大书写速度设置为 10 MB 每秒使用磁盘配额，使用下列语法：

```
[userdemo@demo ~]$ openstack flavor set m2.small --property quota:disk_write_bytes_sec=10485760
```

要将 VM 用户的最大读取速度设置为 10 MB 每秒使用磁盘配额，使用下列语法：

```
[userdemo@demo ~]$ openstack flavor set m2.small --property quota:disk_read_bytes_sec=10485760
```

使用下列命令来验证 flavor 磁盘配额：

```
[userdemo@demo ~]$ penstack flavor show m2.small
```


### 练习：自定义类别属性

1. 在 workstation 上，获取 admin 凭据，以便创建类别。

    a. 提供 overcloudrc 文件以获取 admin 凭据。

    [student@workstation ~]$ source overcloudrc

2. 检查 dev_instance 实例，以验证 extra_specs 已用于 m2.small 类别。

    a. 显示 dev_instance 实例，再检查 extra_specs 元素是否已配置用于 m2.small 类别。

    [student@workstation ~]$ openstack server show dev_instance | grep flavor
    | flavor  | m2.small (85aa04f3-0f67-4eb6-8c70-ed37fc88c33e)

3. 终止 dev_instance 实例。

    a. 使用openstack server delete命令，终止 dev_instance 实例。

    [student@workstation ~]$ openstack server delete dev_instance

4. 将磁盘写入速度上限设为每秒 10MB，磁盘读取速度上限设为每秒 10 MB，以调优类别 m2.small 的磁盘 I/O 配额。

    a. 使用 openstack flavor set 命令，将磁盘写入速度上限设为每秒 10 MB，以调优类别 m2.small 的磁盘 I/O 配额。

    [student@workstation ~]$ openstack flavor set m2.small --property quota:disk_write_bytes_sec=10485760
    +----------------------------+---------------------------------------+
    | Field                      | Value                                 |
    +----------------------------+---------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                 |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                     |
    | disk                       | 10                                    |
    | id                         | 85aa04f3-0f67-4eb6-8c70-ed37fc88c33e  |
    | name                       | m2.small                              |
    | os-flavor-access:is_public | True                                  |
    | properties                 | quota:disk_write_bytes_sec='10485760' |
    | ram                        | 1024                                  |
    | rxtx_factor                | 1.0                                   |
    | swap                       | 1                                     |
    | vcpus                      | 2                                     |
    +----------------------------+---------------------------------------+

    b. 使用 openstack flavor set 命令，将磁盘读取速度上限设为每秒 10 MB，以调优类别 m2.small 的磁盘 I/O 配额。

    [student@workstation ~]$ openstack flavor set m2.small --property quota:disk_read_bytes_sec=10485760
    +----------------------------+---------------------------------------+
    | Field                      | Value                                 |
    +----------------------------+---------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                 |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                     |
    | disk                       | 10                                    |
    | id                         | 85aa04f3-0f67-4eb6-8c70-ed37fc88c33e  |
    | name                       | m2.small                              |
    | os-flavor-access:is_public | True                                  |
    | properties                 | quota:disk_read_bytes_sec='10485760',
                                   quota:disk_write_bytes_sec='10485760' |
    | ram                        | 1024                                  |
    | rxtx_factor                | 1.0                                   |
    | swap                       | 1                                     |
    | vcpus                      | 2                                     |
    +----------------------------+---------------------------------------+
    
5. 启动 extraspecs_instance 实例，利用下表验证读取和写入速度。本实验的映像、网络、安全组规则和密钥对已使用 setup 脚本进行了调配。

参数	值

*    名称	extraspecs_instance
*    映像	small
*    类别	m2.small
*    密钥对	dev_key1
*    安全组	default
*    网络	dev_privnet

```
[student@workstation ~]$ openstack server create --image small --flavor m2.small --key-name dev_key1 --nic net-id=dev_privnet extraspecs_instance --wait

+--------------------------------------+------------------------------------ +
| Field                                | Value                               |
+--------------------------------------+------------------------------------ +
| OS-DCF:diskConfig                    | MANUAL                              |
| OS-EXT-AZ:availability_zone          | nova                                |
| OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-0.localdomain |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-0.localdomain |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000007                   |
| OS-EXT-STS:power_state               | 1                                   |
| OS-EXT-STS:task_state                | None                                |
| OS-EXT-STS:vm_state                  | active                              |
| OS-SRV-USG:launched_at               | 2016-08-05T19:00:47.000000          |
| OS-SRV-USG:terminated_at             | None                                |
| accessIPv4                           |                                     |
| accessIPv6                           |                                     |
| addresses                            | dev_privnet=192.168.1.16            |
| adminPass                            | a9r5XUsPKjMG                        |
| config_drive                         |                                     |
| created                              | 2016-08-05T18:59:52Z                |
...Output omitted...
| key_name                             | dev_key1                            |
| name                                 | dev_instance                        |
| os-extended-volumes:volumes_attached | []                                  |
| progress                             | 0                                   |
| project_id                           | aacb4534d71545d78b3b5375f82cd3c0    |
| properties                           |                                     |
| security_groups                      | [{u'name': u'default'}]             |
| status                               | ACTIVE                              |
| updated                              | 2016-08-05T19:00:47Z                |
| user_id                              | 329f2371a6924716b8a4e6f4a8e7310a    |
+--------------------------------------+------------------------------------ +
```

6. 使用 openstack ip floating add 命令，将 172.25.250.51 浮动 IP 关联到 extraspecs_instance 实例。

```
[student@workstation ~]$ openstack ip floating add 172.25.250.51 extraspecs_instance
```

7. 使用 /home/student/dev/dev_key1.pem 下保存的私钥，以 cloud-user 用户身份发起与实例的 SSH 连接。

```
[student@workstation ~]$ cd dev
[student@workstation dev]$ ssh -i dev_key1.pem cloud-user@172.25.250.51
[cloud-user@extraspecs-instance ~]$ 
```

8. 使用命令 dd，测试实例 extraspecs_instance 的写入速度。它的最大值应当是 10 Mbps。

```
[cloud-user@extraspecs-instance ~]$  dd if=/dev/zero of=anof bs=512MB count=1 oflag=direct
1+0 records in
1+0 records out
512000000 bytes (512 MB) copied, 51.8222 s, 9.9 MB/s
```

9. 使用 hdparm 命令，验证已经为实例 extraspecs_instance 配置了 10 MBps 的最大读取速度。使用 yum 命令安装 hdparm。

```
[cloud-user@extraspecs-instance ~]$ sudo yum -y install hdparm
```

10. 使用 hdparm 命令，测试实例 extraspecs_instance 的读取速度。

```
[cloud-user@extraspecs_instance ~]$ sudo hdparm -t --direct /dev/vda1
/dev/vda1:
Timing O_DIRECT disk reads:  34 MB in  3.11 seconds =  10.94 MB/sec   
```

### 管理类别访问权限

###### 专用类别

Red Hat OpenStack Platform 为云管理员提供根据项目成员资格限制类别访问权限的功能。云管理员可以创建专门为某一项目调优的类别。例如，在具有资源限制的云环境中，管理员可以通过创建专用类别，限制访问类别的项目数量。如果管理员创建了具有 256 GB RAM 的公共类别，则所有项目有权访问该类别。在资源问题严峻的环境中，这可导致云环境很快达到其容量。但是，创建专用类别将限制所有的项目，除非项目被明确授予该类别的访问权限。这也包括 admin 项目。

######### Nova policy.json

此计算服务使用基于角色的访问策略，来指定可由用户访问的对象以及可以访问它们的方式。策略定义在 Nova 的policy.json文件中规定。是否接受对 Nova 的 API 调用则由策略规则决定。这一决定不仅基于策略，也取决于发出 API 调用的用户以及 API 调用作用的对象。

在policy.json中，各项策略通过一行语句进行定义。

```
"target" :
"rule"
```

target 代表 API 调用，rule 则决定在哪些必要条件下允许该 API 调用。如果 API 仅可由 admin 调用，它将通过规则 "role:admin" 进行表述。例如，以下策略规则仅允许 admin 用户在 Identity 数据库中创建新用户。

```
"identity:create_user" : "role:admin"
```

可以通过将 "role:admin" 替换为""修改该规则，从而让所有用户都能够创建新用户。在修改 policy.json 文件后，需要重启该 API 服务。如果在文件/etc/nova/policy.json中修改了规则，您需要重启 openstack-nova-api 服务，使更改生效。例如，如果管理员希望将访问权限限制到仅属于 demoproject 的用户，该管理员应当执行以下操作：

###### 演示：管理类别访问权限

1. 从 workstation 打开一个终端，再提供 /home/student/overcloudrc 文件并创建一个新项目。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ openstack project create demoproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 56aba48550da4b2eafad93fc4495a45b |
| name        | demoproject                      |
+-------------+----------------------------------+
```

2. 在 demoproject 中创建用户 demouser。

```
[student@workstation ~]$ openstack user create --project demoproject --password redhat demouser
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| email      | None                             |
| enabled    | True                             |
| id         | a556bbf334874c98a1a083a7dea8efee |
| name       | demouser                         |
| project_id | 56aba48550da4b2eafad93fc4495a45b |
| username   | demouser                         |
+------------+----------------------------------+
```

3. 从 workstation，为 demouser 创建 /home/student/overcloudrc_demouser 文件。该文件应当如下所示：

```
export OS_NO_CACHE=True
export COMPUTE_API_VERSION=1.1
export OS_USERNAME=demouser
export OS_PASSWORD=redhat
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
export OS_TENANT_NAME=demoproject
export OS_CLOUDNAME=overcloud
export NOVA_VERSION=1.1
```

4. 从 workstation，打开一个新终端，再以 stack 用户身份使用 SSH 连接 director 服务器。登录后，提供 stackrc 文件。

```
[student@workstation ~]$ ssh stack@director
[stack@director ~]$ source stackrc
```

5. 运行 openstack server list 命令，检索 overcloud-controller-0 节点的浮动 IP。

```
[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+------------------------+
| ID                                   | Name                    | Status | Networks               |
+--------------------------------------+-------------------------+--------+------------------------+
...Output omitted...                                    ...Output omitted...
| a7bf4726-363d-4303-88e2-2b9573aca5d0 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24 |
+--------------------------------------+-------------------------+--------+------------------------+
```

6. 以 heat-admin 用户身份使用 SSH 连接控制器节点。务必将 IP 替换为上一命令返回的 IP。运行 sudo -i 命令以切换到 root 用户。

```
[stack@director ~]$ ssh heat-admin@172.25.250.24
[heat-admin@overcloud-controller-0 ~]$ sudo -i
[root@overcloud-controller-0 ~]# 
```

7. 编辑/etc/nova/policy.json文件。通过删除 rule:admin_api，更改 "os_compute_api:os-flavor-manage" 策略。该行应当如下所示：

```
"os_compute_api:os-flavor-manage": "",
```

8. 重新启动 openstack-nova-api 服务，再退出控制器节点。

```
[root@overcloud-controller-0 ~]# sudo systemctl restart openstack-nova-api
[root@overcloud-controller-0 ~]# exit
[heat-admin@overcloud-controller-0 ~]$ exit
```

9. 退出 director 节点。

```
[stack@director ~]$ exit
[student@workstation ~]$ 
```

10. 从 workstation 提供 overcloudrc_demouser 文件，以设置 demouser 的环境变量。

```
[student@workstation ~]$ source overcloudrc_demouser
```

11. 创建一个名为 demoflavor 的新专用类别。

```
[student@workstation ~]$ openstack flavor create --private demoflavor --vcpus 1 --ram 1024 --disk 10 --ephemeral 2 --swap 512
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 2                                    |
| disk                       | 10                                   |
| id                         | e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb |
| name                       | demoflavor                           |
| os-flavor-access:is_public | False                                |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 512                                  |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

12. 为 admin 用户提供 overcloudrc 文件。

```
[student@workstation ~]$ source overcloudrc
```
        
13. 检索 demoproject 和 demoflavor 的 ID。

```
[student@workstation ~]$ openstack project list
+----------------------------------+--------------+
| ID                               | Name         |
+----------------------------------+--------------+
...Output omitted...
| 56aba48550da4b2eafad93fc4495a45b | demoproject  |
...Output omitted...
+----------------------------------+--------------+

[student@workstation ~]$ openstack flavor list --private
+--------------------------------------+----------------+------+------+-----------+-------+-----------+
| ID                                   | Name           |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+----------------+------+------+-----------+-------+-----------+
| e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb | demoflavor     | 1024 |   10 |         2 |     1 | False     |
+--------------------------------------+----------------+------+------+-----------+-------+-----------+
```

14. 使用nova命令，将 demoproject 的项目 ID 与 demoflavor 类别的类别 ID 关联。该命令要求将类别 ID 用作第一参数，项目 ID 用作第二参数。

```
[student@workstation ~]$ nova flavor-access-add e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb
            56aba48550da4b2eafad93fc4495a45b
+--------------------------------------+---------------------------------+
| Flavor_ID                            | Tenant_ID                       |
+--------------------------------------+---------------------------------+
| e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb | ab8815a7d462421191afe42e1a6bc4f3|
+--------------------------------------+---------------------------------+
```

### 练习：管理类别访问权限

1. 提供overcloudrc文件以获取 admin 凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 创建 flavor-access 项目。

```
[student@workstation ~]$ openstack project create flavor-access
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 56aba48550da4b2eafad93fc4495a45b |
| name        | flavor-access                    |
+-------------+----------------------------------+
```

3. 在 flavor-access 项目中，创建 user1 用户并使用 redhat 作为密码。使用openstack user create命令。

```
[student@workstation ~]$ openstack user create --project flavor-access --password redhat user1
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| email      | None                             |
| enabled    | True                             |
| id         | a556bbf334874c98a1a083a7dea8efee |
| name       | user1                            |
| project_id | 56aba48550da4b2eafad93fc4495a45b |
| username   | user1                            |
+------------+----------------------------------+
```

4. 从 workstation，以 student 用户身份为 user1 用户创建 overcloudrc 文件，方法是将包含管理员凭据的现有 /home/student/overcloudrc 文件复制到 /home/student/overcloudrc_user1。

[student@workstation ~]$ cp overcloudrc overcloudrc_user1

5. 更新 overcloudrc_user1 文件。为变量 OS_USERNAME 赋予值 user1，变量 OS_TENANT_NAME 赋予值 flavor-access，以及变量 OS_PASSWORD 赋予值 redhat。确保 overcloudrc_user1 包含以下内容：

```
export OS_NO_CACHE=True
export COMPUTE_API_VERSION=1.1
export OS_USERNAME=user1
export no_proxy=,192.168.0.11
export OS_TENANT_NAME=flavor-access
export OS_CLOUDNAME=overcloud
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
export NOVA_VERSION=1.1
export OS_PASSWORD=redhat
```

6. 配置 Nova，以允许非管理员用户创建和管理类别。

    a. 从 workstation，打开一个新终端，再以 stack 用户身份使用 SSH 连接 director.lab.example.com。提供 stackrc 凭据文件，再运行 openstack server list 命令来检索 overcloud-controller-0 节点的公共 IP。

    [student@workstation ~]$ ssh stack@director
    [stack@director ~]$ source stackrc
    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+------------------------+
    | ID                                   | Name                    | Status | Networks               |
    +--------------------------------------+-------------------------+--------+------------------------+
    ...Output omitted...                                                ...Output omitted...
    | a7bf4726-363d-4303-88e2-2b9573aca5d0 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24 |
    +--------------------------------------+-------------------------+--------+------------------------+
    
    b. 使用 SSH 连接 overcloud-controller-0 节点。将 IP 替换为上一命令返回的 IP。

    [stack@director ~]$ ssh heat-admin@172.25.250.24

    c. 提升权限，方法是使用 sudo 命令修改 /etc/nova/policy.json 文件，将 "os_compute_api:os-flavor-manage": "rule:admin_api", 替换为 "os_compute_api:os-flavor-manage": "", 。

    [heat-admin@overcloud-controller-0 ~]$ sudo cat /etc/nova/policy.json
    ...Output omitted...
    "os_compute_api:os-flavor-manage": "",
    ...Output omitted...

    d. 重新启动 openstack-nova-api 服务。

    [heat-admin@overcloud-controller-0 ~]$ sudo systemctl restart openstack-nova-api

7. 从 workstation 提供 /home/student/overcloudrc_user1 文件，以获取 user1 权限。

[student@workstation ~]$ source overcloudrc_user1
[student@workstation ~]$ 

8. 从 workstation 创建一个名为 newflavor 的专用类别，它具有 1 个 VCPU、1024 MB RAM、10 GB root 文件系统磁盘、2 GB 临时磁盘，以及 512  MB 交换内存。

    a. 使用 openstack flavor create 命令，将 newflavor 创建为专用类别。

    [student@workstation ~]$ openstack flavor create --private newflavor --vcpus 1 --ram 1024 --disk 10 --ephemeral 2 --swap 512
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 2                                    |
    | disk                       | 10                                   |
    | id                         | e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb |
    | name                       | newflavor                            |
    | os-flavor-access:is_public | False                                |
    | ram                        | 1024                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       | 512                                  |
    | vcpus                      | 1                                    |
    +----------------------------+--------------------------------------+

9. 配置访问控制，使得类别 newflavor 仅对 flavor-access 项目可见并由该项目使用。为此，请检索 flavor-access 项目和 newflavor 类别的 UUID。

    a. 提供 /home/student/overcloudrc 文件以获取 admin 权限。

    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ 

    b. 使用 openstack project list 命令列出可用的项目，以检索 flavor-access 类别的 UUID。

    [student@workstation ~]$ openstack project list
    +----------------------------------+--------------+
    | ID                               | Name         |
    +----------------------------------+--------------+
    ...Output omitted...
    | 56aba48550da4b2eafad93fc4495a45b | flavor-access|
    ...Output omitted...
    +----------------------------------+--------------+

    c. 使用 openstack flavor list --private 命令列出可用的专用类别，以检索 newflavor 类别的 UUID。

    [student@workstation ~]$ openstack flavor list --private
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+
    | ID                                   | Name           |  RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+
    | e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb | newflavor      | 1024 |   10 |         2 |     1 | False     |
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+

    d. 将类别 newflavor 与项目 flavor-access 关联。为此，可运行nova flavor-access-add FLAVOR ID TENANT ID命令。务必将 UUID 地址替换为上面两个命令返回的 UUID。

    [student@workstation ~]$ nova flavor-access-add e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb 56aba48550da4b2eafad93fc4495a45b
    +--------------------------------------+---------------------------------+
    | Flavor_ID                            | Tenant_ID                       |
    +--------------------------------------+---------------------------------+
    | e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb | 56aba48550da4b2eafad93fc4495a45b|
    +--------------------------------------+---------------------------------+

    e. 提供/home/student/overcloudrc_user1文件以获取 user1 权限。

    [student@workstation ~]$ source overcloudrc_user1
    [student@workstation ~]$ 

    f. 使用openstack flavor list --private命令，列出可用的专用类别。

    [student@workstation ~]$ openstack flavor list --private
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+
    | ID                                   | Name           |  RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+
    | e9c64432-a2b2-4a9e-86d9-72bae2f8d4bb | newflavor      | 1024 |   10 |         2 |     1 | False     |
    +--------------------------------------+----------------+------+------+-----------+-------+-----------+

10. 尝试使用 devtest 用户显示专用类别 newflavor。类别 newflavor 应当不显示出来。

    a. 提供 /home/student/overcloudrc_devtest 文件以获取 devtest 权限。

    [student@workstation ~]$ source overcloudrc_devtest

    b. 使用 openstack flavor list 命令显示专用类别。确保类别 newflavor 不会返回。

    [student@workstation ~]$ openstack flavor list
    +----+-----------+-------+------+-----------+-------+-----------+
    | ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +----+-----------+-------+------+-----------+-------+-----------+
    | 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
    | 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
    | 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
    | 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
    | 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
    +----+-----------+-------+------+-----------+-------+-----------+


### 实验：管理类别

1. 创建一个名为 prodflavor 的新类别，它具有 1 个 VCPU、2048 MB RAM、15 GB root 文件系统磁盘、3 GB 临时磁盘，以及 512 MB 交换内存。

    a. 提供文件overcloudrc，以获取 admin 凭据。

    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ 

    b. 列出当前可用的类别。您应当会看到列出默认类别，以及由实验脚本所创建的名为 prodsecondflavor 的类别。

    [student@workstation ~]$ openstack flavor list
    +--------------------------------------+-----------------+-------+------+-----------+-------+-----------+
    | ID                                   | Name            |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+-----------------+-------+------+-----------+-------+-----------+
    | 09137934-f246-4798-92d0-e0023f0f52c9 | prodsecondflavor |  2048 |   10 |         2 |     2 | True      |
    | 1                                    | m1.tiny         |   512 |    1 |         0 |     1 | True      |
    | 2                                    | m1.small        |  2048 |   20 |         0 |     1 | True      |
    | 3                                    | m1.medium       |  4096 |   40 |         0 |     2 | True      |
    | 4                                    | m1.large        |  8192 |   80 |         0 |     4 | True      |
    | 5                                    | m1.xlarge       | 16384 |  160 |         0 |     8 | True      |
    +--------------------------------------+-----------------+-------+------+-----------+-------+-----------+

    c. 创建一个类别 prodflavor，它具有 1 个 VCPU、2048 MB RAM、15 GB root 文件系统磁盘、3 GB 临时磁盘，以及 512 MB 交换内存。

    [student@workstation ~]$ openstack flavor create --vcpus 1 --ram 2048 --disk 15 --ephemeral 3 --swap 512 prodflavor
    ...Output omitted...

    d. 列出新建的类别 prodflavor。

    [student@workstation ~]$ openstack flavor show prodflavor
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 3                                    |
    | disk                       | 15                                   |
    | id                         | eeca1d0c-86ae-5804-eb68-c564f220c8bf |
    | name                       | prodflavor                            |
    | os-flavor-access:is_public | True                                 |
    | properties                 |                                      |
    | ram                        | 2048                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       | 512                                  |
    | vcpus                      | 1                                    |
    +----------------------------+--------------------------------------+

2. 使用前面创建的类别，创建一个名为 prodvm 的新实例。使用密钥对 prodkeypair、映像 prodimage 和网络 prodnet 来部署该实例。上述所有资源都已由实验脚本创建。

    a. 列出可用的映像。应当会列出映像 prodimage。

    [student@workstation ~]$ openstack image list
    +--------------------------------------+-----------+
    | ID                                   | Name      |
    +--------------------------------------+-----------+
    | 0e6afa93-0a2d-4698-9c0b-3231a352ef47 | prodimage |
    +--------------------------------------+-----------+

    b. 显示映像 prodimage 的详细信息。您需要使用其 UUID 来执行此操作，这已在上一步中获得。检查映像要求；prodflavor 提供的资源应当同时满足 min_disk 和 min_ram 要求。

    [student@workstation ~]$ openstack image show 0e6afa93-0a2d-4698-9c0b-3231a352ef47
    +------------------+------------------------------------------------------+
    | Field            | Value                                                |
    +------------------+------------------------------------------------------+
    ...Output omitted...
    | id               | 0e6afa93-0a2d-4698-9c0b-3231a352ef47                 |
    | min_disk         | 0                                                    |
    | min_ram          | 0                                                    |
    | name             | prodimage                                            |
    ...Output omitted...
    +------------------+------------------------------------------------------+

    c. 列出可用的密钥对。其中应当列有密钥对 prodkeypair。

    [student@workstation ~]$ openstack keypair list
    +-------------+-------------------------------------------------+
    | Name        | Fingerprint                                     |
    +-------------+-------------------------------------------------+
    | prodkeypair | 0f:ec:e8:2f:c8:d0:95:85:ee:7b:f7:cd:6b:91:87:97 |
    +-------------+-------------------------------------------------+

    d. 列出可用的网络。其中应当列有网络 prodnet。

    [student@workstation ~]$ openstack network list
    +--------------------------------------+---------+--------------------------------------+
    | ID                                   | Name    | Subnets                              |
    +--------------------------------------+---------+--------------------------------------+
    ...Output omitted...
    | ea806142-cd19-47e9-812b-9c2753602b51 | prodnet | acecd647-2875-49ef-b10e-ba12a165115b |
    +--------------------------------------+---------+--------------------------------------+

    e. 使用 openstack server create 命令，创建名为 prodvm 的新实例，它使用前面检查过的资源，包括 prodimage 映像、prodkeypair 密钥对、prodnet 网络和 prodflavor 类别。

    [student@workstation ~]$ openstack server create --flavor prodflavor --key-name prodkeypair --image prodimage --nic net-id=prodnet prodvm
    +-------------+---------------------------------------------------+
    | Field       | Value                                             |
    +-------------+---------------------------------------------------+
    ...Output omitted...
    | created     | 2016-01-20T12:50:09Z                              |
    | flavor      | prodflavor (...)                                  |
    | hostId      |                                                   |
    | image       | prodimage (...)                                   |
    | key_name    | prodkeypair                                       |
    | name        | prodvm                                            |
    ...Output omitted...
    +-------------+---------------------------------------------------+

3. 从 ext 浮动 IP 池中分配一个浮动 IP，并将它关联到 prodvm 实例。

    a. 从 ext 池分配一个新的浮动 IP。该浮动 IP 池已由实验脚本配置。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 5dfda71b-c94b-4f31-93e3-7bc08bececfa |
    | instance_id | None                                 |
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    b. 将之前分配的浮动 IP 关联到 prodvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.27 prodvm

4. 检查实例 prodvm 已经使用类别 prodflavor 中详述的规格正确创建好。使用与 prodkeypair 密钥对关联的 /root/prodprivatekey.key 私钥，访问 prodvm 实例。

    a. 使用与 prodkeypair 密钥对关联的 /home/student/lab/prodkeypair.pem 私钥文件，以 cloud-user 用户身份通过之前分配的浮动 IP 登录 prodvm 实例。

    [student@workstation ~]$ ssh -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.27
    [cloud-user@prodvm ~]$ 

    b. 检查 prodvm实例已部署了 1 个 VCPU，如 prodflavor 类别中所配置。

    [cloud-user@prodvm ~]$ lscpu
    ...Output omitted...
    CPU(s):                1
    ...Output omitted...

    c. 检查 prodvm实例已部署了 2048 MB RAM、512 MB 交换内存和 15 GB 根磁盘，如 prodflavor 类别中所配置。

    [cloud-user@prodvm ~]$ free -m
                  total        used        free      shared  buff/cache   available
    Mem:           1840          79        1619          16         141        1617
    Swap:           511           0         511
    [cloud-user@prodvm ~]$ lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    vda    253:0    0   15G  0 disk
    └─vda1 253:1    0   15G  0 part /
    vdb    253:16   0    3G  0 disk /mnt
    vdc    253:32   0  512M  0 disk [SWAP]

    d. 检查 prodvm实例已部署了 3 GB 临时磁盘，prodflavor 类别中所配置。完成时注销。

    [cloud-user@prodvm ~]$ df -h /mnt
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vdb        3.0G  4.0K  3.0G   1% /mnt
    [cloud-user@prodvm ~]$ logout
    Connection to 172.25.250.27 closed.

5. 删除由实验脚本创建的类别 prodsecondflavor。

    a. 删除类别 prodsecondflavor。

    [student@workstation ~]$ openstack flavor delete prodsecondflavor


### 总结

在本章中，您可以学到：

*    类别定义了由实例使用的资源，包括 VCPU 数、内存或磁盘等资源。
*    由实例使用的映像的要求应当与所用类别提供的资源相符。
*    类别只能由具备管理权限的用户创建。
