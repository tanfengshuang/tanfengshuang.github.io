---
layout: post
title:  "管理浮动 IP 地址(210-4)"
categories: Linux
tags: RHCA 210
---

### 配置外部网络

###### 浮动 IP 地址

在云计算中，浮动 IP 地址是可从外部访问的静态 IP 地址。它们使得管理员能够将所选实例的专用网络路由到外部世界。在 OpenStack 中，浮动 IP 由 Neutron 服务管理，它使用一组底层 Linux 服务（如 Netfilter 或 Open vSwitch）创建必要的配置。Nova 服务可以访问 Neutron 管理的池，从而将浮动 IP 关联到实例。以下示意图演示了如何从外部世界向专用网络中的实例执行这一路由。 

默认情况下，OpenStack 设有每个项目 50 个浮动 IP 的配额。浮动 IP 取消关联之后，它就返回到池中，可以重新关联到其他实例。若要从项目取消分配 IP 地址，该 IP 地址必须在取消关联后 释放。但是，浮动 IP 被释放后，就不保证能够再次分配到同一个 IP 地址。Neutron 将第一个可用浮动 IP 地址用于分配。

> 注意: 创建的浮动 IP 使用无类别域间路由 (CIDR)，后者是针对子网而定义。例如，当您创建 10.0.10.0/24 子网时，浮动 IP 10.0.10.50/32 将可用于分配。然后，管理员必须确保此范围能被公开路由，因为 Neutron 不清楚外部网络拓扑。 

要为实例提供外部连接，管理员必须：

1. 定义 Neutron 外部网络。管理员可以利用neutron net-update --router:external NETWORK命令将网络定义为外部网络。
2. 定义 Neutron 专用网络。
3. 创建一个同时具有这两个网络中的端口的路由器，外部网络中的端口设为网关。
4. 在/etc/nova/nova.conf中定义默认浮动 IP 地址池。该变量名为 default_floating_pool。
5. 使用命令行或 Horizon 控制面板，在池中分配一组浮动 IP 地址。
6. 将浮动 IP 地址关联到该实例。

关联了浮动 IP 地址后，Neutron 将使用一组网桥和 iptables 规则自动配置计算节点和网络节点，以及计算节点上的 IP 地址别名。

> 注意: 管理员还要定义一套安全组规则，确保实例可被访问。若无适当的安全规则，将无法访问实例。

###### 取消分配并释放浮动 IP 地址

浮动 IP 地址可以从实例 取消分配，使它立即可用于后续分配。这样做可腾出该 IP 地址，但也会清除已执行的任何现有系统配置，如 iptables 规则。管理员也可以 释放浮动 IP 地址，使它可用于其他项目。如果在实例终止时依然关联有浮动 IP 地址，则该地址将可用于所在的项目；它不会从浮动 IP 地址池中释放出来。每一浮动 IP 地址占据外部网络中的一个 Neutron 端口。

> 注意: 必须先从实例取消关联浮动 IP 地址，然后才能释放该地址。

###### L3 路由和 NAT

为提供外部连接，Neutron 使用由第 3 层 (L3) 路由器提供的网络地址转换 (NAT) 机制。通过路由器，Neutron 实施来源网络地址转换 (SNAT) 和目的地网络地址转换 (DNAT)。这会创建一个静态一对一映射，从外部网络上的公共 IP 地址映射到路由器可以访问的子网（非路由专用网络）的专用 IP 地址。Neutron L3 代理使用 iptables 实施浮动 IP 地址，来实现网络地址转换。浮动 IP 地址不与实例直接关联；浮动 IP 地址关联的是 Neutron 端口。

> 警告: 由于浮动 IP 地址使用外部网络中的端口，管理员无法删除该网络，直至所有浮动 IP 地址都从池中释放出来。

###### 浮动 IP 地址工作原理

当浮动 IP 地址分配给实例后，便可从租户网络以外的数据中心网络访问实例。登录实例时，它将显示与租户网络/固定 IP 地址关联的网络配置。所有将数据包发送到外部网络的路由发生于计算和控制器节点。

假设设置中具有一个 Nova 计算节点和一个带有物理路由器的控制器节点。Nova 实例是分配有浮动 IP 地址的 Web 服务器，应当能够从公共网络进行访问。

以下步骤涉及到计算与控制器节点：

1. 实例 (VM) 将其数据包转发到 Linux 网桥 qbr。
2. 使用安全组规则通过 qbr处理规则后，数据包转发到 Open vSwitch 集成网桥 br-int。
3. 该集成网桥连接项目网络和隧道网桥 br-tun。
4. 隧道网桥 br-tun 封装数据包，并将它们通过 br-tun 隧道网桥传递到控制器主机。
5. 数据包通过隧道网桥 br-tun 解封，并传递到控制器节主机上的集成网桥 br-int。
6. Open vSwitch 集成网桥 br-int 将数据包转发到项目的 Open vSwitch 路由器命名空间 qrouter。
7. 路由器命名空间 qrouter 为数据包执行 SNAT 运算，其包括将数据包从浮动 IP 地址路由到实例的固定 IP 地址。对于出口流量，数据包被转发到 Open vSwitch 网桥 br-ex。
8. 连接外部网络的 Open vSwitch 网桥 br-ex 转发数据包。

###### 网络问题故障排除

如果实例在关联了浮动 IP 后依然无法访问，则需要检查其配置来进行故障排除。

*    验证浮动 IP 网络是否配置为外部网络。
*    验证实例能否ping路由器 IP 地址。
*    验证 br-ex 是否已正确配置到物理网络，可使用ovs-vsctl show命令进行调查。
*    检查能否从路由器命名空间访问外部 IP 地址。
*    若使用 VLAN，验证交换机是否允许这些 VLAN ID。 


### 练习：配置外部网络

1. 从workstation打开一个终端，再提供 overcloud 凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 使用 neutron net-list 命令，检查创建的网络。

```
[student@workstation ~]$ neutron net-list -c id -c name
+--------------------------------------+-------------+
| id                                   | name        |
+--------------------------------------+-------------+
| 8c75b7d2-385d-4234-9da5-ada1da1943b7 | dev_pubnet  |
| c59d6147-ecb6-455c-8cc5-34fbb70badd9 | dev_privnet |
+--------------------------------------+-------------+
```

3. 在 dev_privnet 网络中部署一个新实例。将实例命名为 dev-floating-vm，并使用类别 m1.small、密钥对 devkey 以及映像名称 web。

```
[student@workstation ~]$ openstack server create --flavor m1.small --image web --key-name devkey --nic net-id=dev_privnet dev-floating-vm
[student@workstation ~]$ openstack server list
+--------------------------------------+-----------------+--------+--------------------------+
| ID                                   | Name            | Status | Networks                 |
+--------------------------------------+-----------------+--------+--------------------------+
| 5321c708-ff05-4fcd-a173-77617f1a7db2 | dev-floating-vm | ACTIVE | dev_privnet=192.168.1.16 |
+--------------------------------------+-----------------+--------+--------------------------+
```

4. 创建网络 dev_pubnet，作为外部网络。

```
[student@workstation ~]$ neutron net-update --router:external dev_pubnet
Updated network: dev_pubnet
```

5. 将公共网络 dev_pubnet 连接为路由器 dev_router1 的网关。

```
[student@workstation ~]$ neutron router-list
+--------------------------------------+-------------+-----------------------+-------------+-------+
| id                                   | name        | external_gateway_info | distributed | ha    |
+--------------------------------------+-------------+-----------------------+-------------+-------+
| c6090649-3ca3-4f42-9537-60e57bd25826 | dev_router1 | null                  | False       | False |
+--------------------------------------+-------------+-----------------------+-------------+-------+

[student@workstation ~]$ neutron router-gateway-set dev_router1 dev_pubnet
Set gateway for router dev_router1
```

6. 创建一个与 dev_pubnet 公共网络关联的浮动 IP 地址。

```
[student@workstation ~]$ openstack ip floating pool list
+------------+
| Name       |
+------------+
| dev_pubnet |
+------------+
[student@workstation ~]$ openstack ip floating create dev_pubnet
+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| fixed_ip    | None                                 |
| id          | 07758d47-85d4-49c3-b943-37ede0f12121 |
| instance_id | None                                 |
| ip          | 172.25.250.51                        |
| pool        | dev_pubnet                           |
+-------------+--------------------------------------+
```

7. 将上一步中创建的浮动 IP 地址关联到 dev-floating-vm 实例。

```
[student@workstation ~]$ openstack ip floating list
+--------------------------------------+------------+---------------+----------+-------------+
| ID                                   | Pool       | IP            | Fixed IP | Instance ID |
+--------------------------------------+------------+---------------+----------+-------------+
| 07758d47-85d4-49c3-b943-37ede0f12121 | dev_pubnet | 172.25.250.51 | None     | None        |
+--------------------------------------+------------+---------------+----------+-------------+
[student@workstation ~]$ openstack ip floating add 172.25.250.51 dev-floating-vm
[student@workstation ~]$ openstack ip floating list
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
| ID                                   | Pool       | IP            | Fixed IP     | Instance ID                          |
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
| 07758d47-85d4-49c3-b943-37ede0f12121 | dev_pubnet | 172.25.250.51 | 192.168.1.16 | 5321c708-ff05-4fcd-a173-77617f1a7db2 |
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
```

8. 验证可以使用浮动 IP 地址 172.25.250.51 从外部访问 dev-floating-vm 实例上的 Web 页面。

```
[student@workstation ~]$ curl http://172.25.250.51
My web page
```

9. 使用其浮动 IP 地址和 /home/student/devkey.pem 私钥进行登录，验证 dev-floating-vm 实例可以访问外部网络。使用 ping 验证实例是否能访问课堂网络。完成时从实例注销。

```
[student@workstation ~]$ ssh -i devkey.pem cloud-user@172.25.250.51
Warning: Permanently added '172.25.250.51' (ECDSA) to the list of known hosts.
[cloud-user@dev-floating-vm ~]$ ping -c3 workstation
PING workstation (172.25.250.254) 56(84) bytes of data.
64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=1 ttl=63 time=0.597 ms
64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=2 ttl=63 time=0.615 ms
64 bytes from workstation.lab.example.com (172.25.250.254): icmp_seq=3 ttl=63 time=0.743 ms

--- workstation ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.597/0.651/0.743/0.071 ms
[cloud-user@dev-floating-vm ~]$ logout
Connection to 172.25.250.51 closed.
```

10. 从 workstation，使用其面向外部的 IP 地址 192.168.0.11 和位于 /home/student/overcloud.pem 下的私钥，连接 overcloud 控制器节点 overcloud-controller-0。

```
[student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
Last login: Mon Jun 13 10:33:10 2016 from director.lab.example.com
[heat-admin@overcloud-controller-0 ~]$ 
```

11. 从 overcloud-controller-0，检索路由器使用的命名空间的名称。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns list
qrouter-c6090649-3ca3-4f42-9537-60e57bd25826
qdhcp-c59d6147-ecb6-455c-8cc5-34fbb70badd9
```

12. 验证路由器命名空间中存在 qr-* 和 qg-* 端口。这两个端口分别将路由器连接到集成网桥 br-int 和提供商网桥 br-ex。

第一接口 qg-2ab66fe3-39 将路由器连接到由 neutron router-gateway-set 命令设置的网关。第二接口 qr-9ffb2be3-c3 将路由器与集成网桥连接：

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qrouter-c6090649-3ca3-4f42-9537-60e57bd25826 ip addr
...command output omitted...
46: qr-9ffb2be3-c3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:25:78:13 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.5/24 brd 192.168.1.255 scope global qr-9ffb2be3-c3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe25:7813/64 scope link
       valid_lft forever preferred_lft forever
47: qg-2ab66fe3-39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
    link/ether fa:16:3e:80:ad:e2 brd ff:ff:ff:ff:ff:ff
    inet 172.25.250.50/24 brd 172.25.250.255 scope global qg-2ab66fe3-39
       valid_lft forever preferred_lft forever
    inet 172.25.250.51/32 brd 172.25.250.51 scope global qg-2ab66fe3-39
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe80:ade2/64 scope link
       valid_lft forever preferred_lft forever
```

13. 路由器命名空间内的 netfilter nat 表负责将浮动 IP 地址与实例关联。

当您将浮动 IP 地址与实例关联时，此表中会创建类似的规则。SNAT 和 DNAT 规则映射浮动地址 172.25.250.51 和专用地址 192.168.1.16 之间的流量。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qrouter-c6090649-3ca3-4f42-9537-60e57bd25826 iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-PREROUTING  all  --  anywhere             anywhere

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-OUTPUT  all  --  anywhere             anywhere

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-POSTROUTING  all  --  anywhere             anywhere
neutron-postrouting-bottom  all  --  anywhere             anywhere

Chain neutron-l3-agent-OUTPUT (1 references)
target     prot opt source               destination
DNAT       all  --  anywhere             172.25.250.51        to:192.168.1.16
...command output omitted...
Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
DNAT       all  --  anywhere             172.25.250.51        to:192.168.1.16
REDIRECT   tcp  --  anywhere             169.254.169.254      tcp dpt:http redir ports 9697

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
SNAT       all  --  192.168.1.16         anywhere             to:172.25.250.51
```

SNAT 规则将实例的来源固定 IP 地址 (192.168.1.16) 转换为分配的浮动 IP 地址 (172.25.250.51)，因而帮助数据包传输到租户网络外部。

DNAT 规则将浮动 IP 地址(172.25.250.51) 转换为实例的固定 IP 地址 (192.168.1.16)，使得传入流量能到达实例。

14. 外部流量通过路由器命名空间中的 qg-2ab66fe3-39 接口流过 br-ex，该接口与 br-ex 连接。

如果查看路由器命名空间中的路由表，我们可以发现外部流量通过路由器命名空间中的 qg-2ab66fe3-39 接口转发。ovs-vsctl show 命令显示 br-ex 关联了 qg-2ab66fe3-39 端口。

完成时，从 overcloud-controller-0 退出。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qrouter-c6090649-3ca3-4f42-9537-60e57bd25826 ip route
default via 172.25.250.254 dev qg-2ab66fe3-39
172.25.250.0/24 dev qg-2ab66fe3-39  proto kernel  scope link  src 172.25.250.50
192.168.1.0/24 dev qr-9ffb2be3-c3  proto kernel  scope link  src 192.168.1.5
[heat-admin@overcloud-controller-0 ~]$ sudo ovs-vsctl show
...command output omitted..
    Bridge br-ex
...command output omitted..
        Port "qg-2ab66fe3-39"
            Interface "qg-2ab66fe3-39"
                type: internal
[heat-admin@overcloud-controller-0 ~]$ exit
```

15. 从 workstation， 取消浮动 IP 地址 172.25.250.51 与 dev-floating-vm 实例的关联。

```
[student@workstation ~]$ openstack ip floating remove 172.25.250.51 dev-floating-vm
```

16. 列出浮动 IP 地址以确认取消关联。

```
[student@workstation ~]$ openstack ip floating list
+--------------------------------------+------------+---------------+----------+-------------+
| ID                                   | Pool       | IP            | Fixed IP | Instance ID |
+--------------------------------------+------------+---------------+----------+-------------+
| 07758d47-85d4-49c3-b943-37ede0f12121 | dev_pubnet | 172.25.250.51 | None     | None        |
+--------------------------------------+------------+---------------+----------+-------------+
```

17. 使用其面向外部的 IP 地址 192.168.0.11 和位于 /home/student/overcloud.pem 下的私钥，连接 overcloud 控制器节点 overcloud-controller-0。

```
[student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
Last login: Mon Jun 13 10:33:10 2016 from director.lab.example.com
[heat-admin@overcloud-controller-0 ~]$ 
```

18. 路由器命名空间内的 netfilter nat 表应当已删除 SNAT 和 DNAT 规则，因为浮动 IP 地址已从实例 dev-floating-vm 取消关联。

完成时，从 overcloud-controller-0 退出。

```
[heat-admin@overcloud-controller-0 ~]$ sudo ip netns exec qrouter-c6090649-3ca3-4f42-9537-60e57bd25826 iptables -t nat -L
...command output omitted...
Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
REDIRECT   tcp  --  anywhere             169.254.169.254      tcp dpt:http redir ports 9697

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
[heat-admin@overcloud-controller-0 ~]$ exit
```


### 管理浮动 IP 地址

###### 关于浮动 IP 地址

如前一节中所述，实例具有动态分配的专用 IP 地址，但也可以拥有公共或浮动 IP 地址。专用 IP 地址用于实例之间的通信，而公共地址则用于和云环境以外的网络进行通信。创建新实例时，假设已经为专用网络配置了 DHCP，则实例将被自动分配专用 IP 地址，它将保持不变，直到实例被显式终止。重新启动实例对专用 IP 地址没有影响。

为了能从外部访问实例，云环境需要浮动 IP 地址 池。池在创建子网期间定义，包含 Neutron 可用于实例的所有可路由公共 IP 地址。在使用浮动 IP 地址时，需要考虑一些事项：

*    在任何给定的时间，实例的每个网络端口只能被分配一个浮动 IP 地址。如果实例具有多个端口 (vNIC)，管理员可以将多个浮动 IP 地址关联到该实例。
*    从实例移除浮动 IP 地址后，该地址立即可供其他实例使用；如果在重新分配后有域名指向实例，则另一实例将提供内容。

> 重要: 在将浮动 IP 地址添加到池前，管理员必须先创建网络并使用 neutron net-update --router:external NETWORK 命令将它声明为外部网络。

###### 管理浮动 IP 地址

管理员可以使用openstack命令管理浮动 IP 地址。可用选项如下：

*    openstack ip floating add: 将浮动 IP 地址添加到实例。
*    openstack ip floating create: 将新的浮动 IP 地址分配到池。
*    openstack ip floating delete: 从池移除浮动 IP 地址。
*    openstack ip floating list: 列出池中现有的浮动 IP 地址。
*    openstack ip floating pool list: 列出现有的浮动 IP 网络。
*    openstack ip floating remove: 释放浮动 IP 地址，将它退回到池。

###### 添加多个浮动 IP 网络

当池中分配的浮动 IP 地址用尽时，可以通过以下方式应对这一状况：

*    更新外部网络的分配池，这将增大浮动 IP 地址范围。
*    在 overcloud 控制器节点上创建额外的浮动 IP 网络，然后在这些网络上添加多个浮动 IP 地址池。

在 Red Hat OpenStack Platform 8 联网环境中，浮动 IP 网络在 br-ex 上配置，后者连接至外部物理网络。

如果 overcloud 部署是用独立于外部网络 (br-ex) 的 VLAN 上的浮动 IP 网络进行的，则部署 overcloud 后需要使用 Neutron 在 overcloud 控制器节点上中继该 VLAN。这样，可以创建与多个网桥连接的多个浮动 IP 地址网络。这些 VLAN 是中继的，而非配置为接口。相反，Neutron 为每个浮动 IP 网络在所选网桥上创建一个具有 VLAN 网段 ID 的 OVS 端口。

######### 使用 OSP-director overcloud 部署添加多个浮动 IP 网络

要创建多个浮动 IP 网络，请确保网络环境文件 (/usr/share/openstack-tripleo-heat-templates/environments/network- environment.yaml) 的参数 NeutronExternalNetworkBridge 设置为空字符串，因为这会将物理接口映射到 br-int，而不直接使用 br-ex。这种模式允许多个浮动 IP 网络使用 VLAN 或多个物理连接。

```
parameter_defaults:
# Set to empty string to enable multiple external networks or VLANs
NeutronExternalNetworkBridge: "''"
```

在 overcloud 部署期间，网络环境文件中设定的参数会更新 controller 节点上 /etc/neutron/l3-agent.ini 中的 external_network_bridge 参数值。此外，overcloud 部署期间 director 节点应该已映射了额外的网桥。

> 注意: 网桥的原生 VLAN 上具有浮动 IP 网络的结果是数据包仅必须遍历一个网桥，而非两个，这可以使流量通过此网络时的 CPU 使用量稍微降低。

> 重要: 下述实施在课堂环境中不可行，因为 director 节点没有从外部关联和连接任何额外的网络设备。 

假设通过 eth0 与第一外部网络 (br-ex) 通信，通过 eth1 与第二外部网络通信，管理员应当配置第二外部网桥 (br-floating) 和接口并添加到 director 节点。

以下步骤演示了如何为 OpenStack overcloud 环境配置具有 br-floating 网桥的网络接口，从而扩展浮动 IP 池。为了能扩展池，需要在 director 节点上创建第二个网络。在下例中，eth1 网络接口与 br-floating Open vSwitch 网桥搭配使用。

    1. 更新使用 br-floating 网桥的 /etc/sysconfig/network-scripts/ifcfg-eth1 网络配置文件。该文件应当如下所示：

    DEVICE=eth1
    TYPE=OVSPort
    DEVICEYPE=ovs
    OVS_BRIDGE=br-floating
    ONBOOT=yes
    NM_CONTROLLED=no
    BOOTPROTO=none

    2. 使用下列内容，创建 /etc/sysconfig/network-scripts/ifcfg-br-floating 配置文件：

    DEVICE=br-floating
    TYPE=OVSBridge
    DEVICETYPE=ovs
    ONBOOT=yes
    NM_CONTROLLED=no
    BOOTPROTO=none

    3. 重新启动 director 节点上的 network 服务。

    [stack@director ~]$ sudo systemctl restart network
              
    4. 在 ~/templates/network-environment.yaml 环境文件中，将 NeutronExternalNetworkBridge 值设为空值 ("''")。

    parameter_defaults:
    # Set to empty string to enable multiple external networks or VLANs
    NeutronExternalNetworkBridge: "''"

    > 注意: 这是两个双引号 (") 括起两个单引号 (')。

    5. 部署 overcloud 环境。

    [stack@director ~]$ openstack overcloud deploy --templates -e \
    > /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml \
    > -e ~/templates/network-environment.yaml \
    > --neutron-bridge-mappings datacenter:br-ex,floating:br-floating

    6. 部署 overcloud 后，提供overcloudrc凭据文件并运行neutron net-create命令来创建外部网络：

    [stack@director ~]$ source ~/overcloudrc
    [stack@director ~]$ neutron net-create pubnet-1  --router:external \
    > --provider:physical_network floating --provider:network_type vlan \
    > --provider:segmentation_id 101

    7. 运行neutron subnet-create，为外部网络创建子网：

    [stack@director ~]$ neutron subnet-create pubnet-1 172.25.250.0/24 \
    > --name pubsubnet-1 \
    > --allocation-pool start=172.25.250.50,end=172.25.250.100 \
    > --gateway 172.25.250.254


### 练习：管理浮动 IP 地址

1. 从 workstation 打开一个终端，再提供 overcloud Keystone 凭据。

```
[student@workstation ~]$ source overcloudrc
```

2. 使用 openstack 命令，确认存在外部网络 dev_pubnet。它是上一节中创建的。

```
[student@workstation ~]$ openstack network list
+--------------------------------------+--------------+--------------------------------------+
| ID                                   | Name         | Subnets                              |
+--------------------------------------+--------------+--------------------------------------+
...Output omitted...
| 42ee2c0a-844a-41e1-9e25-b1de75cb13b9 | dev_pubnet   | 3fce2a47-13e8-463e-a3a1-d3a1c25e02d8 |
+--------------------------------------+--------------+--------------------------------------+
```

3. 检查可用于管理浮动 IP 地址的选项。

```
[student@workstation ~]$ openstack help ip floating
Command "ip" matches:
  ip fixed add
  ip fixed remove
  ip floating add
  ip floating create
  ip floating delete
  ip floating list
  ip floating pool list
  ip floating remove
```

4. 列出为此环境配置的浮动 IP 地址池。

```
[student@workstation ~]$ openstack ip floating pool list
+-------------+
| Name        |
+-------------+
| dev_pubnet  |
+-------------+
```

由于网络 dev_pubnet 已配置为外部网络，这便是用作浮动 IP 地址池的网络。

5. 运行以下命令三次，向这个池分配三个浮动 IP 地址。请注意，您的环境可能使用不同于本例中所示的 IP 地址。

```
[student@workstation ~]$ openstack ip floating create dev_pubnet
+-------------+--------------------------------------+
| Field       | Value                                |
+-------------+--------------------------------------+
| fixed_ip    | None                                 |
| id          | 87e49329-dc77-4887-aa99-a2b9c113ef69 |
| instance_id | None                                 |
| ip          | 172.25.250.52                        |
| pool        | dev_pubnet                           |
+-------------+--------------------------------------+
```

记住要运行此命令三次，从而分配三个浮动 IP 地址。

6. 运行openstack ip floating list命令，检查添加到池中的 IP 地址。

```
[student@workstation ~]$ openstack ip floating list
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
| ID                                   | Pool       | IP            | Fixed IP     | Instance ID                          |
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
| 56f75095-9cbe-4239-8ece-6be77ed46059 | dev_pubnet | 172.25.250.51 | 192.168.1.16 | 6a6ab5b6-9e7a-4398-a0b4-486d3fc0d9ca |
| b78092fc-a5b5-4125-be8f-82e27aa9bbc7 | dev_pubnet | 172.25.250.52 | None         | None                                 |
+--------------------------------------+------------+---------------+--------------+--------------------------------------+
```

> 注意: 如您所见，分配的 IP 地址是相邻的；但是，如果实例已分配有 IP 地址，可能就不是这样。OpenStack 仅分配未关联到任何实例并且未被其他用户分配的浮动 IP 地址。

7. 实验脚本创建了名为 dev_test-remove-floating 的实例，该实例关联了一个浮动 IP 地址。使用 openstack 命令，检索与该实例相关的详细信息。

```
[student@workstation ~]$ openstack server show dev_test-remove-floating
+--------------------+-----------------------------------------+
| Field              | Value                                   |
+--------------------+-----------------------------------------+
| OS-DCF:diskConfig  | MANUAL                                  |
...Output omitted...
| accessIPv6         |                                         |
| addresses          | dev_privnet=192.168.1.16, 172.25.250.51|
...Output omitted...
| name               | dev_test-remove-floating                |
...Output omitted...
+--------------------+-----------------------------------------+
```

您环境中分配给该实例的浮动 IP 地址可能与本例中所示不同。

8. 如下所示，移除关联至实例的浮动 IP 地址：

```
[student@workstation ~]$ openstack ip floating remove 172.25.250.51 dev_test-remove-floating
```

> 注意: 该命令不会返回任何输出。

9. 从池删除浮动 IP 地址是一个两步流程：

    检索与浮动 IP 地址对应的 ID。

```
    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+-------------+---------------+--------------+--------------------------------------+
    | ID                                   | Pool        | IP            | Fixed IP     | Instance ID                          |
    +--------------------------------------+-------------+---------------+--------------+--------------------------------------+
    ...Output omitted...
    | a7827d0d-d313-4867-acee-4141dca1fb48 | dev_pubnet  | 172.25.250.51 | None         | None                                 |
    +--------------------------------------+-------------+---------------+--------------+--------------------------------------+
```

    作为参数传递该浮动 IP 地址的 ID，从实例取消分配您刚才移除的浮动 IP 地址。务必将 ID 替换为上一命令返回的 ID。

```
    [student@workstation ~]$ openstack ip floating delete a7827d0d-d313-4867-acee-4141dca1fb48
```

    > 注意: 该命令不会返回任何输出。

10. 在环境中生成名为 dev_test-floating 的实例。使用类别 m2.small 和映像 small，并关联密钥对 dev_key1。上述类别、映像和密钥对全部已由设置脚本创建。将实例附加到专用网络 dev_privnet。

    在 --nic net-id 参数中使用 dev_privnet 网络，创建服务器。

```
    [student@workstation ~]$ openstack server create --image small --flavor m2.small --key-name dev_key1 --nic net-id=dev_privnet dev_test-floating --wait 
    +--------------------------------------+--------------------------------------+
    | Field                                | Value                                |
    +--------------------------------------+--------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                               |
    | OS-EXT-AZ:availability_zone          | nova                                 |
    | OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-0.localdomain  |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-0.localdomain  |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000088                    |
    | OS-EXT-STS:power_state               | 1                                    |
    | OS-EXT-STS:task_state                | None                                 |
    | OS-EXT-STS:vm_state                  | active                               |
    | OS-SRV-USG:launched_at               | 2016-01-27T02:51:08.000000           |
    | OS-SRV-USG:terminated_at             | None                                 |
    | accessIPv4                           |                                      |
    | accessIPv6                           |                                      |
    | addresses                            | dev_privnet=192.168.1.17             |
    ...Output omitted....
    | name                                 | dev_test-floating                    |
    | os-extended-volumes:volumes_attached | []                                   |
    | progress                             | 0                                    |
    ...Output omitted...
    | security_groups                      | [{u'name': u'default'}]              |
    | status                               | ACTIVE                               |
    | updated                              | 2016-01-27T02:51:08Z                 |
    ...Output omitted...
    +--------------------------------------+--------------------------------------+
```

11. 现在可以将浮动 IP 地址关联到该实例。openstack 命令要求将浮动 IP 地址用作第一参数，实例名称用作第二参数。下例将浮动 IP 地址 172.25.250.52 添加到实例 dev_test-floating。使用 openstack ip floating list 命令检索可用的 IP。

```
[student@workstation ~]$ openstack ip floating add 172.25.250.52 dev_test-floating
```

> 注意: 该命令不会返回任何输出。

12. 确保实例已标记为 ACTIVE。

```
[student@workstation ~]$ openstack server list
+--------------------------------------+--------------------------+--------+-----------------------------------------+
| ID                                   | Name                     | Status | Networks                                |
+--------------------------------------+--------------------------+--------+-----------------------------------------+
| 2ddbec54-cdeb-4763-9826-39da96206fc8 | dev_test-floating        | ACTIVE | dev_privnet=192.168.1.17, 172.25.250.52 |
...Output omitted...                                                ...Output omitted...
+--------------------------------------+--------------------------+--------+-----------------------------------------+
```

### 验证浮动 IP 地址功能

###### 验证浮动 IP 地址

浮动 IP 地址关联到实例后，管理员可以使用 ICMP 或 HTTP 等不同网络协议来访问该实例。对于创建的每一条安全规则，Neutron 为其定义相应的网络路由。例如，如果添加了使用 SSH 访问实例的规则，Neutron 将创建一系列必要的数据包过滤规则，使数据包通过网桥和网络接口路由到端口 22，从物理主机传输到实例。

> 注意: 默认安全组禁止实例被访问，除非定义了显式安全组规则。

使用浮动 IP 地址访问实例涉及了网络路由，而这是由 Neutron 服务执行的。当 IP 地址分配给实例后，Neutron 将执行以下任务：

*    将该浮动 IP 地址分配到物理主机（使用网络命名空间）。
*    创建 Netfilter NAT 规则来路由数据包。此类规则严格依赖于云操作员定义的安全组规则。


### 练习：验证浮动 IP 地址功能

1. 从workstation.lab.example.com打开一个终端，再提供 overcloud Keystone 凭据。确保 dev_test-floating 实例仍在运行，并且确认它已分配有浮动 IP 地址。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ openstack server list
+---------------------------+--------+------------+-------------+------------------------------------------+
| Name                      | Status | Task State | Power State | Networks                                 |
+---------------------------+--------+------------+-------------+------------------------------------------+
| dev_test-floating         | ACTIVE | -          | Running     | dev_privnet=192.168.1.17, 172.25.250.52  |
...Output omitted...
+---------------------------+--------+------------+-------------+------------------------------------------+
```

2. 从 workstation.lab.example.com，使用关联的浮动 IP 地址 ping 该实例。

```
[student@workstation ~]$ ping -c 3 172.25.250.52
PING 172.25.250.52 (172.25.250.52) 56(84) bytes of data.
64 bytes from 172.25.250.52: icmp_seq=1 ttl=63 time=2.42 ms
64 bytes from 172.25.250.52: icmp_seq=2 ttl=63 time=0.648 ms
64 bytes from 172.25.250.52: icmp_seq=3 ttl=63 time=0.617 ms

--- 172.25.250.52 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.617/1.231/2.429/0.847 ms
```

3. 使用 /home/student/dev/dev_key1.pem 下保存的私钥，以 cloud-user 用户身份发起与实例的 SSH 连接。

```
[student@workstation ~]$ cd dev
[student@workstation dev]# ssh -i dev_key1.pem cloud-user@172.25.250.52
[cloud-user@dev-floatingip-attach ~]$ exit
logout
Connection to 172.25.250.52 closed.
```


### 实验：管理浮动 IP 地址

在本实验中，您将在自己的 OpenStack 环境中管理浮动 IP 地址。您将通过列出池中包含的浮动 IP 地址，来检查现有的池。您将分配额外的 IP 地址到这个池，移除与实例关联的浮动 IP 地址，再将浮动 IP 地址关联到实例。最后，您将使用ping和ssh命令，确认您可以通过其 IP 地址访问实例。环境中提供有下列资源：

*    Keystone 凭据文件，位于 /home/student/overcloudrc 下。
*    Glance 映像 small。
*    Nova 类别 m2.small。
*    公共网络 lab_pubnet。
*    专用网络 lab_privnet。
*    专用子网 lab_pubsub，位于 172.25.250.0/24 范围内。该网络用于提供外部连接，也是浮动 IP 地址的池。
*    专用子网 lab_privsub，位于 192.168.1.0/24 范围内。该网络运行供实例使用的 DHCP 服务器。
*    密钥对 lab_key1。其私钥位于/home/student/lab/lab_key1.pem下。
*    两个实例：lab_floatingip-attach，您要向它关联一个浮动 IP 地址并测试其连通性；lab_floatingip-remove，您要从它移除一个浮动 IP 地址。
*    本实验的工作目录是/home/student/lab/。

1. 管理浮动 IP 地址池

从workstation.lab.example.com，提供 overcloud 凭据并列出浮动 IP 地址池。分配三个浮动 IP 地址。

    a. 从workstation打开一个终端，再提供 overcloud 凭据。

```
    [student@workstation ~]$ source overcloudrc
```

    b. 使用openstack命令，列出网络池中分配的浮动 IP 地址。

```
    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
    | ID                                   | Pool       | IP            | Fixed IP     | Instance ID                          |
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
    | cb5455a5-5eb6-44cd-a373-93d2a43ed6fe | lab_pubnet | 172.25.250.51 | 192.168.1.16 | fc4463af-8d42-4f53-90cc-b9bd74275f77 |
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
```

    c. 使用openstack命令，将三个额外的浮动 IP 地址分配到池中。为此，请再执行下列命令两次。

```
    [student@workstation ~]$ openstack ip floating create lab_pubnet
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 8c3d739d-9d2d-4c20-a250-24bc7e0961b5 |
    | instance_id | None                                 |
    | ip          | 172.25.250.52                        |
    | pool        | lab_pubnet                           |
    +-------------+--------------------------------------+
```

    记住要运行上述命令三次，从而分配三个浮动 IP 地址。

    d. 检查前面添加到浮动 IP 地址池中的三个 IP 地址。

```
    [student@workstation ~]$ openstack ip floating list
    +------------------------------------+------------+---------------+--------------+--------------------------------------+
    |ID                                  | Pool       | IP            | Fixed IP     | Instance ID                          |
    +------------------------------------+------------+---------------+--------------+--------------------------------------+
    |bfc8000b-18b4-485e-91a3-91cc09c245b2| lab_pubnet | 172.25.250.53 | None         | None                                 |
    |8c3d739d-9d2d-4c20-a250-24bc7e0961b5| lab_pubnet | 172.25.250.52 | None         | None                                 |
    |cb5455a5-5eb6-44cd-a373-93d2a43ed6fe| lab_pubnet | 172.25.250.51 | 192.168.1.16 | fc4463af-8d42-4f53-90cc-b9bd74275f77 |
    |c1740b74-58cf-4d74-861e-b2140877aeb1| lab_pubnet | 172.25.250.54 | None         | None                                 |
    +------------------------------------+------------+---------------+--------------+--------------------------------------+
```

    如上所示，共计有四个浮动 IP 地址，其中第三个已分配到某一实例。

2. 取消关联浮动 IP 地址

从 lab_floatingip-remove 实例移除浮动 IP 地址。将浮动 IP 地址关联到实例 lab_floatingip-attach。最后，从浮动 IP 地址池释放之前关联到实例 lab_floatingip-remove 的浮动 IP 地址。

    a. 取消关联并释放与实例 lab_floatingip-attach 关联的浮动 IP 地址。使用 openstack server show INSTANCE 命令，检索实例的浮动 IP 地址。

```
    [student@workstation ~]$ openstack server show lab_floatingip-remove
    +------------- +----------------------------------------------------------+
    | Field        | Value                                                    |
    +------------- +----------------------------------------------------------+
    ...Output omitted...
    | accessIPv6   |                                                          |
    | addresses    | lab_privnet=192.168.1.16, 172.25.250.51                  |
    | config_drive |                                                          |
    ...Output omitted...
    | name         | lab_floatingip-remove                                    |
    ...Output omitted...
    +------------- +----------------------------------------------------------+
```

    请注意，您的实例使用的 IP 地址可能不同于本例所示。

    b. 使用openstack命令，取消关联浮动 IP 地址。

```
    [student@workstation ~]$ openstack ip floating remove 172.25.250.51 lab_floatingip-remove
```

    c. 再次运行openstack server show命令，确认浮动 IP 地址已被移除。

```
    [student@workstation ~]$ openstack server show lab_floatingip-remove
    +------------- +----------------------------------------------------------+
    | Field        | Value                                                    |
    +------------- +----------------------------------------------------------+
    ...Output omitted...
    | accessIPv6   |                                                          |
    | addresses    | lab_privnet=192.268.1.16                                 |
    | config_drive |                                                          |
    ...Output omitted...
    | name         | lab_floatingip-remove                                    |
    ...Output omitted...
    +------------- +----------------------------------------------------------+
```

    如上所示，该实例不再关联有浮动 IP 地址。

    d. 从浮动 IP 地址池释放之前关联到实例 lab_floatingip-remove 的浮动 IP 地址。首先，检索浮动 IP 地址的 ID。

```
    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
    | ID                                   | Pool       | IP            | Fixed IP     | Instance ID                          |
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
    ...Output omitted...
    | cb5455a5-5eb6-44cd-a373-93d2a43ed6fe | lab_pubnet | 172.25.250.51 | None         | None                                 |
    ...Output omitted...
    +--------------------------------------+------------+---------------+--------------+--------------------------------------+
```

    e. 将该 ID 作为参数传递给 openstack 命令。确保替换 ID。

```
    [student@workstation ~]$ openstack ip floating delete cb5455a5-5eb6-44cd-a373-93d2a43ed6fe
```

    > 注意: 该命令不会返回任何输出。

3. 关联浮动 IP 地址

将浮动 IP 地址 172.25.250.53 关联到名为 lab_floatingip-attach 的实例。

    a. 使用 openstack ip floating add 命令，将浮动 IP 地址 172.25.250.53 关联到名为 lab_floatingip-attach 的实例。

```
    [student@workstation ~]$ openstack ip floating add 172.25.250.53 lab_floatingip-attach
```

    b. 验证浮动 IP 地址 172.25.250.53 已关联到名为 lab_floatingip-attach 的实例。

```
    [student@workstation ~]$ openstack server show lab_floatingip-attach
    +------------- +----------------------------------------------------------+
    | Field        | Value                                                    |
    +------------- +----------------------------------------------------------+
    ...Output omitted...
    | accessIPv6   |                                                          |
    | addresses    | lab_privnet=192.268.1.17, 172.25.250.53                  |
    | config_drive |                                                          |
    ...Output omitted...
    | name         | lab_floatingip-attach                                    |
    ...Output omitted...
    +------------- +----------------------------------------------------------+
```

4. 配置安全规则并验证与实例的连接

在 default 安全组中添加两条安全规则：一条允许来自任何位置的所有 ICMP 类型，另一条允许从任何位置通过 SSH 访问实例。使用 ping 命令，确认可以通过实例的浮动 IP 地址访问该实例。使用 /home/student/lab/lab_key1.pem 下保存的私钥，以 cloud-user 用户身份发起 SSH 连接。

    a. 首先添加 ICMP 流量规则，允许从任何位置进行访问。

```
    [student@workstation ~]$ openstack security group rule create --proto icmp --src-ip 0.0.0.0/0 --dst-port -1 default
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 38984d0f-161e-486d-bb7b-88832f846344 |
    | ip_protocol     | icmp                                 |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 182a1124-4745-4b74-8628-50f48a9782e1 |
    | port_range      |                                      |
    +-----------------+--------------------------------------+
```

    b. 针对 SSH 规则重复相同通操作，允许从任何位置进行访问。

```
    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 22:22 default
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 625aeb1c-4de3-4794-ba2a-4f0f88f76014 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 182a1124-4745-4b74-8628-50f48a9782e1 |
    | port_range      | 22:22                                |
    +-----------------+--------------------------------------+
```

    c. 从workstation.lab.example.com，使用ping命令确认可通过您关联的浮动 IP 地址访问该实例。下例使用浮动 IP 地址 172.25.250.53。务必将此地址替换为您的实例使用的浮动 IP 地址。

```
    [student@workstation ~]$ ping -c 3 172.25.250.53
    PING 172.25.250.53 (172.25.250.53) 56(84) bytes of data.
    64 bytes from 172.25.250.53: icmp_seq=1 ttl=63 time=2.42 ms
    64 bytes from 172.25.250.53: icmp_seq=2 ttl=63 time=0.648 ms
    64 bytes from 172.25.250.53: icmp_seq=3 ttl=63 time=0.617 ms

    --- 172.25.250.53 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 0.617/1.231/2.429/0.847 ms
```

    d. 使用/home/student/lab/lab_key1.pem下保存的私钥，以 cloud-user 用户身份发起与实例的 SSH 连接。

```
    [student@workstation ~]$ cd lab
    [student@workstation lab]# ssh -i lab_key1.pem cloud-user@172.25.250.53
    [cloud-user@lab_floatingip-attach ~]$ exit
    logout
    Connection to 172.25.250.53 closed.
```


### 总结

在本章中，您可以学到：

*    在云计算中，浮动 IP 地址是可从外部访问的静态 IP 地址。它们使得管理员能够将所选实例的专用网络路由到外部世界。
*    默认情况下，OpenStack 设有每个项目 50 个浮动 IP 地址的配额。在浮动 IP 被取消分配后，它将返回到池中，可以重新关联到其他实例。
*    管理员还必须定义一套安全组规则，确保实例可被访问。若无适当的安全规则，就无法访问实例。
*    为了能从外部访问实例，云环境需要浮动 IP 地址池。池在创建子网期间定义，包含 Neutron 可用于实例的所有可路由公共 IP 地址。
*    在红帽企业 Linux 中，管理员可以使用ip命令清空系统上的 ARP 缓存。
*    管理员必须创建网络并将它声明为外部网络，然后才能分配浮动 IP 地址到池中。
*    如果需要多种浮动 IP 地址，可以创建额外的浮动 IP 网络来支持多个浮动 IP 地址池。
*    浮动 IP 地址关联到实例后，管理员可以使用 ICMP 或 HTTP 等不同网络协议来访问实例。
