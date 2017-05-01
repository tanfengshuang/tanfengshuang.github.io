---
layout: post
title:  "Controlling Access to Network Services(413-15)"
categories: Linux
tags: 413 
---

### Review of iptables Basics


[iptables详解](http://www.cnblogs.com/metoy/p/4320813.html)
[iptables的表与链](http://www.cnblogs.com/wangkangluo1/archive/2012/04/19/2457072.html)

iptables只是Linux防火墙的管理工具而已，位于/sbin/iptables。真正实现防火墙功能的是 netfilter，它是Linux内核中实现包过滤的内部结构。

iptables的规则表和链：

*    表（tables）提供特定的功能，iptables内置了4个表，即filter表、nat表、mangle表和raw表，分别用于实现包过滤，网络地址转换、包重构(修改)和数据跟踪处理。
*    链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定义的默认策略来处理数据包。
   
规则表：

1. filter表——三个链：INPUT、FORWARD、OUTPUT        -   作用：过滤数据包  内核模块：iptables_filter.
2. Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT  -   作用：用于网络地址转换（IP、端口） 内核模块：iptable_nat
3. Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD -   作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS内核模块：iptable_mangle(别看这个表这么麻烦，咱们设置策略时几乎都不会用到它)
4. Raw表——两个链：OUTPUT、PREROUTING  -   作用：决定数据包是否被状态跟踪机制处理  内核模块：iptable_raw (这个是REHL4没有的，用的不多)

规则链：

1. INPUT——进来的数据包应用此规则链中的策略
2. OUTPUT——外出的数据包应用此规则链中的策略
3. FORWARD——转发数据包时应用此规则链中的策略
4. PREROUTING——对数据包作路由选择前应用此链中的规则（记住！所有的数据包进来的时侯都先由这个链处理）
5. POSTROUTING——对数据包作路由选择后应用此链中的规则（所有的数据包出来的时侯都先由这个链处理）
  
iptables的基本语法格式

> iptables [-t 表名] 命令选项 ［链名］ ［条件匹配］ ［-j 目标动作或跳转］

说明：表名、链名用于指定 iptables命令所操作的表和链，命令选项用于指定管理iptables规则的方式（比如：插入、增加、删除、查看等；条件匹配用于指定对符合什么样 条件的数据包进行处理；目标动作或跳转用于指定数据包的处理方式（比如允许通过、拒绝、丢弃、跳转（Jump）给其它链处理。

 

iptables命令的管理控制选项

*    -A 在指定链的末尾添加（append）一条新的规则
*    -D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
*    -I 在指定链中插入（insert）一条新的规则，默认在第一行添加
*    -R 修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
*    -L 列出（list）指定链中所有的规则进行查看
*    -E 重命名用户定义的链，不改变链本身
*    -F 清空（flush）
*    -N 新建（new-chain）一条用户自己定义的规则链
*    -X 删除指定表中用户自定义的规则链（delete-chain）
*    -P 设置指定链的默认策略（policy）
*    -Z 将所有表的所有链的字节和数据包计数器清零
*    -n 使用数字形式（numeric）显示输出结果
*    -v 查看规则表详细信息（verbose）的信息
*    -V 查看版本(version)
*    -h 获取帮助（help）


防火墙处理数据包的四种方式

*    ACCEPT 允许数据包通过
*    DROP 直接丢弃数据包，不给任何回应信息
*    REJECT 拒绝数据包通过，必要时会给数据发送端一个响应的信息。
*    LOG在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则


通用匹配：源地址目标地址的匹配
*	-s (source)：指定作为源地址匹配，这里不能指定主机名称，必须是IP(IP | IP/MASK | 0.0.0.0/0.0.0.0), 而且地址可以取反，加一个“!”表示除了哪个IP之外
*	-d (destination)：表示匹配目标地址
*	-p (protocol)：用于匹配协议的（这里的协议通常有3种，TCP/UDP/ICMP）
*	-i (input interface)：从这块网卡流入的数据, 流入一般用在INPUT和PREROUTING上
*	-o (output interface): 从这块网卡流出的数据, 流出一般在OUTPUT和POSTROUTING上
*	-j (jump to target): 指定了当与规则(Rule)匹配时如何处理数据包, 可能的值是ACCEPT, DROP, QUEUE, RETURN, 还可以指定其他链（Chain）作为目标


描述规则的扩展参数

对规则有了一个基本描述之后，有时候我们还希望指定端口、TCP标志、ICMP类型等内容。

*    --sport, --source-port 源端口: 针对 -p tcp 或者 -p udp, 可以指定端口号或者端口名称，例如"--sport 22"与"--sport ssh", 使用冒号可以匹配端口范围，如--sport 22:100″
*    --dport, --destination-port 目的端口: 针对-p tcp 或者 -p udp, 与参数--sport类似
*    --tcp-flags TCP标志 针对-p tcp，可以指定由逗号分隔的多个参数，有效值可以是：SYN, ACK, FIN, RST, URG, PSH，可以使用ALL或者NONE
*    --icmp-type ICMP类型 针对-p icmp， --icmp-type 0 表示Echo Reply； --icmp-type 8 表示Echo Request


iptables防火墙规则的保存与恢复

iptables-save把规则保存到文件中，再由目录rc.d下的脚本（/etc/rc.d/init.d/iptables）自动装载

1. iptables-save > /etc/sysconfig/iptables     -> 生成保存规则的文件 /etc/sysconfig/iptables，
2. service iptables save                       ->  把规则自动保存在/etc/sysconfig/iptables中。

当计算机启动时，rc.d下的脚本将用命令iptables-restore调用这个文件，从而就自动恢复了规则。


###### Workshop: Establish a Stateful Host Firewall


###### Quiz: netfilter Basics

### Advanced iptables

###### ICMP (Internet Control Message Protocol )

*    echo-reply: 回显应答（Ping应答）别人ping我时，我给出的反馈信息，状态是ESTABLISHED
*    echo-request： 回显请求（Ping请求），第一次ping时状态是NEW，以后一直都是ESTABLISHED


```
# iptables -F
# iptables -A INPUT -p icmp -j DROP         -> 将有关icmp协议的包都drop掉

# iptables -F
# iptables -A INPUT -p icmp -m icmp --icmp-type echo-reply -j ACCEPT        -> 这一条可以不写，默认就是ACCEPT的
# iptables -A INPUT -p icmp -m icmp --icmp-type echo-request -j DROP

# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     icmp --  anywhere             anywhere            icmp echo-reply 
DROP       icmp --  anywhere             anywhere            icmp echo-request 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# iptables -L -v
Chain INPUT (policy ACCEPT 47 packets, 3288 bytes)
 pkts bytes target     prot opt in     out     source               destination
    3   252 ACCEPT     icmp --  any    any     anywhere             anywhere            icmp echo-reply 
  133 11172 DROP       icmp --  any    any     anywhere             anywhere            icmp echo-request 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 28 packets, 3296 bytes)
 pkts bytes target     prot opt in     out     source               destination

# iptables -L -v -n
Chain INPUT (policy ACCEPT 60 packets, 4164 bytes)
 pkts bytes target     prot opt in     out     source               destination
    3   252 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           icmp type 0 
  137 11508 DROP       icmp --  *      *       0.0.0.0/0            0.0.0.0/0           icmp type 8 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 35 packets, 4628 bytes)
 pkts bytes target     prot opt in     out     source               destination

# iptables -L -v -n --line-number
Chain INPUT (policy ACCEPT 110 packets, 7724 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        3   252 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           icmp type 0 
2      150 12600 DROP       icmp --  *      *       0.0.0.0/0            0.0.0.0/0           icmp type 8 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 61 packets, 7708 bytes)
num   pkts bytes target     prot opt in     out     source               destination



# iptables -F
# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# iptables -A INPUT -i lo -j ACCEPT
# iptables -P INPUT DROP
# iptables -L -v -n --line-number
Chain INPUT (policy DROP 28 packets, 2280 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      142 10268 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:22 
2        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 12 packets, 1312 bytes)
num   pkts bytes target     prot opt in     out     source               destination 


# iptables -t filter -A INPUT -p icmp -m state --state NEW -j ACCEPT            -> 开启了这一条，其他机器能ping通这台机器一次
# iptables -t filter -D INPUT -p icmp -m state --state NEW -j ACCEPT            -> 删除上面刚刚添加的规则
# iptables -t filter -A INPUT -p icmp -m state --state NEW,ESTABLISHED -j ACCEPT    -> 添加此规则后，能被其他机器能一直ping通

# iptables -P OUTPUT DROP
# iptables -t filter -A INPUT -p icmp -m state --state ESTABLISHED -j ACCEPT

```

###### HTTPD

```
【服务器端】                      【客户端】
   80端口                         >1024的随机端口
      <---------------- SYN 第一次握手       （NEW）
      ----------------> SYN/ACK  第二次握手  （ESTABLISHED）
      <---------------- ACK 第三次握手       （ESTABLISHED）
      <---------------> Data 开始数据传送     （ESTABLISHED）
      <---------------> Data 数据传送        （ESTABLISHED）   
```

```
# iptables -A OUTPUT -i lo -j ACCEPT
# iptables -A OUTPUT -o lo -j ACCEPT

# iptables -A INPUT -p tcp -dport 80 -m state --state NEW,ESTABLISHED
# iptables -A OUTPUT -p tcp -sport 80 -m state --state ESTABLISHED
```

```
开启DNS服务器访问
# iptables -A INPUT -p udp --dport 53 -j ACCEPT
# iptables -A OUTPUT -p udp --sport 53 -j ACCEPT
```

###### FTP

the initial connection to an FTP server is on port 21/tcp, but the data delivery is done from a dynamic port. What we need is some sort of “helper” to let netfilter know that this is part of the same RELATED conversation.

There are a number of netfilter modules that can be optionally loaded. The modules can be found in /lib/modules/$(uname -r)/kernel/net/netfilter/ or /lib/modules/$(uname -r)/kernel/net/ipv4/netfilter/. 

```
【服务器】                       【客户端】
  21端口                          > 1024的随机端口
        <------------------ SYN     (NEW)
        ------------------> SYN/ACK (ESTABLISHED)
        <------------------ ACK     (ESTABLISHED)
        <-----------------> DATA    (ESTABLISHED)

  17892端口，这个端口是随机的，服务器指定一个端口，让客户端往这个口发送数据
        <------------------ SYN     (RELATED)
        ------------------> SYN/ACK (ESTABLISHED)
        <------------------ ACK     (ESTABLISHED)
        <-----------------> DATA    (ESTABLISHED)
        
        
即：
INPUT   N(21端口), E(随机端口), R(随机端口)
OUTPUT  E(随机端口)
```

```
# iptables -F
# iptables -P INPUT DROP
# iptables -P OUTPUT DROP

# iptables -A INPUT -i lo -j ACCEPT
# iptables -A OUTPUT -o lo -j ACCEPT

# iptables -A INPUT -p tcp -dport 21 -m state --state NEW -j ACCEPT
# iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
# iptables -A OUTPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

# rmmod nf_conntrack_ftp            -> 当没有加载nf_conntrack_ftp模块时，ftp服务器仍然不能使用
# lsmod | grep ftp

Dynamically load the nf_conntrack_ftp module
# modprobe nf_conntrack_ftp

All these changes need to be made permanent, /etc/sysconfig/iptables-config - edit this file to add the nf_conntrack_ftp module
# vim /etc/sysconfig/iptables-config
IPTABLES_MODULES="nf_conntrack_ftp"

Save the current set of rules in memory to /etc/sysconfig/iptables.
# service iptables save

```

###### Discussion: iptables Rule Management Pratices

###### Demonstration: Troubleshoot Missing iptables Module

###### Performance Checklist: Script a Stateful Host Firewall

###### Unit Test: Protect Remote Management from Firewall Changes

