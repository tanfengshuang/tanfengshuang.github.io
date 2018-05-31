---
layout: post
title:  "管理安全组(210-5)"
categories: Linux
tags: RHCA 210
---

### 管理安全组和规则

###### 管理安全组

在任何云环境中，无论是混合云、私有云还是公共云，安全性都极为重要。红帽 OpenStack 平台提供了相关的功能，管理员可以借助实施规则组来控制传入和传出流量的访问。这些规则组被称为安全组。一个安全组含有多条网络访问规则，它们用于限制不同类型的流量对虚拟机实例的访问。实例在启动时可被分配一个或多个安全组。如果云管理员不希望创建新的安全组，可以向实例分配默认安全组。如果安全组指定为规则的来源或目的地，该规则将影响与安全组关联的所有实例。可以在新创建的安全组中添加或移除规则，也可以修改默认安全组中的规则。安全组规则通过来源 IP 前缀、目的地 IP 前缀、IP 协议、L4 来源端口、L4 目的地端口、ICMP 类型、ICMP 代码和方向识别流量。源自虚拟机的流量通过对照出口规则进行检查，而以虚拟机为目的地的流量则通过对照入口规则进行检查。下方列出其行为...

1. 每一项目定义有一个默认安全组

*    每一默认安全组具允许与该默认安全组关联的主机之间通信的规则。
*    来自默认安全组外部的入口流量将被丢弃。
*    出口流量和主机之间相互的通信与默认安全组关联。

2. 入口流量以实例为目的地

*    如果未定义规则，则所有流量将被丢弃。
*    只有符合入口安全组规则的流量才被允许。

3. 出口流量从实例传出

*    只有符合出口安全组规则的流量才被允许。
*    如果未定义规则，则所有出口流量将被丢弃。
*    创建新的安全组时，将自动添加出口流量规则。


安全组将分配有 Neutron 网络端口。Neutron 网络端口与虚拟机实例绑定。安全组规则将丢弃任何未明确允许的转发流量，无论流量的方向是前往还是来自虚拟机。对于返回流量，不需要与安全组规则相符。专用于反欺骗的规则将自动应用到面向虚拟机的网络端口。创建端口时，其 MAC 和 IP 地址由 Neutron 选定。下表中列出了数据包在从虚拟机发出时如何被防火墙处理。

*    欺骗检查：如果数据包的来源 MAC 和 IP 地址不匹配虚拟机网络端口创建之时为虚拟机预留的地址，则丢弃该数据包。
*    如果数据包是允许发往虚拟机的答复流量，则允许该数据包。
*    如果流量与分配给虚拟机网络端口的安全组中任何出站规则匹配，则允许该数据包。
*    如果数据包没有在第 2 或第 3 步中获得许可，它将被丢弃。

借助 Nova CLI 和 secgroup-add-group-rule 子命令，可以在与安全组关联的规则中指定端口范围。在下例中，端口 20 到 22 可以被属于 demosg 安全组的任何实例访问。

```
      [student@workstation ~]$ nova secgroup-add-group-rule demosg 0.0.0.0/0 tcp 20 22
```

openstack命令使用端口:端口表示法来显示安全组规则的端口范围。它显示该规则所应用到的端口范围。类似地，novaCLI 可以显示来源端口和目标端口字段，来指明规则端口范围的首个端口和末尾端口。

使用openstack命令进行安全组管理

*    子命令 	                        详情
*    security group create 	        创建安全组
*    security group delete 	        删除安全组
*    security group list 	        列出安全组
*    security group rule create 	创建安全组规则
*    security group rule delete 	删除安全组规则
*    security group rule list 	    列出安全组规则

在部署时，实例可以添加到某一安全组中，但其生命周期的任意时点上都可以更改与实例关联的安全组。如果实例创建之时没有指定安全组，它将与默认安全组关联。 
    

### 练习：管理安全组和规则

1. 在workstation上，通过提供文件/home/student/overcloudrc获取 admin 凭据。

    a. 提供文件overcloudrc，以获取 admin 凭据。

    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ 

2. 新建一个安全组，取名为 devsecgroup。列出新安全组中包含的规则（应该为空）。

    a. 新建一个安全组，取名为 devsecgroup。使用openstack security group create命令。

    [student@workstation ~]$ openstack security group create devsecgroup
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | devsecgroup                          |
    | id          | 4cf0b831-7a2c-4814-bd8a-fe771ecc392c |
    | name        | devsecgroup                          |
    | rules       | []                                   |
    | tenant_id   | 18b9750851884544b527a0afacee02f5     |
    +-------------+--------------------------------------+

    b. 检查 devsecgroup 是否未关联有任何规则。使用openstack security group rule list命令。

    [student@workstation ~]$ openstack security group rule list devsecgroup

3. 在 devsecgroup 安全组中创建两条新规则，一条允许从任何位置到 TCP/22 端口的入口流量，另一条允许从与 devsecondsecgroup 安全组关联的实例到 TCP/80 端口的入口流量。列出该安全组中的规则，确保正确添加了规则。

    a. 在 devsecgroup 安全组内创建一条规则，它使用 tcp 作为协议、0.0.0.0/0 作为来源 IP 地址，以及端口 22 作为目的地端口。使用openstack security group rule create命令。

    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 22 devsecgroup
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | cc34f92d-c298-4fca-9daa-88ae554da895 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 4cf0b831-7a2c-4814-bd8a-fe771ecc392c |
    | port_range      | 22:22                                |
    +-----------------+--------------------------------------+

    b. 在 devsecgroup 安全组内创建另一条规则，它使用 tcp 作为协议、devsecondsecgroup 作为来源安全组，以及端口 80 作为目的地端口。由于 openstack CLI 不支持在创建安全组时将另一个安全组用作来源组，因此必须使用 nova CLI。

    [student@workstation ~]$ nova secgroup-add-group-rule devsecgroup devsecondsecgroup tcp 80 80
    +-------------+-----------+---------+----------+-------------------------------+
    | IP Protocol | From Port | To Port | IP Range |         Source Group          |
    +-------------+-----------+---------+----------+-------------------------------+
    | tcp         | 80        | 80      |          | devsecondsecgroup             |
    +-------------+-----------+---------+----------+-------------------------------+

    c. 检查 devsecgroup 安全组是否关联了上述规则。

    [student@workstation ~]$ openstack security group rule list devsecgroup
    +--------------------------------------+-------------+-----------+------------+
    | ID                                   | IP Protocol | IP Range  | Port Range |
    +--------------------------------------+-------------+-----------+------------+
    | cc34f92d-c298-4fca-9daa-88ae554da895 | tcp         | 0.0.0.0/0 | 22:22      |
    | 9daa88ae-554d-a895-cc34-222dc2984fca | tcp         |           | 80:80      |
    +--------------------------------------+-------------+-----------+------------+

4. 删除与 devsecondsecgroup 安全组关联的规则。在删除此规则之前和之后，列出安全组规则，以确保它已被移除。

    a. 检查与 devsecondsecgroup 安全组关联的规则。

    [student@workstation ~]$ openstack security group rule list devsecondsecgroup
    +--------------------------------------+-------------+-----------+------------+
    | ID                                   | IP Protocol | IP Range  | Port Range |
    +--------------------------------------+-------------+-----------+------------+
    | 969d83f4-9a7d-44aa-a4d6-ec6f848c96f1 | tcp         | 0.0.0.0/0 | 80:80      |
    +--------------------------------------+-------------+-----------+------------+

    b. 使用其关联的 UUID，从 devsecondsecgroup 安全组移除上述规则。使用openstack security group rule delete命令。

    [student@workstation ~]$ openstack security group rule delete 969d83f4-9a7d-44aa-a4d6-ec6f848c96f1

    c. 检查 devsecondsecgroup 是否未关联有任何规则。使用openstack security group rule list命令。

    [student@workstation ~]$ openstack security group rule list devsecondsecgroup

5. 删除 devthirdsecgroup 安全组。

    a. 删除 devthirdsecgroup 安全组。使用openstack security group delete命令。

    [student@workstation ~]$ openstack security group delete devthirdsecgroup


### 验证安全组规则

###### 验证安全组

在验证安全组的部署是否正确时，造成它不能被访问的原因有许多。安全组是根本原因分析中扮演重要角色的一大元素。如果在访问实例时遇到问题，则要确保在实例所关联的安全组中配置了适当的访问规则（例如，如果尝试通过 SSH 访问某一实例但有安全组阻止对其 22/TCP 的访问，则可能会发现 SSH 连接超时）。

对安全组相关的问题进行故障排除时，系统管理员可能需要直接检查 iptables 规则。在托管有问题的实例的计算节点上，列出其 iptables 规则。应该会存在一系列规则，它们通常分为两大类：一类用于源自实例的出站流量（出口流量），另一类用于前往实例的入站流量（入口流量）。分析这些规则或许有助于对实例问题进行故障排除。

在对实例访问问题故障排除时，要执行的另一项检查是实例操作系统本身的防火墙配置。确保其中配置的防火墙规则与需要的实例访问权限一致。

如果实例创建之时没有指定安全组，它将与 default 安全组关联。此组默认为不允许实例的入口或出口流量，直到开放所需的端口。 


### 练习：验证安全组规则

1. 在workstation上，通过提供文件/home/student/overcloudrc获取 admin 凭据。

    a. 提供 overcloudrc 文件，以获取 admin 凭据。

    [student@workstation ~]$ source overcloudrc

2. 创建一个名为 devvm 的实例，该实例使用 default 安全组。您可以从命令行界面传递此安全组；不过，如果您彻底不管安全组选项，系统将使用 default 安全组。使用 devimage 映像、m2.small 类别、devkeypair 密钥对，以及 devnet 网络。

[student@workstation ~]$ openstack server create --flavor m2.small --image devimage --key-name devkeypair --nic net-id=devnet devvm
+--------------+-------------------------------------------------+
| Field        | Value                                           |
+--------------+-------------------------------------------------+
...Output omitted...
| flavor       | m2.small (2)                                    |
...Output omitted...
| image        | devimage (...)                                  |
| key_name     | devkeypair                                      |
| name         | devvm                                           |
...Output omitted...
+--------------+-------------------------------------------------+

3. 等待 devvm 实例状态变为 ACTIVE。

[student@workstation ~]$ openstack server list
+--------------------------------------+-----------+--------+--------------------------------------+
| ID                                   | Name      | Status | Networks                             |
+--------------------------------------+-----------+--------+--------------------------------------+
| 3840437e-f32f-45c5-aaf0-4d6d851f5407 | devvm     | ACTIVE | devnet=192.168.1.16                  |
+--------------------------------------+-----------+--------+--------------------------------------+

4. 从 ext 浮动 IP 池中分配一个浮动 IP，并将它关联到 devvm 实例。

    a. 从 ext 浮动 IP 池分配一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 9162e47f-3bc5-4a9b-91ad-bae6ac94f889 |
    | instance_id | None                                 |
    | ip          | 172.25.250.51                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    b. 将前面获取的浮动 IP 关联到 devvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.51 devvm

5. 使用与 devkeypair 密钥对关联的私钥 /home/student/dev/devkeypair.pem，尝试打开与 devvm 实例的 SSH 会话。它将失败。SSH 超时会将您返回到提示符。

[student@workstation ~]$ ssh  -o ConnectTimeout=5 -i /home/student/dev/devkeypair.pem cloud-user@172.25.250.51
ssh: connect to host 172.25.250.51 port 22: Connection timed out
[student@workstation ~]$ 

6. 创建另一个名为 devsecondvm, 实例，该实例使用 devsecgroup 安全组。使用 devimage 映像、m2.small 类别、devkeypair 密钥对，以及 devnet 网络。

[student@workstation ~]$ openstack server create --flavor m2.small --security-group devsecgroup --image devimage --key-name devkeypair --nic net-id=devnet devsecondvm
+--------------------------------------+-------------------------------------------------+
| Field                                | Value                                           |
+--------------------------------------+-------------------------------------------------+
...Output omitted...
| flavor                               | m2.small (2)                                    |
...Output omitted...
| image                                | devimage (e167ac1a-cc46-4975-9aba-a49066677dea) |
| key_name                             | devkeypair                                      |
| name                                 | devsecondvm                                           |
...Output omitted...
| security_groups                      | [{u'name': u'devsecgroup'}]                     |
...Output omitted...
+--------------------------------------+-------------------------------------------------+

7. 从 ext 浮动 IP 池中分配一个浮动 IP，并将它关联到 devsecondvm 实例。

    a. 从 ext 浮动 IP 池分配一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 05a17088-7383-4bad-aa5a-9856583961c8 |
    | instance_id | None                                 |
    | ip          | 172.25.250.52                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    b. 将前面获取的浮动 IP 关联到 devsecondvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.52 devsecondvm

8. 使用与 devkeypair 密钥对关联的私钥 /home/student/dev/devkeypair.pem，打开与 devsecondvm 实例的 SSH 会话。它应当成功，因为 devsecgroup 允许从任何位置到 devsecondvm 实例的端口 TCP/22 (SSH) 的入口流量。完成时注销。

[student@workstation ~]$ ssh -i /home/student/dev/devkeypair.pem cloud-user@172.25.250.52
[cloud-user@devsecondvm ~]$ logout
Connection to 172.25.250.52 closed.


### 实验：管理安全组

1. 在 workstation 上，获取 admin 凭据。使用含有此类凭据的 /home/student/overcloudrc 文件。

    a. 提供文件overcloudrc，以获取 admin 凭据。

    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ 

2. 新建一个安全组，取名为 prodsshsg。

    a. 新建一个安全组，取名为 prodsshsg。

    [student@workstation ~]$ openstack security group create prodsshsg
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | prodsshsg                            |
    | id          | 988f7c90-ea6d-4e32-8a62-2ae004b4c5a8 |
    | name        | prodsshsg                            |
    | rules       | []                                   |
    | tenant_id   | 4b44f462430a4f9cbd2950f6c31546ab     |
    +-------------+--------------------------------------+

3. 在 prodsshsg 安全组中新建一条规则，以允许从任何位置到 TCP/22 端口的入口流量。

    a. 在 prodsshsg 安全组内创建一条规则，它使用 tcp 作为协议、0.0.0.0/0 作为来源 IP 地址，以及端口 22 (SSH) 作为目的地端口。

    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 22 prodsshsg
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 32274213-dc88-4cd2-888a-2179a723ca46 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 988f7c90-ea6d-4e32-8a62-2ae004b4c5a8 |
    | port_range      | 22:22                                |
    +-----------------+--------------------------------------+

4. 新建一个安全组，取名为 prodwebsg。

    a. 新建一个安全组，取名为 prodwebsg。

    [student@workstation ~]$ openstack security group create prodwebsg
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | prodwebsg                            |
    | id          | 5b9fdc7a-025d-4b6c-b4d0-dfb825f3be7a |
    | name        | prodwebsg                            |
    | rules       | []                                   |
    | tenant_id   | 4b44f462430a4f9cbd2950f6c31546ab     |
    +-------------+--------------------------------------+

5. 在 prodwebsg 安全组中新建两条规则。第一条规则允许从任何位置到 TCP/80 的入口流量，第二条规则允许从与 prodsshsg 安全组关联的实例到 TCP/22 的入口流量。

    a. 在 prodwebsg 安全组内创建一条规则，它使用 tcp 作为协议、0.0.0.0/0 作为来源 IP 地址，以及端口 80 作为目的地端口。

    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 80 prodwebsg
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 36bee3f2-3adb-4a0a-a003-1046d0157553 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 5b9fdc7a-025d-4b6c-b4d0-dfb825f3be7a |
    | port_range      | 80:80                                |
    +-----------------+--------------------------------------+

    b. 在 prodwebsg 安全组内创建另一条规则，它使用 tcp 作为协议、prodsshsg 作为来源安全组，以及端口 22 作为目的地端口。

    [student@workstation ~]$ nova secgroup-add-group-rule prodwebsg prodsshsg tcp 22 22
    +-------------+-----------+---------+----------+--------------+
    | IP Protocol | From Port | To Port | IP Range | Source Group |
    +-------------+-----------+---------+----------+--------------+
    | tcp         | 22        | 22      |          | prodsshsg    |
    +-------------+-----------+---------+----------+--------------+

6. 新建一个实例，取名为 prodsshvm。将 prodsshvm 关联到 prodsshsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。分配一个新的浮动 IP，并将它关联到 prodsshvm 实例。

    a. 创建一个名为 prodsshvm 实例，该实例使用 prodsshsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。

    [student@workstation ~]$ openstack server create --flavor m2.small --security-group prodsshsg --image prodimage --key-name prodkeypair --nic net-id=prodnet prodsshvm
    +--------------------------------------+-------------------------------------------------+
    | Field                                | Value                                           |
    +--------------------------------------+-------------------------------------------------+
    ...Output omitted...
    | flavor                               | m2.small (2)                                    |
    ...Output omitted...
    | image                                | prodimage (e167ac1a-cc46-4975-9aba-a49066677dea) |
    | key_name                             | prodkeypair                                      |
    | name                                 | prodsshvm                                           |
    ...Output omitted...
    | security_groups                      | [{u'name': u'prodsshsg'}]                     |
    ...Output omitted...
    +--------------------------------------+-------------------------------------------------+

    b. 从 ext 浮动 IP 池分配一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 79ec2726-9769-4db4-adcf-2cf22dd7fab0 |
    | instance_id | None                                 |
    | ip          | 172.25.250.51                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    c. 将前面获取的浮动 IP 关联到 prodsshvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.51 prodsshvm

7. 再新建一个实例，取名为 prodwebvm。将 prodwebvm 关联到 prodwebsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。分配一个新的浮动 IP，并将它关联到 prodwebvm 实例。

    a. 创建一个名为 prodwebvm 的实例，该实例使用 prodwebsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。

    [student@workstation ~]$ openstack server create --flavor m2.small --security-group prodwebsg --image prodimage --key-name prodkeypair --nic net-id=prodnet prodwebvm
    +--------------------------------------+-------------------------------------------------+
    | Field                                | Value                                           |
    +--------------------------------------+-------------------------------------------------+
    ...Output omitted...
    | flavor                               | m2.small (2)                                    |
    ...Output omitted...
    | image                                | prodimage (e167ac1a-cc46-4975-9aba-a49066677dea) |
    | key_name                             | prodkeypair                                      |
    | name                                 | prodwebvm                                           |
    ...Output omitted...
    | security_groups                      | [{u'name': u'prodwebsg'}]                     |
    ...Output omitted...
    +--------------------------------------+-------------------------------------------------+

    b. 从 ext 浮动 IP 池分配另一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 73848fc3-c9b6-4f33-b113-8ede35e1bd03 |
    | instance_id | None                                 |
    | ip          | 172.25.250.52                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    c. 将前面获取的浮动 IP 关联到 prodwebvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.52 prodwebvm

8. 为 prodwebvm 实例获取专用 IP 地址。它将在 192.168.1.0/24 子网中。

    a. 列出当前部署的两个实例的详细信息。将显示 prodwebvm 实例的专用 IP 地址。我们将使用此专用 IP 地址测试从 SSH 服务器到 Web 服务器的 SSH 连接。

    [student@workstation ~]$ openstack server list
    +--------------------------------------+-----------+--------+--------------------------------------+
    | ID                                   | Name      | Status | Networks                             |
    +--------------------------------------+-----------+--------+--------------------------------------+
    | 3840437e-f32f-45c5-aaf0-4d6d851f5407 | prodwebvm | ACTIVE | prodnet=192.168.1.17, 172.25.250.52 |
    | 99118fce-529f-4085-8653-511eaa8b86e5 | prodsshvm | ACTIVE | prodnet=192.168.1.16, 172.25.250.51 |
    +--------------------------------------+-----------+--------+--------------------------------------+

9. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem 并通过 SSH 连接实例，验证 prodsshvm 实例中 TCP/22 端口的访问权限已经开启。成功登录后退出。

    a. 使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem，打开与 prodsshvm 实例的 SSH 会话。它应当成功，因为 prodsshsg 允许从任何位置到 prodsshvm 实例的端口 TCP/22 (SSH) 的入口流量。完成时注销。

    [student@workstation ~]$ ssh -o ConnectTimeout=5 -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.51
    [cloud-user@prodsshvm ~]$ logout
    Connection to 172.25.250.51 closed.
    [student@workstation ~]$ 

10. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem，验证 prodwebvm 实例中 TCP/22 端口的访问权限已经关闭。

    a. 使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem，尝试打开与 prodwebvm 实例的 SSH 会话。它将失败。SSH 超时会将您返回到提示符。

    [student@workstation ~]$ ssh -o ConnectTimeout=5 -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.52
    ssh: connect to host 172.25.250.52 port 22: Connection timed out

11. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem，验证从 prodsshsg 安全组到 prodwebvm 实例中端口 TCP/22 的访问权限已经开启。您需要将此文件从workstation复制到 prodsshvm 实例。

    a. 将与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem 复制到 prodsshvm 实例。

    [student@workstation ~]$ scp -i /home/student/lab/prodkeypair.pem /home/student/lab/prodkeypair.pem cloud-user@172.25.250.51:
    prodkeypair.pem                            100% 1684     1.6KB/s   00:00

    b. 打开与 prodsshvm 实例的 SSH 会话；从该处，使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem 以及前面获取的专用 IP 地址，打开与 prodwebvm 的 SSH 会话。它应当成功，因为 prodwebsg 允许从与 prodsshsg 安全组关联的实例到 prodwebvm 实例的端口 TCP/22 (SSH) 的入口流量。完成时注销。

    [student@workstation ~]$ ssh -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.51
    [cloud-user@prodsshvm ~]$ ssh -i /home/cloud-user/prodkeypair.pem cloud-user@192.168.1.17
    The authenticity of host '192.168.1.17 (192.168.1.17)' can't be established.
    ECDSA key fingerprint is 55:03:93:b6:bb:29:73:4c:aa:7e:0a:b9:9e:14:18:a5.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.1.17' (ECDSA) to the list of known hosts.
    [cloud-user@prodwebvm ~]$ logout
    Connection to 192.168.1.17 closed.
    [cloud-user@prodsshvm ~]$ logout
    Connection to 172.25.250.51 closed.
    [student@workstation ~]$ 

12. 删除与 prodsecondsecgroup 安全组关联的规则。安全组及安全组规则均由实验脚本创建。

    a. 列出 prodsecondsecgroup 安全组中配置的规则。

    [student@workstation ~]$ openstack security group rule list prodsecondsecgroup
    +--------------------------------------+-------------+-----------+------------+
    | ID                                   | IP Protocol | IP Range  | Port Range |
    +--------------------------------------+-------------+-----------+------------+
    | f2c9d9df-5dce-4b10-ada7-47c0ecab8a3f | tcp         | 0.0.0.0/0 | 80:80      |
    +--------------------------------------+-------------+-----------+------------+

    b. 使用前面获取的规则 UUID，删除与 prodsecondsecgroup 安全组关联的规则。

    [student@workstation ~]$ openstack security group rule delete f2c9d9df-5dce-4b10-ada7-47c0ecab8a3f

13. 删除 prodthirdsecgroup 安全组。

    a. 删除 prodthirdsecgroup 安全组。

    [student@workstation ~]$ openstack security group delete prodthirdsecgroup


### 总结

在本章中，您可以学到：

*    默认情况下，任何在部署时未指定安全组的实例将关联到默认安全组。
*    新创建的安全组默认为不关联任何规则，因此所有入口流量将被拦截。若要通过 SSH 连接计算机，需要开放其 TCP/22 端口。
*    可以基于来源 IP、端口范围和协议，在安全组中创建规则。
