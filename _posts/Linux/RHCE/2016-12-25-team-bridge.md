---
layout: post
title:  "链路聚合和桥接"
categories: Linux
tags: RHCE
---

### 什么是链路聚合

*    链路聚合(Link Aggregation),是指多个物理端口捆绑在一起，称为一个逻辑端口，以实现出/入流量在各成员端口中的负荷分担，交换机根据用户配置的端口负荷分担策略决定报文从哪一个成员端口发送到对端的交换机上。当交换机检测到其中一个成员端口的链路发生故障时，就停止在此端口上发送报文，并根据负荷分担策略在剩下链路中重新计算报文发送的端口，故障端口恢复后再次重新计算报文发送端口。链路聚合在增加链路带宽、实现链路传输弹性和冗余等方面是一项很重要的技术
*    如果通过聚合的每个链路都遵循不同的物理路径，则聚合链路也提供冗余和容错。通过聚合调制解调器链路或者数字线路，链路聚合可用于改善对公共网络的访问。链路聚合也可用于企业网络，以便在吉比特网以太网交换机之间构建多吉比特的主干链路

### HREL7中链路聚合的工作模式

*    active-backup	主备
*    loadbalance	负载均衡
*    RR RoundRobin	轮询


```
添加两块网卡
# nmcli device show | grep -i device
GENERAL.DEVICE:		eno16777736
GENERAL.DEVICE:		eno33554984
GENERAL.DEVICE:		eno50332208


# nmcli connection show		-> 只有默认网卡的连接配置文件，刚才新添加的网卡没有对应的配置文件

为两个新网卡添加连接配置文件
# nmcli connection add con-name eth1 type thernet ifname eno33554984
# nmcli connection add con-name eth2 type thernet ifname eno50332208
# nmcli connection show         -> 显示出刚才新建的两个连接配置文件

新建team设备和连接配置文件
# nmcli connection add type 
adsl        bond        cdma        gsm         ip-tunnel   olpc-mesh   team        vlan
vxlan       wimax       bluetooth   bridge      ethernet    infiniband  macvlan     pppoe
tun         vpn         wifi        
# nmcli connection add type team con-name qinteamfile ifname qinteamdevice config '{"runner":{"name":"activebackup"}}'
# nmcli connection show		-> 出现qinteamdevice设备并和qinteamfile配置文件关联在一起
# nmcli connection modify qinteamfile ipv4.method manual ipv4.addresses 192.168.100.88 ipv4.gateway 192.168.100.1 ipv4.dns 192.168.100.1 connection.autoconnect yes

# nmcli connection add type team-slave con-name qinteamslave1 ifname eno33554984 master qinteamfile
# nmcli connection add type team-slave con-name qinteamslave2 ifname eno50332208 master qinteamfile
# nmcli connection down qinteamfile
# nmcli connection up qinteamfile
# nmcli connection up qinteamslave1
# nmcli connection up qinteamslave2
# nmcli connection show 
# ifconfig | grep ether		-> 后来新建的qinteamfile qinteamslave1 qinteamslave2三个网卡的MAC地址一样

# teamdctl qinteamdevice state

# teamdnl qinteamdevice -h
# teamdnl qinteamdevice getoption activeport
3
# teamdnl qinteamdevice ports
4: ...
3: ...
# nmcli device disconnect eno33554984
# teamdnl qinteamdevice getoption activeport
4
# teamdctl qinteamdevice state

```

### 什么是桥接

*    桥接(Bridging)是指依据OSI网络模型的链路层的地址，对网络数据包进行转发的过程，工作在OSI的第二层。一般的交换机，网桥就有桥接作用
*    就交换机来说，本身就有一个端口于MAC的映射表，通过这些，隔离了冲突域(collision)。简单的说就是通过网桥可以把两个不同的物理局域网连接起来，是一种在链路层实现局域网互连的存储转发设备。网桥从一个局域网接收MAC帧，拆封、校对、校验之后，按另一个局域网的格式重新组装，发往它的物理曾，通俗的说就是通过一台设备(可能不止一个)把几个网络串起来形成的连接


```
# nmcli connection add type bridge con-name qinbrfile ifname qinbrdevice
# nmcli connection modify qinbrfile ipv4.method manual ipv4.addresses 192.168.100.88 ipv4.gateway 192.168.100.1 ipv4.dns 192.168.100.1 connection.autoconnect yes
# nmcli connection show		-> 多了一个bridge设备
# nmcli connection add type bridge-slave con-name qinbrslave ifname eno16777736 master qinbrdevice

# nmcli connection down qinbrfile
# nmcli connection up qinbrfile
# nmcli connection up qinbrslave
# nmcli connection show

# ping -I qinbrdevice 192.168.100.2

```
