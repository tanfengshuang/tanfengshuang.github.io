---
layout: post
title:  "Implementing External Puppet Modules(405-12)"
categories: Linux
tags: RHCA 405
---

### Puppet Forge 中的 Puppet 模块

###### Puppet Forge 中的 Puppet 模块

Puppet Forge是一个 Puppet 模块公共资源库，这些模块由许许多多 Ansible 管理员和用户编写。它是一个包含数千 Puppet 模块的档案库，具有可搜索的数据库，可帮助 Puppet 用户确定或许有助于他们完成管理任务的模块。Puppet Forge 包含面向新 Puppet 模块开发人员的文档的链接。它也包含可帮助编写 Puppet 模块的各种工具的链接。 

1. 已核准模块

Puppet Forge 上发布的一些模块已标记为“已核准”模块。这些模块被 Puppet Labs 认定为实用性强、编写精良、文档记录完好，并且提供包含有许可和联系信息的元数据。

已核准模块得到了积极开发和增强。它们已经过检查，确定不含有害代码。模块作者和贡献者为已核准模块提供非官方支持。这些模块没有官方支持。

2. 受支持的模块

“受支持的”模块是 Puppet Labs 提供商业支持的模块。它们已经过测试，能够在各种平台上搭配 Puppet Enterprise 产品使用。欢迎提供错误报告，也可访问 JIRA 页面的链接；该页面有助于 Puppet Labs 管理这些模块的维护。

3. Puppet Forge 的益处

Puppet Forge 有助于开发人员将开源理念运用到 Puppet 模块开发中。如果管理员的系统配置需求能通过 Puppet 解决，通常不必从头开始开发新模块。他们可以搜索 Puppet Forge，了解别人是否发布过能够执行该任务的模块。

Puppet Forge 允许若干名管理员和开发人员协同工作。他们可以团队形式开发、测试和记录自认为有用的模块。这样的协作能够让整个 DevOps 社区受益。

任何人都可以在 Puppet Forge 注册页上创建 Puppet Forge 帐户。使用 Puppet Forge 不需要帐户，但如果开发人员希望发布自己的 Puppet 模块供他人使用，则需要拥有帐户。

在创建 Puppet Forge 帐户时，必须提供以下信息：

*    Username—由字母数字字符组成的名称，用作模块名称中的作者部分。
*    Display Name—通常是持有该帐户的用户的全名。这是其他 Puppet Forge 用户看到的名称。
*    Email—用于联系帐户所有者的电子邮件地址。
*    Password（和 Confirmation）—所有者在访问其帐户时用于身份验证的密码。
    
###### 在 Puppet Forge 中搜索模块

Puppet Forge 主页的顶部具有一个搜索框。指定的搜索词与模块名称、描述和可能分配到的标签进行比对。找到匹配项时，显示模块作者和名称的列表。这些名称带有超链接，可以用于访问包含模块或作者的更多详情的页面，具体取决于链接的内容。 

每个模块都分配有 Puppet Labs 的质量评分。其他 Puppet Forge 社区成员也可评价模块。还提供额外的超链接，连至开发人员所列待处理问题和兼容性问题的列表，以及说明如何对模块进行评分/打分的页面。

也可以通过 puppet 命令行实用工具进行搜索。唯一的要求是搜索要在连接了互联网的 Puppet 主机上执行。以下示例输出中显示了搜索词语 “firewall” 时生成的输出。

```
[root@host ~]# puppet module search firewall | head
Notice: Searching https://forgeapi.puppetlabs.com ...
NAME                           DESCRIPTION           AUTHOR            KEYWORDS
gildas-firewall                Unified Firewall ...  @gildas
puppetlabs-firewall            Manages Firewalls...  @puppetlabs       firewall
example42-firewall             Puppet firewall a...  @example42        firewall
sschneid-firewall              UNKNOWN               @sschneid         firewall
liamjbennett-windows_firewall  Module that will ...  @liamjbennett     firewall
camptocamp-firewall_c2c        Add autorequire t...  @camptocamp       firewall
thoward-windows_firewall       puppet windows fi...  @thoward
andrewkroh-base_firewall       Puppet module tha...  @andrewkroh
```

与 Puppet Forge Web 界面类似，搜索基于模块的名称、描述和关键词或标签匹配模块。


### 从 Puppet Forge 实施模块

虽然 Puppet Forge Web 界面在视觉上更吸引人，但最好通过命令行从 Puppet Forge 安装模块。在各个描述 Puppet 模块的页面中，概览框的底部都有一个 download latest tar.gz 链接。此链接可用于下载模块的 tar 存档，然后通过 puppet module install MODULE.tar.gz 命令安装模块，但不安装任何依赖项。必须首先下载和安装依赖项；或者可以通过 --ignore-dependencies 选项使用 puppet module install 强制安装模块。

当完整模块名称（包括作者）用于 puppet module install 时，该命令将以单一命令从 Puppet Forge 下载并安装模块及其依赖项。以下示例输出中利用 puppetlabs-firewall 模块进行了演示。

```
[root@host ~]# puppet module install puppetlabs-firewall
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── puppetlabs-firewall (v1.7.1)
```

puppet module install 的另一个有用的选项是 --module_repository REPOPATH 选项。它将 puppet 命令指向 Puppet Forge 之外的其他 Puppet 模块存储库。

### 练习：从 Puppet Forge 实施 Puppet 模块

1. 以 root 身份登录 servera，再安装 tree 应用。稍后，将使用它来显示下载的模块的文件层次结构。

```
[root@servera ~]# yum -y install tree
```

2. 在线搜索当日消息模块 (“motd”)，并从命令行使用 puppet module 命令进行同样的搜索。

    a. 打开 Web 浏览器并前往 Puppet Forge网站。在搜索框中输入 motd，然后单击 Find 按钮。

    b. 使用 puppet module search 命令，搜索与字符串 “motd” 匹配的模块。应当会显示几个类似于如下输出的匹配项。

```
    [root@servera ~]# puppet module search motd
    Notice: Searching https://forgeapi.puppetlabs.com ...
    NAME                      DESCRIPTION              AUTHOR            KEYWORDS   
    saz-motd                  Manage 'Message Of T...  @saz              rhel motd  
    attachmentgenie-motd      Puppet motd Module       @attachmentgenie  motd       
    CERNOps-motd              Manages entries in a...  @CERNOps          motd       
    fvoges-motd               Simple MOTD Puppet m...  @fvoges           motd unix  
    ... Output omitted ...
```

3. 查看 Puppet Labs 是否提供了用于管理当日消息的模块。下载并安装到 root 的主目录。

```
[root@servera ~]# puppet module search motd | grep @puppetlabs
puppetlabs-motd           A simple module to d...  @puppetlabs       testing
```

下方的示例输出中显示了安装 puppetlabs-motd 模块时的情况。使用 --modulepath=. 选项可使 Puppet 将模块安装到当前目录下。

```
[root@servera ~]# puppet module install --modulepath=. puppetlabs-motd
Notice: Preparing to install into /root ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/root
└── puppetlabs-motd (v1.2.0)
```

puppet module install 会从 Puppet Forge 下载模块及依赖项，然后安装它们。

4. 对安装模块时创建的新目录运行 tree 命令。保存模块的目录中含有模块名称但去除了前置作者信息。

```
[root@servera ~]# tree -d -L 1 motd
motd
├── manifests
├── spec
├── templates
└── tests

4 directories
```

5. 利用 motd 模块中提供的烟雾测试清单，应用 motd 类。

```
[root@servera ~]# puppet apply --modulepath=. motd/tests/init.pp
Warning: Config file /etc/puppet/hiera.yaml not found, using Hiera defaults
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 0.57 seconds
Notice: /Stage[main]/Motd/File[/etc/motd]/content: content changed 
 '{md5}d41d8cd98f00b204e9800998ecf8427e' to '{md5}88e0d501d7e3aa61c2675f2ac47e59a4'
Notice: Finished catalog run in 0.14 seconds
```

模块更新了当日消息文件 /etc/motd。

```
[root@servera ~]# cat /etc/motd
The operating system is RedHat
The free memory is 1.47 GB
The domain is lab.example.com
```


### 实验：实施外部 Puppet 模块

1. 搜索 Puppet Forge，再找到 Puppet Labs 发布的 Apache Web 服务器模块。

由于 Puppet Labs 是发布者，因此可使用 puppet module search 来查看是否有 Puppet Labs “apache” 模块。

```
[root@servera ~]# puppet module search apache | grep @puppetlabs
Notice: Searching https://forgeapi.puppetlabs.com ...
NAME               DESCRIPTION                     AUTHOR        KEYWORDS
puppetlabs-apache  Installs, configures, and m...  @puppetlabs   web ssl rhel

Puppet Forge 中有一个名为 puppetlabs-apache 的模块，可以安装和管理该服务器。
```

2. 将 Puppet Labs Apache 模块及所有依赖项从 Puppet Forge 安装到 /root 目录。

使用 puppet module install 命令执行此步骤。添加 --modulepath=. 选项可使 Puppet 将模块安装到当前目录中。

```
[root@servera ~]# puppet module install --modulepath=. puppetlabs-apache
Notice: Preparing to install into /root ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/root
└─┬ puppetlabs-apache (v1.6.0)
  ├── puppetlabs-concat (v1.2.4)
  └── puppetlabs-stdlib (v4.9.0)
```

注意已安装了 puppetlabs-concat 和 puppetlabs-stdlib 模块，因为 puppetlabs-apache 依赖于它们。

3. 创建烟雾测试文件 ~/apache/tests/apache-test.pp，它将测试 apache 模块。应用它，并确认其正常工作。

```
[root@servera ~]# mkdir apache/tests
[root@servera ~]# echo 'include apache' > apache/tests/apache-test.pp
[root@servera ~]# puppet apply --modulepath=. apache/tests/apache-test.pp
Warning: Config file /etc/puppet/hiera.yaml not found, using Hiera
 defaults
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 3.14 seconds
... Output omitted ... 
```

4. 打开相关的防火墙端口。

默认的 Apache 行为是在端口 80 上侦听 HTTP 连接。以下 firewall-cmd 命令将立即且永久打开该端口。

```
[root@servera ~]# firewall-cmd --add-service=http
success
[root@servera ~]# firewall-cmd --permanent --add-service=http
success
```

5. 创建包含以下内容的文件 /var/www/html/index.html：

The external Puppet lab is done.

```
[root@servera ~]# cd /var/www/html
[root@servera html]# echo 'The external Puppet lab is done.' > index.html
```

6. 确认 Web 内容正在发布。

打开浏览器，然后在地址框中提供以下 URL：http://serverafqdn;。

```
[student@workstation ~]$ curl http://servera.lab.example.com
The external Puppet lab is done.
```


### 总结

在本章中，您学到了：

*    Puppet Forge中为开发人员和系统管理员提供 Puppet 模块资源。
*    Puppet Forge 根据名称、描述和分配的标签来搜索模块。
*    在可以访问互联网时，puppet module search 命令可在 Puppet Forge 中搜索模块。
*    在指定完整模块名称而非 tar 存档时，puppet module install 命令将从 Puppet Forge 下载并安装 Puppet 模块及其依赖项。
*    --ignore-dependencies 选项可强制 Puppet 从 tar 存档安装模块。


