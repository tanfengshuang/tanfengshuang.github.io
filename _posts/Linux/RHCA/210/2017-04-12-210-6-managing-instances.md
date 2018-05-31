---
layout: post
title:  "管理实例(210-6)"
categories: Linux
tags: RHCA 210
---

### 启动和删除实例

###### 关于实例

在云计算中，实例是在提供计算功能的物理硬件上运行的虚拟资源。实例通常为一组隔离且独立的资源，例如在物理资源上运行的网卡或虚拟磁盘。虚拟服务器可以在私有云（位于组织内部且由其管理的云）或公共云（由外部提供商管理的云）上运行。在云计算中，一个硬件由多个虚拟资源使用，而且因为实施了扩展而具有高度动态性，自动管理资源的使用与分配，因此允许用户根据需要来部署和使用虚拟资源。资源用尽时，用户可以通过部署更多虚拟实例来扩展底层的基础架构。这种实施方式缩短了服务硬件相关的停机时间。此外，实例可以从一台物理计算机轻松地无缝移动到另一台上，将所有数据从一个点传输到另一个点。以下示意图显示了一个提供商的基础架构，它由用于构建公共云的硬件资源组成。用户消耗大量资源，作为其虚拟基础架构的一部分。 

云计算提出了按需使用资源并将一套基础硬件用于多个虚拟资源的模型，制定了一个新的业界标准。这一模型通过将用户资源与基础系统隔离来提高安全性。云计算并不是软件或特定的技术，而是一种方法，实践与实施的标准化，其宗旨皆是为了改善异构硬件之间的兼容性和支持。它提供某种程度的抽象，允许部署可以堆叠并且就绪可用的各种硬件。各家供应商提出的不同云计算实施之间存在相似性：

*    云计算可以划分到三个层级：

        1. 基础架构即服务 (IaaS)：提供商管理基础硬件，用户使用直接利用资源的虚拟资源。
        2. 平台即服务 (PaaS)：提供商管理基础硬件，以及其上运行的虚拟基础架构。用户购买虚拟资源但不管理它们。PaaS 允许公司运行 Web 应用或软件，不必操心资源分配或资源管理。
        3. 软件即服务 (SaaS)：提供商管理一套捆绑的资源，其包含计算资源和应用。向用户提供预配置的解决方案（如客户关系管理，CRM），或云服务；例如，在线存储解决方案通常在 PaaS 基础上运行。

*    资源被划分成定义“为服务”的功能角色，供用户使用；例如，虚拟服务器、虚拟存储或虚拟网络等，它们都符合以上所述的层级。随着云计算机模型的演进，提供的服务也越来越多，如 DNS 即服务 (DNSaaS)、 电子邮件即服务 (EaaS)或 数据库即服务 (DBaaS)。此类功能满足了公司从实际资源抽象出所用服务并利用云计算功能的需求。
*    云计算实施了租户的概念，或者也称为项目，它们是相同物理资源（如网络路由器、系统管理程序或存储服务器）在多个客户之间的逻辑分割。这种实践促进了安全模型的隔离与开发，确保一个客户无法访问另一客户的数据（除非需要）。例如，通常使用 VLAN 来实现网络隔离。
*    基础资源重新分组为资源池。例如，一组系统管理程序属于同一环境的一个部分，由多个 CPU 组成。用户被分配此类资源的一个子集。例如，消费者可以订购具有两个 VCPU 的两台虚拟服务器，分别分配自具有四个 VCPU 的同一系统管理程序。用户将看到两台独立的服务器，但实际上在同一计算机上运行。
*    资源的访问借助于 应用编程接口 (API)。API 是基于 Web 的接口，与软件进行交互。通过 API 通信实现了实践的标准化，资源可以按照规范的方式加以使用和访问。
*    由于使用了 API，因此可以动态插入额外的服务，如记账和监控解决方案。通过 API 进行的每一个调用均可记录下来，满足各种各样的用途。
*    当用户不再需要时，资源可以快速收回到池中，从而提高使用效率。例如，如果用户不再需要某一实例，可以在数秒内将它终止，它所使用的资源便可提供给其他实例。
*    云计算模型提供了某种程度的灵活性，使得用户可以自由地仅使用所需资源。这有助于降低与基础架构相关的成本。
*    云计算模型可以促进市场上的标准化。用户可以轻松将自己的工作负载从一家提供商迁移到另一家。开发商仅需要更新 API 的端点；而对于使用命令行界面的用户，相关的命令也趋向于类似（例如， 服务器创建 MY SERVER 或 实例部署 MY SERVER）。

> 注意: 云计算并非一种解决方案，而是与物理硬件交互的一系列做法、实施、解决方案和标准化。

###### OpenStack 中的实例

云计算中的实例是虚拟资源。在 OpenStack 中，管理虚拟实例的服务是 Nova。Nova 搭载 Swift（对象存储服务），是 OpenStack 中率先实施的项目（在 2010 年发布的 Austin 版本中）。尽管最初仅支持 libvirt（虚拟化 API）和 KVM（开源虚拟化解决方案），它得到了快速演进，已经支持市面上的大多数虚拟化解决方案，不论是专有还是开源。云管理员使用其 API 与 Nova 交互，可以通过其原生应用，或者借助命令行接口。例如，openstack 和 nova 实例是大小不一的虚拟资源，它们需要一种 类别，类别就是分配到实例的资源集合（如磁盘和内存）。要部署实例，需要配置至少一个类别、一个网络和一个映像。默认情况下，Red Hat OpenStack Platform 中已配置了一组类别，但没有配置映像。以下示意图显示了 OpenStack Nova 所包含的各种服务，以及它们如何与物理资源交互来提供给用户使用。通过一套驱动程序编配硬件资源，再通过面向用户的 API 使得这样的资源可被使用。 

> 注意: 要将类别与映像搭配使用，该类别必须满足映像的最低要求，如磁盘和内存的最小大小。

通过 openstack server image create 命令，可以上传要使用的映像来启动实例。此命令与 openstack image create 的作用相同。openstack 命令提供了多个子命令，比如以下所示。

使用 openstack 命令管理实例

*    子命令 	        详情
*    server create 	创建实例
*    server delete 	删除实例
*    server list 	列出实例
*    server show 	显示实例详细信息
*    server set 	设置实例属性（如 root 密码）

通过 openstack server create 命令启动实例部署后，Red Hat OpenStack Platform 的计算服务 Nova 将寻找所含资源足以部署该实例的计算节点，并将它声明为符合创建虚拟服务器的条件。实例要求的资源在使用的类别中指定。选定了计算节点时，映像将传输到此计算节点上。在传输完成后，实例便被启动。 

启动实例时的请求流

*    请求 	                    详情
*    客户端 -> Keystone 	            身份验证请求
*    Keystone -> 令牌存储        	    保存令牌
*    Keystone -> 客户端 	            传递身份验证令牌
*    客户端 -> Nova.api 	            启动实例
*    Nova.api -> 数据库 	            创建实例的初始条目
*    Nova.api -> RabbitMQ 	        发出 rpc.cast 以请求新实例
*    Nova.api -> 客户端 	            实例请求完成
*    Nova.schedular -> RabbitMQ 	    订阅新实例请求
*    Nova.schedular -> 数据库 	    读取筛选和权重信息
*    Nova.schedular -> 数据库 	    读取群集状态
*    Nova.schedular -> 数据库 	    保存实例状态
*    Nova.schedular -> RabbitMQ 	    Rpc.cast 以启动实例
*    Nova-compute -> RabbitMQ 	    订阅新实例请求
*    Nova-compute -> RabbitMQ 	    Rpc.call 到 Nova-conductor 以获取实例信息
*    Nova-conductor -> RabbitMQ 	    订阅新实例请求
*    Nova-conductor -> RabbitMQ 	    读取实例状态
*    Nova-conductor -> RabbitMQ 	    发布新实例状态
*    Nova-compute -> RabbitMQ 	    订阅新实例请求
*    Nova-compute -> Glance.api 	    [REST] 通过映像 ID 从 Glance 获取映像 URI
*    Glance.api -> Nova-compute 	    返回映像 URI
*    Nova-compute -> Ceph_mon 	    检索群集映射
*    Ceph_mon -> Nova-compute 	    返回群集映射
*    Nova-compute -> Ceph_rgw 	    [REST] 请求对象
*    Ceph_rgw -> Ceph_osd 	        [Socket] 获取对象
*    Ceph_rgw -> Nova-compute 	    返回对象
*    Nova-compute -> Neutron-server 	为实例分配和配置网络
*    Neutron-server -> RabbitMQ 	    请求 IP 地址
*    Neutron-server -> RabbitMQ 	    请求 L2 配置
*    Neutron-DHCP-agent -> RabbitMQ 	地区请求 IP 地址
*    Neutron-DHCP-agent -> Dnsmasq 	分配 IP 地址
*    Dnsmasq -> Neutron-DHCP-agent 	答复
*    Neutron-DHCP-agent -> RabbitMQ 	答复 IP 地址
*    Neutron-server -> RabbitMQ 	    读取 IP 地址
*    Neutron-L2-Agent -> RabbitMQ 	读取 L2 配置请求
*    Neutron-L2-Agent -> Libvirt 	配置 L2
*    Neutron-L2-Agent -> RabbitMQ 	答复 L2 配置
*    Neutron-server -> 数据库 	    保存实例网络状态
*    Neutron-server -> Nova-compute 	传递网络信息
*    Nova-compute -> Cinder.api 	    [REST] 获取卷数据
*    Cinder.api -> Keystone 	        验证令牌和权限
*    Keystone -> Cinder.api      	利用角色和 acl 更新身份验证标题
*    Cinder.api -> Nova-compute 	    返回卷信息
*    Nova-compute -> Libvirt 	    启动虚拟机
*    Nova-compute -> Libvirt 	    更新端口信息
*    Nova-compute -> RabbitMQ 	    Rpc.call 到 Nova-conductor 以获取实例信息
*    Nova-conductor -> RabbitMQ 	    订阅新实例请求
*    Nova-conductor -> RabbitMQ 	    发布新实例状态
*    Nova-compute -> Libvirt 	    传递卷信息
*    Libvirt -> Ceph_mon 	        获取群集映射
*    Ceph_mon -> Libvirt 	        返回群集映射
*    Libvirt -> Ceph_osd 	        挂载卷
*    VM-instance -> Neutron_metadata_proxy 	        http rest 169.254.169.254
*    Neutron_metadata_proxy -> Nova-api-metadata 	http rest add uuid into X-headers
*    Neutron_metadata_proxy -> VM-instance 	        返回元数据
*    客户端 -> Nova-api 	            轮询实例状态
*    Nova.api -> 数据库 	            读取实例状态
*    数据库 -> Nova.api 	            返回状态
*    Nova-api -> 客户端 	            返回实例状态

> 注意: 如果未指定安全组，实例将部署在默认安全组中。默认情况下，此安全组将允许实例发出的所有传出流量，也允许来自部署在默认安全组中其他实例的所有传入流量。要启用外部来源（如 SSH）的传入流量，必须向默认安全组添加相应的规则。

一些参数是强制使用的，例如要使用的类别或映像，Nova 假定了一套默认设置。设置包括：

*    安全组 default。
*    如果项目中只有一个专用网络，则 Nova 将实例连接到此网络。

当实例被删除（或 终止）时，将发生以下情况：

*    Nova 指示系统管理程序通过其驱动程序（如 libvirt）关闭实例。
*    创建并分配给实例的磁盘从系统管理程序删除。
*    如果实例关联了 Cinder 卷，它将被分离。
*    如果实例关联了浮动 IP 地址，它将被从实例分离并可用于重新分配。
*    Neutron 释放分配给实例的所有端口。
*    Nova 更新其数据库，以指示该实例不再为活动状态。

### 练习：启动和删除实例

1. 从workstation 打开一个终端，再提供 Keystone overcloud 凭据。

[student@workstation ~]$ source overcloudrc

2. 列出所有可用的密钥对。确保存在密钥 dev_key1。您将在后续步骤中使用此密钥对启动实例。

[student@workstation ~]$ openstack keypair list
+-----------+-------------------------------------------------+
| Name      | Fingerprint                                     |
+-----------+-------------------------------------------------+
| dev_key1  | d4:ee:71:be:2e:e3:49:b4:8a:f8:f9:3f:48:42:0c:22  |
+-----------+-------------------------------------------------+

3. 确保存在映像 small。您将在后续步骤中使用此映像启动实例。

[student@workstation ~]$ openstack image list
+--------------------------------------+-------+
| ID                                   | Name  |
+--------------------------------------+-------+
| 51cd79db-30a1-46e9-95c9-04df6501a286 | small |
+--------------------------------------+-------+

4. 最后，确保存在您要用于实例的类别 m2.small。

[student@workstation ~]$ openstack flavor list
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+
| ID                                   | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+
...Output omitted...
| f6c3b4f7-51cd-4781-9d28-2d506e7281aa | m2.small  |  1024 |   10 |         0 |     2 | True      |
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+

5. 列出可用的网络。应当会显示 dev_pubnet 和 dev_privnet 这两个网络。如前文所述，如果未指定网络且项目中仅存在一个内部网络，实例将连接到该内部网络；本例中为 dev_privnet，因此从该子网的 IP 范围内获取一个 IP 地址。

[student@workstation ~]$ openstack network list
+--------------------------------------+-------------+--------------------------------------+
| ID                                   | Name        | Subnets                              |
+--------------------------------------+-------------+--------------------------------------+
| cafb27eb-fefc-433d-8ede-8e5101b81058 | dev_privnet | d4b2b2e0-4b5e-420c-bcdc-50c43fb5820c |
| 3921448f-aaf5-4926-90d3-d8d384e53da2 | dev_pubnet  | 8f11e6a4-a380-4747-ad1e-0ca1e2edd1fc |
+--------------------------------------+-------------+--------------------------------------+

6. 创建一个名为 dev_instance 的新实例，创建时使用类别 m2.small、密钥对 dev_key1、映像 small 以及专用网络 dev_privnet。

[student@workstation ~]$ openstack server create --flavor m2.small --key-name dev_key1 --image small --nic net-id=dev_privnet dev_instance
+-------------------------------+------------------------------+
| Field                         | Value                        |
+-------------------------------+------------------------------+
| OS-DCF:diskConfig             | MANUAL                       |
| OS-EXT-AZ:availability_zone   |                              |
| OS-EXT-SRV-ATTR:host          | None                         |
| OS-EXT-SRV-ATTR:instance_name | instance-00000029            |
| OS-EXT-STS:power_state        | 0                            |
| OS-EXT-STS:task_state         | scheduling                   |
| OS-EXT-STS:vm_state           | building                     |
...Output omitted...
+-------------------------------+------------------------------+

7. 使用命令 openstack，检查该实例的状态。等待 Status 过渡到 ACTIVE。

[student@workstation ~]$ openstack server list
+--------------------------------------+--------------+--------+------------+-------------+--------------------------+
| ID                                   | Name         | Status | Task State | Power State | Networks                 |
+--------------------------------------+--------------+--------+------------+-------------+--------------------------+
...Output omitted...                                           ...Output omitted...
| 2fd92399-b563-4a98-beba-4ab7915fbd78 | dev_instance | ACTIVE | -          | Running     | dev_privnet=192.168.1.17 |
+--------------------------------------+--------------+--------+------------+-------------+--------------------------+

8. 在默认浮动 IP 池 dev_pubnet 中创建一个新的浮动 IP。

[student@workstation ~]$ openstack ip floating create dev_pubnet
+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| fixed_ip    | None                                 |
| id          | b305e18d-dab5-4edf-a37c-1fcd874fe376 |
| instance_id | None                                 |
| ip          | 172.25.250.51                        |
| pool        | dev_pubnet                           |
+-------------+--------------------------------------+

9. 将此浮动 IP 地址关联到实例 dev_instance。

[student@workstation ~]$ openstack ip floating add 172.25.250.51 dev_instance

> 注意: 该命令不会产生任何输出。

10. 验证该浮动 IP 地址现已关联到实例 dev_instance。

[student@workstation ~]$ openstack server list
+--------------------------------------+---------------+--------+-----------------------------------------+
| ID                                   | Name          | Status | Networks                                |
+--------------------------------------+---------------+--------+-----------------------------------------+
...Output omitted...                                   ...Output omitted...
| 2fd92399-b563-4a98-beba-4ab7915fbd78 | dev_instance  | ACTIVE | dev_privnet=192.168.1.17, 172.25.250.51 |
+--------------------------------------+---------------+--------+-----------------------------------------+

11. 删除 dev_removeme 实例。

    a. 设置脚本创建了实例 dev_removeme。使用命令 openstack，确保存在该实例。

    [student@workstation ~]$ openstack server list
    +--------------------------------------+--------------+--------+-------------------------+
    | ID                                   | Name         | Status | Networks                |
    +--------------------------------------+--------------+--------+-------------------------+
    ...Output omitted...                                  ...Output omitted...
    | 76447251-ae31-48e9-ba5b-4d927ee571e5 | dev_removeme | ACTIVE | dev_privnet=192.168.1.16|
    +--------------------------------------+--------------+--------+-------------------------+

    b. 移除实例。

    [student@workstation ~]$ openstack server delete dev_removeme

    > 注意: 该命令不会产生任何输出。

    c. 使用命令 openstack，确保该实例已被移除。

    [student@workstation ~]$ openstack server list


### 自定义实例

###### 自定义实例

在操作实例时，用户可以利用模板（即预先自定义的映像）或者使用 cloud-init。两种方式都可根据需要用于自定义实例。自定义包括：

*    创建用户
*    安装软件包
*    配置服务

以及各种其他自定义，其方式与用户登录实例并执行手动操作一样。虽然可以利用映像模板来加快调配，云用户有时可能想要动态自定义实例，而不必去修改基础映像。为此，可以使用 cloud-init。cloud-init 是处理实例初期初始化的一款开源软件。安装此软件后，管理员可将它用于：

*    设定默认的区域设置。
*    更新实例主机名。
*    生成或注入 SSH 私钥。
*    设置临时挂载点。

cloud-init 可以通过 user-data调用。user-data 是用户提供的数据，在实例启动时由 cloud-init 读取并解析，从而对实例进行自定义。OpenStack 通过 cloud-init 实施实例管理。user-data 脚本可以从 CLI 或 Horizon 调用。以下是从 CLI 使用 user-data 参数的示例。

```
[root@demo ~]# openstack server create --flavor m2.small --key-name demo_key1 --nic net-id=demo_privnet --user-data /home/student/demo/user-data --image small demo_instance
```

在使用 Horizon 控制面板操作时，用户可以生成实例，再使用 Post-Creation 选项卡来指定要应用到实例的自定义。以下屏幕截图演示了 cloud-init 和 Horizon 的实施，它允许用户在引导时自定义实例。 

OpenStack 随后将信息转换为可由 cloud-init 读取的格式。下方流程图中演示了用户数据如何从 Horizon 中的创建后步骤传递到使用 user-data 指令的实际实例自定义中。

cloud-init 支持多种格式：

*    格式 	                描述
*    使用 gzip 压缩的数据 	    先解压缩数据，然后读取。管理员可以使用 gzip 压缩的数据来发送大小超过 16384 字节（user-data 大小限制）的信息。
*    MIME 多部件存档 	        管理员可以使用多部件文件来指定多种类型的数据。例如，可以同时指定用户数据脚本和 cloud-config 类型。
*    user-data 脚本。 	    开头为 #! 或 Content-Type: text/x-shellscript 的脚本。该脚本将于实例第一次引导期间在 rc.local 级别执行。
*    include 文件 	        开头为 #include 或 Content-Type: text/x-include-url 的数据声明。此声明指定要包含的文件。文件中包含 URL 列表，每行一个。每一个 URL 都将被读取，其内容将通过同一组规则进行传递；即，从 URL 读取的内容可以是 gzip 压缩数据、MIME 多部件或纯文本。
*    cloud-config 数据 	    数据声明的开头必须为 #cloud-config 或 Content-Type: text/cloud-config。
*    upstart 作业 	        数据声明的开头必须为 #upstart-job 或 Content-Type: text/upstart-job。该内容将放入 /etc/init 下的文件中，与任何其他的 upstart 脚本一起供 upstart 使用。
*    部件处理程序 	        数据声明的开头必须为 #part-handler 或 Content-Type: text/part-handler。该数据将根据其文件名写入到 /var/lib/cloud/data 下的文件中。此处理程序必须用 Python 编写。

###### user-data 脚本

user-data scripts 提供了一种便捷的方式，以供在实例创建时向实例发送指令集合。该脚本将在 rc.local 级别调用，这是最后一个级别。下列脚本将使用 Bash 脚本更改当日消息：

```
#!/bin/bash
echo "This instance has been customized by cloud-init at $(date -R)!" >> /etc/motd
```

管理员可以使用 user-data 脚本安装新的软件包，更改系统设置，更新系统，以及确保文件存在等。在使用命令行生成实例时，user-data 脚本利用 --user-data USER-DATA 参数进行传递。

###### cloud-config

尽管管理员可以使用脚本来自定义实例， cloud-config 语法包含了各种可以使用的指令。借助 cloud-config 语法，管理员可以用户友好的格式指定指令。此类指令包括：

*    在第一次引导时使用 Yum 更新系统。
*    添加新的 Yum 存储库。
*    更新现有的 Yum 存储库。
*    导入 SSH 密钥。
*    创建用户。

> 警告: 文件必须使用 YAML 语法，以便它能够被 cloud-init 解析并执行。请注意文本前面的空格数；此处只能有一个空格。

文件必须使用有效的 YAML 语法。下例演示如何通过添加新的用户并运行一组命令来自定义系统。

```
#cloud-config
groups:
  - cloud-users: [john,doe]

users:
  - default
  - name: barfoo
    gecos: Bar B. Foo
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    ssh-import-id: None
    lock-passwd: true
    ssh-authorized-keys:
      - <SSH public key>

runcmd:
 - [ wget, "http://materials.example.com", -O, /tmp/index.html ]
 - [ sh, -xc, "echo $(date) ': hello world!'" ]
```

> 警告: 此文件需要值的具体位置。请注意键之前有一个空格。如果参数之前未插入空格，将无法解译该指令。


### 练习：自定义实例

1. 从workstation打开一个终端，再提供 Keystone overcloud 凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 将目录更改为/home/student/dev。常见 user-data 文件，取名为 dev_customization。该文件应当如下所示：

```
#!/bin/bash
yum -y install httpd
systemctl start httpd
systemctl enable httpd
echo "This instance has been customized by cloud-init at $(date -R)!" >> /etc/motd
```

此 user-data 脚本将安装 httpd 软件包，并且运行一组命令以启动并启用 Web 服务器。它还将自定义当日消息。

3. 设置脚本创建了一个可以生成实例的基础环境。与上一节中使用的相同，专用网络名为 dev_privnet，私钥是 dev_key1；要使用的类别则是 m2.small 类别。最后，浮动 IP 池已命名为 dev_pubnet。在环境中启动名为 dev_custom-instance 的实例，再检查它是否已正确过渡到 ACTIVE 状态。使用 --user-data 传递用户数据文件 /home/student/dev/dev_customization。

    a. 设置脚本创建了一个可以生成实例的基础环境。与上一节中使用的相同，专用网络名为 dev_privnet，私钥是 dev_key1；要使用的类别则是 m2.small 类别，映像是 small。最后，使用 --user-data 传递用户数据文件 /home/student/dev/dev_customization。启动名为 dev_custom-instance 的实例。

    [student@workstation dev]$ openstack server create --flavor m2.small \ 
    > --nic net-id=dev_privnet \ 
    > --key-name dev_key1 \
    > --image small \
    > --user-data /home/student/dev/dev_customization \
    > dev_custom-instance
    +-------------------------------+-------------------------------+
    | Field                         | Value                         |
    +-------------------------------+-------------------------------+
    | OS-DCF:diskConfig             | MANUAL                        |
    | OS-EXT-AZ:availability_zone   |                               |
    | OS-EXT-SRV-ATTR:host          | None                          |
    | OS-EXT-SRV-ATTR:instance_name | instance-0000002b             |
    | OS-EXT-STS:power_state        | 0                             |
    | OS-EXT-STS:task_state         | scheduling                    |
    | OS-EXT-STS:vm_state           | building                      |
    ...Output omitted...
    +-------------------------------+-------------------------------+

    b. 使用 openstack 命令，检查该实例的状态。等待 Status 过渡到 ACTIVE。

    [student@workstation dev]$ openstack server list
    +--------------------------------------+---------------------+--------+------------+-------------+--------------------------+
    | ID                                   | Name                | Status | Task State | Power State | Networks                 |
    +--------------------------------------+---------------------+--------+------------+-------------+--------------------------+
    | 2fd92399-b563-4a98-beba-4ab7915fbd78 | dev_custom-instance | ACTIVE | -          | Running     | dev_privnet=192.168.1.16 |
    +--------------------------------------+---------------------+--------+------------+-------------+--------------------------+
    
4. 从默认浮动 IP 池 dev_pubnet 创建一个新的浮动 IP，再将它关联到 dev_custom-instance 实例。确保浮动 IP 已正确关联到实例。

    a. 从默认浮动 IP 池 dev_pubnet 中创建一个新的浮动 IP。

    [student@workstation dev]$ openstack ip floating create dev_pubnet
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 62be57e1-6fe6-47bb-b3d4-8d7c407fb5bc |
    | instance_id | None                                 |
    | ip          | 172.25.250.51                        |
    | pool        | dev_pubnet                           |
    +-------------+--------------------------------------+

    b. 将之前创建的浮动 IP 关联到 dev_custom-instance 实例。务必将浮动 IP 地址替换为环境中所分配的地址。

    [student@workstation dev]$ openstack ip floating add 172.25.250.51 dev_custom-instance

    c. 确保 dev_custom-instance 实例已关联有浮动 IP。

    [student@workstation dev]# openstack server list
    +--------------------------------------+---------------------+--------+-----------------------------------------+
    | ID                                   | Name                | Status | Networks                                |
    +--------------------------------------+---------------------+--------+-----------------------------------------+
    | 2fd92399-b563-4a98-beba-4ab7915fbd78 | dev_custom-instance | ACTIVE | dev_privnet=192.168.1.16, 172.25.250.51 |
    +--------------------------------------+---------------------+--------+-----------------------------------------+

    > 注意: 下一节中将验证 user-data 脚本执行的自定义。
    

### 验证实例自定义

实例生成之后，云用户可以登录该实例，并验证其自定义已成功应用。在上一节中，实例已通过 user-data 文件进行了自定义。user-data 文件中包含安装和启动 httpd 服务以及自定义 Message of The Day (MOTD) 的指令。

此外，cloud-init 日志文件位于 /var/log/cloud-init.log 目录中，包含关于相关服务所执行的任务的信息。若出现失败，管理员可以检查该文件以进行故障排除。例如，这有助于确定服务未按指示启动的原因，或者 cloud-init 未能安装某一软件包的原因。该文件记录 cloud-init 执行的每一个操作，同时还记录相关服务执行的所有命令的输出。下列屏幕显示了日志文件在各个阶段上的信息。

```
Jan 24 19:15:42 localhost cloud-init: Cloud-init v. 0.7.6 running 'init-local' at Mon, 25 Jan 2016 00:15:42 +0000. Up 5.40 seconds.1
...Output omitted...
Feb  8 22:16:32 localhost cloud-init: ---> Package httpd.x86_64 0:2.4.6-40.el7 will be installed2
...Output omitted...
Feb  8 22:16:44 localhost cloud-init: Cloud-init v. 0.7.6 finished at Tue, 09 Feb 2016 03:16:44 +0000. Datasource DataSourceOpenStack [net,ver=2].  Up 32.53 seconds3
```

1. 开始 cloud-init 引导序列。
2. 安装 httpd 软件包。
3. 结束 cloud-init 引导序列。


### 练习：验证实例自定义

1. 从workstation打开一个终端，再提供 Keystone overcloud 凭据。

[student@workstation ~]$ source overcloudrc

2. 使用 openstack 命令，确保上一节中创建的实例 dev_custom-instance 依然在运行中。

[student@workstation ~]$ openstack server list
+--------------------------------------+---------------------+--------+-----------------------------------------+
| ID                                   | Name                | Status | Networks                                |
+--------------------------------------+---------------------+--------+-----------------------------------------+
| 2fd92399-b563-4a98-beba-4ab7915fbd78 | dev_custom-instance | ACTIVE | dev_privnet=192.168.1.16, 172.25.250.51 |
+--------------------------------------+---------------------+--------+-----------------------------------------+

3. 设置脚本创建了安全规则，可以使用 ping 和 SSH 访问该实例。使用 ping，确认可通过其公共 IP 访问该实例。在前面的输出中，其公共 IP 为 172.25.250.51。务必替换为其公共 IP，即 172.25.250.0/24 范围中的 IP。

```
[student@workstation ~]$ ping -c 3 172.25.250.51
PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.71 ms
64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=1.10 ms
64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.729 ms

--- 172.25.250.51 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.729/1.183/1.718/0.409 ms
```

4. 使用 systemctl 和 curl，确保 httpd 服务正在运行并已在 dev_custom-instance 实例中激活。

    a. 使用位于 /home/student/dev/dev_key1.pem 中的私钥，以 cloud-user 用户身份发起与实例的 SSH 连接。

    [student@workstation ~]$ ssh -i dev/dev_key1.pem cloud-user@172.25.250.51
    This instance has been customized by cloud-init at Fri, 05 Feb 2016 21:45:31 -0800!
    [cloud-user@dev-custom-instance ~]$ 

    如您所见，Message of The Day 已经使用 user-data 脚本中包含的信息进行了更新。

    b. 使用 systemctl，确保 httpd 服务正在运行并已激活。完成时注销。

    [cloud-user@dev-custom-instance ~]$ sudo systemctl status httpd.service
    ● httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
       Active: active (running)since Sat 2016-02-06 00:46:45 EST; 2 days ago
    ....
    [cloud-user@dev-custom-instance ~]$ logout
    Connection to 172.25.250.51 closed.

    c. 使用 curl，检查 httpd 服务可从 workstation 使用。该调用应当返回默认 Web 页面的 HTML 代码。

    [student@workstation ~]$ curl http://172.25.250.51
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
      <head>
        <title>Test Page for the Apache HTTP Server on Red Hat Enterprise Linux</title>
    ...Output omitted...
    
    
### 实验：管理实例

在本实验中，您将在 OpenStack 环境中启动实例。您将检查现有的配置，如网络环境。您将安装一些软件包来自定义实例。最后，您将关联一个浮动 IP 到实例，以便登录实例并确保自定义已成功应用。该环境将提供有下列资源：

*    Keystone 凭据文件，位于/home/student/overcloudrc下。
*    Glance 映像 small。
*    Nova 类别 m2.small。
*    公共网络 lab_pubnet。
*    专用网络 lab_privnet。
*    专用子网 lab_pubsub，位于 172.25.250.0/24 范围内。该网络用于提供外部连接，也是浮动 IP 的池。
*    专用子网 lab_privsub，位于 192.168.1.0/24 范围内。该网络运行供实例使用的 DHCP 服务器。
*    密钥对 lab_key1。其私钥将位于 /home/student/lab/lab_key1.pem 下。

本实验的工作目录是 /home/student/lab/。 

1. 收集环境的相关信息

从workstation.lab.example.com检查环境。确保密钥对、网络、映像和类别都已存在。

    a. 从workstation打开一个终端，再提供 overcloud 凭据。

    [student@workstation ~]$ source overcloudrc

    b. 列出所有可用的密钥对。确保存在密钥对 lab_key1。

    [student@workstation ~]$ openstack keypair list
    +----------+-------------------------------------------------+
    | Name     | Fingerprint                                     |
    +----------+-------------------------------------------------+
    | lab_key1 | d4:ee:71:be:2e:e3:49:b4:8a:f8:f9:3f:48:42:0c:22 |
    +----------+-------------------------------------------------+

    c. 检查与 lab_key1 密钥对关联且位于 /home/student/lab/lab_key1.pem 的私钥已经存在。

    [student@workstation ~]$ cat /home/student/lab/lab_key1.pem
    -----BEGIN RSA PRIVATE KEY-----
    ...Output omitted...
    -----END RSA PRIVATE KEY-----

    d. 列出可用的网络。确保专用网络 lab_privnet 和公共网络 lab_pubnet 都存在。

    [student@workstation ~]$ openstack network list
    +--------------------------------------+-------------+--------------------------------------+
    | ID                                   | Name        | Subnets                              |
    +--------------------------------------+-------------+--------------------------------------+
    | cafb27eb-fefc-433d-8ede-8e5101b81058 | lab_privnet | d4b2b2e0-4b5e-420c-bcdc-50c43fb5820c |
    | 3921448f-aaf5-4926-90d3-d8d384e53da2 | lab_pubnet  | 8f11e6a4-a380-4747-ad1e-0ca1e2edd1fc |
    +--------------------------------------+-------------+--------------------------------------+

    e. 确保存在映像 small。

    [studen@workstation ~]$ openstack image list
    +--------------------------------------+-------+
    | ID                                   | Name  |
    +--------------------------------------+-------+
    | 51cd79db-30a1-46e9-95c9-04df6501a286 | small |
    +--------------------------------------+-------+

    f. 最后，确保存在您要用于实例的类别 m2.small。

    [student@workstation ~]$ openstack flavor list
    +--------------------------------------+-----------+-------+------+-----------+-------+-----------+
    | ID                                   | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
    +--------------------------------------+-----------+-------+------+-----------+-------+-----------+
    ...Output omitted...
    | f6c3b4f7-51cd-4781-9d28-2d506e7281aa | m2.small  |  1024 |   10 |         0 |     2 | True      |
    +--------------------------------------+-----------+-------+------+-----------+-------+-----------+

2. 创建用户文件

创建 web-db-server 文件，其中包含安装 httpd 软件包和 mariadb-server 软件包的指令。在 /home/student/lab/ 目录中创建 web-db-server 文件。

    a. 创建 web-db-server 文件，它将安装 httpd 软件包和 mariadb-server 软件包。将文件保存到 /home/student/lab/web-db-server 下。该文件应当包含以下内容：

    #cloud-config
    packages:
     - httpd
     - mariadb-server
    runcmd:
     - [ systemctl, enable, httpd.service ]
     - [ systemctl, start, httpd.service ]
     - [ systemctl, enable, mariadb-server.service ]
     - [ systemctl, start, mariadb-server.service ]

3. 启动实例

创建好自定义文件后，使用命令行在您的环境中启动实例。使用 m2.small 类别、small 映像、lab_privnet 网络和 lab_key1 密钥对，并将该自定义文件用作参数。web-db-server 文件将使用 user-data 参数进行传递。

    a. 启动名为 lab_instance 的实例。将 m2.small 用作类别，lab_key1 用作密钥对，lab_privnet 用作网络，并将 small 用作映像。包含 /home/student/lab/web-db-server 文件，以便 user-data 脚本自定义实例。

    [student@workstation lab]$ openstack server create --flavor m2.small \
    > --key-name lab_key1 --nic net-id=lab_privnet \
    > --user-data /home/student/lab/web-db-server --image small lab_instance
                  
    +-------------------------------+-------------------------------+
    | Field                         | Value                         |
    +-------------------------------+-------------------------------+
    | OS-DCF:diskConfig             | MANUAL                        |
    | OS-EXT-AZ:availability_zone   |                               |
    | OS-EXT-SRV-ATTR:host          | None                          |
    | OS-EXT-SRV-ATTR:instance_name | instance-0000001e             |
    | OS-EXT-STS:power_state        | 0                             |
    | OS-EXT-STS:task_state         | scheduling                    |
    | OS-EXT-STS:vm_state           | building                      |
    ...Output omitted...
    +-------------------------------+-------------------------------+

    b. 检查实例的状态。等待 Status 过渡到 ACTIVE。需要稍等片刻，它才会过渡到此状态。

    [student@workstation lab]$ openstack server list
    +--------------------------------------+--------------+--------+------------+-------------+--------------------------+
    | ID                                   | Name         | Status | Task State | Power State | Networks                 |
    +--------------------------------------+--------------+--------+------------+-------------+--------------------------+
    | 2fd92399-b563-4a98-beba-4ab7915fbd78 | lab_instance | ACTIVE | -          | Running     | lab_privnet=192.168.1.17 |
    ...Output omitted...                                           ...Output omitted...
    +--------------------------------------+--------------+--------+------------+-------------+--------------------------+

4. 关联浮动 IP

从 lab_pubnet 池创建一个浮动 IP，并将该浮动 IP 关联到实例。验证浮动 IP 地址已正确关联到实例。

    a. 从默认浮动 IP 池 lab_pubnet 中创建一个新的浮动 IP。

    [student@workstation lab]$ openstack ip floating create lab_pubnet
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | b305e18d-dab5-4edf-a37c-1fcd874fe376 |
    | instance_id | None                                 |
    | ip          | 172.25.250.51                        |
    | pool        | lab_pubnet                           |
    +-------------+--------------------------------------+

    b. 将之前创建的浮动 IP 关联到 lab_instance 实例。

    [student@workstation lab]$ openstack ip floating add 172.25.250.51 lab_instance

    c. 确保上一节中创建的 lab_instance 实例已关联有浮动 IP。

    [student@workstation lab]# openstack server list
    +--------------------------------------+---------------------+--------+-----------------------------------------+
    | ID                                   | Name                | Status | Networks                                |
    +--------------------------------------+---------------------+--------+-----------------------------------------+
    | 2fd92399-b563-4a98-beba-4ab7915fbd78 | lab_instance        | ACTIVE | lab_privnet=192.168.1.17, 172.25.250.51 |
    ...Output omitted...                                         ...Output omitted...
    +--------------------------------------+---------------------+--------+-----------------------------------------+

5. 访问实例并验证自定义

发起与实例的 SSH 连接，确保软件包已正确安装。

    a. 使用与密钥对 lab_key1 关联并存储在 /home/student/lab/lab_key1.pem 下的私钥，通过浮动 IP 172.25.250.51 发起与实例的 SSH 连接。

    [student@workstation lab]$ ssh -i lab_key1.pem cloud-user@172.25.250.51
    [cloud-user@lab_instance ~]$ 

    b. 使用 yum，确保 httpd 和 mariadb-server 软件包已经安装成功，然后从实例注销。

    [cloud-user@lab_instance ~]$ yum list installed | grep httpd
    httpd.x86_64               2.4.6-40.el7      @rhel_dvd
    httpd-tools.x86_64         2.4.6-40.el7      @rhel_dvd
    [cloud-user@lab_instance ~]$ yum list installed | grep mariadb-server
    mariadb-server.x86_64      1:5.5.47-1.el7_2  @errata
    [cloud-user@lab_instance ~]$ exit

6. 移除实例

设置脚本创建了实例 lab_removeme。使用 openstack 命令，删除该实例。

    a. 确保实例存在。

    [student@workstation lab]$ openstack server list
    +--------------------------------------+--------------+--------+--------------------------+
    | ID                                   | Name         | Status | Networks                 |
    +--------------------------------------+--------------+--------+--------------------------+
    ...Output omitted...                                  ...Output omitted...
    | 76447251-ae31-48e9-ba5b-4d927ee571e5 | lab_removeme | ACTIVE | lab_privnet=192.168.1.16 |
    +--------------------------------------+--------------+--------+--------------------------+

    b. 移除实例。

    [student@workstation lab]$ openstack server delete lab_removeme

    c. 确保实例已被删除。

    [student@workstation lab]$ openstack server list

### 总结

在本章中，您可以学到：

*    在云计算中，实例是在提供计算功能的物理硬件上运行的虚拟资源。
*    实例通常为一组隔离且独立的资源，例如在物理资源上运行的网卡或虚拟磁盘。
*    在操作实例时，管理员可以利用模板（即预先自定义的映像）或者使用 cloud-init。两种方式都可根据需要用于自定义实例。
*    cloud-init 支持各种不同格式，如 gzip 压缩的数据、user-data 脚本或 cloud-config 脚本。
*    实例生成之后，云用户可以登录该实例，并验证其自定义已成功应用。
*    cloud-init 日志文件位于 /var/log/cloud-init.log 目录中，包含关于相关服务所执行的任务的信息。若出现失败，管理员可以检查该文件以进行故障排除。
