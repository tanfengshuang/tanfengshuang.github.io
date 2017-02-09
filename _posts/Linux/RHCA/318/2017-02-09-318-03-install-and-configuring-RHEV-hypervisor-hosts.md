---
layout: post
title:  "Install and Configuring RHEV Hypervisor Hosts(318-3)"
categories: Linux
tags: RHCA 318
---

### 安装 RHEV 系统管理程序

1. 以 PXE 方式引导 servera.podX.example.com 机器，按 Escape 键，然后在 PXE boot: 提示符处输入 rhevh

> 需要 1 分钟才会开始安装，因为机器要下载 RHEV-H 映像。等待安装完成期间，注意如何获取 RHEV-H 的安装媒体：

可按照如下方式创建 RHEV-H 安装媒体（可以是 CD-ROM ISO 或通过网络进行 PXE 启动）：

*    安装 rhev-hypervisor6 或 rhev-hypervisor7 软件包，以创建 RHEV-H 的启动媒体。该操作会将 .iso 文件置于 /usr/share/rhev-hypervisor 中。可使用像 wodim（之前为 cdrecord的）这样的工具或者您选择的任何其他工具，将 ISO 刻录到 CD 上。
*    通过分别使用 rhev-hypervisor{6,7}-tools 包中的 rhevh-iso-to-disk 或 rhevh-iso-to-pxeboot，ISO 还可用于制作可启动的 USB 记忆棒或 PXE 启动装置。

2. 选择安装系统管理程序 7.1-版本，然后按 Enter。
3. 选择键盘布局并按 Enter。
4. 选择本地的内部硬盘驱动器作为启动设备并选择<继续>。

> 使用方向键和 Tab/Shift+Tab 来选择选项和按钮。

5. 选择本地内部硬盘驱动器作为安装设备并选择<继续>。 RHEV-H 7.1 可以控制分区的大小。获取默认值并选择<继续>，然后选择<确认>。
6. 在密码和确认密码字段输入 redhat，然后选择<安装>。
7. 安装完成之后，按重新引导。 


### 配置 RHEV 系统管理程序

1. 完成安装并启动 RHEV-H 机器之后，以 admin 身份使用之前创建的密码 (redhat) 登录。
2. 


### 练习：安装 RHEV Hypervisor

1. 重新启动 servera VM，并在出现 BIOS 屏幕时选择 PXE 启动选项。出现 PXE 菜单时，在 boot: 提示符处键入 rhevh。

    boot: rhevh

2. RHEV Hypervisor 启动器启动后，选择安装 Hypervisor 7.1-版本，然后按 Enter 键。

选择键盘布局并按 Enter。    
安装程序询问启动设备时应选择内部硬盘驱动器。确认已选择内置硬盘驱动器，然后选择<继续>并按 Enter 键。     
安装程序询问安装设备时应选择内部硬盘驱动器。确认已选择内置硬盘驱动器，然后选择<继续>并按 Enter 键。接受默认分区大小，然后选择<继续>。     

3. RHEV-H 安装程序将提示您输入 RHEV-H 主机管理员密码。在密码：和确认密码：字段中键入 redhat123，选择 <安装>，然后按 Enter 键。    
此时将格式化硬盘驱动器，随后安装 RHEV Hypervisor。安装完成后，出现<重新启动>提示时，按 Enter 键。现在 RHEV-H 软件已完成安装，接下来到了配置系统的时候了。

4. RHEV 系统管理程序系统启动后，作为 admin，利用密码 redhat123 登录。状态和配置项目的主菜单将出现在屏幕的左侧。通过使用上下箭头键选择各菜单项以将其突出显示，然后按 Enter 键，可以选择它们。 
5. 从主菜单中选择网络。出现网络配置屏幕时，设置主机名和 DNS 及 NTP 设置。提供以下值：

    主机名：servera.podX.example.com
    DNS 服务器 1：172.25.254.254
    NTP 服务器 1：172.25.254.254

选择<保存>以确认选择。

接下来配置网络接口。使用 Tab 键将光标移动到网络设备并确保突出显示 eth0（或第一个接口），然后按 Enter 键。

出现网络接口配置屏幕时，为 IPv4 设置：选择静态单选按钮，然后提供以下值：

    IP 地址：172.25.X.10
    网络掩码：255.255.255.0
    网关：172.25.X.254

选择<保存>以确认选择。出现确认框时，选择关闭。 

6. 只有在联机学习环境中运行时，才需要执行此步骤，因为云的嵌套虚拟化缺少一些运行 RHEV-H 所需的信息。    
打开虚拟键盘将 F2 发送到系统，以打开 Rescue Shell。选择确定以显示提示符。

[root@servera admin]# mkdir -p /config/usr/share/libvirt
[root@servera admin]# wget http://classroom.example.com/materials/cpu_map.xml -P /config/usr/share/libvirt
[root@servera admin]# echo "/usr/share/libvirt/cpu_map.xml" >> /config/files
[root@servera admin]# uuidgen -r > /etc/vdsm/vdsm.id
[root@servera admin]# persist /etc/vdsm/vdsm.id
[root@servera admin]# reboot

在启动完成后，重新登录 RHEV-H 配置界面。 

7. 从主菜单中选择安全。导航到启用 ssh 密码验证字段并按空格键进行切换。选择 <保存>，然后选择关闭以确认您的设置。

8. 从主菜单中选择 RHEV-M。指定允许您的 RHEV-H 主机与 RHEV-M 服务器进行通信的值。

    管理服务器：rhevm.podX.example.com
    管理服务器端口：443

使用提供的信息填写相关字段。提供 RHEV-M 密码 redhat 并确认。选择<保存并注册>，以保存您的更改。    
当 RHEV-H 主机联系 RHEV-M 服务器时，它应显示数字指纹。选择 <接受>，以继续 RHEV-M 注册。注册成功后，选择关闭。    
从主菜单中选择状态，然后选择<注销>，以退出配置工具并回到 RHEV-H 登录提示。 

9. 在 example.com 域中利用密码 redhat 作为 rhevadmin 登录 RHEV-M 管理门户。选择主机选项卡。主机应呈现 待批准 状态。单击 servera.podX.example.com 以突出显示它，然后单击批准按钮。审核 编辑 & 批准主机 窗口。单击确定。

在导出的电源管理配置窗口中，选择确定以跳过电源管理。    
主机应将状态更改为正在安装，然后快速通过未分配到启动。    
如果 RHEV-H 主机进入不可操作状态，请重新启动 RHEV-H 主机并检查其 BIOS 设置。确保已启用“执行禁用内存保护技术”。






