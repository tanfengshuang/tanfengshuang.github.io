---
layout: post
title:  "Implementing Puppet in Red Hat Satellite 6(405-14)"
categories: Linux
tags: RHCA 405
---

### 创建 Puppet 存储库

###### 自定义产品和存储库

红帽卫星 6 使用 Puppet 来配置它管理的主机。每一卫星或胶囊服务器充当客户端系统的 Puppet 宿主。卫星将 Puppet 类发布到客户端主机。虽然类是系统配置的基本单元，但它们作为 Puppet 模块载入到卫星服务器上。

> 注意: 红帽卫星胶囊服务器可在大规模环境中扩展红帽卫星服务器。它们提供软件内容，并且充当它们管理的卫星客户端的 Puppet 宿主。

当红帽软件内容添加到卫星服务器时，将自动创建红帽存储库及其父级产品。但是，若要在卫星服务器上托管自定义软件内容或 Puppet 模块，管理员必须创建存储库和产品。简单而言，产品是存储库的集合，根据管理员的偏好进行分组。这样，管理员可以将不同软件供应商的软件存储库或相关的 Puppet 模块分组为独立的产品。

与红帽内容一样，自定义产品和存储库是卫星服务器上的内容实体。它们在组织化上下文中创建和管理。在组织上下文下创建的产品和存储库仅对该组织可见。

###### 创建自定义产品

自定义软件和 Puppet 模块软件包托管在卫星服务器上的存储库中。存储库不能以自身的实体单独存在，必须在产品内创建；因此，管理员必须首先创建产品。

管理员可以在卫星 Web UI 中以 admin 用户身份执行下列步骤来创建产品。以下示例演示了在 ACME_Corporation 组织中创建产品。

1. 从左上角的上下文菜单，单击 Any Context → Any Organization → Acme_Corporation，以进入 Acme_Corporation 组织上下文。
2. 单击 Content → Products。
3. 单击 New Product 按钮。
4. 输入该产品的以下信息。

*    在必填的 Name 字段中输入名称。
*    在必填的 Label 字段中输入标签。
*    GPG Key 字段在验证软件包时使用。创建用于管理 Puppet 模块的产品时用不到此字段。
*    Sync Plan 字段在维护软件包时使用。创建用于管理 Puppet 模块的产品时用不到此字段。
*    此外，也可以在 Description 字段中输入更为详细的产品描述。

5. 单击 Save 按钮。

###### 创建 Puppet 存储库

大多数卫星用户已熟知卫星如何将软件发布到客户单系统。Yum 存储库包含客户端系统可以安装的软件包。类似地，红帽卫星 6 引入了 Puppet 存储库的概念，这些存储库是 Puppet 模块的集合。在红帽卫星管理 Puppet 存储库与管理 Yum 存储库的方式相似。

Puppet 存储库必须是红帽卫星服务器中某一组织内某产品的一个部分。存储库可以作为现有产品的一部分创建，或者为满足拥有该 Puppet 存储库的需要而专门创建一个新产品。

下列步骤介绍了如何在红帽卫星中创建 Puppet 存储库：

1. 选择默认的组织。单击左侧的 Any Context 选项卡，再从下拉菜单中选择相应的组织。
2. 选择 Content → Products 选项卡，以导航到产品管理屏幕。
3. 选择或创建产品以包含 Puppet 存储库。
4. 创建 Puppet 存储库。在所选产品中选择 Repositories 选项卡，再单击 Create Repository 按钮。
5. 完成显示的表单。至少填写存储库的名称，然后选择 puppet 作为存储库类型。
6. 单击 Save 按钮，以确认选择并创建该存储库。

创建了 Puppet 存储库后，Puppet 模块就可加载到卫星服务器中。

###### 将 Puppet 模块上传到 Puppet 存储库

下列步骤介绍了如何将 Puppet 模块导入到红帽卫星中的 Puppet 存储库：

1. 选择含有 Puppet 存储库的组织，该存储库应包含要上传的模块。
2. 选择 Content → Products 选项卡。
3. 选择与包含 Puppet 存储库的产品对应的链接。
4. 选择与存储库对应的链接。
5. 在 Upload Puppet Module 框中，单击 Browse 按钮。
6. 导航并选择含有 Puppet 模块的文件，然后单击 Upload 按钮。 


### 练习：创建 Puppet 存储库

1. 将 chrony Puppet 模块下载到本地目录。启动 Web 浏览器并导航到 http://materials.example.com/modules。右键单击 rht-chrony Puppet 模块的名称，将它保存到本地文件。
2. rht-chrony 模块具有一个依赖项。它使用 puppetlabs-stdlib Puppet 模块提供的函数。将 stdlib Puppet 模块下载到本地目录。启动 Web 浏览器并导航到 http://materials.example.com/modules。右键单击 puppetlabs-stdlib 模块的名称，将它保存到本地文件。
3. 打开 Web 浏览器并导航到 http://&rhsfqdn;。以 admin 身份并使用密码 redhat 登录卫星 Web UI。
4. 单击左侧的 Any Context 选项卡，将 Operations 选为默认的组织。在出现的下拉菜单中，选择 Operations。
5. 选择 Content → Products 选项卡，以导航到 Products 管理屏幕。
6. 创建用于包含 Puppet 存储库的产品。 

    a. 单击 New Product 按钮。
    b. 完成显示的表单。按照以下表格填写字段：
    *    字段	值
    *    Name	Operations Product
    *    Label	Operations_Product（自动生成）

    剩余的字段保持不变或留空。

    c. 单击 Save 按钮，以确认选择并创建该产品。 

7. 创建 Puppet 存储库。

    a. 选择 Repositories 选项卡，再单击 Create Repository 按钮。

    b. 完成显示的表单。按照以下表格填写字段：
    
    *    字段	值
    *    Name	Operations Modules
    *    Label	Operations_Modules（自动生成）
    *    Type	puppet

    剩余的字段保持不变或留空。

    c. 单击 Save 按钮，以确认选择并创建该存储库。 

8. 将 rht-chrony Puppet 模块上传到该存储库。

    a. 选择 Operations Modules 链接。
    b. 在 Upload Puppet Module 框中，单击 Browse... 按钮。
    c. 导航并选择包含 chrony Puppet 模块的文件 rht-chrony-0.1.0.tar.gz。
    d. 单击 Upload 按钮，以上传 Puppet 模块。 

9. 执行上述步骤，将 puppetlabs-stdlib Puppet 模块上传到该存储库。

10. 可以通过几种方式验证上传成功。完成每次上传时，Upload Puppet Module 框中将出现对话框，指出 “Content successfully uploaded”。

若要通过另一种方式确认上传成功，可导航到 Products 管理屏幕，再选择 Operations Product 链接。选择 Repositories 选项卡后，Content 列应当会显示链接 2 Puppet Modules。单击该链接将显示 chrony-0.1.0 和 stdlib-4.9.0 模块的信息。 


### 连接 Puppet 客户端

###### 安装和配置 Puppet 代理

在 Puppet 模块上传到红帽卫星服务器（或胶囊）后，需要在客户端系统上部署和配置 Puppet 代理软件，从而与作为 Puppet 宿主的卫星服务器配合。要完成 Puppet 代理配置，必须完成下列一般步骤：

1. 定义提供必要软件包和 Puppet 模块的内容视图。
2. 将客户端注册到红帽卫星。
3. 安装 Puppet 代理软件。
4. 签署客户端主机证书。
5. 启动 Puppet 代理。 

###### 定义提供必要软件包和 Puppet 模块的内容视图

红帽卫星内容视图决定哪些软件包和哪些 Puppet 模块可以安装到客户端系统上。由于 Puppet 可能会安装软件包，与系统关联的内容视图应当提供包括依赖项在内的必要软件包，供 Puppet 用于配置系统上的服务。内容视图不决定系统的配置方式，它提供 Puppet 配置系统时所需软件包的访问权限。

尽管如何定义提供软件包访问权限的内容视图不在本课程范围内，但了解在使用卫星服务器时应当向 Puppet 客户端系统提供什么软件也会有所帮助。必须创建内容视图，提供对 puppet 软件包及其依赖项的访问权限。至少，应由红帽卫星工具 6.1和红帽企业 Linux 存储库提供这些软件包。

此外，也要考虑 Puppet 类所需的其他软件。Puppet 类可以包含用于安装软件的资源。例如，配置 Web 服务器的 Puppet 类可能会包含 httpd 软件包及其依赖项。

> 重要： 可以在红帽卫星中创建内容过滤器，限制客户端主机有权访问的软件包。在定义内容过滤器时请谨慎操作。请记住，Puppet 类必须具有它们要安装的软件包的访问权限，从而配置客户端系统以提供功能。

内容视图也决定客户端主机有权访问的 Puppet 模块。在内容视图中定义时不会激活模块中的 Puppet 类，而仅仅是使其可用。在编辑内容视图时，请选择 Puppet Modules 选项卡。单击 Add New Module 按钮，可以从之前加载到卫星服务器的模块中选择，以发布到客户端主机。在创建内容视图后，必须发布并提升到客户端将要注册的软件环境中。

在内容视图创建、发布并提升后，红帽卫星 6 将在相关组织的每一生命周期环境中为该内容视图创建 Puppet 环境。Puppet 环境名称具有下列结构：

```
KT_ORG_ENV_VIEW_#
```

其中，ORG 是组织名称，ENV 是生命周期环境，VIEW 是内容视图名称，# 则是内部序列号。如果尚未分配到 Puppet 环境，则将主机组分配到主机时一些菜单项将不处于有效状态。

###### 将客户端注册到红帽卫星

为 Puppet 客户端创建了内容视图后，客户端应当注册到红帽卫星服务器，从而能与该内容视图关联。自动执行此过程的最佳方式是使用激活密钥。

为 Puppet 客户端创建激活密钥时，密钥应当将客户端绑定到软件环境以及提供所需 Puppet 模块的内容视图。在选择了适当的组织上下文后，选择 Content → Activation keys。选择现有激活密钥的 Details 选项卡，或者单击 New Activation Key 按钮。这将显示表单，从中可以选择软件环境以及内容视图。

在注册客户端系统后，必须在客户端上安装提供 CA 证书的 RPM，该 CA 证书用于签署卫星服务器主机证书。就位之后，subscription-manager 命令可将系统注册到使用刚才创建的激活密钥的组织。

```
[root@host ~]# yum -y install http://SATELLITE.FQDN/pub/katello-ca-consumer-latest.noarch.rpm
[root@host ~]# subscription-manager register --org=ORG --activationkey='KEY'
```

###### 安装 Puppet 代理软件

使用 yum 安装 Puppet 代理。在注册时通常会安装的另一软件包是 katello-agent。借助 Katello Agent 这款软件，红帽卫星可以执行客户端主机的远程软件管理。

```
[root@host ~]# yum -y install puppet katello-agent
```

这应当在客户端系统注册了激活密钥并且可以访问相应的存储库之后执行。在构建虚拟机基础映像时，也可以提前安装这些软件包。

> 注意: 我们课堂环境中的红帽卫星服务器尚未配置成提供软件包。已经配置了 Yum，以用于访问不属于卫星安装一部分的存储库。

安装了 Puppet 代理软件后，必须修改 Puppet 主配置文件，以便 Puppet 代理配置为将卫星服务器/胶囊用作 Puppet 宿主。将以下行添加到 /etc/puppet/puppet.conf：

```
server=satellite.FQDN
```

###### 签署客户端主机证书

完成客户端注册事务还需要两个步骤：与 Puppet 宿主连接，然后签署 Puppet 代理主机证书。使 Puppet 代理联系 Puppet 宿主。这可以通过 puppet 命令手动执行：

```
[root@host ~]# puppet agent --test --noop
```

Puppet 代理将发送主机证书到 Puppet 宿主，以允许安全通信。默认情况下，Puppet 代理每隔 120 秒连接宿主，并要求它签署其主机客户端请求，直到获得了签名后的证书。可以通过 puppet agent 命令的 --waitforcert=DELAY 选项更改默认延迟。

登录卫星界面，再执行以下步骤来签署 Puppet 代理出示的主机证书。

1. 管理 Puppet 证书并不属于特定红帽卫星组织的职能。将组织上下文设为 Any Organization。
2. 选择 Infrastructure → Capsules 下拉菜单项。
3. 单击胶囊服务器主机名右侧的 Certificates 按钮。
4. 出现主机证书列表时，单击位于需要签署其证书的主机右侧的 Sign 按钮。

红帽卫星可以配置为自动签署 Puppet 主机证书。显示胶囊的主机证书列表时，单击列表上方出现的 Autosign Entries 按钮。显示的屏幕允许管理员创建主机名条目（可以包含通配符），使得在主机第一次连接卫星服务器时自动签署其主机证书。
警告

将红帽卫星配置为自动签署 Puppet 主机证书可造成安全隐患。在这种情形中，任何主机可以连接并请求 Puppet 清单（其中可能包含特权信息，如密码、共享密钥和证书）。 

###### 启动 Puppet 代理

下列命令将在 Red Hat Enterprise Linux 7 系统上启动并启用 Puppet 代理作为守护进程：

```
[root@host ~]# systemctl start puppet.service
[root@host ~]# systemctl enable puppet.service
ln -s '/usr/lib/systemd/system/puppet.service' '/etc/systemd/multi-user.target.wants/puppet.service'
```


### 练习：连接 Puppet 客户端

1. 为客户端主机创建内容视图。它应当包含具有 chrony 和 stdlib Puppet 模块的 Puppet 存储库。

    a. 选择 Any Context → Any Organization → Operations，以将组织上下文设为 Operations。

    b. 选择 Content → Content Views，然后单击 Create New View 按钮。

    使用下方的值完成显示的表单：
    
    *    字段	            值
    *    Name	            Operations Content
    *    Label	            Operations_Content（自动生成）
    *    Description	    Content view for Operations servers
    *    Composite View?	不选中

    单击 Save 按钮确认。

    c. 单击 Puppet Modules 选项卡，然后单击 Add New Module 按钮。

    在 Actions 列中，单击与 chrony 和 stdlib 模块对应的 Select a Version 按钮。

    单击 chrony 模块的 Use Latest (currently 0.1.0) 版本的 Select Version 按钮。对 stdlib 模块的 Use Latest (currently 4.9.0) 版本执行同样的操作。

    d. 发布内容视图，并将它提升到 Dev 环境。

    单击 Publish New Version 按钮，然后单击 Save 以确认您要进行发布。

    等待发布完成，然后在 Actions 列中单击该内容视图 1.0 版本的 Promote 按钮。选择 Dev 环境，然后单击 Promote Version 按钮。

    完成时，您应当看到 Library 和 Dev 都显示在 Environments 列中。 

2. 确认正确的位置已分配给 Operations 组织的 Puppet 环境。

    a. 若有必要，将 Operations 选择为默认的组织上下文。

    b. 选择 Configure → Environments，以导航到 Puppet Environments 屏幕。

    c. 选择 Operations 内容库的环境 KT_Operations_Library_Operations_Content_#。

    d. 单击 Locations 子选项卡，然后将 Boston 和 Default Location 放入 Selected items 窗口。

    e. 单击 Submit 以应用更改。

    f. 对 Operations 组织的 Dev Puppet 环境重复上述步骤。选择 Operations 开发内容的环境 KT_Operations_Dev_Operations_Content_#。

    g. 单击 Locations 子选项卡，然后将 Boston 和 Default Location 放入 Selected items 窗口。

    h. 单击 Submit 以应用更改。 

3. 创建激活密钥，以注册 Puppet 客户端计算机。

    a. 确保 Operations 选定为当前组织上下文。

    b. 选择 Content → Activation keys，再单击 New Activation Key 按钮。

    使用下方的值完成显示的表单：
    
    *    字段	                值
    *    Name	                Operations Host
    *    Content Host Limit: Unlimited Content Hosts	保留选中
    *    Description	        Activation key that registers Operations hosts
    *    Environment	        选择 Dev
    *    Content View	        从下拉菜单中选择 Operations Content

    单击 Save 以确认。

    c. 选择 Subscriptions 选项卡，取消选中 Auto-Attach 复选框，然后单击 Save 按钮。这可防止红帽卫星将软件订阅自动关联到使用此激活密钥注册的内容主机。

    单击 Add 子选项卡，然后选择 Operations Product 订阅。单击 Add Selected 按钮确认。这将授予客户端系统访问产品提供的 Puppet 存储库的权限。 

4. 配置 Puppet 客户端主机以供红帽卫星使用。在 servera 上以 root 身份登录，再利用 Operations Host 激活密钥注册。

```
[root@servera ~]# yum -y install http://satellite.lab.example.com/pub/katello-ca-consumer-latest.noarch.rpm
... Output omitted ...
[root@servera ~]# subscription-manager register --org 'Operations' \
                      --activationkey 'Operations Host'
The system has been registered with ID: 3badb391-9eaa-448c-91f7-bc4fae059510

Installed Product Current Status:
Product Name: Red Hat Enterprise Linux Server
Status:       Not Subscribed

Unable to find available subscriptions for all your installed products.
```

subscription-manager 无法为客户端找到可用的订阅。这是因为我们实验环境中的红帽卫星服务器尚未配置提供红帽软件订阅的清单。 

5. 通过浏览器访问红帽卫星 Web 界面，再确认客户端主机已在红帽卫星中注册。

    a. 将组织上下文设为 Operations。

    b. 选择 Hosts → Content Hosts。

    c. 出现的主机列表中应当有 servera.lab.example.com 的一个条目。它应当具有最近的注册时间。 

6. 返回到客户端系统上的 root 会话。在主机上安装 Puppet 和 Katello Agent 软件。

```
[root@servera ~]# yum -y install puppet katello-agent
... Output omitted ...
Installed:
  katello-agent.noarch 0:2.2.5-1.el7sat      puppet.noarch 0:3.6.2-4.el7sat     
Dependency Installed:
... Output omitted ...
Complete!
```

7. 将 Puppet 配置为使用红帽卫星服务器。

```
[root@servera ~]# echo 'server=satellite.lab.example.com' >> /etc/puppet/puppet.conf
```

8. 显示 Puppet 将执行的操作（不要应用它们）。注意这里存在一个证书问题。我们必须在卫星服务器上解决该问题。

```
[root@servera ~]# puppet agent --test --noop
Info: Creating a new SSL key for servera.lab.example.com
Info: Caching certificate for ca
Info: csr_attributes file loading from /etc/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for servera.lab.example.com
Info: Certificate Request fingerprint (SHA256): E5:04:15:71:F6:9B:A7:EF:
 88:84:88:53:E1:C2:5F:9E:97:35:E3:22:31:5F:13:3A:10:8E:4F:0E:61:3E:AD:C5
Info: Caching certificate for ca
Exiting; no certificate found and waitforcert is disabled
```

9. 在卫星中签署该客户端系统的主机证书。

    a. 返回到显示有卫星用户界面的 Web 浏览器，再选择 Infrastructure → Capsules。

    b. 单击 satellite.lab.example.com 胶囊主机名右侧的 Certificates 按钮。

    c. 此时将显示主机证书列表。单击 servera 主机名右侧的 Sign 按钮。 

10. 在卫星客户端系统上，显示 Puppet 将执行的操作（不要应用它们）。请注意这里存在一个目录问题，但下一练习中会解决此问题。

```
[root@servera ~]# puppet agent --test --noop
Info: Caching certificate for servera.lab.example.com
Info: Caching certificate_revocation_list for ca
Info: Caching certificate for servera.lab.example.com
Warning: Unable to fetch my node definition, but the agent run will continue:
Warning: Error 400 on SERVER: Failed to find servera.lab.example.com via exec:
 Execution of '/etc/puppet/node.rb servera.lab.example.com' returned 1:
Info: Retrieving plugin
Error: /File[/var/lib/puppet/lib]: Could not evaluate: Could
 not retrieve information from environment production source(s)
 puppet://satellite.lab.example.com/plugins
Info: Caching catalog for servera.lab.example.com
Info: Applying configuration version '1441218498'
Notice: Finished catalog run in 0.07 seconds
```


### 通过主机组标准化配置

###### 通过 Puppet 标准化系统

在 Puppet 模块上传到红帽卫星中的 Puppet 存储库后，这些模块需要关联到待配置的客户端计算机。这通过使用主机组来完成。主机组定义了一组默认值，主机在被放入该组时将继承这些值。主机仅可从属于一个主机组。例如，可能有一个主机组配置数据中心内的 Web 服务器。另一主机组可能配置 DNS 服务器。

主机组创建后，可以选择与该主机组内容视图关联的模块中定义的 Puppet 类来部署该主机组中的客户端系统。这些 Puppet 类将生效并配置该主机组中的服务器。它们可以创建用户、安装软件、部署配置文件、启动服务，以及执行许多其他任务。

Puppet 类可能会依赖于其他模块提供的功能。许多 Puppet 类使用由 stdlib 类提供的功能，该类由 puppetlabs-stdlib 模块提供。在定义主机组时，卫星管理员必须知道这些依赖项，并且在将类包含在主机组中时确保满足这些依赖关系。本节下文中介绍的配置组可以简化相关 Puppet 类的部署。

######### 创建主机组

每一卫星组织具有自己专用的一个主机组集合。将组织上下文设为主机组将要从属于的组织。选择 Configure → Host groups，再单击 New Host Group 按钮。指定 Lifecycle Environment、Content View、Puppet Environment、Content Source、Puppet CA 和 Puppet Master。为主机组选择一个描述性名称，然后单击 Submit 来创建该主机组。

创建了主机组后，必须选择 Puppet 类，以定义如何配置该主机组中的服务器。选择 Puppet Classes 选项卡。展开包含您要包括的类的 Puppet 模块链接。单击类名称右侧的 +，使它们显示在 Included Classes 列中。定义了配置组时，也可以选择它们。选择一个配置组将激活该配置组中的所有 Puppet 类。

创建了主机组时，它便可分配到单独的主机。将 Any Organization 设为默认组织上下文，然后选择 Hosts → All hosts 选项卡。确定所需的主机在主机列表中，然后单击主机名链接。这将调出该服务器的 Puppet 详情页。单击屏幕顶部的 Edit 按钮，然后从 Host Group 下拉菜单中选择所需的主机组。

主机组可以关联使用某一激活密钥注册的计算机。在创建主机组时，选择 Activation Key 选项卡，再选择现有的激活密钥以便和该主机组关联。

######### 查看 Puppet 审核和报告活动

一旦 Puppet 配置为作为守护进程在客户端上运行，它将每隔半小时签入到红帽卫星服务器，以查看是否有要进行的更改。所有这些信息都会记录到卫星服务器的数据库，因此可以进行审核。

要查看某一主机的 Puppet 事务相关信息，可更改到该主机从属的组织。选择 Hosts → All hosts。单击所需主机的链接，以查看该主机的 Puppet 详情页。这是关于该主机 Puppet 活动的信息的通用仪表板。

单击 Reports 按钮。应用到主机的 Puppet 操作进行了编号，这些编号链接到该操作更为详细的报告。在查看报告时，单击 View Diff 链接可显示 Puppet 事务之前和之后状态之间的差别。 


###### 配置组

一些 Puppet 模块包含的 Puppet 类依赖于其他类才能执行其配置。配置组使得这些类分组在一起，从而作为一个整体进行管理，而不必逐一管理。

######### 创建配置组

下列步骤描述了如何创建配置组，并将 Puppet 类添加到其中：

1. 选择应当包含该配置组的组织。
2. 选择 Configure → Config groups 选项卡。
3. 单击 New Config Group 按钮。
4. 指定配置组的名称。
5. 在 Available Classes 列中，单击/展开所需的模块。
6. 针对要包含在配置组中的类，单击其右侧的 +。添加了这些类后，它们将显示在 Included Classes 列中。
7. 单击 Submit 按钮以确认选择。 

### 练习：通过主机组标准化配置

1. 在 Operations 组织中创建名为 NTP Classes 的配置组，它包含 chrony、chrony::params 和 stdlib Puppet 类。

    a. 选择 Any Context → Any Organization → Operations，以将组织上下文设为 Operations。

    b. 选择 Configure → Config groups。

    c. 单击 New Config Group 按钮。在显示的表单中，在 Name 字段中输入 NTP Classes。

    d. 单击 chrony 和 stdlib 模块的名称，以显示它们的类。选择 chrony、chrony::params 和 stdlib 类，使它们显示在页面上的 Included Classes 列中。

    rht-chrony Puppet 模块同时包含 chrony 和 chrony::params 类。chrony::params 类定义 chrony 类的类参数值，因此它是必须要包含的依赖项，这样才能使用 chrony。

    e. 单击 Submit 按钮，以确认选择并创建该配置组。 
    
2. 在 Operations 组织中，创建名为 Operations Host Group 的主机组。

    a. 选择 Configure → Host groups，再单击 New Host Group 按钮。

    使用下方的值完成显示的表单：
    
    *    字段	                值
    *    Name	                Operations Host Group
    *    Lifecycle Environment	Dev
    *    Content View	        Operations Content
    *    Puppet Environment	    KT_Operations_Dev_Operations_Content_#
    *    Content Source	        satellite.lab.example.com
    *    Puppet CA	            satellite.lab.example.com
    *    Puppet Master	        satellite.lab.example.com

    b. 选择 Puppet Classes 选项卡。

    在 Available Config Groups 列中，单击列出的 NTP Classes 配置组旁边的 Add 按钮。这会选中该配置组，并将它移入 Included Config Groups 列中。

    c. 单击 Network 选项卡、Operating System 选项卡和 Parameters 选项卡，探索它们的选项，但不要进行任何更改。另一节中将探索 Parameters 选项卡提供的选项。

    d. 选择 Locations 选项卡。将 Boston 和 Default Location 移到 Selected items 列表中。

    e. 选择 Organizations 选项卡。确保已选中 Operations。

    f. 单击底部的 Submit 按钮，以确认选择并创建新的主机组。 

3. 将新主机组配置文件分配到 servera.lab.example.com。

    a. 将 Any Organization 设置为默认的组织上下文。

    b. 选择 Hosts → All hosts 选项卡。

    c. 确认 servera.lab.example.com 在主机列表中。注意，此主机的 Host group 列为空。单击复选框以选中此主机。

    d. 单击 Select Action 下拉菜单，再选择 Assign Organization 菜单项。单击 Select Organization 按钮，再选择 Operations。选择 Fix Organization on Mismatch 按钮来覆盖现有的组织分配。单击 Submit 以提交选择。

    e. 单击 Select Action 下拉菜单，再选择 Assign Location 菜单项。单击 Select Location 按钮，选择 Boston，单击 Fix Location on Mismatch 按钮，然后单击 Submit 以提交选择。

    f. 单击 Select Action 下拉菜单，再选择 Change Environment 菜单项。单击 Select environment 按钮，选择 KT_Operations_Dev_Operations_Content_#，然后单击 Submit 以提交选择。

    g. 单击 Select Action 下拉菜单，再选择 Change Group 菜单项。单击 Select host group 按钮，选择 Operations Host Group，然后单击 Submit 以提交选择。 

4. 确认客户端主机可以看到其 NTP 配置中的差别。以 root 身份登录 servera，再注意 Puppet 如何更改本地环境并提议对 /etc/chrony.conf 的更改。

```
[root@servera ~]# puppet agent --test --noop
Warning: Local environment: "production" doesn't match server specified
 node environment "KT_Operations_Dev_Operations_Content_8", switching
 agent to "KT_Operations_Dev_Operations_Content_8".
... Output omitted ...
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Caching catalog for servera.lab.example.com
... Output omitted ...
Info: Applying configuration version '1441288571'
Notice: /Stage[main]/Chrony/File[/etc/chrony.keys]/mode: current_value 0640,
 should be 0644 (noop)
Notice: /Stage[main]/Chrony/File[/var/log/chrony]/owner: current_value chrony,
 should be root (noop)
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/ensure: current_value link,
 should be file (noop)
Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Scheduling refresh of
 Service[chronyd]
Notice: /Stage[main]/Chrony/Service[chronyd]: Would have triggered 'refresh'
 from 1 events
Notice: Class[Chrony]: Would have triggered 'refresh' from 4 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.62 seconds
```

注意原始配置文件是一个符号链接。这是我们课堂标准构建的一部分。

```
[root@servera ~]# ls -l /etc/chrony.conf
lrwxrwxrwx. 1 root root 15 Aug 26 17:44 /etc/chrony.conf -> chrony.conf-rht

/etc/chrony.conf 是主 chrony 配置文件。Puppet 将管理该文件并重新创建它。 
```

5. 使 Puppet 应用更改到主机系统。

```
[root@servera ~]# puppet agent --test
... Output omitted ...
Info: Applying configuration version '1441290722'
Notice: /Stage[main]/Chrony/File[/etc/chrony.keys]/mode: mode changed '0640'
 to '0644'
Notice: /Stage[main]/Chrony/File[/var/log/chrony]/owner: owner changed
 'chrony' to 'root'
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/ensure: defined content as
 '{md5}e814ce39347f1431253f933967bcdffb'
Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Scheduling refresh of
 Service[chronyd]
Notice: /Stage[main]/Chrony/Service[chronyd]: Triggered 'refresh' from 1
 events
Notice: Finished catalog run in 0.89 seconds
[root@servera ~]# ls -l /etc/chrony.conf
-rw-r--r--. 1 root root 928 Sep  3 10:32 /etc/chrony.conf
```

Puppet 将 /etc/chrony.conf 转换为它将进行管理的简单文件。注意 Puppet 将文件所有权从 chrony 更改为 root，并将权限更改为 644。 

6. 返回到红帽卫星 Web 界面，再审核 Puppet 对配置文件进行的更改。

    a. 将组织上下文设为 Operations。

    b. 选择 Hosts → All hosts。

    c. 单击 servera.lab.example.com 的链接，以查看该主机的 Puppet 详情页。

    d. 单击 Reports 按钮。

    e. 在 Applied 列中，单击具有 4 的报告的链接。一组消息将显示 /etc/chrony.conf 的属性和内容已被更改。查看对 /etc/chrony.conf 进行的所有权和权限更改。消息也会显示 chrony 服务已被刷新（重新启动）。 


### 实施智能类参数

###### 智能类参数简介

Puppet 参数化类使得变量可以使用类定义标头中的默认值进行定义。在 Puppet 清单或目录中使用这种类时，可以指定不同的值来覆盖默认值。

以下是获取参数的 Puppet 类声明的简单示例：

```
class ntp ($servers = undef, $enable = true, $ensure = 'running') {
  ...
```

$servers、$enable 和 $ensure 是可以传递到该类中的参数。如果未在使用此类的目录中指定它们，那么 $servers 未定义，$enable 被设为 true（或许激活相关服务），而 $ensure 则设为 'running'（导致 NTP 服务启动）。

以下是该类使用方法的描述：

```
class { 'ntp':
    enable => false,
    ensure => 'stopped',
}
```

class 的标题决定所要使用的 Puppet 类。指定的参数覆盖原先在 Puppet 类定义中指定的默认值。在上例中，enable 参数设为 false，而 ensure 参数则设为 'stopped'。 


###### 在红帽卫星 6 中使用 Puppet 类

在红帽卫星 6 中，智能类参数是一种将具体值传输到所配置系统的 Puppet 模块的方式。导入模块时，初始状态下使用的是默认值，而且类参数为固定值。若要覆盖它们，必须打上相应的标志。可以根据与基于主机相关事实的表达式相符的主机覆盖这些值。也可以通过卫星 6 Web 界面，逐个主机覆盖自定义的值。

要覆盖智能类参数的默认值，可将组织上下文设置为具有相关 Puppet 模块的组织。选择 Configure → Puppet Classes，然后单击要操作的 Puppet 类的 Class name 列中显示的名称。

Smart Class Parameter 子选项卡允许调节参数。选择了参数时，可通过 Override 复选框启用对类参数默认值的更改。可以使用新的 Default value 进行全局更改。

可以使用 Matcher-Value 表达式将智能类参数的不同值应用到有限数量的主机（其中一个 Matcher 表达式为 true）。这些表达式可根据卫星/Foreman 变量和值使用，或者可基于与客户端主机相关的系统事实。Matcher 表达式是以下形式的比较：variable = value。通配符和正则表达式对 Matcher 表达式无效。以下是 Matcher 表达式的几个简单示例：

```
ipaddress = '192.168.10.250'

hostgroup = 'Web Servers' 
```


### 练习：实施智能类参数

1. 调整 chrony Puppet 类的类参数，使得默认 NTP 服务器为 0.rhel.pool.ntp.org。配置为 Operations Host Group 服务器的主机的 NTP 服务器应当是 classroom.example.com。

    a. 打开浏览器，并进入红帽卫星 Web 用户界面。选择 Any Context → Any Organization → Operations，以将组织上下文设为 Operations。

    b. 选择 Configure → Puppet classes，然后单击 Class name 列中的 chrony。

    c. 选择 Smart Class Parameter 子选项卡。

    d. 滚动、查找并单击左侧的 servers 链接。那是类参数的名称。选中 Override 框，以允许输入自定义值。

    e. chrony 类将 servers 参数定义为字符串数组。Puppet 模块的 init.pp 清单的注释中找到了以下文本。

```
    # [*servers*]
    #   An array of strings containing the NTP servers chrony should use and the
    #   additional configuration settings for each one.  Each entry in the array
    #   starts with the server's ip address or fqdn followed by {any additional
    #   options accepted by the chrony server directive}
    #   [http://chrony.tuxfamily.org/manual.html#server-directive].
```

    从 Parameter type 的下拉菜单中，选择 array。将 Default value 设置为如下：

```
    ["0.rhel.pool.ntp.org"]
```

    f. 单击 Add Matcher-Value 按钮。使用下方的值完成显示的表单：
    
    *    字段	值
    *    Match	hostgroup=Operations Host Group
    *    Value	["classroom.example.com"]

    单击 Submit 按钮以确认更改。 

2. 在 servera 上以 root 身份登录，再显示 /etc/chrony.conf 中的原始 server 配置。确认客户端主机可以看到其 chrony 配置中的差别。

```
[root@servera ~]# grep '^server' /etc/chrony.conf
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst
[root@servera ~]# puppet agent --test --noop
Warning: Local environment: "production" doesn't match server specified
 node environment "KT_Operations_Dev_Operations_Content_8", switching
 agent to "KT_Operations_Dev_Operations_Content_8".
Info: Retrieving plugin
... Output omitted ...
Info: Applying configuration version '1441292816'
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content: 
--- /etc/chrony.conf    2015-09-03 10:32:04.471043566 -0400
+++ /tmp/puppet-file20150903-2429-91igrw        2015-09-03 11:06:57.671851980 -0400
@@ -1,7 +1,4 @@
-server 0.rhel.pool.ntp.org iburst
-server 1.rhel.pool.ntp.org iburst
-server 2.rhel.pool.ntp.org iburst
-server 3.rhel.pool.ntp.org iburst
+server classroom.example.com iburst
 
 
 # Ignore stratum in source selection.

Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content: current_value
 {md5}e814ce39347f1431253f933967bcdffb, should be
 {md5}345a7419db5320fd7c6c6c6395105070 (noop)
Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Scheduling refresh
 of Service[chronyd]
Notice: /Stage[main]/Chrony/Service[chronyd]: Would have triggered
 'refresh' from 1 events
Notice: Class[Chrony]: Would have triggered 'refresh' from 2 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.68 seconds
```

Puppet 有意将 chrony 服务器定义更改为 classroom.example.com。它优先选择这个值，而不是 0.rhel.pool.ntp.org，因为 Matcher-Value 对基于其分配的主机组匹配了该 Puppet 客户端。 

3. 使 Puppet 将更改应用到主机系统，然后重新显示 /etc/chrony.conf 中的 server 配置。

```
[root@servera ~]# puppet agent --test
Warning: Local environment: "production" doesn't match server specified
 node environment "KT_Operations_Dev_Operations_Content_8", switching
 agent to "KT_Operations_Dev_Operations_Content_8".
Info: Retrieving plugin
Info: Loading facts in /var/lib/puppet/lib/facter/facter_dot_d.rb
Info: Loading facts in /var/lib/puppet/lib/facter/root_home.rb
Info: Loading facts in /var/lib/puppet/lib/facter/puppet_vardir.rb
Info: Loading facts in /var/lib/puppet/lib/facter/pe_version.rb
Info: Caching catalog for servera.lab.example.com
... Output omitted ...
Info: Applying configuration version '1441292816'
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content: 
--- /etc/chrony.conf    2015-09-03 10:32:04.471043566 -0400
+++ /tmp/puppet-file20150903-2579-sl3kwe        2015-09-03 11:07:23.044912553 -0400
@@ -1,7 +1,4 @@
-server 0.rhel.pool.ntp.org iburst
-server 1.rhel.pool.ntp.org iburst
-server 2.rhel.pool.ntp.org iburst
-server 3.rhel.pool.ntp.org iburst
+server classroom.example.com iburst
 
 
 # Ignore stratum in source selection.

Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Filebucketed
 /etc/chrony.conf to puppet with sum e814ce39347f1431253f933967bcdffb
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content: content
 changed '{md5}e814ce39347f1431253f933967bcdffb' to
 '{md5}345a7419db5320fd7c6c6c6395105070'
Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Scheduling refresh
 of Service[chronyd]
Notice: /Stage[main]/Chrony/Service[chronyd]: Triggered 'refresh' from
 1 events
Notice: Finished catalog run in 0.97 seconds
[root@servera ~]# grep '^server' /etc/chrony.conf
server classroom.example.com iburst
```


### 实验：在红帽卫星 6 中实施 Puppet

1. 将 chrony 和 stdlib Puppet 模块下载到本地目录。

启动 Web 浏览器并导航到 http://&clrmfqdn;/materials/puppet。右键单击 chrony Puppet 模块的名称，将它保存到本地文件。对 stdlib 模块执行相同的操作。 

2. 登录卫星 Web UI，并使 Finance 成为当前的组织。

以 admin 身份登录卫星 Web UI。选择 Finance 作为当前的组织。单击左侧的 Any Context 选项卡，再从下拉菜单中选择 Finance。 

3. 创建名为 Finance Product 的产品，以包含 Puppet 存储库。

选择 Content → Products 选项卡。单击 New Product 按钮。

完成显示的表单。按照以下表格填写字段：

*    字段	值
*    Name	Finance Product
*    Label	Finance_Product（自动生成）

剩余的字段保持不变或留空。单击 Save 按钮，以确认选择并创建该产品。 

4. 创建 Finance Modules Puppet 存储库。

选择 Repositories 选项卡，再单击 Create Repository 按钮。完成显示的表单。按照以下表格填写字段：

*    字段	值
*    Name	Finance Modules
*    Label	Finance_Modules（自动生成）
*    Type	puppet

剩余的字段保持不变或留空。单击 Save 按钮，以确认选择并创建该存储库。 

5. 将 chrony 和 stdlib Puppet 模块上传到该存储库。

    a. 选择 Finance Modules 链接。

    b. 在 Upload Puppet Module 框中，单击 Browse... 按钮。

    c. 导航并选择包含 chrony Puppet 模块的文件 rht-chrony-0.1.0.tar.gz。

    d. 单击 Upload 按钮，以上传 Puppet 模块。

    e. 为 puppetlabs-stdlib-4.9.0.tar.gz 文件中的 stdlib 模块重复上述步骤。

    f. 确认指定的模块已上传到服务器。单击页面上 Content Type 部分中 Puppet Modules 右侧的数字。 

6. 创建名为 Finance Content 的内容视图。它必须包含具有 chrony 和 stdlib Puppet 模块的 Puppet 存储库。发布并提升内容视图，使它可供分配至 Dev 软件生命周期环境的 Finance 主机使用。

    a. 以 admin 用户身份登录卫星 Web 界面。将组织上下文设为 Finance。选择 Any Context → Any Organization → Finance。

    b. 选择 Content → Content Views，然后单击 Create New View 按钮。使用下方的值完成显示的表单：
    
    *    字段	            值
    *    Name	            Finance Content
    *    Label	            Finance_Content（自动生成）
    *    Description	    Content view for Finance servers
    *    Composite View?	不选中

    单击 Save 按钮确认。

    c. 单击 Puppet Modules 选项卡，然后单击 Add New Module 按钮。

    在 Actions 列中，选择与 chrony 模块对应的 Select a Version 按钮。单击 chrony 模块的 Use Latest (currently 0.1.0) 版本的 Select Version 按钮。

    重复上述步骤，以使用 stdlib Puppet 模块的最新版本。

    d. 发布内容视图，并将它提升到 Dev 环境。单击 Publish New Version 按钮，然后单击 Save 以确认您要进行发布。

    e. 当内容视图发布完成时，在 Actions 列中单击该内容视图 1.0 版本的 Promote 按钮。选择 Dev 环境，然后单击 Promote Version 按钮。

    当提升完成时，Library 和 Dev 都应当列在 Environments 列中。这确认发布和提升已成功。 

7. 确认正确的位置已分配给 Finance 组织的 Puppet 环境。

    a. 确保 Finance 是默认的组织上下文。选择 Any Context → Any Organization → Finance。

    b. 选择 Configure → Environments，以导航到 Puppet Environments 屏幕。

    c. 选择 Finance 内容库的环境 KT_Finance_Library_Finance_Content_#。

    d. 单击 Locations 子选项卡，然后将 Boston 和 Default Location 放入 Selected items 窗口。

    e. 单击 Submit 以应用更改。

    f. 对 Finance 组织的 Dev Puppet 环境重复上述步骤。 

8. 在 Finance 组织中创建激活密钥 Finance Host，以注册 Puppet 客户端计算机。Finance Content 内容视图应当分配至使用该密钥注册的主机，它们的软件生命周期环境应当是 Dev。由于课堂红帽卫星服务器不提供软件包，激活密钥应当不自动关联任何红帽软件订阅。激活密钥应当将主机订阅到自定义的 Finance Product 产品，这样它们有权访问它所提供的 Puppet 存储库。

    a. 确保 Finance 是默认的组织上下文。选择 Any Context → Any Organization → Finance。

    b. 选择 Content → Activation keys，再单击 New Activation Key 按钮。使用下方的值完成显示的表单：
    
    *    字段	        值
    *    Name	        Finance Host
    *    Content Host Limit: Unlimited Content Hosts	保留选中
    *    Description	Activation key that registers Finance development servers
    *    Environment	选择 Dev
    *    Content View	从下拉菜单中选择 Finance Content

    单击 Save 以确认。

    c. 选择 Subscriptions 选项卡，将 Auto-Attach 更改为 No，然后单击 Save 。

    单击 Add 子选项卡，然后选择 Finance Product 订阅。单击 Add Selected 按钮确认。 

9. 配置 serverb 上的 Puppet 客户端以供红帽卫星使用。使用 Finance Host 激活密钥，将它注册到 Finance 组织。

```
[root@serverb ~]# yum -y  install http://satellite.lab.example.com/pub/katello-ca-consumer-latest.noarch.rpm
... Output omitted ...
[root@serverb ~]# subscription-manager register --org='Finance' \
                    --activationkey='Finance Host'
The system has been registered with ID: 4cbf205c-96ca-4524-b1f3-1b1b37ed7ab4 
Installed Product Current Status:
Product Name: Red Hat Enterprise Linux Server
Status:       Not Subscribed

Unable to find available subscriptions for all your installed products.
```

10. 在红帽卫星客户端主机 serverb 上安装 Puppet 和 Katello Agent 软件。

```
[root@serverb ~]# yum -y install puppet katello-agent
```

11. 将 serverb 上的 Puppet 配置为将红帽卫星服务器用作 Puppet 宿主。在测试模式中运行一次 puppet，以便它发送证书到卫星服务器。

如果 /etc/puppet/puppet.conf 包含 server= 行，将它修改为指向卫星服务器。否则，添加 server= 指令到配置文件。

```
[root@serverb ~]# echo 'server=satellite.lab.example.com' >> /etc/puppet/puppet.conf
[root@serverb ~]# puppet agent --test
```

12. 使用卫星 Web 用户界面，为客户端系统签署主机证书。

    a. 选择 Infrastructure → Capsules。

    b. 单击 satellite.lab.example.com 胶囊主机名右侧的 Certificates 按钮。此时将显示主机证书列表。单击 serverb 主机名右侧的 Sign 按钮。

    c. 在 serverb 上，使用 puppet 命令签入到卫星服务器，以检索客户端的已签名主机证书。

```
    [root@serverb ~]# puppet agent --test --noop
```

13. 在 Operations 组织中，创建名为 Finance Host Group 的主机组。

    a. 确保 Finance 是默认的组织上下文。选择 Any Context → Any Organization → Finance。

    b. 选择 Configure → Host groups，再单击 New Host Group 按钮。使用下方的值完成显示的表单：
    
    *    字段	            值
    *    Name	            Finance Host Group
    *    Lifecycle          Environment	Dev
    *    Content View	    Finance Content
    *    Puppet             Environment	KT_Finance_Dev_Finance_Content_#
    *    Content Source	    satellite.lab.example.com
    *    Puppet CA	        satellite.lab.example.com
    *    Puppet Master	    satellite.lab.example.com

    c. 选择 Puppet Classes 选项卡，然后选择 chrony、chrony::params 和 stdlib 类，以便它们出现在 Included Classes 列中。

    d. 选择 Locations 选项卡。将 Boston 和 Default Location 移到 Selected items 列表中。

    e. 选择 Organizations 选项卡。确保已选中 Finance。

    f. 单击底部的 Submit 按钮，以确认选择并创建新的主机组。 

14. 将 Finance Host Group 主机组配置文件分配到 serverb.lab.example.com。serverb 应当分配到 Finance 组织和 Boston 位置。将 Finance 内容视图分配到主机，并使它成为 Dev 生命周期环境的成员。它还应当分配到 KT_Finance_Dev_Finance_Content_# Puppet 环境。卫星服务器应当充当 serverb 的内容来源、Puppet 证书认证机构和 Puppet 宿主。

    a. 将 Any Organization 设置为默认的组织上下文。选择 Any Context → Finance → Any Organization。

    b. 选择 Hosts → All hosts 选项卡。确认 serverb.lab.example.com 在主机列表中。注意 Host group 列为空。单击复选框以选中此主机。

    c. 单击 Select Action 下拉菜单，再选择 Assign Organization 菜单项。单击 Select Organization 按钮，再选择 Finance，然后单击 Fix Organization on Mismatch 按钮来确认更改生效。单击 Submit 以提交选择。

    d. 单击 Select Action 下拉菜单，再选择 Assign Location 菜单项。单击 Select Location 按钮，选择 Boston，单击 Fix Location on Mismatch 按钮，然后单击 Submit 以提交选择。

    e. 单击 Select Action 下拉菜单，再选择 Change Environment 菜单项。单击 Select environment 按钮，选择 KT_Finance_Dev_Finance_Content_#，然后单击 Submit 以提交选择。

    f. 单击列表中的 serverb.lab.example.com 主机名链接。这将调出该服务器的 Puppet 详情页。

    单击屏幕顶部的 Edit 按钮。从下拉菜单中选择下列值：
    
    *    字段	                值
    *    Host Group	            Finance Host Group
    *    Lifecycle Environment	Dev
    *    Content View	        Finance Content
    *    Puppet Environment	    KT_Finance_Dev_Finance_Content_#
    *    Content Source	        satellite.lab.example.com
    *    Puppet CA	            satellite.lab.example.com
    *    Puppet Master	        satellite.lab.example.com

    单击 Submit 按钮以确认选择。 

15. 定义智能类参数，使其使用仅包含 classroom.example.com 和 satellite.lab.example.com 主机的服务器列表填充 chrony Puppet 类的 servers 参数。此配置应当仅应用到属于 Finance Host Group 主机组成员的主机。

    a. 确保 Finance 是默认的组织上下文。选择 Any Context → Any Organization → Finance。

    b. 选择 Configure → Puppet classes，然后单击 Class name 列中的 chrony。

    c. 选择 Smart Class Parameter 子选项卡。向下滚动、查找并单击左侧的 servers 类参数的链接。选中 Override 框，以允许输入自定义值。

    保留 servers 智能类参数的默认值。Puppet 需要进行配置，以仅将指定的服务器配置应用到 Finance 部门的服务器。

    d. 单击 Add Matcher-Value 按钮。使用下方的值完成显示的表单：
    
    *    字段	值
    *    Match	hostgroup=Finance Host Group
    *    Value	["classroom.example.com","satellite.lab.example.com"]

    单击 Submit 按钮以确认更改。

    您定义的 Matcher-Value 对仅正确设置 Finance Host Group 类型主机的 NTP 服务器。 

16. 使 Puppet 将更改应用到主机系统，以确认配置正确。

```
[root@serverb ~]# puppet agent --test
... Output omitted ...
Info: Applying configuration version '1431439702'
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content: 
--- /etc/chrony.conf    2015-05-12 05:43:32.585655052 +0000
+++ /tmp/puppet-file20150512-1226-zzpdwj     2015-05-12 14:08:24.518777983 +0000
@@ -1,4 +1,5 @@
-server 0.rhel.pool.ntp.org iburst
+server classroom.example.com iburst
+server satellite.lab.example.com iburst
 
 
 # Ignore stratum in source selection.

Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Filebucketed
/etc/chrony.conf to puppet with sum d90283ded230b6354ca9f4afe8361edd
Notice: /Stage[main]/Chrony/File[/etc/chrony.conf]/content:
content changed '{md5}d90283ded230b6354ca9f4afe8361edd' to
'{md5}051dd44a2dff92e14e2cb19eb90876c8'
Info: /Stage[main]/Chrony/File[/etc/chrony.conf]: Scheduling refresh
of Service[chronyd]
Notice: /Stage[main]/Chrony/Service[chronyd]: Triggered 'refresh' from 1 events
Notice: Finished catalog run in 1.06 seconds
```

17. 在 serverb 上永久配置 Puppet 作为服务运行。

```
[root@serverb ~]# systemctl start puppet.service
[root@serverb ~]# systemctl enable puppet.service
ln -s '/usr/lib/systemd/system/puppet.service' '/etc/systemd/multi-user.target.wants/puppet.service'
```


### 总结

在本章中，您学到了：

*    红帽卫星 6 可以向客户端系统提供 Puppet 模块。必须创建产品，以包含利用 puppet 类型定义的存储库。
*    在将 Puppet 与红帽卫星 6 搭配使用时，必须定义内容视图，以发布 Puppet 模块可能要用于安装所需软件的 Puppet 存储库（及其他软件存储库）。
*    可以定义激活密钥，来自动化注册客户端系统。
*    由卫星服务器发布的 katello-ca-consumer-latest 软件包可以对客户端进行配置，从而将卫星服务器用于软件。必须先安装它，然后才能用 subscription-manager 注册系统。
*    将 server=satellite.FQDN 添加到 /etc/puppet/puppet.conf ，要搭配红帽卫星 6 的 Puppet。
*    Infrastructure → Capsules 中的 Certificates 按钮用于签署 Puppet 代理证书。
*    Configure → Host groups 菜单用于定义红帽卫星 6 上的标准系统定义。
*    选择了 Puppet 模块时，可以利用 Configure → Puppet Classes 菜单更改智能类参数的默认值。


