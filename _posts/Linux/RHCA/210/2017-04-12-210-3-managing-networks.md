---
layout: post
title:  "管理网络(210-3)"
categories: Linux
tags: RHCA 210
---

### 管理网络和子网

###### 联网服务 Red Hat OpenStack Platform

代号为 Neutron 的联网服务提供 API，可帮助定义 Red Hat OpenStack Platform 中受 Nova 管理的系统以及 Red Hat OpenStack Platform 8 中的 overcloud 节点的网络连接和寻址。OpenStack Neutron 允许创建和管理虚拟网络基础架构，其包含网络、交换机、子网和路由器，供其他 OpenStack 服务使用。联网服务也提供 API，用于配置和管理各种联网服务，范围涉及 L3 转发和 NAT 以及负载平衡、边界防火墙和虚拟专用网络等。

###### OpenStack 联网概念

Neutron 可以用于配置网络和子网，而 Nova 等其他 OpenStack 服务则可将虚拟设备连接到这些网络上的端口。OpenStack Neutron 定义两种类型的网络，即租户和提供商网络。可以在项目/租户之间共享任何这些类型的网络，作为网络创建流程的一个部分。由 Neutron 管理的部分联网概念有：

*    租户网络: OpenStack 用户创建租户网络，实现项目内的连接。默认情况下，这些网络完全孤立，不在项目之间共享。OpenStack Neutron 支持下列类型的网络隔离和覆盖技术：

        1. 扁平：所有实例驻留在同一网络上，可以和底层主机共享。没有任何 VLAN 标记或网络分隔。
        2. VLAN：此类联网允许用户利用 VLAN ID 创建多个租户网络，实现网络分隔。Web 层实例流量与数据库层实例隔开，便是这样的用例。
        3. GRE 和 VXLAN：这些网络提供封装，供覆盖网络来激活和控制计算实例之间的通信。

*    提供商网络: 这些网络映射到数据中心内的现有物理网络，通常为扁平或 VLAN 网络。
*    子网: 子网是一个 IP 地址区块，由租户和提供商网络在创建了新端口时提供。
*    端口: 端口是一种连接，将实例的虚拟 NIC 等单一设备关联到虚拟网络。端口也提供其上使用的相关配置，如 MAC 地址和 IP 地址。
*    路由器:
 
        1. 路由器在网络之间转发数据包。它们为租户网络上的虚拟机提供到外部网络的 L3 和 NAT 转发。需要路由器才能将流量发送到租户网络外部。路由器也用于通过浮动 IP 地址将租户网络与外部网络连接。
        2. 路由器由项目内验明身份的用户创建，归该项目所有。当租户实例需要外部访问时，用户可以将 OpenStack 管理员声明为外部网络的网络分配到其项目所有的路由器。
        3. 路由器通过实施来源网络地址转换 (SNAT) 提供出站外部连接，实施目的地网络地址转换 (DNAT) 来提供入站外部连接。

*    安全组: 安全组是实例的虚拟防火墙，可以控制出站和入站流量。它包含一组安全组规则，数据包进入或离开实例时将解析这些规则。

> 注意: 虚拟可扩展局域网 (VXLAN) 提供了解决方案，可解决与使用 VLAN 相关的扩展性问题，即并发租户网络数量限制为最多 4096 个。VXLAN 通过添加 24 位网段 ID，将它扩展到最多 1600 万个并发租户网络。VXLAN 在 L3 网络上封装 L2 网络。 


###### 创建网络

在 Nova 上启动实例之前，需要创建该实例要连接的虚拟网络基础架构。在创建网络之前，务必要考虑将要使用的子网是什么。路由器用于将一个子网中的流量定向到另一个子网中。

###### 创建提供商网络

提供商网络提供对实例的外部访问权限。它通过网络地址转换 (NAT) 并使用浮动 IP 地址和适当的安全组规则实现对实例的外部访问。要创建提供商网络，可执行下列步骤：

1. 提供overcloudrc文件。

```
[student@demo ~]$ source ~/overcloudrc
```
      
2. 创建提供商网络。

```
[student@demo ~]$ neutron net-create public --router:external
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 6ae19ebd-3a30-4839-9616-a482673c65a3 |
| mtu                       | 0                                    |
| name                      | public                               |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 40                                   |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | b13d4428224247ed99e8a7f688c55a9d     |
+---------------------------+--------------------------------------+
```

3. 与物理网络类似，虚拟网络需要创建并分配子网。提供商网络和它所连接的物理网络共享相同的子网和关联网关。要为提供商网络创建子网，请使用 --allocation-pool 指定浮动 IP 地址片段。

```
[student@demo ~]$ neutron subnet-create public 172.25.250.0/24 \
> --name public_subnet \
> --disable-dhcp --allocation-pool start=172.25.250.130,end=172.25.250.199 \
> --gateway 172.25.250.254
Created a new subnet:
+-------------------+------------------------------------------------------+
| Field             | Value                                                |
+-------------------+------------------------------------------------------+
| allocation_pools  | {"start": "172.25.250.130", "end": "172.25.250.199"} |
| cidr              | 172.25.250.0/24                                      |
| dns_nameservers   |                                                      |
| enable_dhcp       | False                                                |
| gateway_ip        | 172.25.250.254                                       |
| host_routes       |                                                      |
| id                | e4af19b7-f1b6-4f82-b20f-63003fcf2d24                 |
| ip_version        | 4                                                    |
| ipv6_address_mode |                                                      |
| ipv6_ra_mode      |                                                      |
| name              | public_subnet                                        |
| network_id        | 6ae19ebd-3a30-4839-9616-a482673c65a3                 |
| subnetpool_id     |                                                      |
| tenant_id         | b13d4428224247ed99e8a7f688c55a9d                     |
+-------------------+------------------------------------------------------+
```

###### 创建租户网络

租户网络为实例提供对特定项目/租户的内部网络访问。下例演示了如何创建租户网络。

1. 创建基于 VXLAN 的内部租户网络：

```
    [student@demo ~]$ neutron net-create internal
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 184887f9-4a0a-4ace-a1fd-8bc4b2f21e9e |
    | mtu                       | 0                                    |
    | name                      | internal                             |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 82                                   |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | b13d4428224247ed99e8a7f688c55a9d     |
    +---------------------------+--------------------------------------+
```

2. 通过指定租户网络 CIDR，为租户网络创建对应的子网。默认情况下，此子网使用 DHCP，以便实例可以获取 IP 地址。子网的第一个 IP 地址保留为网关 IP 地址。

```
    [student@demo ~]$ neutron subnet-create internal 192.168.0.0/24 --name internal_subnet
    Created a new subnet:
    +-------------------+--------------------------------------------------+
    | Field             | Value                                            |
    +-------------------+--------------------------------------------------+
    | allocation_pools  | {"start": "192.168.0.2", "end": "192.168.0.254"} |
    | cidr              | 192.168.0.0/24                                   |
    | dns_nameservers   |                                                  |
    | enable_dhcp       | True                                             |
    | gateway_ip        | 192.168.0.1                                      |
    | host_routes       |                                                  |
    | id                | 4443e73f-77ea-43a0-8f81-8f6ad60a30b6             |
    | ip_version        | 4                                                |
    | ipv6_address_mode |                                                  |
    | ipv6_ra_mode      |                                                  |
    | name              | internal_subnet                                  |
    | network_id        | 184887f9-4a0a-4ace-a1fd-8bc4b2f21e9e             |
    | subnetpool_id     |                                                  |
    | tenant_id         | b13d4428224247ed99e8a7f688c55a9d                 |
    +-------------------+--------------------------------------------------+
```

###### 更新网络

无论是出于开发、扩展还是调配的需要，在某一个时点上可能必须要更新某一网络。如果网络尚未连接到路由器也未用于实例，则可对它进行更新。不过，如果网络已连接了实例的路由器，需要将它从路由器分离，然后才能更新。在下例中，为 public_subnet 子网更新了 DNS 名称服务器。

```
[student@demo ~]$ neutron subnet-update --dns-nameserver 172.25.250.254 public_subnet
```

###### 使用 Neutron 管理网络

已验证身份的用户可以通过neutron net-*子命令创建、更新、列出和删除 L2 网络。

neutron net-* 子命令

*    子命令 	                                功能
*    net-create NAME [--shared] [--admin- state-down] 	创建网络。默认情况下，admin 状态为 up，并且只有创建它的租户才能使用它。
*    net-delete NAME 	                    删除网络。
*    net-show NAME [--show-details|-D] 	    显示网络的详细信息。
*    net-list [--show-details|-D] 	        列出租户拥有的所有网络。
*    net-update [--shared True|False] NAME 	更改 admin 状态或 shared 状态。
*    net-external-list [--show-details|-D] 	列出租户拥有的外部网络。

###### 使用 Neutron 管理子网

用户可用使用neutron subnet-*子命令执行创建、更新、删除和列举操作。

neutron subnet-* 子命令

*    子命令 	                                    功能
*    subnet-create（见下列选项） 	                创建子网。
*    subnet-delete NAME|ID 	                    删除子网。如果分配了使用该子网的端口，则需要先删除这些端口。
*    subnet-show [--show-details|-D] NAME|ID 	显示子网的详细信息。
*    subnet-list [--show-details|-D] 	        列出租户拥有的所有子网。
*    subnet-update NAME|ID 	                    更新子网的信息。

neutron subnet-create 选项

*    选项 	                        目的
*    NAME 	                        子网的名称。
*    CIDR 	                        子网地址。
*    --gateway GATEWAY_IP 	        DHCP 网关。
*    --no-gateway 	                不使用网关。
*    --allocation-pool start=IP_ADDR,end=IP_ADDR 	DHCP 分配池。
*    --host-route 	                用于向实例播发的静态路由器。
*    --dns-nameserver DNS_IP 	    名称服务器的 IP 地址。
*    --disable-dhcp 	            在这一子网上禁用 DHCP。
*    --ip-version {4,6} IP 	        设置 IP 版本。默认为 IPv4。

### 练习：管理网络和子网

1. 从 workstation 打开一个终端，再提供 /home/student/overcloudrc 文件中找到的 overcloud 管理凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 所用 neutron 命令行界面，创建公共网络 dev_pubnet。

```
[student@workstation ~]$ neutron net-create dev_pubnet
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 6ae19ebd-3a30-4839-9616-a482673c65a3 |
| mtu                       | 0                                    |
| name                      | dev_pubnet                           |
| port_security_enabled     | True                                 |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 40                                   |
| qos_policy_id             |                                      |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | b13d4428224247ed99e8a7f688c55a9d     |
+---------------------------+--------------------------------------+
```

3. 将该网络声明为外部网络，以进行更新。

```
[student@workstation ~]$ neutron net-update --router:external dev_pubnet
Updated network: dev_pubnet
```

4. 为外部网络 dev_pubnet 创建对应的子网，取名为 dev_pubsubnet，它使用 CIDR 172.25.250.0/24、从 172,25.250.130 到 172.25.250.199 的分配池，以及指向 172.25.250.254 的网关。DHCP 应当禁用。

```
[student@workstation ~]$ neutron subnet-create dev_pubnet 172.25.250.0/24 \
> --name dev_pubsubnet --disable-dhcp \
> --allocation-pool start=172.25.250.130,end=172.25.250.199 \
> --gateway 172.25.250.254
Created a new subnet:
+-------------------+------------------------------------------------------+
| Field             | Value                                                |
+-------------------+------------------------------------------------------+
| allocation_pools  | {"start": "172.25.250.130", "end": "172.25.250.199"} |
| cidr              | 172.25.250.0/24                                      |
| dns_nameservers   |                                                      |
| enable_dhcp       | False                                                |
| gateway_ip        | 172.25.250.254                                       |
| host_routes       |                                                      |
| id                | e4af19b7-f1b6-4f82-b20f-63003fcf2d24                 |
| ip_version        | 4                                                    |
| ipv6_address_mode |                                                      |
| ipv6_ra_mode      |                                                      |
| name              | dev_pubsubnet                                        |
| network_id        | 6ae19ebd-3a30-4839-9616-a482673c65a3                 |
| subnetpool_id     |                                                      |
| tenant_id         | b13d4428224247ed99e8a7f688c55a9d                     |
+-------------------+------------------------------------------------------+
```

5. 将 dev_pubsubnet 的 DNS 名称服务器更新为 172.25.250.254。

```
[student@workstation ~]$ neutron subnet-update --dns-nameserver 172.25.250.254 dev_pubsubnet
Updated subnet: dev_pubsubnet
```

6. 使用 neutron net-list 命令，检查通过 neutron subnet-list 创建的网络和子网。

```
[student@workstation ~]$ neutron net-list
+--------------+------------------------------------------------------+
| name         | subnets                                              |
+--------------+------------------------------------------------------+
...Output omitted...
| dev_pubnet   | e4af19b7-f1b6-4f82-b20f-63003fcf2d24 172.25.250.0/24 |
...Output omitted...
+--------------+------------------------------------------------------+
[student@workstation ~]$ neutron subnet-list
+--------------------------------------+-----------------+-----------------+------------------------------------------------------+
| id                                   | name            | cidr            | allocation_pools                                     |
+--------------------------------------+-----------------+-----------------+------------------------------------------------------+
...Output omitted...
| e4af19b7-f1b6-4f82-b20f-63003fcf2d24 | dev_pubsubnet   | 172.25.250.0/24 | {"start": "172.25.250.130", "end": "172.25.250.199"} |
...Output omitted...
+--------------------------------------+-----------------+-----------------+------------------------------------------------------+
```

7. 使用 neutron net-delete，删除 prod-oldnet 网络。不过，为了能删除该网络，需要先删除子网 dev_secondsubnet。

```
[student@workstation ~]$ neutron subnet-delete dev_secondsubnet
Deleted subnet: dev_secondsubnet
[student@workstation ~]$ neutron net-delete dev_secondnet
Deleted network: dev_secondnet
```


### 管理路由器

###### 在两个隔离的网络之间路由流量

在现实部署中，可能会创建不同类型的网络来用于不同的流量流。例如，Web 流量应当和数据库流量隔开，但同时也要在这两个网络之间路由流量，从而让 Web 请求能访问数据库服务器以获取响应。由于这两个网络在不同的 IP范围内，我们需要使用路由器将它们相连。

在连接了公共网络时，路由器也帮助实现外部路由，从而通过浮动 IP 从外部访问或者从外部网络访问 Nova 实例。 

###### 使用 Neutron 管理路由器

已验证身份的用户可以通过 neutron router-* 子命令创建、删除和管理路由器。

neutron router-* 子命令

*    子命令 	功能
*    router-create NAME [--admin- state-down] 	创建路由器。默认情况下，admin 状态为 up，并且只有创建它的租户才能使用它。
*    router-delete NAME|ID 	删除路由器。要删除的路由器的 ID 或名称。
*    router-gateway-set NAME|ID EXT-NETWORK 	设置路由器的外部网络网关。EXT-NETWORK 是网关的外部网络名称或 ID。
*    router-interface-add NAME|ID INTERFACE 	向路由器添加内部网络接口。INTERFACE：必须指定子网 ID 或端口 ID；也可以使用名称。
*    router-show NAME|ID 	显示给定路由器的信息。
*    router-update NAME|ID 	更新路由器的信息。
*    router-port-list NAME|ID 	列出属于给定租户并具有指定路由器的端口。


### 练习：管理路由器

1. 从workstation打开一个终端，再提供/home/student/overcloudrc文件中找到的 overcloud 管理凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 创建两个专用/租户网络，以及对应的子网。使用下表创建网络和子网。

*    网络/子网名称	        详情	                CIDR 块
*    dev_privnet1	    专用网络 1	        -
*    dev_privnet2	    专用网络 2	        -
*    dev_privsubnet1	    租户网络 1 的子网	    10.10.10.0/24
*    dev_privsubnet2	    租户网络 2 的子网	    20.20.20.0/24

    a. 创建 dev_privnet1 网络。

```
    [student@workstation ~]$ neutron net-create dev_privnet1
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 511c9f28-2137-4817-abe1-15d44beb826c |
    | mtu                       | 0                                    |
    | name                      | dev_privnet1                         |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 99                                   |
    | qos_policy_id             |                                      |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

    b. 创建 dev_privnet1 网络的子网，名称为 dev_privsubnet1，CIDR 为 10.10.10.0/24。

```
    [student@workstation ~]$ neutron subnet-create dev_privnet1 10.10.10.0/24 --name dev_privsubnet1
    Created a new subnet:
    +-------------------+------------------------------------------------+
    | Field             | Value                                          |
    +-------------------+------------------------------------------------+
    | allocation_pools  | {"start": "10.10.10.2", "end": "10.10.10.254"} |
    | cidr              | 10.10.10.0/24                                  |
    | dns_nameservers   |                                                |
    | enable_dhcp       | True                                           |
    | gateway_ip        | 10.10.10.1                                     |
    | host_routes       |                                                |
    | id                | ecc02880-10b1-4675-b824-27384cbb7979           |
    | ip_version        | 4                                              |
    | ipv6_address_mode |                                                |
    | ipv6_ra_mode      |                                                |
    | name              | dev_privsubnet1                                |
    | network_id        | 511c9f28-2137-4817-abe1-15d44beb826c           |
    | subnetpool_id     |                                                |
    | tenant_id         | 5e0c6cdea7694c1db57951aba78f0578               |
    +-------------------+------------------------------------------------+
```

    c. 创建 dev_privnet2 网络。

```
    [student@workstation ~]$ neutron net-create dev_privnet2
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 8f0b7667-f724-4983-8258-a7d8be2ce2a4 |
    | mtu                       | 0                                    |
    | name                      | dev_privnet2                         |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 51                                   |
    | qos_policy_id             |                                      |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

    d. 创建 dev_privnet2 网络的子网，名称为 dev_privsubnet2，CIDR 为 20.20.20.0/24。

```
    [student@workstation ~]$ neutron subnet-create dev_privnet2 20.20.20.0/24 --name dev_privsubnet2
    Created a new subnet:
    +-------------------+------------------------------------------------+
    | Field             | Value                                          |
    +-------------------+------------------------------------------------+
    | allocation_pools  | {"start": "20.20.20.2", "end": "20.20.20.254"} |
    | cidr              | 20.20.20.0/24                                  |
    | dns_nameservers   |                                                |
    | enable_dhcp       | True                                           |
    | gateway_ip        | 20.20.20.1                                     |
    | host_routes       |                                                |
    | id                | 3b730a0c-848c-4b50-8acf-3937bf188582           |
    | ip_version        | 4                                              |
    | ipv6_address_mode |                                                |
    | ipv6_ra_mode      |                                                |
    | name              | dev_privsubnet2                                |
    | network_id        | 8f0b7667-f724-4983-8258-a7d8be2ce2a4           |
    | subnetpool_id     |                                                |
    | tenant_id         | 5e0c6cdea7694c1db57951aba78f0578               |
    +-------------------+------------------------------------------------+
```

3. 创建名为 dev_router 的路由器。

```
[student@workstation ~]$ neutron router-create dev_router
Created a new router:
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| admin_state_up        | True                                 |
| distributed           | False                                |
| external_gateway_info |                                      |
| ha                    | False                                |
| id                    | 950ec629-4e27-4f10-86ad-94e7538df3ff |
| name                  | dev_router                           |
| routes                |                                      |
| status                | ACTIVE                               |
| tenant_id             | 5e0c6cdea7694c1db57951aba78f0578     |
+-----------------------+--------------------------------------+
```

4. 将创建的子网 dev_privsubnet1 和 dev_privsubnet2 连接到路由器 dev_router。

```
[student@workstation ~]$ neutron router-interface-add dev_router dev_privsubnet1
Added interface 26378b9a-60be-4295-9cbd-5c8aae05504c to router dev_router.
[student@workstation ~]$ neutron router-interface-add dev_router dev_privsubnet2
Added interface 5f8343e1-c65f-458a-8375-d2f08eff6b26 to router dev_router.
```

5. 确保 dev_pubnet 网络可用，并将它定义为 dev_router 路由器的网关。

```
[student@workstation ~]$ neutron net-list
+--------------------------------------+---------------+------------------------------------------------------+
| id                                   | name          | subnets                                              |
+--------------------------------------+---------------+------------------------------------------------------+
...Output omitted...
| 7a9e2cfe-d7c6-4e4c-b29c-44b41e6b04f6 | dev_pubnet    | 46cb0222-5f43-4dbe-a949-7e96cedd5192 172.25.250.0/24 |
+--------------------------------------+---------------+------------------------------------------------------+

[student@workstation ~]$ neutron router-gateway-set dev_router dev_pubnet
Set gateway for router dev_router
```

6. 使用命令 neutron router-port-list 列出与路由器连接的端口，检查网关是否已正确添加。

```
[student@workstation ~]$ neutron router-port-list dev_router
+--------------------------------------+------+-------------------+----------------------------------------------------------------------------------------+
| id                                   | name | mac_address       | fixed_ips                                                                              |
+--------------------------------------+------+-------------------+----------------------------------------------------------------------------------------+
| 26378b9a-60be-4295-9cbd-5c8aae05504c |      | fa:16:3e:0a:b1:34 | {"subnet_id": "ecc02880-10b1-4675-b824-27384cbb7979",  "ip_address": "10.10.10.1"}     |
| 5f8343e1-c65f-458a-8375-d2f08eff6b26 |      | fa:16:3e:9b:ee:fb | {"subnet_id": "3b730a0c-848c-4b50-8acf-3937bf188582",  "ip_address": "20.20.20.1"}     |
| b00a417c-377a-4beb-a114-98db97fb9ac1 |      | fa:16:3e:a8:cd:db | {"subnet_id": "51176d1f-c3db-4568-ac20-5452117f2b96",  "ip_address": "172.25.250.130"} |
+--------------------------------------+------+-------------------+----------------------------------------------------------------------------------------+
```

7. 删除之前调配的 dev_oldrouter 路由器。由于已经为 dev_oldpubnet 设置了网关，需要先将它清除，然后才能删除路由器。

    a. 使用neutron router-list命令，列出本实验环境中存在的路由器。列出连接到 dev_oldrouter 路由器的端口。

```
    [student@workstation ~]$ neutron router-list -c id -c name
    +--------------------------------------+-----------------+
    | id                                   | name            |
    +--------------------------------------+-----------------+
    | 950ec629-4e27-4f10-86ad-94e7538df3ff | dev_router      |
    | b1998821-e734-47de-b785-5b5f7156885e | dev_oldrouter   |
    +--------------------------------------+-----------------+
    [student@workstation ~]$ neutron router-port-list dev_oldrouter
    +--------------------------------------+------+-------------------+-------------------------------------------------------+
    | id                                   | name | mac_address       | fixed_ips                                             |
    +--------------------------------------+------+-------------------+-------------------------------------------------------+
    | db0131c8-f9f2-45bf-92ec-026233510a4a |      | fa:16:3e:43:e8:0a | {"subnet_id": "2f4a9333-1e4f-4a3c-9167-83732f87d47c", |
    |                                      |      |                   | "ip_address": "172.24.250.130"}                       |
    +--------------------------------------+------+-------------------+-------------------------------------------------------+
```

    b. 检查之前列出的端口是否为 dev_oldrouter 路由器的网关。在以下屏幕输出中，external_gateway_info 将 network_id cbd6cf30-0c13-4efe-a6b6-76bc2faf1f8f 标记为 dev_oldrouter 的网关。

```
    [student@workstation ~]$ neutron router-show dev_oldrouter
    +-----------------------+----------------------------------------------------+
    | Field                 | Value                                              |
    +-----------------------+----------------------------------------------------+
    | admin_state_up        | True                                               |
    | distributed           | False                                              |
    | external_gateway_info | {"network_id":                                     |
    |                       | "cbd6cf30-0c13-4efe-a6b6-76bc2faf1f8f",            |
    |                       | "enable_snat": true, "external_fixed_ips":[        |
    |                       | "subnet_id":"2f4a9333-1e4f-4a3c-9167-83732f87d47c",|
    |                       | "ip_address": "172.24.250.130"}]}                  |
    | ha                    | False                                              |
    | id                    | b1998821-e734-47de-b785-5b5f7156885e               |
    | name                  | dev_oldrouter                                      |
    | routes                |                                                    |
    | status                | ACTIVE                                             |
    | tenant_id             | 5e0c6cdea7694c1db57951aba78f0578                   |
    +-----------------------+----------------------------------------------------+
```

    c. 使用命令 neutron net-show，通过网络 ID 查找其网络名称。

```
    [student@workstation ~]$ neutron net-show cbd6cf30-0c13-4efe-a6b6-76bc2faf1f8f
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | cbd6cf30-0c13-4efe-a6b6-76bc2faf1f8f |
    | mtu                       | 0                                    |
    | name                      | dev_oldpubnet                        |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 56                                   |
    | qos_policy_id             |                                      |
    | router:external           | True                                 |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   | 2f4a9333-1e4f-4a3c-9167-83732f87d47c |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

    d. 要删除 dev_oldrouter 路由器，需要先移除与它连接的网关。使用 neutron router-gateway-clear 移除该网关。

```
    [student@workstation ~]$ neutron router-gateway-clear dev_oldrouter dev_oldpubnet
    Removed gateway from router dev_oldrouter
```

    e. 删除 dev_oldrouter 路由器。

```
    [student@workstation ~]$ neutron router-delete dev_oldrouter
    Deleted router: dev_oldrouter
```

    f. 删除子网 dev_oldpubsubnet，然后删除网络 dev_oldpubnet。

```
    [student@workstation ~]$ neutron subnet-delete dev_oldpubsubnet
    Deleted subnet: dev_oldpubsubnet
    [student@workstation ~]$ neutron net-delete dev_oldpubnet
    Deleted network: dev_oldpubnet
```

    g. 使用neutron router-list命令，列出本实验环境中存在的路由器。dev_oldrouter 路由器应当不再可用。

```
    [student@workstation ~]$ neutron router-list -c id -c name
    +--------------------------------------+-----------------+
    | id                                   | name            |
    +--------------------------------------+-----------------+
    | 950ec629-4e27-4f10-86ad-94e7538df3ff | dev_router      |
    +--------------------------------------+-----------------+
```


### 验证 OpenStack 联网功能

###### 网络命名空间 (netns)

网络命名空间是被隔离的容器，可以保管不可由命名空间外部访问的网络配置。命名空间让管理员能够拥有不同且单独的网络接口实例和路由表，它们能够互相独立运作。网络命名空间拥有自己的网络路由、自己的防火墙规则，以及自己的网络设备。

ip netns 命令可自动化处理命名空间配置。

```
[student@workstation ~]$ ip netns help
Usage: ip netns list
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id
```

例如，列出给定命名空间中的所有网络接口：

```
ip netns exec NAME ip link show
```

ip netns 可以在命名空间中执行任何命令，但只有网络型命令才会受到影响，如 ping、 ip 和 traceroute 等。

###### 虚拟网络设备

Linux 支持各种虚拟网络设备。相较于由物理网络适配器支持的硬件设备，虚拟设备是完全在软件中支持的设备。虚拟设备模拟物理适配器支持的大部分功能；通过虚拟设备传输的数据包通常传送到本身与虚拟设备通信的用户空间程序。虚拟设备的优点包括：

*    在受限的硬件环境中提供复杂的网络拓扑。
*    在更广的环境中实施架构前构建概念验证。
*    热插拔网络接口；按需创建。
*    软件驱动的拓扑，使构建网络环境变得更加简单。

由于虚拟设备不使用自己的适配器，性能可能会成为一个问题，尤其是有多个网络设备堆叠并交换流量时，或者对流量应用了网络过滤时。虚拟设备通常与虚拟化软件一同使用，后者控制网络资源的底层调配。OpenStack 管理下列网络设备：

OpenStack 联网设备

*    网络设备 	用法
*    vnet 	与实例连接的虚拟接口。
*    qbr 	Linux 网桥。
*    br-int/br-ex 	Open vSwitch 网桥。
*    qvb 和 qvo 	与 Linux 网桥和 Open vSwitch 网桥连接的 vEth 对。
*    虚拟路由器和命名空间 	它们与虚拟网桥连接。路由器用于在外部路由网络。命名空间对 Linux 系统上的网络使用进行分区。


*    TAP 设备：TAP 设备是虚拟网络内核设备；它们模拟链路层设备，并处理以太网帧等第 2 层数据包。TAP 设备可用于创建网桥。
*    vEth 设备：这些设备可以和其他虚拟网络设备连接，它们基本上是虚拟跳接线缆。在 OpenStack 中，这种虚拟设备可用于将租户网络连接到基础架构网络。通信既可通过在两端分配 IP 地址并在主机上执行路由而将该对用作点对点直连链路，也可以通过桥接。

###### 连接两个命名空间

若要连接两个或更多命名空间，Linux 提供了各种不同的技巧。

*    第一个方法中包含创建两个 vEth 对并将它们的端口连接到 Linux 网桥。

```
    +------------+                 +------------+
    | Namespace1 |                 | Namespace2 |
    |       TAP1 |                 |       TAP2 |
    +------------+                 +------------+
              |                           |
              |         vEth pairs        |
              |                           |
          +---------+--------------+---------+    
          | br-tap1 | Linux bridge | br-tap2 |
          +---------+--------------+---------+
```

*    另一种办法中包含将 Linux 网桥替换为 Open vSwitch 网桥。配置相同，但桥接由 Open vSwitch 而不是 Linux 网桥进行管理： 

```
    +------------+                 +------------+
    | Namespace1 |                 | Namespace2 |
    |       TAP1 |                 |       TAP2 |
    +------------+                 +------------+
              |                           |
              |         vEth pairs        |
              |                           |
          +---------+--------------+---------+    
          | br-tap1 |  OVS bridge  | br-tap2 |
          +---------+--------------+---------+
```

*    最后，管理员还可以使用 Open vSwitch 内部端口来连接两个命名空间。这可以避免使用 vEth 对。 

```
    +------------+                 +------------+
    | Namespace1 |                 | Namespace2 |
    |       TAP1 |                 |       TAP2 |
    +------------+                 +------------+
              |                           |
              |         OVS ports         |
              |                           |
          +----------------------------------+    
          |            OVS bridge            |
          +----------------------------------+
```

###### OpenStack 命名空间管理

在使用主机实施路由时，各种 OpenStack 租户网络子网可能会与物理网络的子网重叠。如果各种租户的最终用户有权创建自己的逻辑网络和子网，则管理员必须对系统进行设计，以防止此冲突发生。联网功能使用 Linux 网络命名空间防止网络主机上的物理网络和虚拟机使用的逻辑网络之间发生冲突。它也能防止不互相路由的不同逻辑网络之间发生冲突。 

*    DHCP 服务器使用自己的命名空间 qdhcp，上面将关联 TAP 设备。dnsmasq 进程侦听该接口，从而为实例提供 DHCP 服务。使用命名空间可允许不同的租户之间存在重叠 IP 地址。
*    路由器使用自己的命名空间，它们包含特定的路由 qr。路由器网关 qg 通过 TAP 设备连接到额外的网桥 br-ex，后者本身连接到外部网络。

###### OpenStack 中实例和外部网络间的数据包流

在网络中创建新实例时，OpenStack 将创建下列虚拟网络设备标识符：

1. 接口 eth0 映射到虚拟端口 vnet0。vnet0 是 TAP 设备。
2. 端口连接到 Linux 网桥 qbrX。
3. 网桥通过 vEth 对（qvoX 和 qvbX）连接到集成网桥 br-int。
4. 流量通过 Open vSwitch 端口 phy-br-ethX 在物理接口来回路由。

###### 来回于外部网络的示例数据包流

*    离开实例前往外部目的地的数据包首先从实例的eth0 设备传递到 Linux 网桥 qbrX，其中将筛选出口防火墙规则。接下来，数据包传递到集成网桥br-int，它将辨别外部目的地地址并将数据包传递到路由器 qrA。该路由器对数据包应用 SNAT 运算，并将它通过网关 ggB 传递到外部网桥 br-ex。最后，数据包通过网络节点的接口 eth0（与 br-ex关联）传递到外部路由器（未显示），后者将数据包传递到通向其外部目的地的路径上的下一跃点。
*    来自外部来源并以实例为目的地的数据包将遵循与上述相反的流程，但有一个例外：在路由器 qrA 处应用 DNAT 运算并在 Linux 网桥 qbrX 筛选入口防火墙规则。


### 练习：验证 OpenStack 联网功能

1. 从workstation打开一个终端，再提供/home/student/overcloudrc文件中找到的 overcloud 管理凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 检查上一练习中创建的网络。

```
[student@workstation ~]$ neutron net-list
+--------------------------------------+---------------+------------------------------------------------------+
| id                                   | name          | subnets                                              |
+--------------------------------------+---------------+------------------------------------------------------+
| 511c9f28-2137-4817-abe1-15d44beb826c | dev_privnet1  | ecc02880-10b1-4675-b824-27384cbb7979 10.10.10.0/24   |
| 8f0b7667-f724-4983-8258-a7d8be2ce2a4 | dev_privnet2  | 3b730a0c-848c-4b50-8acf-3937bf188582 20.20.20.0/24   |
| 8d5808fc-a39b-454d-be84-0c68719d382e | dev_pubnet    | 51176d1f-c3db-4568-ac20-5452117f2b96 172.25.250.0/24 |
+--------------------------------------+---------------+------------------------------------------------------+

```

3. 确保上一练习中创建的子网可用。

```
[student@workstation ~]$ neutron subnet-list
+--------------------------------------+------------------+-----------------+------------------------------------------------------+
| id                                   | name             | cidr            | allocation_pools                                     |
+--------------------------------------+------------------+-----------------+------------------------------------------------------+
| ecc02880-10b1-4675-b824-27384cbb7979 | dev_privsubnet1  | 10.10.10.0/24   | {"start": "10.10.10.2", "end": "10.10.10.254"}       |
| 3b730a0c-848c-4b50-8acf-3937bf188582 | dev_privsubnet2  | 20.20.20.0/24   | {"start": "20.20.20.2", "end": "20.20.20.254"}       |
| 51176d1f-c3db-4568-ac20-5452117f2b96 | dev_pubsubnet    | 172.25.250.0/24 | {"start": "172.25.250.130", "end": "172.25.250.199"} |
+--------------------------------------+------------------+-----------------+------------------------------------------------------+
```

4. 确保上一练习中创建的路由器可用。

```
[student@workstation ~]$ neutron router-list -c id -c name
+--------------------------------------+-------------+
| id                                   | name        |
+--------------------------------------+-------------+
| 950ec629-4e27-4f10-86ad-94e7538df3ff | dev_router  |
+--------------------------------------+-------------+
```

5. 使用ip netns list命令，从 overcloud-controller-0 访问与网络和路由器关联的命名空间。

    a. 从 workstation，使用私钥 /home/student/overcloud.pem 和用户名 heat-admin 通过 SSH 连接 overcloud 控制器节点 overcloud-controller-0。IP 地址是来自 overcloudrc 文件的 OS_AUTH_URL 的值，因为控制器节点 overcloud-controller-0 上部署了 Keystone。

```
    [student@workstation ~]$ echo $OS_AUTH_URL
    http://192.168.0.11:5000/v2.0/
    [student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
    Last login: Tue Feb  9 08:50:30 2016 from workstation.lab.example.com
    [heat-admin@overcloud-controller-0 ~]$ 
```

    b. 使用 ip netns list 命令列出控制器节点上创建的命名空间。

```
    [heat-admin@overcloud-controller-0 ~]$ sudo ip netns list
    qrouter-5d6a1e56-419a-4826-b882-3a64e5f83a1d
    qdhcp-15f94f78-8bc7-469e-a676-e8ac04445a69
    qdhcp-6d8016c0-e605-4614-99c3-bf9de795131a
```

6. 列出 dev_privnet1 网络的命名空间中可用的网络接口，该网络关联有 CIDR 块为 10.10.10.0/24 的子网。该命名空间可以用于访问连接到 dev_privsubnet1 子网的所有网络设备。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qdhcp-6d8016c0-e605-4614-99c3-bf9de795131a ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
14: tap51feda31-05: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:16:86:cd brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.2/24 brd 10.10.10.255 scope global tap51feda31-05
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe16:86cd/64 scope link
       valid_lft forever preferred_lft forever
```

TAP 设备 tap51feda31-05 被提供了 IP 地址 10.10.10.2/24，因为它是 dev_privsubnet1 子网中的接口。 

7. 列出 dev_privnet2 网络的命名空间中可用的网络接口，该网络关联有 CIDR 块为 20.20.20.0/24 的子网。该命名空间可以用于访问连接到 dev_privsubnet2 子网的所有网络设备。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qdhcp-15f94f78-8bc7-469e-a676-e8ac04445a69 ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
15: tapfc6c9905-bd: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:20:85:15 brd ff:ff:ff:ff:ff:ff
    inet 20.20.20.2/24 brd 20.20.20.255 scope global tapfc6c9905-bd
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe20:8515/64 scope link
       valid_lft forever preferred_lft forever
```

TAP 设备 tapfc6c9905-bd 被提供了 IP 地址 20.20.20.2/24，因为它是 dev_privsubnet2 子网中的接口。

8. 列出 dev_router 命名空间 qr-XXX 中可用的端口。由于该路由器连接了两个网络，它已关联有两个 IP 地址；第一个地址是 10.10.10.1/24，来自 CIDR 块 10.10.10.0/24；第二个地址则是 20.20.20.1/24，来自 CIDR 块 20.20.20.0/24。请注意，qg-XXX 是与 dev_router 路由器关联的网关。

前面列出的 XXX 对应于该路由器具有的端口的 UUID，它们连接了这两个子网。可以运行 neutron router-port-list dev_router 命令来访问这些 UUID。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qrouter-5d6a1e56-419a-4826-b882-3a64e5f83a1d ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
16: qr-bd7c9b8f-0f:<BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:c6:59:7d brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global qr-bd7c9b8f-0f
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec6:597d/64 scope link
       valid_lft forever preferred_lft forever
17: qr-63885b85-6a:<BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:2d:a9:7b brd ff:ff:ff:ff:ff:ff
    inet 20.20.20.1/24 brd 20.20.20.255 scope global qr-63885b85-6a
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe2d:a97b/64 scope link
       valid_lft forever preferred_lft forever
18: qg-bae4ccce-8d:<BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:71:a4:a4 brd ff:ff:ff:ff:ff:ff
    inet 172.25.250.130/24 brd 172.25.250.255 scope global qg-bae4ccce-8d
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe71:a4a4/64 scope link
       valid_lft forever preferred_lft forever
```

9. 从 workstation，使用下列参数在 dev_privnet1 网络中启动名为 dev_instance1 的实例：

*    参数	            值
*    --映像	            small
*    --类别	            m2.small
*    --security-group	dev_sg
*    --密钥对	        dev_key
*    --nic 	            net-id=dev_privnet1

```
[student@workstation ~]$ openstack server create --image small --flavor m2.small \
> --security-group dev_sg --key-name dev_key \
> --nic net-id=dev_privnet1 dev_instance1 --wait
```

10. 从 workstation，使用下列参数在 dev_privnet2 网络中启动名为 dev_instance2 的实例：

*    参数	            值
*    --映像	            small
*    --类别	            m2.small
*    --security-group	dev_sg
*    --密钥对	        dev_key
*    --nic 	            net-id=dev_privnet2

```
[student@workstation ~]$ openstack server create --image small --flavor m2.small \
> --security-group dev_sg --key-name dev_key \
> --nic net-id=dev_privnet2 \
> dev_instance2 --wait
```

11. 使用openstack server list命令，查找分配给这些调配的实例的 IP 地址。

```
[student@workstation ~]$ openstack server list
+--------------------------------------+---------------+--------+--------------------------+
| ID                                   | Name          | Status | Networks                 |
+--------------------------------------+---------------+--------+--------------------------+
| 01117214-6c7f-43cc-9e9c-e96ceca511e2 | dev_instance2 | ACTIVE | dev_privnet2=20.20.20.3  |
| 736ea92c-ceeb-4fdd-a439-46019a082d6f | dev_instance1 | ACTIVE | dev_privnet1=10.10.10.3  |
+--------------------------------------+---------------+--------+--------------------------+
```

12. 从 overcloud-controller-0，检索与网络 dev_privnet1 和 dev_privnet2 以及路由器 dev_router 关联的命名空间 UUID。

```
[student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
Last login: Wed Feb 10 00:46:21 2016 from workstation.lab.example.com
[heat-admin@overcloud-controller-0 ~]$ ip netns list
qrouter-5d6a1e56-419a-4826-b882-3a64e5f83a1d
qdhcp-15f94f78-8bc7-469e-a676-e8ac04445a69
qdhcp-6d8016c0-e605-4614-99c3-bf9de795131a
```

13. 在 dev_privnet1 命名空间内，ping 从 dev_privnet1 网络分配的 dev_instance1 的 IP 地址进行验证。该 IP 地址在上一步中获得。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qdhcp-6d8016c0-e605-4614-99c3-bf9de795131a ping -c4 10.10.10.3
PING 10.10.10.3 (10.10.10.3) 56(84) bytes of data.
64 bytes from 10.10.10.3: icmp_seq=1 ttl=64 time=0.478 ms
64 bytes from 10.10.10.3: icmp_seq=2 ttl=64 time=0.534 ms
64 bytes from 10.10.10.3: icmp_seq=3 ttl=64 time=0.496 ms
64 bytes from 10.10.10.3: icmp_seq=4 ttl=64 time=0.565 ms

--- 10.10.10.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 0.478/0.518/0.565/0.037 ms
```

14. 在 dev_privnet2 命名空间内，ping 从 dev_privnet2 网络分配的 dev_instance2 的 IP 地址进行验证。该 IP 地址在上一步中获得。完成时注销。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qdhcp-15f94f78-8bc7-469e-a676-e8ac04445a69 ping -c4 20.20.20.3
PING 20.20.20.3 (20.20.20.3) 56(84) bytes of data.
64 bytes from 20.20.20.3: icmp_seq=1 ttl=64 time=0.992 ms
64 bytes from 20.20.20.3: icmp_seq=2 ttl=64 time=0.484 ms
64 bytes from 20.20.20.3: icmp_seq=3 ttl=64 time=0.496 ms
64 bytes from 20.20.20.3: icmp_seq=4 ttl=64 time=0.447 ms

--- 20.20.20.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3001ms
rtt min/avg/max/mdev = 0.447/0.604/0.992/0.226 ms
[heat-admin@overcloud-controller-0 ~]$ exit
logout
Connection to 172.25.250.100 closed.
[student@workstation ~]$ 
```

15. 为使用 ping 从命名空间外部进行测试，这些实例需要能够从 workstation 进行访问。因此，必须向这些实例添加来自 dev_pubnet 外部网络的外部 IP 地址。在 workstation上，使用 openstack ip floating add 向 dev_instance1 和 dev_instance2 添加外部 IP 地址。

本实验的外部浮动 IP 地址已经由实验的 setup 动词进行了调配。

```
[student@workstation ~]$ openstack ip floating list
+--------------------------------------+-------------+----------------+----------+-------------+
| ID                                   | Pool        | IP             | Fixed IP | Instance ID |
+--------------------------------------+-------------+----------------+----------+-------------+
| 282b691d-6395-4afb-a56b-ad0657627c21 | dev_pubnet  | 172.25.250.131 | None     | None        |
| 471d2a8b-f982-4b8b-b72a-f8ed49b80c8f | dev_pubnet  | 172.25.250.132 | None     | None        |
+--------------------------------------+-------------+----------------+----------+-------------+

[student@workstation ~]$ openstack ip floating add 172.25.250.131 dev_instance1
[student@workstation ~]$ openstack ip floating add 172.25.250.132 dev_instance2
[student@workstation ~]$ openstack server list
+--------------------------------------+---------------+--------+------------------------------------------+
| ID                                   | Name          | Status | Networks                                 |
+--------------------------------------+---------------+--------+------------------------------------------+
| 01117214-6c7f-43cc-9e9c-e96ceca511e2 | dev_instance2 | ACTIVE | dev_privnet2=20.20.20.3, 172.25.250.132  |
| 736ea92c-ceeb-4fdd-a439-46019a082d6f | dev_instance1 | ACTIVE | dev_privnet1=10.10.10.3, 172.25.250.131  |
+--------------------------------------+---------------+--------+------------------------------------------+
```

16. 从 dev_instance1，pingdev_instance2 的 dev_pubnet2 网络 IP 地址。从 workstation，使用其外部地址、与 dev_key 密钥对关联的 /home/student/dev/dev_key.pem 私钥，以及用户名 cloud-user 进行 SSH 连接。

```
[student@workstation ~]$ ssh -i dev/dev_key.pem cloud-user@172.25.250.131
Last login: Wed Feb 10 01:35:56 2016 from 172.25.250.254
[cloud-user@dev_instance1 ~]$ ping -c4 20.20.20.3
PING 20.20.20.3 (20.20.20.3) 56(84) bytes of data.
64 bytes from 20.20.20.3: icmp_seq=1 ttl=63 time=0.740 ms
64 bytes from 20.20.20.3: icmp_seq=2 ttl=63 time=4.78 ms
64 bytes from 20.20.20.3: icmp_seq=3 ttl=63 time=1.02 ms
64 bytes from 20.20.20.3: icmp_seq=4 ttl=63 time=0.875 ms

--- 20.20.20.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.740/1.857/4.787/1.694 ms
```

17. 从 dev_instance1，使用其 IP 地址 172.25.250.254 ping workstation，验证该实例使用 dev_router路由到外部网络。使用 tracepath 验证路由。完成时注销。

```
[cloud-user@dev_instance1 ~]$ ping -c4 172.25.250.254
PING 172.25.250.254 (172.25.250.254) 56(84) bytes of data.
64 bytes from 172.25.250.254: icmp_seq=1 ttl=63 time=0.484 ms
64 bytes from 172.25.250.254: icmp_seq=2 ttl=63 time=0.796 ms
64 bytes from 172.25.250.254: icmp_seq=3 ttl=63 time=0.711 ms
64 bytes from 172.25.250.254: icmp_seq=4 ttl=63 time=0.676 ms

--- 172.25.250.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3001ms
rtt min/avg/max/mdev = 0.484/0.666/0.796/0.118 ms
[cloud-user@dev_instance1 ~]$ tracepath 172.25.250.254
 1?: [LOCALHOST]                                         pmtu 1300
 1:  gateway                                               0.929ms
 1:  gateway                                               0.688ms
 2:  172.25.250.254                                        0.691ms reached
     Resume: pmtu 1300 hops 2 back 2
[cloud-user@dev_instance1 ~]$ exit
logout
Connection to 172.25.250.131 closed.
[student@workstation ~]$ 
```


### 实验：管理网络

1. 从workstation打开一个终端，再提供/home/student/overcloudrc文件中找到的 overcloud 管理凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 创建三个 VXLAN 网络，取名为 prodnet1、prodnet2 和 prodpubnet。

    a. 创建 prodnet1 网络。

```
    [student@workstation ~]$ neutron net-create prodnet1
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 31fff225-d3a7-4e5e-ac6c-24375f3c1bc9 |
    | mtu                       | 0                                    |
    | name                      | prodnet1                             |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 21                                   |
    | qos_policy_id             |                                      |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

    b. 创建 prodnet2 网络。

```
    [student@workstation ~]$ neutron net-create prodnet2
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 09f135e4-f6e6-4e79-8458-d07f1b6ebe00 |
    | mtu                       | 0                                    |
    | name                      | prodnet2                             |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 62                                   |
    | qos_policy_id             |                                      |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

    c. 创建 prodpubnet 网络。

```
    [student@workstation ~]$ neutron net-create prodpubnet
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | 6ee540ac-714b-4995-9d03-dd154fc875f5 |
    | mtu                       | 0                                    |
    | name                      | prodpubnet                           |
    | port_security_enabled     | True                                 |
    | provider:network_type     | vxlan                                |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 16                                   |
    | qos_policy_id             |                                      |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 5e0c6cdea7694c1db57951aba78f0578     |
    +---------------------------+--------------------------------------+
```

3. 创建与 prodnet1 和 prodnet2 对应的子网。使用下表配置子网：

*    prodnet1 的子网
*    参数	    值
*    姓名	    prodsubnet1
*    CIDR 块	    192.168.1.0/24
*    启用 DHCP	对


*    prodnet2 的子网
*    参数	    值
*    名称	    prodsubnet2
*    CIDR 块 	192.168.2.0/24
*    启用 DHCP	对

    a. 为 prodnet1 网络创建子网 prodsubnet1。将 CIDR 块 192.168.1.0/24 用于子网。

```
    [student@workstation ~]$ neutron subnet-create prodnet1 192.168.1.0/24 --name prodsubnet1
    Created a new subnet:
    +-------------------+--------------------------------------------------+
    | Field             | Value                                            |
    +-------------------+--------------------------------------------------+
    | allocation_pools  | {"start": "192.168.1.2", "end": "192.168.1.254"} |
    | cidr              | 192.168.1.0/24                                   |
    | dns_nameservers   |                                                  |
    | enable_dhcp       | True                                             |
    | gateway_ip        | 192.168.1.1                                      |
    | host_routes       |                                                  |
    | id                | 61cbab9f-601c-4d72-8f2f-b99cfa3f0cb5             |
    | ip_version        | 4                                                |
    | ipv6_address_mode |                                                  |
    | ipv6_ra_mode      |                                                  |
    | name              | prodsubnet1                                      |
    | network_id        | 31fff225-d3a7-4e5e-ac6c-24375f3c1bc9             |
    | subnetpool_id     |                                                  |
    | tenant_id         | 5e0c6cdea7694c1db57951aba78f0578                 |
    +-------------------+--------------------------------------------------+
```

    b. 为 prodnet2 网络创建名为 prodsubnet2 的子网。将 CIDR 块 192.168.2.0/24 用于子网。

```
    [student@workstation ~]$ neutron subnet-create prodnet2 192.168.2.0/24 --name prodsubnet2
    Created a new subnet:
    +-------------------+--------------------------------------------------+
    | Field             | Value                                            |
    +-------------------+--------------------------------------------------+
    | allocation_pools  | {"start": "192.168.2.2", "end": "192.168.2.254"} |
    | cidr              | 192.168.2.0/24                                   |
    | dns_nameservers   |                                                  |
    | enable_dhcp       | True                                             |
    | gateway_ip        | 192.168.2.1                                      |
    | host_routes       |                                                  |
    | id                | cf0da213-32d1-4c1b-8daa-374ea871e9cd             |
    | ip_version        | 4                                                |
    | ipv6_address_mode |                                                  |
    | ipv6_ra_mode      |                                                  |
    | name              | prodsubnet2                                      |
    | network_id        | 09f135e4-f6e6-4e79-8458-d07f1b6ebe00             |
    | subnetpool_id     |                                                  |
    | tenant_id         | 5e0c6cdea7694c1db57951aba78f0578                 |
    +-------------------+--------------------------------------------------+
```

4. 更新子网 prodsubnet1 和 prodsubnet2，将 172.25.250.254 添加为它们的 DNS 名称服务器。

    a. 更新子网 prodsubnet1 ，将 DNS 名称服务器添加为 172.25.250.254。

```
    [student@workstation ~]$ neutron subnet-update prodsubnet1 --dns-nameserver 172.25.250.254
    Updated subnet: prodsubnet1
```

    b. 更新子网 prodsubnet2 ，将 DNS 名称服务器添加为 172.25.250.254。

```
    [student@workstation ~]$ neutron subnet-update prodsubnet2 --dns-nameserver 172.25.250.254
    Updated subnet: prodsubnet2
```

5. 更新网络 prodpubnet，使它成为面向外部的网络。

    a. 更新网络 prodpubnet，使它成为面向外部的网络。使用 --router:external 参数。

```
    [student@workstation ~]$ neutron net-update prodpubnet --router:external
    Updated network: prodpubnet
```

6. 使用下列参数，为 prodpubnet 创建对应的子网。

*    参数	        值
*    名称	        prodpubsubnet
*    CIDR 块	        172.25.250.0/24
*    分配池	        172.25.250.130 到 172.25.250.199
*    启用 DHCP	    错
*    网关	        172.25.250.254
*    DNS 名称服务器	172.25.250.254

    a. 为 prodpubnet 创建名为 prodpubsubnet 的子网。使用 CIDR 块 172.25.250.0/24，以及从 172.25.250.130 到 172.25.250.199 的分配池。禁用 DHCP，并将 172.25.250.254 用作子网的网关和 DNS 名称服务器。

```
    [student@workstation ~]$ neutron subnet-create prodpubnet 172.25.250.0/24 \
    > --name prodpubsubnet --disable-dhcp \
    > --allocation-pool start=172.25.250.130,end=172.25.250.199 \
    > --gateway 172.25.250.254 --dns-nameserver 172.25.250.254
    Created a new subnet:
    +-------------------+------------------------------------------------------+
    | Field             | Value                                                |
    +-------------------+------------------------------------------------------+
    | allocation_pools  | {"start": "172.25.250.130", "end": "172.25.250.199"} |
    | cidr              | 172.25.250.0/24                                      |
    | dns_nameservers   | 172.25.250.254                                       |
    | enable_dhcp       | False                                                |
    | gateway_ip        | 172.25.250.254                                       |
    | host_routes       |                                                      |
    | id                | c20bbe88-66e7-465a-a338-1b8b8fb93916                 |
    | ip_version        | 4                                                    |
    | ipv6_address_mode |                                                      |
    | ipv6_ra_mode      |                                                      |
    | name              | prodpubsubnet                                        |
    | network_id        | 6ee540ac-714b-4995-9d03-dd154fc875f5                 |
    | subnetpool_id     |                                                      |
    | tenant_id         | 5e0c6cdea7694c1db57951aba78f0578                     |
    +-------------------+------------------------------------------------------+
```

7. 创建名为 prodrouter 的路由器。

    a. 创建一个路由器，将它命名为 prodrouter。

```
    [student@workstation ~]$ neutron router-create prodrouter
    Created a new router:
    +-----------------------+--------------------------------------+
    | Field                 | Value                                |
    +-----------------------+--------------------------------------+
    | admin_state_up        | True                                 |
    | distributed           | False                                |
    | external_gateway_info |                                      |
    | ha                    | False                                |
    | id                    | bf98a467-dd76-4b0f-8368-512e051dd350 |
    | name                  | prodrouter                           |
    | routes                |                                      |
    | status                | ACTIVE                               |
    | tenant_id             | 5e0c6cdea7694c1db57951aba78f0578     |
    +-----------------------+--------------------------------------+
```

8. 将两个租户子网 prodsubnet1 和 prodsubnet2 连接到 prodrouter 路由器。

    a. 将子网 prodsubnet1 和 prodsubnet2 连接到 prodrouter 路由器。

```
    [student@workstation ~]$ neutron router-interface-add prodrouter prodsubnet1
    Added interface 99b2020c-80a5-4fcf-a831-85b8b3003a18 to router prodrouter.

    [student@workstation ~]$ neutron router-interface-add prodrouter prodsubnet2
    Added interface 7bfe110e-3620-43ce-92fa-f3066bbb9140 to router prodrouter.
```

9. 将面向外部的网络 prodpubnet 作为网关连接到路由器 prodrouter。

    a. 将外部网络 prodpubnet 作为网关连接到路由器 prodrouter。

```
    [student@workstation ~]$ neutron router-gateway-set prodrouter prodpubnet
    Set gateway for router prodrouter
```

10. 使用以下参数，在 prodnet1 网络中启动 prodinstance1。本实验的映像、安全组和规则以及密钥对已使用 setup 脚本进行了调配。

*    参数	值
*    名称	prodinstance1
*    映像	small
*    类别	m2.small
*    密钥对	prodkey
*    安全组	prodsg
*    网络	prodnet1

    a. 使用类别 m2.small、映像 small、安全组 prodsg 以及密钥对 prodkey 在 prodnet1 网络内启动实例 prodinstance1。本实验的映像、安全组和规则以及密钥对已进行了调配。

```
    [student@workstation ~]$ openstack server create \
    > --image small --flavor m2.small --security-group prodsg \
    > --key-name prodkey --nic net-id=prodnet1 prodinstance1
```

11. 使用以下参数，在 prodnet2 网络中启动 prodinstance2。本实验的映像、安全组和规则以及密钥对已使用 setup 脚本进行了调配。

*    参数	值
*    名称	prodinstance2
*    映像	small
*    类别	m2.small
*    密钥对	prodkey
*    安全组	prodsg
*    网络	prodnet2

    a. 使用类别 m2.small、映像 small、安全组 prodsg 以及密钥对 prodkey 在 prodnet2 网络内启动实例 prodinstance2。本实验的映像、安全组和规则以及密钥对已进行了调配。

```
    [student@workstation ~]$ openstack server create \
    > --image small --flavor m2.small --security-group prodsg \
    > --key-name prodkey --nic net-id=prodnet2 prodinstance2
```

12. 创建两个面向外部的浮动 IP 地址，并将它们关联到 prodinstance1 和 prodinstance2。在关联 IP 地址之前，这两个实例的状态应当是 ACTIVE。通过 openstack server list 进行验证。

使用 openstack ip floating create PUBLIC-NETWORK 命令。要分配浮动 IP 地址，可使用openstack ip floating add IPADDR INSTANCENAME。使用openstack ip floating list验证其关联。

    a. 使用外部网络 prodpubnet 创建两个面向外部的浮动 IP 地址。使用openstack ip floating create命令两次，以生成两个浮动 IP 地址。

```
    [student@workstation ~]$ openstack ip floating create prodpubnet
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | c39b3a18-5e9e-49e5-b18d-7b69ca55a49b |
    | instance_id | None                                 |
    | ip          | 172.25.250.131                       |
    | pool        | prodpubnet                           |
    +-------------+--------------------------------------+
    [student@workstation ~]$ openstack ip floating create prodpubnet
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | c2e4463c-226b-4a61-ac36-fe0484259489 |
    | instance_id | None                                 |
    | ip          | 172.25.250.132                       |
    | pool        | prodpubnet                           |
    +-------------+--------------------------------------+
```

    b. 在继续下一步之前，检查 prodinstance1 和 prodinstance2 实例的状态是否为 ACTIVE。使用 openstack server list 命令。

```
    [student@workstation ~]$ openstack server list
    +--------------------------------------+---------------+--------+----------------------+
    | ID                                   | Name          | Status | Networks             |
    +--------------------------------------+---------------+--------+----------------------+
    | 025d52a5-055e-464b-936a-5f0eff383ff2 | prodinstance2 | ACTIVE | prodnet2=192.168.2.3 |
    | 92fbfadd-b6f5-4f06-9abb-c0a309b44505 | prodinstance1 | ACTIVE | prodnet1=192.168.1.3 |
    +--------------------------------------+---------------+--------+----------------------+
```

    c. 将浮动 IP 地址关联到 prodinstance1 和 prodinstance2。使用 openstack ip floating add 命令；这不会产生任何输出。使用openstack ip floating list验证其关联。

```
    [student@workstation ~]$ openstack ip floating add 172.25.250.131 prodinstance1
    [student@workstation ~]$ openstack ip floating add 172.25.250.132 prodinstance2
    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+------------+----------------+-------------+--------------------------------------+
    | ID                                   | Pool       | IP             | Fixed IP    | Instance ID                          |
    +--------------------------------------+------------+----------------+-------------+--------------------------------------+
    | c2e4463c-226b-4a61-ac36-fe0484259489 | prodpubnet | 172.25.250.132 | 192.168.2.3 | 025d52a5-055e-464b-936a-5f0eff383ff2 |
    | c39b3a18-5e9e-49e5-b18d-7b69ca55a49b | prodpubnet | 172.25.250.131 | 192.168.1.3 | 92fbfadd-b6f5-4f06-9abb-c0a309b44505 |
    +--------------------------------------+------------+----------------+-------------+--------------------------------------+ 
```

13. 使用openstack server list，检查分配给 prodinstance1 和 prodinstance2 的租户网络和外部 IP 地址。

    a. 使用openstack server list，检查分配给 prodinstance1 和 prodinstance2 的租户网络和外部 IP 地址。

```
    [student@workstation ~]$ openstack server list
    +--------------------------------------+---------------+--------+--------------------------------------+
    | ID                                   | Name          | Status | Networks                             |
    +--------------------------------------+---------------+--------+--------------------------------------+
    | 025d52a5-055e-464b-936a-5f0eff383ff2 | prodinstance2 | ACTIVE | prodnet2=192.168.2.3, 172.25.250.132 |
    | 92fbfadd-b6f5-4f06-9abb-c0a309b44505 | prodinstance1 | ACTIVE | prodnet1=192.168.1.3, 172.25.250.131 |
    +--------------------------------------+---------------+--------+--------------------------------------+
```

14. 从workstation，通过登录 prodinstance1 并且 ping 分配自 prodnet2 网络的 prodinstance2 IP 地址来进行验证。

使用 prodkey 密钥对的私钥文件/home/student/prodkey.pem并以 cloud-user用户身份获取对 prodinstance1 和 prodinstance2 的访问权限。

    a. 从workstation，以 cloud-user 用户身份并使用私钥/home/student/prodkey.pem，通过 SSH 连接 prodinstance1。

```
    [student@workstation ~]$ ssh -i prodkey.pem cloud-user@172.25.250.131
    Last login: Wed Feb 11 01:35:56 2016 from 172.25.250.254
    [cloud-user@prodinstance1 ~]$ 
```

    b. 使用其分配自 prodnet2 网络的租户 IP 地址 ping prodinstance2。它应当能成功发送使用 prodrouter 进行路由的数据包。

```
    [cloud-user@prodinstance1 ~]$ ping -c4 192.168.2.3
    PING 192.168.2.3 (192.168.2.3) 56(84) bytes of data.
    64 bytes from 192.168.2.3: icmp_seq=1 ttl=63 time=0.740 ms
    64 bytes from 192.168.2.3: icmp_seq=2 ttl=63 time=4.78 ms
    64 bytes from 192.168.2.3: icmp_seq=3 ttl=63 time=1.02 ms
    64 bytes from 192.168.2.3: icmp_seq=4 ttl=63 time=0.875 ms

    --- 192.168.2.3 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 0.740/1.857/4.787/1.694 ms
```

15. 使用其主机名从 prodinstance1 pingworkstation.lab.example.com进行验证。这应当会成功，因为路由器 prodrouter 已连接有外部网络 prodpubnet 作为其网关并且它使用了 NAT，因而能被访问到。与 prodsubnet1 连接的名称服务器 172.25.250.254 帮助执行名称解析。

验证之后，从 prodinstance1 的远程登录会话退出。

    a. 从 prodinstance1 ping workstation.lab.example.com 进行验证。

```
    [cloud-user@prodinstance1 ~]$ ping -c4 workstation.lab.example.com
    PING workstation.lab.example.com (172.25.250.254) 56(84) bytes of data.
    64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=1 ttl=64 time=0.169 ms
    64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=2 ttl=64 time=0.139 ms
    64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=3 ttl=64 time=0.225 ms
    64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=4 ttl=64 time=0.252 ms

    --- workstation.lab.example.com ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3001ms
    rtt min/avg/max/mdev = 0.139/0.196/0.252/0.045 ms
    [cloud-user@prodinstance1 ~]$ exit
```

16. 从控制器节点访问其命名空间，验证 prodrouter 上设置的 NAT 规则。使用ip netns exec检查 NAT 规则。

使用 /home/student/overcloudrc 文件的 OS_AUTH_URL 环境变量中列出的 IP 地址，并使用用户名 heat-admin 和私钥文件 /home/student/overcloud.pem 连接到 overcloud 控制器节点。

验证之后，从节点 overcloud-controller-0 的远程登录会话退出。

    a. 从 workstation，使用私钥 /home/student/overcloud.pem 和用户名 heat-admin 通过 SSH 连接 overcloud 控制器节点 overcloud-controller-0。IP 地址将是来自 overcloudrc 文件的 OS_AUTH_URL，因为控制器节点 overcloud-controller-0 上部署了 Keystone。

```
    [student@workstation ~]$ echo $OS_AUTH_URL
    http://192.168.0.11:5000/v2.0/
    [student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
    Last login: Thu Feb 11 06:51:49 2016 from workstation.lab.example.com
    [heat-admin@overcloud-controller-0 ~]$ 
```

    b. 使用 ip netns list 命令列出创建的命名空间。

```
    [heat-admin@overcloud-controller-0 ~]$ sudo ip netns list
    qrouter-2c719f0b-93b5-4d13-9557-9472b241e7ea
    qdhcp-8b12921e-f51a-4808-8243-bb2220b25b0d
    qrouter-77a5930b-fcf3-47ec-8634-90d028d3409c
    qdhcp-09f135e4-f6e6-4e79-8458-d07f1b6ebe00
    qdhcp-31fff225-d3a7-4e5e-ac6c-24375f3c1bc9
```

    c. 验证与 prodrouter 关联的 qrouter-XXX。
    
    具有 qr-XXX 的 prodsubnet1 拥有来自 CIDR 块 192.168.1.0/24 的 IP 地址 192.168.1.1/24，prodsubnet2 则拥有来自 CIDR 块 192.168.2.0/24 的IP 地址 192.168.2.1/24。也请注意，qg-XXX 是与 prodrouter 关联的网关。

    前面列出的 XXX 对应于这些子网所连接的路由器中端口的 UUID (neutron router-port-list prodrouter)。

```
    [heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec \
    > qrouter-77a5930b-fcf3-47ec-8634-90d028d3409c \
    > ip addr show
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    16: qr-b88e70e0-dd: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
        link/ether fa:16:3e:a5:3a:7d brd ff:ff:ff:ff:ff:ff
        inet 192.168.2.1/24 brd 192.168.2.255 scope global qr-b88e70e0-dd
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fea5:3a7d/64 scope link 
           valid_lft forever preferred_lft forever
    17: qr-c1b2c814-3f: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
        link/ether fa:16:3e:18:28:20 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.1/24 brd 192.168.1.255 scope global qr-c1b2c814-3f
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe18:2820/64 scope link 
           valid_lft forever preferred_lft forever
    18: qg-6b395073-f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
        link/ether fa:16:3e:40:d0:92 brd ff:ff:ff:ff:ff:ff
        inet 172.25.250.133/24 brd 172.25.250.255 scope global qg-6b395073-f0
           valid_lft forever preferred_lft forever
        inet 172.25.250.131/32 brd 172.25.250.134 scope global qg-6b395073-f0
           valid_lft forever preferred_lft forever
        inet 172.25.250.132/32 brd 172.25.250.135 scope global qg-6b395073-f0
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe40:d092/64 scope link 
           valid_lft forever preferred_lft forever
```

    d. 使用ip netns exec验证路由器上设置的 NAT 规则。

```
    [heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec \
    > qrouter-77a5930b-fcf3-47ec-8634-90d028d3409c \
    > iptables -t nat -L
    ...Output omitted...
    Chain neutron-l3-agent-OUTPUT (1 references)
    target     prot opt source               destination
    DNAT       all  --  anywhere             172.25.250.132       to:192.168.2.3
    DNAT       all  --  anywhere             172.25.250.131       to:192.168.1.3

    Chain neutron-l3-agent-POSTROUTING (1 references)
    target     prot opt source               destination
    ACCEPT     all  --  anywhere             anywhere             ! ctstate DNAT

    Chain neutron-l3-agent-PREROUTING (1 references)
    target     prot opt source               destination
    REDIRECT   tcp  --  anywhere             169.254.169.254      tcp dpt:http redir ports 9697
    DNAT       all  --  anywhere             172.25.250.132       to:192.168.2.3
    DNAT       all  --  anywhere             172.25.250.131       to:192.168.1.3

    Chain neutron-l3-agent-float-snat (1 references)
    target     prot opt source               destination
    SNAT       all  --  192.168.2.3          anywhere             to:172.25.250.132
    SNAT       all  --  192.168.1.3          anywhere             to:172.25.250.131

    Chain neutron-l3-agent-snat (1 references)
    target     prot opt source               destination
    neutron-l3-agent-float-snat  all  --  anywhere             anywhere
    SNAT       all  --  anywhere             anywhere             to:172.25.250.130
    SNAT       all  --  anywhere             anywhere             mark match ! 0x2/0xffff ctstate DNAT to:172.25.250.130

    Chain neutron-postrouting-bottom (1 references)
    target     prot opt source               destination
    neutron-l3-agent-snat  all  --  anywhere             anywhere             /* Perform source NAT on outgoing traffic. */
```

17. 从 workstation，删除现有的网络 prod-oldnet 及其关联的子网 prod-oldsubnet。因为子网已作为端口连接到路由器 prod-oldrouter，因此需要先删除它。最后，验证没有关联任何其他端口后，将 prod-oldrouter 路由器删除。

    a. 从 workstation，列出实验环境中存在的网络、子网和路由器。

```
    [student@workstation ~]$ neutron net-list
    +--------------------------------------+-------------+------------------------------------------------------+
    | id                                   | name        | subnets                                              |
    +--------------------------------------+-------------+------------------------------------------------------+
    | 8b12921e-f51a-4808-8243-bb2220b25b0d | prod-oldnet | e35850a1-f1fb-47dc-ba49-a773480b919c 10.10.10.0/24   |
    | 31fff225-d3a7-4e5e-ac6c-24375f3c1bc9 | prodnet1    | 61cbab9f-601c-4d72-8f2f-b99cfa3f0cb5 192.168.1.0/24  |
    | 09f135e4-f6e6-4e79-8458-d07f1b6ebe00 | prodnet2    | cf0da213-32d1-4c1b-8daa-374ea871e9cd 192.168.2.0/24  |
    | 6ee540ac-714b-4995-9d03-dd154fc875f5 | prodpubnet  | c20bbe88-66e7-465a-a338-1b8b8fb93916 172.25.250.0/24 |
    +--------------------------------------+-------------+------------------------------------------------------+
    
    [student@workstation ~]$ neutron subnet-list
    +--------------------------------------+---------------+-----------------+------------------------------------------------------+
    | id                                   | name          | cidr            | allocation_pools                                     |
    +--------------------------------------+---------------+-----------------+------------------------------------------------------+
    | e35850a1-f1fb-47dc-ba49-a773480b919c | prod-oldsunet | 10.10.10.0/24   | {"start": "10.10.10.2", "end": "10.10.10.254"}       |
    | 61cbab9f-601c-4d72-8f2f-b99cfa3f0cb5 | prodsubnet1   | 192.168.1.0/24  | {"start": "192.168.1.2", "end": "192.168.1.254"}     |
    | cf0da213-32d1-4c1b-8daa-374ea871e9cd | prodsubnet2   | 192.168.2.0/24  | {"start": "192.168.2.2", "end": "192.168.2.254"}     |
    | c20bbe88-66e7-465a-a338-1b8b8fb93916 | prodpubsubnet | 172.25.250.0/24 | {"start": "172.25.250.130", "end": "172.25.250.199"} |
    +--------------------------------------+---------------+-----------------+------------------------------------------------------+
    
    [student@workstation ~]$ neutron router-list -c id -c name
    +--------------------------------------+----------------+
    | id                                   | name           |
    +--------------------------------------+----------------+
    | 2c719f0b-93b5-4d13-9557-9472b241e7ea | prod-oldrouter |
    | 77a5930b-fcf3-47ec-8634-90d028d3409c | prodrouter     |
    +--------------------------------------+----------------+
```

    b. 使用 neutron router-port-list 命令，列出与 prod-oldrouter 连接的端口。

```
    [student@workstation ~]$ neutron router-port-list prod-oldrouter
    +--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
    | id                                   | name | mac_address       | fixed_ips                                                                           |
    +--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
    | 297d2525-b06c-4c47-b22e-e8315130edf3 |      | fa:16:3e:44:73:61 | {"subnet_id": "e35850a1-f1fb-47dc-ba49-a773480b919c",  "ip_address": "10.10.10.1"}  |
    +--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
```

    c. 因为 prod-oldsubnet 已作为端口连接到 prod-oldrouter，先从路由器删除该接口。

```
    [student@workstation ~]$ neutron router-interface-delete prod-oldrouter prod-oldsubnet
    Removed interface from router prod-oldrouter.
```

    d. 删除子网 prod-oldsubnet，再删除该网络。

```
    [student@workstation ~]$ neutron subnet-delete prod-oldsubnet
    Deleted subnet: prod-oldsubnet
```

    e. 删除 prod-oldnet 网络。

```
    [student@workstation ~]$ neutron net-delete prod-oldnet
    Deleted network: prod-oldnet
```

    f. 路由器 prod-oldrouter 现在可以删除，因为已没有连接的端口。

```
    [student@workstation ~]$ neutron router-delete prod-oldrouter
    Deleted router: prod-oldrouter
```


### 总结

在本章中，您可以学到：

*    租户网络可通过各种底层技术进行调配，如 VLAN、GRE 和 VXLAN。
*    软件 L2 插件依赖于 Linux 桥接模块或 Open vSwitch 为各个租户提供网络隔离，并提供对网络资源的共享访问。
*    虚拟可扩展局域网 (VXLAN) 提供了解决方案，可解决与使用 VLAN 相关的扩展性问题，即并发租户网络数量限制为最多 4096 个。VXLAN 通过添加 24 位网段 ID，将它扩展到最多 1600 万个并发租户网络。VXLAN 在 L3 网络上封装 L2 网络。
*    网络命名空间拥有自己的网络路由、自己的防火墙规则，以及自己的网络设备。
*    在 OpenStack 中，TAP 设备由支持的系统管理程序（如 KVM 和 Xen）创建，它们实施虚拟网络接口卡，通常称为 VIF 或 VNIC。
