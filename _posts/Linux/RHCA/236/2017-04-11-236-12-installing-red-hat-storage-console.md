---
layout: post
title:  "Installing Red Hat Storage Console(236-12)"
categories: Linux
tags: RHCA 236
---

### 安装红帽 Gluster 存储控制台

###### 红帽 Gluster 控制台

红帽存储控制台提供基于 Web 的管理控制台和 API，以管理称为群集的红帽存储环境，包括相关的节点、用户和其上创建的卷。除了管理这些项目外，它也允许调配和监控红帽存储环境。 

红帽存储控制台需要安装一个节点，该节点不包含在红帽 Gluster 存储环境中。它还要求在该节点上配置如下额外频道：jb-eap-7-for-rhel-7-server-rpms、rhsc-3-for-rhel-7-server-rpms 和 rhs-nagios-3-for-rhel-7-server-rpms。配置了上述频道后，还应使用 yum 安装 rhsc 软件包。

    [root@demo-a ~]# yum install rhsc
      
完成安装时，执行 rhsc-setup 脚本以对红帽存储控制台执行安装后配置。在这一过程中，提供关于将要部署红帽存储控制台的环境的一些额外信息（如红帽存储控制台数据库后端的位置）。rhsc-setup 脚本执行以下操作：

    [root@demo-a ~]# rhsc-setup

1. 启动安装脚本: 该脚本提示确认将红帽 Gluster 控制台安装到本地主机上。

    [root@demo-a ~]# rhsc-setup
    [root@manager ~]# rhsc-setup
    [ INFO  ] Stage: Initializing
    [ INFO  ] Stage: Environment setup
              Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging.conf', '/etc/ovirt-engine-setup.conf.d/20-rhsc-packaging.conf']
              Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20160523195106-846j0i.log
              Version: otopi-1.4.0_master (otopi-1.4.0-0.0.1.master.el6ev)
    [ INFO  ] Stage: Environment packages setup
    [ INFO  ] Stage: Programs detection
    [ INFO  ] Stage: Environment setup
    [ INFO  ] Stage: Environment customization

              --== PRODUCT OPTIONS ==--

              Configure Engine on this host (Yes, No) [Yes]:
          
2. 产品选项: 检查要执行新的安装还是升级。

    ...
              --== PACKAGES ==--

    [ INFO  ] Checking for product updates...
    [ INFO  ] No product updates found

3. 防火墙配置: 检测活动且已启用的防火墙，并提示确认自动配置。

            ...
             --== NETWORK CONFIGURATION ==--

              Setup can automatically configure the firewall on this system.
              Note: automatic configuration of the firewall may overwrite current settings.
              Do you want Setup to configure the firewall? (Yes, No) [Yes]: 

4. 主机名配置: 检查主机名称，并提示确认或进行必要的修改。

    ...
             Host fully qualified DNS name of this server [manager.lab.example.com]:

5. 数据库配置: 提示提供 PostgreSQL 数据位置；可以自动配置该数据库，包括添加用户和数据库。或者，可以手动配置。

    ...
             --== DATABASE CONFIGURATION ==--

              Where is the Engine database located? (Local, Remote) [Local]:
              Setup can configure the local PostgreSQL server automatically for the engine to run. This may conflict with existing applications.
              Would you like Setup to automatically configure PostgreSQL and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 

6. 管理员凭据: 提示为自动创建的红帽 Gluster 存储控制台管理用户 admin@internal 提供管理员密码。使用弱密码时需要确认。

    ...
             --== OVIRT ENGINE CONFIGURATION ==--

              Engine admin password:
              Confirm engine admin password:
    [WARNING] Password is weak: it is based on a dictionary word
              Use weak password? (Yes, No) [No]: 

6. 证书配置: 证书提供控制台与主机之间的安全通信。

    ...
             --== PKI CONFIGURATION ==--
              Organization name for certificate [lab.example.com]:

7. Web 服务器配置: 提示使用默认的自签名证书，以及将控制台的登录页设为 Apache 提供的默认页面。也可以将其他证书用于外部 HTTPS 连接，而不影响控制台与主机的通信。

     ...
          --== APACHE CONFIGURATION ==--

              Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
              Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]:
              Setup can configure Apache to use SSL using a certificate issued from the internal CA.
              Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 

8. 证书配置: 默认情况下，glusterfs 使用应用模式，因此将跳过 NFS 配置。

    ...
               --== SYSTEM CONFIGURATION ==--

    [ INFO  ] NFS configuration skipped with application mode Gluster

9. 控制台设置: 提示确认是否使用 Red Hat Access 插件。

    ...
             --== MISC CONFIGURATION ==--

              Would you like transactions from the Red Hat Access Plug-in sent from Red Hat Gluster Storage Console to be brokered through a proxy server? (Yes, No) [No]: 

10. 监控: 提示确认要启用还是禁用监控。

    ...
              --== END OF CONFIGURATION ==--

              Would you like external monitoring to be enabled? (Yes, No) [Yes]:
    [ INFO  ] Stage: Setup validation
    [WARNING] Less than 16384MB of memory is available

              --== CONFIGURATION PREVIEW ==--

              Application mode                        : gluster
              Firewall manager                        : iptables
              Update Firewall                         : True
              Host FQDN                               : manager.lab.example.com
              Engine database name                    : engine
              Engine database secured connection      : False
              Engine database host                    : localhost
              Engine database user name               : engine
              Engine database host name validation    : False
              Engine database port                    : 5432
              Engine installation                     : True
              PKI organization                        : lab.example.com
              Configure local Engine database         : True
              Set application as default page         : True
              Configure Apache SSL                    : True
              Nagios monitoring enabled for gluster hosts: True

              Please confirm installation settings (OK, Cancel) [OK]: 

11. 总结: 在完成安装脚本时，记下提供的额外信息。复制 ssh 证书指纹、ssh 公钥指纹和红帽 Gluster 存储控制台 url，以备将来参考。


           [ INFO  ] Stage: Transaction setup
    [ INFO  ] Stopping engine service
    [ INFO  ] Stage: Misc configuration
    [ INFO  ] Stage: Package installation
    [ INFO  ] Stage: Misc configuration
    [ INFO  ] Creating PostgreSQL 'engine' database
    [ INFO  ] Configuring PostgreSQL
    [ INFO  ] Creating/refreshing Engine database schema
    [ INFO  ] Creating CA
    [ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
    [ INFO  ] Stage: Transaction commit
    [ INFO  ] Stage: Closing up

              --== SUMMARY ==--

    [ INFO  ] To enable monitoring, ensure that the managed nodes are migrated to Red Hat Gluster Storage 3.0 or above. Also ensure that the auto-discovery command (configure-gluster-nagios) is executed to start monitoring the Red Hat Gluster Storage Nodes after the nodes are added to Red Hat Gluster Storage Console. For more details, refer Red Hat Gluster Storage Console Administration Guide.
    [WARNING] Less than 16384MB of memory is available
              Engine database resources:
                  Database name:      engine_20160523210515
                  Database user name: engine_20160523210515
              SSH fingerprint: 11:8C:93:7C:5D:44:99:63:E8:D5:6D:76:F5:F0:A3:31
              Internal CA BF:25:F4:F1:82:BC:A9:CF:79:A3:46:AD:04:A5:D0:67:1A:B9:4B:6E
              Web access is enabled at:
                  http://manager.lab.example.com:80/ovirt-engine
                  https://manager.lab.example.com:443/ovirt-engine
              Please use the user "admin" and password specified in order to login

              --== END OF SUMMARY ==--

    [ INFO  ] Starting engine service
    [ INFO  ] Restarting httpd
    [ INFO  ] Stage: Clean up
              Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20160523195106-846j0i.log
    [ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20160523210904-setup.conf'
    [ INFO  ] Stage: Pre-termination
    [ INFO  ] Stage: Termination
    [ INFO  ] Execution of setup completed successfully
    [root@manager ~]#
            
完成 rhsc-setup 配置后，可以通过 https://manager.lab.example.com/ovirt-engine 并使用 admin/redhat 凭据访问红帽存储控制台 Web 管理界面。 

###### 导入现有群集到控制台

可以将现有的群集及从属于该群集的所有主机导入到红帽 Gluster 存储控制台。

1. 在树形窗格中，单击 System 选项卡，再单击 Clusters 选项卡。 
2. 单击 New 以打开 New Cluster 对话框。
3. 输入群集的名称、描述和兼容版本。名称中不能包含空格。
4. 选择 Import existing gluster configuration 以导入该群集。
5. 在 Address 字段中，输入群集中主机的主机名称或 IP 地址。将显示主机指纹，以指示存在主机连接。如果主机不可访问，则 Fingerprint 字段中将显示 Error in fetching fingerprint。
6. 在 Password 字段中输入主机的 root 密码，再单击 OK。 
7. 这时将打开 Add Hosts 弹出窗口，并显示属于该群集的主机列表。 
8. 输入每一主机的名称和 root 密码。或者，若要对所有主机使用相同的密码，可选择 Use a common password 并输入密码。 
9. 单击 apply 为所有数据设置密码，然后单击 OK。 

### 实验：安装红帽 Gluster 控制台

安装和配置红帽 Gluster 控制台

1. 在 manager 上，使用 yum 安装 rhsc 软件包。

    [root@manager ~]# yum -y install rhsc

2. 运行 rhsc-setup 脚本。

    安装完成时，使用 rhsc-setup 命令启动控制台配置。

    [root@manager ~]# rhsc-setup
    [ INFO  ] Stage: Initializing
    [ INFO  ] Stage: Environment setup
              Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging.conf', '/etc/ovirt-engine-setup.conf.d/20-rhsc-packaging.conf', '/root/rhsc-install.conf']
              Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20160601054304-syadix.log
              Version: otopi-1.4.0_master (otopi-1.4.0-0.0.1.master.el6ev)
    [ INFO  ] Stage: Environment packages setup
    [ INFO  ] Stage: Programs detection
    [ INFO  ] Stage: Environment setup
    [ INFO  ] Stage: Environment customization

              --== PRODUCT OPTIONS ==--

              Configure Engine on this host (Yes, No) [Yes]: Enter

              --== PACKAGES ==--

    [ INFO  ] Checking for product updates...
    [ INFO  ] No product updates found

              --== NETWORK CONFIGURATION ==--

              Setup can automatically configure the firewall on this system.
              Note: automatic configuration of the firewall may overwrite current settings.
                     Do you want Setup to configure the firewall? (Yes, No) [Yes]: Enter
    [ INFO  ] iptables will be configured as firewall manager.
              Host fully qualified DNS name of this server [manager.lab.example.com]: Enter

               --== DATABASE CONFIGURATION ==--

                     Where is the Engine database located? (Local, Remote) [Local]: Enter
              Setup can configure the local PostgreSQL server automatically for the engine to run. This may conflict with existing applications.
              Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Enter

              --== OVIRT ENGINE CONFIGURATION ==--

              Engine admin password: redhat
              Confirm engine admin password: redhat
    [WARNING] Password is weak: it is based on a dictionary word
                     Use weak password? (Yes, No) [No]: Yes

              --== PKI CONFIGURATION ==--

                      Organization name for certificate [lab.example.com]: Enter

              --== APACHE CONFIGURATION ==--

              Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
              Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: Enter
              Setup can configure apache to use SSL using a certificate issued from the internal CA.
                      Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Enter

              --== SYSTEM CONFIGURATION ==--

    [ INFO  ] NFS configuration skipped with application mode Gluster

              --== MISC CONFIGURATION ==--

              Would you like transactions from the Red Hat Access Plugin sent from Red Hat Gluster Storage Console to be brokered through a proxy server? (yes, No) [No]: Enter


              --== END OF CONFIGURATION ==--

                       Would you like external monitoring to be enabled? (Yes, No) [Yes]: Enter
    [ INFO  ] Stage: Setup validation
    [ WARNING ] Less than 16384MB of memory is available

              --== CONFIGURATION PREVIEW ==--

              Application mode                        : gluster
              Firewall manager                        : iptables
              Update Firewall                         : True
              Host FQDN                               : manager.lab.example.com
              Engine database name                    : engine
              Engine database secured connection      : False
              Engine database host                    : localhost
              Engine database user name               : engine
              Engine database host name validation    : False
              Engine database port                    : 5432
              PKI organization                        : lab.example.com
              Configure local Engine database         : True
              Set application as default page         : True
              Configure Apache SSL                    : True
              Nagios monitoring enabled for gluster hosts: True

                      Please confirm installation settings (OK, Cancel) [OK]: Enter
    [ INFO  ] Stage: Transaction setup
    [ INFO  ] Stopping engine service
    [ INFO  ] Stage: Misc configuration
    [ INFO  ] Stage: Package installation
    [ INFO  ] Stage: Misc configuration
    [ INFO  ] Initializing PostgreSQL
    [ INFO  ] Creating PostgreSQL 'engine' database
    [ INFO  ] Configuring PostgreSQL
    [ INFO  ] Creating/refreshing Engine database schema
    [ INFO  ] Creating CA
    [ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
    [ INFO  ] Stage: Transaction commit
    [ INFO  ] Stage: Closing up

              --== SUMMARY ==--

    [ INFO  ] To enable monitoring, ensure that the managed nodes are migrated to Red Hat Gluster Storage 3.0 or above. Also ensure that the auto-discovery command (configure-gluster-nagios) is executed tos tart monitoring the Red Hat Gluster Storage Nodes after the nodes are added to Red Hat Gluster Storage Console. For more details, refer Red Hat Gluster Storage Console Administration Guide.
    [WARNING] Less than 16384MB of memory is available.
              SSH fingerprint: 8D:57:83:3A:D9:1D:DE:FC:25:BC:1D:6C:C1:08:EE:2A
              Internal CA E1:99:1B:B9:D6:48:25:A7:6E:34:0F:EF:7A:9F:45:BE:EA:C6:A0:90
              Web access is enabled at:
                  http://manager.lab.example.com:80/ovirt-engine
                  https://manager.lab.example.com:443/ovirt-engine
              Please use the user "admin" and password specified in order to login

              --== END OF SUMMARY ==--

    [ INFO  ] Starting engine service
    [ INFO  ] Restarting httpd
              Make sure the managed nodes get migrated to RHS-3.0 to get monitoring enabled for them.
    [ INFO  ] Stage: Clean up
              Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20141218035553.log
    [ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20141218040251-setup.conf'
    [ INFO  ] Stage: Pre-termination
    [ INFO  ] Stage: Termination
    [ INFO  ] Execution of setup completed successfully

3. 将现有的 Gluster 环境导入到红帽 Gluster 控制台。

    a. 在 workstation 上打开浏览器并前往 https://manager.lab.example.com，单击 admin portal 登录，再单击 I Understand the Risks、Add Exception 和 Confirm Security Exception 以接受自签名 SSL 证书。使用 admin 用户和密码 redhat 登录。

    b. 前往 Clusters 选项卡，再单击 New 按钮。这时将显示标题为 New Cluster 的弹出窗口。输入 gluster-cluster 作为名称，添加描述和下方的红帽  Gluster 存储环境信息，然后单击 OK。
    
    新群集
    导入现有 gluster 配置	        选中
    地址	                        servera.lab.example.com
    密码	                        redhat

    c. 在下一屏幕中，选择 Use a common password，输入密码 redhat，再单击 Apply，然后单击 OK。前往 Hosts 选项卡，在所有主机都显示 Up 状态时，浏览所有选项卡并检查红帽  Gluster 存储元素的详细信息。 


### 总结

在本章中，您已学会如何：

*    使用脚本 rhsc-setup 安装红帽存储控制台，以提供基于 Web 的管理控制台。
*    导入现有的红帽 Gluster 存储环境，使其由红帽 Gluster 存储控制台管理。 
