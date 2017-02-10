---
layout: post
title:  "Installing and Configuring RHEL Hosts(318-12)"
categories: Linux
tags: RHCA 318
---


### 将 RHEL 转换为 RHEV 主机

```
1. 
# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.1 (Maipo)

确保 /proc/cpuinfo 中存在 lm、vmx/svm 和 nx 标志。
# egrep --color 'lm|vmx|svm|nx' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm dca lahf_lm dts tpr_shadow vnmi flexpriority

2. 
# yum install -y vdsm

3. 将 RHEV-M 机器的条目添加至 /etc/hosts，以便 hypervisor 始终能够找到 RHEV-M 机器，即使 DNS 已关闭也是如此。
# echo '172.25.X.15  rhevm.podX.example.com rhevm' >> /etc/hosts

4. 禁用 NetworkManager 服务。vdsmd 将配置桥接网络并且与 NetworkManager 不兼容。
# systemctl stop NetworkManager
# systemctl disable NetworkManager
# systemctl start network
# systemctl enable network

5. 确保 root 能够利用密码验证通过 SSH 进行登录。如果需要，不要忘记打开防火墙上的端口 22 TCP。向 RHEV-M 注册主机后，可再次关闭密码验证。
# ssh root@serverb.podX.example.com
```


### 向 RHEV-M 中添加红帽企业 Linux 主机

1. 登录 RHEV-M Web 界面并导航到主机选项卡。
2. 单击新建按钮，以显示新建主机对话框。

3. 
1). 常规选项卡
*    数据中心/主机集群: 此主机将添加到的数据中心和集群。Data Center: dcenter0, Host Cluster: cluster0
*    名称: 应在 RHEV-M 界面中出现的主机名称。serverb.podX.example.com
*    地址: 该主机的 IP 地址。不要在该字段内填写主机名称，因为如果填写主机名称，SPICE 连接将不工作。172.25.X.11
*    Root 密码: 该主机的 root 密码。RHEV-M 在初始设置期间使用该密码通过 SSH 登录到该机器。redhat
*    自动配置主机防火墙: 如果选择此设置，RHEV-M 将更新主机上的防火墙规则，以允许所有必需连接。如果目前未配置防火墙（或者不需要防火墙），那么可以安全地取消选中此框。Deselect

2). “电源管理”选项卡

*    启用电源管理:	选中此框可让 RHEV-M 能够远程重新启动该 hypervisor。这对高可用性配置而言非常必要（稍后讨论）。机器必须具有兼容硬件。
*    地址: 填写该字段，以与远程管理卡的设置和类型相匹配。
*    用户名: 填写该字段，以与远程管理卡的设置和类型相匹配。
*    密码: 填写该字段，以与远程管理卡的设置和类型相匹配。
*    类型: 填写该字段，以与远程管理卡的设置和类型相匹配。
*    选项: 填写该字段，以与远程管理卡的设置和类型相匹配。
*    测试: 使用此按钮可测试新主机上的电源管理。

3). SPM 选项卡

*    SPM 优先权: 主机将发挥存储池管理器 (SPM) 作用的可能性。选择报考“低”、“正常”和“高”。 

4. 单击确定按钮，以开始安装主机。RHEV-M 将通过 SSH 与主机连接并安装所有必需程序包，并做出任何必要的配置更改。如果在该阶段报告任何错误，请从 RHEV-M 界面中删除主机、修复报告的问题，然后重新开始。
5. 导航至 RHEV-M 网络界面的主机选项卡，并等待新主机出现。最开始将报告其状态为正在安装，但稍后会从无响应变为启动。

### 删除托管的 RHEL 主机

删除托管 Red Hat Enterprise Linux 7 主机的步骤与删除 RHEV-H 节点的步骤相同。

1. 导航到 RHEV-M 中的主机选项卡。
2. 通过选择托管的 Red Hat Enterprise Linux 7 主机，右键单击它并选择维护，将其置于维护模式。单击确定以确认。主机更改为维护模式时，该 hypervisor 上运行的任何虚拟机都将自动移动到不同的 hypervisor。如果该主机是存储池管理器 (SPM)，将会为新 SPM 保持新的选择。

> 如果这是集群中唯一的主机且该集群中有虚拟机正在运行，则该主机不能进入维护模式。要使该主机进入维护模式，请先手动关闭在该主机上运行的所有虚拟机。

3. 主机报告其状态为维护后，通过选择该 hypervisor，右键单击它并选择删除，将其删除。单击确定以确认操作。新删除的主机可以移至其他数据中心或集群，也可以用于其他目的。
