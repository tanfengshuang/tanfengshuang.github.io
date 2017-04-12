---
layout: post
title:  "Install Configuring and Test RHEV Manager(318-2)"
categories: Linux
tags: RHCA 318
---

### Install RHEV Manager

```
1. 
# ssh root@rhevm.podX.example.com

2. 
# wget http://classroom.example.com/materials/rhevm.repo -P /etc/yum.repos.d/
...
# yum repolist

3. RHEV-M 应安装在红帽企业 Linux 6 的最新版本上，以便使用 yum 更新所有操作系统软件包
# yum update -y 

4. 安装所需的 RHEV-M 程序包。至少要使用 yum 安装 rhevm 程序包。如果管理员计划使用 RHEV-M 报告工具，那么还需要安装 rhevm-dwh 和 rhevm-reports 软件包
# yum -y install rhevm rhevm-dwh rhevm-reports

5. 配置 RHEV Manager 以及 RHEV Reports 数据仓库。管理员可以通过生成应答文件来自动执行 RHEV-M 安装。可在首次安装时创建应答文件，然后重复使用。
# engine-setup --generate-answer=/root/answers.txt
传递 generate-answer 标志将允许指定配置文件的名称和位置。
    在此主机上配置引擎（是，否）[是]：Enter
    在此主机上配置数据仓库（是，否）[是]：Enter
    在此主机上配置报告（是，否）[是]：Enter
    在此主机上配置 WebSocket 代理（是，否）[是]：Enter
    是否要安装程序配置防火墙？（是，否）[是]：no
    此服务器的主机 DNS 名称 [rhevm.podX.example.com]：Enter
    报告数据库的所在位置：local
    是否希望安装程序自动配置 postgresql 并创建报告数据库，还是更喜欢手动执行？（自动，手动）：automatic
    DWH 数据库的所在位置：local
    是否希望安装程序自动配置 postgresql 并创建 DWH 数据库，还是更喜欢手动执行？automatic
    引擎数据库在哪里？（本地，远程）[本地]：local
    是否希望安装程序自动配置 postgresql 并创建引擎数据库，还是更喜欢手动执行？automatic
    引擎管理员密码：redhat
    确认引擎管理员密码：redhat
    是否使用弱密码？（是，否）[否]：是
    应用模式（Virt，Gluster，两者）[两者]：Enter
    DWH 数据库安全连接（是，否）[否]：Enter
    证书 [podX] 的组织名称：Enter
    是否希望将该应用程序设置为 Web 服务器的默认页面？（是，否）[是]：Enter
    是希望安装程序配置此选项，还是更喜欢手动执行？（自动，手动）[自动]：Enter
    是否在此服务器上配置要用作 ISO 域的 NFS 共享？（是，否）[是]：Enter
    本地 ISO 域路径 [/var/lib/exports/iso]: /exports/rhevisos
    本地 ISO 域 ACL - 请注意，出于安全原因，默认值将仅限于访问 rhevm.podX.example.com，[rhevm.podX.example.com(rw)]：Enter
    本地 ISO 域名 [ISO_DOMAIN]：isoX
    报告高级用户密码：redhat
    确认报告高级用户密码: redhat
    是否使用弱密码？（是，否）[否]：是
    是否希望通过代理服务器代理从 RHEV Manager 中发送的红帽访问插件 (Red Hat Access Plugin) 中的事务？（是，否）[否]：Enter
    ... 请确认安装设置（确定，取消）[确定]：Enter

以后可以使用此文件来自动进行安装。要加载应答文件以自动执行安装，需要传递 --config-append=file 标志：
# engine-setup --config-append=/root/anwers.txt
# cat /root/anwers.txt
#action=setup
[environment:default]
OVESETUP_CORE/engineStop=none:None
OVESETUP_DIALOG/confirmSettings=bool:True
OVESETUP_DB/database=str:engine
OVESETUP_DB/fixDbViolations=none:None
OVESETUP_DB/secured=bool:False
OVESETUP_DB/host=str:localhost
OVESETUP_DB/user=str:engine
OVESETUP_DB/securedHostValidation=bool:False
OVESETUP_DB/password=str:0056jKkY
OVESETUP_DB/port=int:5432
OVESETUP_SYSTEM/nfsConfigEnabled=bool:True
...
OVESETUP_APACHE/configureSsl=bool:True
OSETUP_RPMDISTRO/requireRollback=none:None
OSETUP_RPMDISTRO/enableUpgrade=none:None
OVESETUP_AIO/configure=none:None
OVESETUP_AIO/storageDomainDir=none:None

# engine- <Tab> <Tab>
engine-backup          engine-image-uploader  engine-manage-domains
engine-cleanup         engine-iso-uploader    engine-setup
engine-config          engine-log-collector   engine-upgrade-check
# rhevm- <Tab> <Tab>
rhevm-cleanup         rhevm-iso-uploader    rhevm-setup
rhevm-config          rhevm-log-collector   rhevm-shell
rhevm-image-uploader  rhevm-manage-domains
```

###### 应用版本更新

正常的 yum 更新 将不会更新 RHEV，因为 RHEV 安装将使用 versionlock 插件锁定 yum 中的 RHEV 包。可在 /etc/yum/pluginconf.d/versionlock.list 中找到锁定包的列表。管理员可以使用 engine-upgrade-check 命令来检查 RHEV Manager 更新的可用性。

```
# engine-upgrade-check
VERB: queue package engine-setup for update
VERB: Building transaction
VERB: Empty transaction
VERB: Transaction Summary:
No upgrade
```

将 engine-setup 软件包更新到最新版本，然后以 root 身份执行 engine-setup 命令（不带任何参数）。这将停止所有必需 RHEV Manager 服务、升级底层软件，然后在升级完成时重新启动该服务。

```
# yum update -y engine-setup
...
# engine-setup
```

### Test RHEV Manager

1. 验证 ovirt-engine 服务是否在运行：

```
# service ovirt-engine status
ovirt-engine (pid  7387) is running...
```

2. 验证 ISO NFS 共享是否可用：

```
# showmount -e localhost | grep rhevisos
/exports/rhevisos rhevm.podX.example.com
```

3. 在 workstation 上，打开 Web 浏览器并检查主门户：

```
# firefox https://rhevm.podX.example.com &
```

4. 添加安全例外，以允许 firefox 使用安装 RHEV-M 时生成的自签名 SSL 证书。单击我了解风险 → 添加例外... → “确认安全异常”。

5. 单击管理门户超链接。

6. 如果一切正常，则将显示 RHEV 登录屏幕。使用密码 redhat 并将域设为 internal，以 admin 身份登录。成功登录 RHEV-M 后，将显示管理员门户屏幕


###### 练习：安装和配置 RHEV Manager

1. 要安装所需软件包，将该系统转变为红帽企业虚拟化管理器，您需要为它订阅新的 RHN 频道或者 yum 软件仓库。在课堂上，您有一个自定义 yum 软件仓库。要激活此软件仓库，请从 http://classroom.example.com/materials/rhevm.repo 下载文件 rhevm.repo 并将其放入目录 /etc/yum.repos.d/。

    # wget http://classroom.example.com/materials/rhevm.repo -P /etc/yum.repos.d/
    ...
    # yum repolist
    ...

2. 更新所有现有的红帽企业 Linux 软件包（如果需要，请重新启动）。

    # yum update -y

3. 安装 rhevm、rhevm-dwh 和 rhevm-reports 软件包：

    # yum -y install rhevm rhevm-dwh rhevm-reports

4. 执行 engine-setup --generate-answer=/root/answers.txt 
    
```
# engine-setup --generate-answer=/root/answers.txt
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-dwh.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging-wsp.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf', '/etc/ovirt-engine-setup.conf.d/20-packaging-rhevm-reports.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20141211130705-o82mc9.log
          Version: otopi-1.3.0 (otopi-1.3.0-1.el6ev)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup
[ INFO  ] Stage: Environment customization

          --== PRODUCT OPTIONS ==--

          Configure Engine on this host (Yes, No) [Yes]: <Enter>
          Configure Data Warehouse on this host (Yes, No) [Yes]: <Enter>
          Configure Reports on this host (Yes, No) [Yes]: <Enter>
          Configure WebSocket Proxy on this host (Yes, No) [Yes]: <Enter>

          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found

          --== ALL IN ONE CONFIGURATION ==--


          --== NETWORK CONFIGURATION ==--

          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: No
[ INFO  ] iptables will be configured as firewall manager.
          Host fully qualified DNS name of this server [rhevm.podX.example.com]: <Enter>

          --== DATABASE CONFIGURATION ==--

          Where is the Reports database located? (Local, Remote) [Local]: <Enter>
          Setup can configure the local postgresql server automatically for the Reports to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Reports database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: <Enter>
          Where is the DWH database located? (Local, Remote) [Local]: <Enter>
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: <Enter>
          Where is the Engine database located? (Local, Remote) [Local]: <Enter>
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: <Enter>

          --== OVIRT ENGINE CONFIGURATION ==--

          Engine admin password: redhat
          Confirm engine admin password: redhat
[WARNING] Password is weak: it is based on a dictionary word
          Use weak password? (Yes, No) [No]: yes
          Application mode (Virt, Gluster, Both) [Both]: <Enter>

          --== PKI CONFIGURATION ==--

          Organization name for certificate [podX.example.com]: <Enter>

          --== APACHE CONFIGURATION ==--

          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: <Enter>
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: <Enter>

          --== SYSTEM CONFIGURATION ==--

          Configure an NFS share on this server to be used as an ISO Domain? (Yes, No) [Yes]: <Enter>
          Local ISO domain path [/var/lib/exports/iso]: /exports/rhevisos
          Local ISO domain ACL - note that the default will restrict access to rhevm.pod0.example.com only, for security reasons [rhevm.pod0.example.com(rw)]: <Enter>
          Local ISO domain name [ISO_DOMAIN]: isoX

          --== MISC CONFIGURATION ==--

          Reports power users password: redhat
          Confirm Reports power users password: redhat
[WARNING] Password is weak: it is based on a dictionary word
          Use weak password? (Yes, No) [No]: yes
          Would you like transactions from the Red Hat Access Plugin sent from the RHEV Manager to be brokered through a proxy server? (Yes, No) [No]: <Enter>

          --== END OF CONFIGURATION ==--

[ INFO  ] Stage: Setup validation
[WARNING] Less than 16384MB of memory is available

          --== CONFIGURATION PREVIEW ==--

          Application mode                        : both
          Firewall manager                        : iptables
          Update Firewall                         : False
          Host FQDN                               : rhevm.podX.example.com
          Engine database name                    : engine
          Engine database secured connection      : False
          Engine database host                    : localhost
          Engine database user name               : engine
          Engine database host name validation    : False
          Engine database port                    : 5432
          Engine installation                     : True
          NFS setup                               : True
          PKI organization                        : podX
          NFS mount point                         : /exports/rhevisos
          NFS export ACL                          : rhevm.podX.example.com(rw)
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          DWH installation                        : True
          DWH database name                       : ovirt_engine_history
          DWH database secured connection         : False
          DWH database host                       : localhost
          DWH database user name                  : ovirt_engine_history
          DWH database host name validation       : False
          DWH database port                       : 5432
          Configure local DWH database            : True
          Reports installation                    : True
          Reports database name                   : ovirt_engine_reports
          Reports database secured connection     : False
          Reports database host                   : localhost
          Reports database user name              : ovirt_engine_reports
          Reports database host name validation   : False
          Reports database port                   : 5432
          Configure local Reports database        : True
          Engine Host FQDN                        : rhevm.podX.example.com
          Configure WebSocket Proxy               : True

          Please confirm installation settings (OK, Cancel) [OK]: <Enter>
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping reports service
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_reports' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Creating CA
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Deploying Jasper
[ INFO  ] Importing data into Jasper
[ INFO  ] Configuring Jasper Java resources
[ INFO  ] Configuring Jasper Database resources
[ INFO  ] Customizing Jasper
[ INFO  ] Customizing Jasper metadata
[ INFO  ] Customizing Jasper Pro Parts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up

          --== SUMMARY ==--

[WARNING] Less than 16384MB of memory is available
          SSH fingerprint: B8:D9:4E:E6:5F:5C:86:F1:47:17:B2:E5:D1:A1:42:42
          Internal CA 25:B0:D0:02:30:1E:B2:1C:EA:04:BD:3B:AE:9E:05:29:87:08:80:4A
          Web access is enabled at:
              http://rhevm.podX.example.com:80/ovirt-engine
              https://rhevm.podX.example.com:443/ovirt-engine
          Please use the user "admin" and password specified in order to login

          --== END OF SUMMARY ==--

[ INFO  ] Starting engine service
[ INFO  ] Restarting httpd
[ INFO  ] Restarting nfs services
[ INFO  ] Starting dwh service
[ INFO  ] Starting reports service
[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20141211130705-o82mc9.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20141211131733-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
```

5. 将生成的回答文件复制到 workstation 来加以保存。

    # scp /root/answers.txt student@workstation.podX.example.com:/
    ...

6. 验证 ovirt-engine 服务是否在运行：

    # service ovirt-engine status
    ovirt-engine (pid  7387) is running...

7. 验证 ISO NFS 共享是否可用：

    # showmount -e localhost | grep rhevisos
    /exports/rhevisos rhevm.podX.example.com

8. 在 workstation 上，打开 Web 浏览器：

    # firefox https://rhevm.podX.example.com &

9. 添加安全例外，以允许 firefox 使用安装 RHEV-M 时生成的自签名 SSL 证书。
10. 单击我了解风险 → 添加例外... → “确认安全异常”。
11. 单击管理门户超链接。
12. 如果一切正常，则将显示 RHEV 登录屏幕。使用密码 redhat 并将域设为 internal，以 admin 身份登录。成功登录 RHEV-M 时，将显示管理门户屏幕。

### 管理用户、角色和权限

###### 红帽企业虚拟化授权模型

红帽企业虚拟化拥有扩展的授权模型以限制和控制用户可以对对象执行的操作。尽管红帽企业虚拟化处理授权，外部目录服务也可处理授权。红帽企业虚拟化的目前可用目录服务包括 Microsoft Active Directory、Red Hat Identity Management (IdM)、Red Hat Directory Server 9 和 OpenLDAP。

> 和红帽 IdM 一样，目前无法在同一系统上安装红帽企业虚拟化管理器，因为存在与 mod_ssl 有关的冲突。

红帽企业虚拟化可通过在 RHEV-M 主机运行命令行 engine-manage-domains 来绑定到一个或多个目录服务。engine-setup 命令定义本地 RHEV-M 管理员帐户 (admin)。对于其他用户定义，RHEV-M 可被绑定至外部目录服务。管理员可以使用 engine-manage-domains 命令来绑定至目录服务, 使用 engine-manage-domains 做出更改之后，应重新启动 ovirt-engine 服务，以便它能够识别更改。

```
# engine-manage-domains add --domain=DOMAIN --user=USER --provider=PROVIDER
```

###### 管理角色

角色分为两种主要类型：允许访问管理员门户的管理员角色和允许访问用户门户的用户角色。

> 例如，如果管理员拥有群集上的管理员角色，则可使用管理门户来管理群集中的所有虚拟机。但是，他们无法访问用户门户中的任何虚拟机；如果想要访问，则需要用户角色。

要管理角色，请登录 RHEV-M Web 界面，并单击屏幕顶部的配置链接

###### 目录用户

RHEV 需要连接到至少一个目录服务器，才能添加新用户，并且在初始安装期间，将创建用户 admin@internal。此帐户供初始配置环境和/或故障排除时使用。用户将通过其用户主体名称 (UPN)（格式为 user@domain）来标识。可以将多个目录服务器附加到 RHEV-M 并且支持此操作。如果管理员附加多个目录服务器，他们将能够通过从下拉菜单中选择正确的域来选择用于身份验证的域。

###### 管理多级别管理

要在红帽企业虚拟化中为对象设置细粒度的权限，请在 RHEV-M Web 界面上浏览到对象并将其选中。在下面的窗格中，导航到权限选项卡。在这里，管理员可以看到各用户对此对象拥有的访问级别，以及这些权限是直接设置的，还是从树中的父级继承。

要添加新用户，请单击权限选项卡中的添加按钮并使用显示的将权限添加到用户对话框来搜索所需用户并为此用户分配一个角色。还可以将权限分配到每个人，尽管通常不要这样做。

###### 用户门户

除了到目前为止使用的管理门户，RHEV-M 还提供用户门户。用户门户是普通用户连接来管理并使用虚拟机的位置。用户门户地址为 https://rhevm.podX.example.com/ovirt-engine/userportal。用户首次连接到用户门户时，他们需要在浏览器中设置安全例外以信赖 HTTPS 证书，或必须将来自 http://rhevm.podX.example.com/ca.crt 的 RHEV-M CA 证书安装作为浏览器中值得信赖的 CA 证书。

为了使用 SPICE 协议以连接到虚拟机控制台，必须安装浏览器扩展。在使用 Firefox 浏览器的 Red Hat Enterprise Linux 客户端上，可通过安装 spice-xpi 程序包来实现。 spice-xpi 程序包安装完成之后，重新启动 Firefox 以激活新扩展。在运行的 Internet Explorer 的 Windows 客户端上，首次尝试连接到控制台时，系统将要求安装 SPICE ActiveX 扩展。遵循屏幕上的说明以安装客户端。

###### 演示：管理用户、角色和权限

1. 加入 example.com 域：

```
# engine-manage-domains add --domain=example.com --user=rhevadmin --provider=IPA                -> 提示输入密码时，请使用 redhat
Enter password: redhat
The domain example.com has been added to the engine as an authentication source but no users from that domain have been granted permissions within the oVirt Manager.
Users from this domain can be granted permissions by editing the domain using action edit and specifying --add-permissions or from the Web administration interface logging in as admin@internal user.
oVirt Engine restart is required in order for the changes to take place (service ovirt-engine restart).
Manage Domains completed successfully

# service ovirt-engine restart      -> 重新启动 RHEV-M 以激活更改
```

2. 要确保 RHEV 实例绑定到 example.com 域，请运行：

```
# engine-manage-domains list
Domain: example.com
        User: rhevadmin@EXAMPLE.COM
Manage Domains completed successfully
```

3. 向用户 rhevadmin 授予超级用户权限。
使用 admin 用户名和 redhat 密码来登录 RHEV-M 管理门户。在顶部栏中，选择配置，再选择系统权限选项卡，然后单击添加按钮以创建新用户权限。
选择 example.com (example.com) 域，然后在搜索字段中键入 rhevadmin。用户显示在搜索字段中后，将其选中。
从下拉列表中，选择超级用户角色。单击确定以设置用户权限。

4. 以 rhevadmin 身份登录来测试新的超级用户。
从右上角中，单击用户 example.com，然后单击注销以注销。使用 rhevadmin 用户名和 redhat 密码重新登录。选择 example.com 作为域。

5. 只在虚拟机上创建允许基本操作和远程登录的新角色。
从右上角中，单击配置。在顶部栏中，从角色选项卡中，单击新建以定义新角色。
将新角色命名为 VMUserNoCD 并使用描述 use VMs, no CD。保持帐户类型设置为用户。扩展虚拟机树和基本操作树。确保仅选中基本操作和远程登录。
单击确定，以确认添加此新角色。您将在下一步将此权限添加至对象。

6. 创建另外一个用户并为其授予相应权限。
配置对话框仍然打开时，导航到系统权限选项卡。单击添加以添加新系统用户。
确保将搜索设置为 example.com 并输入 vmadminX 作为搜索项，其中 X 为您的 pod 号。在结果窗口中，在 vmadminX 前放置选中标记。从分配角色到用户下拉菜单中选择 PowerUserRole。单击确定以确认添加。

7. 创建另外一个用户并为其授予相应权限。
单击系统权限选项卡，然后单击添加以添加新用户。在 example.com 域中搜索 rheluserX（确保将 X 替换为您的工作站编号），并给此帐户分配先前创建的 VMUserNoCD 角色。单击确定之前，不要忘记选中 rheluserX 前面的复选框。

8. 在 workstation.podX.example.com 上，安装 spice-xpi 软件包。

    # yum -y install spice-xpi

9. 如果您已在 workstation.podX.example.com 上打开 Firefox，则将其关闭。

10. 在 workstation.podX.example.com 上启动新的 Firefox 实例并导航到 https://rhevm.podX.example.com。单击用户门户，以导航到 RHEV-M 用户门户。
在 Firefox 提供的错误页面上：

*    单击我了解风险以扩展 Firefox 中的证书异常对话框。
*    选择添加例外...。
*    在出现的弹出框中，单击获取证书按钮。
*    选择弹出窗口底部的确认安全例外。

11. 确保您可以使用新定义的角色来登录。
使用 vmadminX 用户名和 redhat 密码登录。确保选择 example.com 作为域。



### Remove RHEV Manager

从服务器中删除 RHEV 管理器软件需要 4 个步骤：

*    关闭 RHEV-M 并删除其配置。
*    删除 RHEV Manager 和 JBoss 软件包。
*    删除所有 ovirt-engine 目录并删除 PostgreSQL 数据库。
*    清除在 RHEV-M 安装过程中创建的任何 NFS 导出。

1. 首先，以 root 身份登录 RHEV-M 服务器并运行 engine-cleanup 命令，以关闭 RHEV-M 并删除其配置。此命令将销毁数据，因此请仅在从服务器中删除 RHEV-M 时执行此命令：

```
# engine-cleanup
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-dwh.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging-wsp.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf', '/etc/ovirt-engine-setup.conf.d/20-packaging-rhevm-reports.conf', '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-remove-20141216123409-fxre6z.log
          Version: otopi-1.3.0 (otopi-1.3.0-1.el6ev)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment customization
          Do you want to remove all components? (Yes, No) [Yes]: yes

          --== PRODUCT OPTIONS ==--

          Do you want to remove Engine database content? All data will be lost (Yes, No) [No]: yes
[ INFO  ] Stage: Setup validation
          During execution engine service will be stopped (OK, Cancel) [OK]: ok
          All the installed ovirt components are about to be removed, data will be lost (OK, Cancel) [Cancel]: ok
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping reports service
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Backing up PKI configuration and keys
[ INFO  ] Backing up database localhost:engine to '/var/lib/ovirt-engine/backups/engine-20141216123457.ImPi6v.dump'.
[ INFO  ] Clearing Engine database engine
[ INFO  ] Backing up database localhost:ovirt_engine_history to '/var/lib/ovirt-engine-dwh/backups/dwh-20141216123517.MG0DBs.dump'.
[ INFO  ] Clearing DWH database ovirt_engine_history
[ INFO  ] Backing up database localhost:ovirt_engine_reports to '/var/lib/ovirt-engine-reports/backups/reports-20141216123523.XBUKrE.dump'.
[ INFO  ] Clearing Reports database ovirt_engine_reports
[ INFO  ] Removing files
[ INFO  ] Reverting changes to files
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up

          --== SUMMARY ==--

          A backup of the Reports database is available at /var/lib/ovirt-engine-reports/backups/reports-20141216123523.XBUKrE.dump
          A backup of the DWH database is available at /var/lib/ovirt-engine-dwh/backups/dwh-20141216123517.MG0DBs.dump
          A backup of the Engine database is available at /var/lib/ovirt-engine/backups/engine-20141216123457.ImPi6v.dump
          ovirt-engine has been removed
          A backup of PKI configuration and keys is available at /var/lib/ovirt-engine/backups/engine-pki-20141216123457cVti4Q.tar.gz
          Engine setup successfully cleaned up

          --== END OF SUMMARY ==--

[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-remove-20141216123409-fxre6z.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20141216123530-cleanup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of cleanup completed successfully
``` 

2. 在关闭 RHEV-M 后，删除所有相关 RPM：

    # yum remove rhevm* vdsm-bootstrap *jboss* postgresql-server
          
3. 删除旧的 ovirt-engine 目录并删除所有剩下的 PostgreSQL 数据库。这包含以下配置和数据目录：

    /etc/ovirt-engine/
    /usr/share/ovirt-engine*
    /var/lib/pgsql/

4. 最后，清除由配置工具创建的任何 NFS 共享。删除 /etc/exports 中的相关条目并重新加载 nfs 服务，以重新读取其配置：

    # vim /etc/exports
    # service nfs reload

5. 清理包括删除 NFS 导出的目录，特别是如果重新安装 RHEV-M，则将重新使用相同的目录名。


### Troubleshooting RHEV-M Installation Issues

###### 防火墙配置

在先前安装中，engine-setup 命令会自动配置防火墙以允许通信。/etc/ovirt-engine/ 中已经包含了一些可以与 iptables 配合使用的行的示例。

```
# cat /etc/ovirt-engine/iptables.example
# Generated by ovirt-engine installer
#filtering rules
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type any -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 111 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 111 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 662 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 662 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 875 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 875 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 892 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 892 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 2049 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 32769 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 32803 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 5432 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 7410 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 6100 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT

#drop all rule
-A INPUT -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

###### 确认系统时间

安装 RHEV-M 时，将创建一个证书颁发机构证书，用于签署 RHEV-M 呈现的 Web 界面使用的主机证书。如果使用 SSL 加密时出现问题，安装 RHEV-M 时，如果用于托管 RHEV-M 的红帽企业 Linux 服务器的系统时钟不准确，则在使用 SSL 加密时可能会出现问题。 调整时区是不足以解决问题。确保将系统时钟和硬件时钟设置为正确的时间。

> 当 NTP 服务器在本地主机上运行时，ntpdate 将不工作。在此种情况下，使用 service 命令停止 ntpd 服务、与准确的时间服务器同步，然后重新启动 ntpd 服务。

如有必要，使用 ntpdate 命令与 NTP 时间服务器同步。hwclock 命令显示 BIOS 实时时钟中存储的时间。如果指定了 -w 选项，hwclock 会持续不断地将当前系统时间写入服务器的实时时钟。

```
# service ntpd stop
Shutting down ntpd:                                        [  OK  ]
# ntpdate classroom.example.com
16 Dec 23:41:17 ntpdate[30767]: adjust time server 172.25.254.254 offset 0.000541 sec
# hwclock -w
# hwclock
Tue 16 Jan 2014 11:41:30 PM PST  -0.937889 seconds
# service ntpd start
```

###### 内部管理帐户

所有 RHEV-M 管理功能都必须由管理帐户执行。RHEV-M 身份验证通常由外部 Windows Active Directory 服务器或 LDAP/IPA 服务器提供。 外部身份验证的成功存在疑问时，应使用名为 admin@internal 的内部管理帐户。如果此帐户密码丢失或更改，可通过重置密码恢复管理访问权限。

要重置内部管理帐户，请执行以下步骤：

1. 作为 RHEV-M Linux 服务器上的 root 登录.
2. 使用 engine-config -s AdminPassword=interactive 命令重置内部管理密码。出现提示时，输入新密码：

    # engine-config -s AdminPassword=interactive
    Please enter a password: New password
    Please reenter password: New password

3. 发送 service ovirt-engine restart 命令，以使更改生效。


###### 演示：更新系统时间并重置管理帐户密码

1. 要设置系统时钟，请使用 ntpdate 和 hwclock。请注意，还应配置 ntpd，但是在发出 ntpdate 命令期间，其不能运行。在 rhevm.podX.example.com 上，确保系统时钟设置正确。在课堂上，使用 classroom.example.com 作为 NTP 服务器。

```
[root@rhevm ~]# service ntpd stop
[root@rhevm ~]# ntpdate classroom.example.com
[root@rhevm ~]# grep server /etc/ntp.conf
...
server 172.25.254.254
...
[root@rhevm ~]# service ntpd start
[root@rhevm ~]# hwclock --systohc
```

2. 演示如何在 rhevm.podX.example.com 机器上将 admin@internal 密码设置为 root，然后将 admin@internal 密码重置为 test123：

```
[root@rhevm ~]# engine-config -s AdminPassword=interactive
Please enter a password: test123
Please reenter password: test123
[root@rhevm ~]# service ovirt-engine restart
Stopping engine-service:                                   [  OK  ]
Starting engine-service:                                   [  OK  ]
```

3. 退出 RHEV-M web 界面并使用新的密码登录。重新启动 ovirt-engine 服务时，web 界面允许登录之前将花费几分钟时间。
4. 重置 redhat 的 admin@internal 密码。

```
[root@rhevm ~]# engine-config -s AdminPassword=interactive
Please enter a password: redhat
Please reenter password: redhat
[root@rhevm ~]# service ovirt-engine restart
Stopping engine-service:                                   [  OK  ]
Starting engine-service:                                   [  OK  ]
```

###### 使用 RHEV 配置工具检索 RHEV 配置值

engine-config 工具使管理员可以更新 RHEV 安装的值。例如，可以重置密码，也可以更新配置设置。要更新值，需要使用 -s 标志。考虑以下用于设置用户自定义值的示例：

> 配置值存储在 RHEV-M 数据库中。除非数据库正在运行，否则更改将不会保存，重新启动 JBoss 后才应用更改。修改要生效的任何更改值之后，重新启动 ovirt-engine 服务。

```
# engine-config -s "UserDefinedVMProperties=macspoof=(true|false)"      -> 将创建一个新的自定义值，用户在使用其虚拟机时可以使用此值
# engine-config -a          -> 管理员可以运行 engine-config -a 以检索所有现有值并在需要时更新这些值
AbortMigrationOnError: false version: 3.0
AbortMigrationOnError: false version: 3.1
AbortMigrationOnError: false version: 3.2
AbortMigrationOnError: false version: 3.3
AsyncTaskPollingRate: 10 version: general
AsyncTaskZombieTaskLifeInMinutes: 3000 version: general
AuditLogAgingThreshold: 30 version: general
...
ClusterRequiredRngSourcesDefault:  version: 3.2
ClusterRequiredRngSourcesDefault:  version: 3.3
ClusterRequiredRngSourcesDefault:  version: 3.4
ClusterRequiredRngSourcesDefault:  version: 3.5
DefaultMTU: 1500 version: general

# engine-config -g "AuditLogCleanupTime"            -> -g 标志可用于检索特定键的值
AuditLogCleanupTime: 03:35:35 version: general

使用 engine-config 工具时，可以附加 --log-file 标志以将查询保存在文件中，这对于调试很有用。可以将配置文件作为参数传递，而不必手动输入要更新的键和值。
# engine-config -s PasswordEntry --admin-pass-file=/tmp/mypass
```

###### 演示：使用 RHEV 配置工具更新 RHEV 配置值

```
1. 显示可调 RHEV-M 值的列表。寻找会决定用户会话超时的值
# engine-config -l | grep -i timeout             
... output omitted ...
UserSessionTimeOutInterval: Timeout interval in minutes, after which inactive user sessions expire. A negative value indicates that sessions never expire. (Value Type: Integer)... output omitted ...

2. 获取 UserSessionTimeOutInterval 设置的当前值：
# engine-config -g UserSessionTimeOutInterval
UserSessionTimeOutInterval: 30 version: general

3. 30 是 RHEV-M 会话超时之前静止的分钟数。general 是要应用的 RHEV 版本（2.2、3.0、3.1、3.5）。将 UserSessionTimeOutInterval 设置的值更改为不活动两分钟之后超时：
# engine-config -s UserSessionTimeOutInterval=2
# engine-config -g UserSessionTimeOutInterval
UserSessionTimeOutInterval: 1 version: general

4. 重新启动 ovirt-engine 服务以应用更改。
# service ovirt-engine restart

5. 等待 web 界面变得可用并作为 rhevadmin 通过密码 redhat 在域 example.com 中登录 RHEV-M 管理门户。等待至少两分钟，期间不要操作键盘或鼠标，以查看会话超时。

6. 将 UserSessionTimeOutInterval 设置的值调整为不活动一小时之后超时：
# engine-config -s UserSessionTimeOutInterval=60
# service ovirt-engine restart
```

###### 读取 RHEV 日志文件

出于调试和故障排除目的，管理员可以使用 RHEV 日志文件，这些文件均位于 /var/log/ 目录下。通常有三个目录：

*    ovirt-engine 包含引擎活动的日志，比如与操作或系统相关的任务。
*    ovirt-engine-dwh 包含数据仓库的日志。此目录包含与 RHEV 数据仓库模块相关的日志。
*    ovirt-engine-reports 包含报告模块的日志。

日志行的格式如下：DATE TIME LOG_LEVEL MODULE (ThreadID/TaskID ) MESSAGE 虽然使用日志文件可能很繁琐，但管理员可以使用 grep 来查找某一特定模块发生的特定错误。例如，如果用户无法登录，那么 server.log 文件可能包含原因：

```
# grep ERROR /var/log/ovirt-engine/engine.log
2014-12-16 11:18:29,594 ERROR [org.ovirt.engine.core.dal.dbbroker.auditloghandling.AuditLogDirector] (ajp-/127.0.0.1:8702-3) Correlation ID: null, Call Stack: null, Custom Event ID: -1, Message: User rhevadmin cannot login, please verify the username and password.
```

出于故障排除目的，还可以附加 --log-file 标志以在特定文件中输出消息，以便于查明在设置阶段可能出现的根本原因错误：

```
# grep ERROR /root/debug.log
...
Error:  exception message: Integrity check on decrypted field failed (31) - PREAUTH_FAILED
```

如果当前日志级别不包含错误，那么可以使用 --log-level 标志来调整，然后重新运行该命令。允许的值包括：

*    DEBUG（默认日志级别）
*    INFO
*    WARN
*    ERROR

###### 使用日志收集器

红帽企业虚拟化管理器提供了一个实用程序，该实用程序可收集所有日志并将其存储在存档中：engine-log-collector。该命令是 engine-log-collector 软件包的一部分。所有系统信息、日志和数据库备份文件都放置到一个压缩文件中。要在运行 RHEV-M 的服务器上启动日志收集器，请以 root 身份登录，并执行 engine-log-collector 命令。命令输出将保存在 /tmp/logcollector/ 目录中。

通过更改 /etc/rhevm/logcollector.conf 文件永久指定选项。原始文件包括带有样本变量设置的注释行。几个有用的参数包括：rhevm 和 user。前者指向要查询的 RHEV 管理器主机，后者指定要进行身份验证的 RHEV 用户

红帽企业虚拟化管理日志

|日志文件 	                                           |描述                                                          |
|/var/log/ovirt-engine/engine-setup-*.log 	           |提供有关 RHEV 管理器安装和配置流程的信息。                         |
|/var/log/ovirt-engine/rhevm-dwh-setup-*.log 	       |包含有关 RHEV 管理器数据仓库 rhevm-history 数据库的安装和配置的信息。|
|/var/log/ovirt-engine/ovirt-engine-reports-setup-*.log|来自用于安装 RHEV 管理器 Reports 模块的日志。                     |
|/var/log/ovirt-engine/engine.log 	                   |反映所有 RHEV-Manager GUI 崩溃，目录查找、数据库和其他问题。        |

```
# engine-log-collector list
Please provide the REST API password for the admin@internal RHEV-M user (CTRL+D to skip): redhat
Host list (datacenter=None, cluster=None, host=None):
Data Center          | Cluster              | Hostname/IP Address
dcenter1             | cluster1             | 172.25.X.15
# engine-log-collector collect
Please provide the REST API password for the admin@internal RHEV-M user (CTRL+D to skip): redhat
About to collect information from 1 hypervisors. Continue? (Y/n): Y
INFO: Gathering information from selected hypervisors...
INFO: collecting information from 172.24.1.201
INFO: finished collecting information from 172.24.1.201
Please provide the password for the PostgreSQL user, postgres, to dump the engine PostgreSQL database instance (CTRL+D to skip): redhat
INFO: Gathering PostgreSQL the RHEV-M database and log files from localhost...
INFO: Gathering RHEV-M information...
INFO: Log files have been collected and placed in /tmp/logcollector/sosreport-LogCollector-rhevm-20121129060015-7050.tar.xz.
      The MD5 for this file is dc38ba4dddd051f1309fd8625a6b7050 and its size is 35.0M
```
