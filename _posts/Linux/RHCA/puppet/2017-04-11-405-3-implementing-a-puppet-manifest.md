---
layout: post
title:  "Implementing a Puppet Manifest(405-3)"
categories: Linux
tags: RHCA 405
---

### Puppet 域特定语言 (DSL)

Puppet 对主机进行更改，使其进入所需的最终状态。系统管理员利用 Puppet 域特定语言 (DSL) 定义他们对系统的需求，而不是定义更改的方式。Puppet 为所需的最终状态提供一个抽象层，—即配置用户、文件和服务—，以忽略特定于操作系统的实施细节。这将产生一个跨平台解决方案。

计算机配置由清单指定。Puppet 清单是使用 Puppet DSL 编写的文本文档，它们描述所需的客户端主机最终状态。

Puppet 域特定语言 (DSL) 的开发目的是为了简化 Puppet 目标的表达。它设计为可供编程基础有限的系统管理员使用。Puppet DSL 不属于编程语言。它与 Ruby 编程语言相似，但不是 Ruby。

Puppet DSL 具有以下语法：

```
resource_type { 'resource_title':
  attr1 => value1,
  attr2 => value2,
  attr3 => value3,
  ...
  attrN => valueN,
}
```

在声明资源时，首先声明资源的类型。Puppet 包含许多内置的类型，如 file、user、package 和 service。资源类型后紧跟花括号，花括号内是资源标题和属性/值对列表。资源标题是字符串常量，字符串常量用单引号括起。通过冒号分隔资源标题和属性分配列表。

每一资源类型拥有自己的属性，也称为参数。例如，file 资源包含 ensure、path、owner、group 和 mode 属性。这些属性可以通过 => 运算符分配值。通过逗号分隔各个属性/值对，但管理员通常以逗号终止所有属性/值对，从而能在需要时更加轻松地添加额外的值分配。

###### 资源标题注意事项

在定义资源标题时必须要谨慎。标题是一种标识性字符串，向 Puppet 编译器标识该资源。它不需要与实际的目标主机有任何关系。资源标题可以包含任何字符，但它们区分大小写。

标题必须在一种资源类型内唯一。可以存在标题都为 “http” 的软件包和服务，但只能有一项服务名为 “http”。标题重复可导致编译错误。

多个类似的资源可以通过指定由字符串列表组成的标题来缩写。下例显示了四个具有相同属性的 file 资源如何合并到一个资源定义中：

```
file { [ '/etc/firewalld',
         '/etc/firewalld/icmptypes',
         '/etc/firewalld/services',
         '/etc/firewalld/zones' ]:
  ensure => 'directory',
  owner  => 'root',
  group  => 'root',
  mode   => '750',
}
```

###### Puppet DSL 简写

如果一个属性块的结尾是分号，而不是逗号，则可以再指定另一个标题、冒号加属性块。Puppet 会将它视为一种类型的多个资源。下例演示了两个目录规格如何合并到一个 file 资源定义中：

```
file {
  '/etc/rc.d':
    ensure => 'directory',
    owner  => 'root',
    group  => 'root',
    mode   => '755';
 
  '/etc/rc.d/init.d':
    ensure => 'directory',
    owner  => 'root',
    group  => 'root',
    mode   => '755';
}
```


### Puppet 资源

Puppet 资源包括系统对象，如文件、用户、组、软件包、服务和 SSH 密钥。Puppet 语言允许管理员以独立于操作系统的方式管理系统资源。Puppet 的资源抽象层 (RAL) 提供在特定操作系统上实施低级功能的库。

以下 Puppet 资源声明指定 httpd 服务应当在系统上运行并且永久启用。

```
service { 'httpd':
  ensure => 'running',
  enable => true,
}
```

Puppet RAL 的职责是使特定的系统进入该状态。以下命令将在 Red Hat Enterprise Linux 6 主机上进行这一实施。

```
service httpd start ; chkconfig httpd on
```

在 Red Hat Enterprise Linux 7 系统上，应当要改为使用以下命令。

```
systemctl start httpd ; systemctl enable httpd
```

资源抽象层使用与它所运行的系统对应的命令。这使得 Puppet 管理员能够在高层面上管理资源，而不必担心底层操作系统的细枝末节。实施相关的细节由 RAL 负责处理，因此 Puppet 管理员可以编写独立于系统的规格。

###### puppet resource 命令

puppet resource 命令显示 Puppet DSL，该语言描述本地系统上某一对象的当前状态。利用此命令显示现有对象的 DSL 时，可以显示资源的属性（及当前的值）。这将帮助管理员了解如何使用 Puppet DSL 描述自己的资源。

puppet resource --type 命令列出 Puppet 主机上可用的资源类型。

```
[root@workstation ~]# yum install puppet

[root@workstation ~]# puppet resource --type
anchor
augeas
computer
cron
exec
file
file_line
... Output omitted ...
```

以下部分将重复使用 puppet resource 命令来显示 Red Hat Enterprise Linux 系统上各种资源的属性。

###### 资源示例

1. notify 资源: notify 资源用于在 Puppet 日志中生成输出。下例演示了如何在日志中显示消息 “Enabling NTP service”：

```
notify { 'log-ntp-enable' :
  message => 'Enabling NTP service',
}
```

2. package 资源: package 资源定义是否应当在主机操作系统中安装软件包。如果应当要删除某一软件包，其 ensure 属性应当设置为值 'absent'。如果应当要安装某一软件包，其值应当是 'present'。

```
[root@workstation ~]# rpm -q vsftpd
package vsftpd is not installed
[root@workstation ~]# puppet resource package vsftpd
package { 'vsftpd':
  ensure => 'absent',
}
[root@workstation ~]# rpm -q coreutils
coreutils-8.22-11.el7.x86_64
[root@workstation ~]# puppet resource package coreutils
package { 'coreutils':
  ensure => '8.22-11.el7',
}
```

puppet resource package 命令显示已安装软件包的当前版本，作为 ensure 属性的值。

3. file 资源: file 资源定义 Puppet 主机上文件或目录的属性和内容。ensure 属性设为 'file' 或 'directory'；如果它的值是 'present'，则 type 属性可以定义它的类型。owner 和 group 属性定义文件的所有权和组访问权限。mode 属性定义文件或目录的权限。

```
[root@workstation ~]# puppet resource file /etc/notafile
file { '/etc/notafile':
  ensure => 'absent',
}
[root@workstation ~]# puppet resource file /root/anaconda-ks.cfg
file { '/root/anaconda-ks.cfg':
  ensure   => 'file',
  content  => '{md5}e08d48c9c39de018140f5f9e2dd99add',
  ctime    => '2015-02-03 17:38:25 -0500',
  group    => '0',
  mode     => '600',
  mtime    => '2015-02-03 17:38:25 -0500',
  owner    => '0',
  selrange => 's0',
  selrole  => 'object_r',
  seltype  => 'admin_home_t',
  seluser  => 'system_u',
  type     => 'file',
}
[root@workstation ~]# puppet resource file /var
file { '/var':
  ensure   => 'directory',
  ctime    => '2015-08-03 00:10:49 -0400',
  group    => '0',
  mode     => '755',
  mtime    => '2015-08-03 00:10:43 -0400',
  owner    => '0',
  selrange => 's0',
  selrole  => 'object_r',
  seltype  => 'var_t',
  seluser  => 'system_u',
  type     => 'directory',
}
```

4. service 资源: service 资源定义系统服务的当前状态和引导时状态。ensure 属性可以取值 'running' 或 'stopped'，用于定义服务的当前状态。enable 属性定义其引导时行为。

```
[root@workstation ~]# puppet resource service sshd
service { 'sshd':
  ensure => 'running',
  enable => true,
}
[root@workstation ~]# puppet resource service nfsd
service { 'nfsd':
  ensure => 'stopped',
  enable => false,
}
```

5. user 资源: user 资源定义主机上应当存在或不存在哪些用户帐户。user 资源的属性指定系统用户的特性。uid 和 gid 属性设置为数值，分别对应于用户 ID 和组 ID。home 属性定义用户的主目录。

> 虽然 home 属性指定用户的主目录，但它不会真正创建该目录。那必须通过 file 资源指定。

> 用户的加密密码哈希通过 password 属性指定。还可以通过指定额外的属性来定义用户密码的过期值。

```
[root@workstation ~]# puppet resource user notauser
user { 'notauser':
  ensure => 'absent',
}
[root@workstation ~]# puppet resource user root
user { 'root':
  ensure           => 'present',
  comment          => 'root',
  gid              => '0',
  home             => '/root',
  password         => '$6$QLji1eoc$ZJhjFcPu6EJBEyls//8pGgfskDOe0PWrLfcXVkmajAnqZsm4xD0I/g236TMNjbQAER3FXcfkhlSwSO76qPqx5/',
  password_max_age => '99999',
  password_min_age => '0',
  shell            => '/bin/bash',
  uid              => '0',
}
```

6. exec 资源: exec 资源用于在 Puppet 主机上执行命令。要执行的命令可以指定为资源的标题，或指定为 command 属性。必须指定可执行文件的完整路径名，或者将 path 属性定义为针对该命令搜索的目录列表。cwd 属性指定执行命令时所要位于的当前工作目录。

通常，exec 资源会以静默方式执行命令。如果命令返回错误，Puppet 将显示输出。当 logoutput 属性设置为 true 时，Puppet 将显示所执行命令的所有输出。

###### OpenStack 清单回顾

本课程开始时我们介绍了 Puppet DSL。现在，再来看看 OpenStack 安装程序所使用的一些 Puppet 资源。

以下清单指定了一个 package 资源。它利用变量指定软件包名称（资源标题），以及是否应当安装该软件包（ensure 属性）。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/manifests/install.pp
#
class ntp::install inherits ntp {

  if $ntp::package_manage {

    package { $ntp::package_name:
      ensure => $ntp::package_ensure,
    }

  }

}
```

以下 Puppet 清单使用 service 资源定义 NTP 网络服务的配置方式。它利用变量指定服务名称（name 属性），该服务是否应当处于运行状态，以及它是否应当在引导时启用。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/manifests/service.pp
#
class ntp::service inherits ntp {

  if ! ($ntp::service_ensure in [ 'running', 'stopped' ]) {
    fail('service_ensure parameter must be running or stopped')
  }

  if $ntp::service_manage == true {
    service { 'ntp':
      ensure     => $ntp::service_ensure,
      enable     => $ntp::service_enable,
      name       => $ntp::service_name,
      hasstatus  => true,
      hasrestart => true,
    }
  }

}
```

###### 默认资源值

Puppet 的资源属性具有默认值。例如，file 资源创建归 root 所有、且默认权限为 0644 的普通文件。目录在创建时的默认权限为 0755。如果在资源中明确指定属性，它们会覆盖默认值。

Puppet DSL 具有为资源定义不同默认值的机制。实现方法是：大写方式指定资源类型，后面跟上包含属性及新默认值的无标题段落。以下 Puppet DSL 为file 资源定义 ensure、owner、group 和 mode 属性的新默认值。

```
File {
  ensure => 'file',
  owner  => 'apache',
  group  => 'apache',
  mode   => '640',
}
```


### 实施清单

###### 安装 Puppet 软件

必须在系统中安装 Puppet 软件，才能将它用于应用 Puppet DSL 清单。在红帽系统上，这可通过安装 puppet 软件包来完成。系统必须订阅以下红帽存储库之一才能安装 Puppet 软件：

*    红帽企业 Linux 7 服务器 - RH Common (RPM)
*    红帽卫星工具 6.1（适用于 RHEL 7 服务器）(RPM)
*    适用于红帽企业 Linux 7 服务器的红帽 Ceph 存储工具 1.3 (RPM)

###### 使用 Puppet 清单

Puppet 清单的创建非常简单。使用文本编辑器创建以 .pp 扩展名结尾的文件，该文件用于定义资源声明。通过运行 puppet 及 apply 子命令应用更改。以下清单 nojack.pp 将导致 Puppet 从系统中删除名为 jack 的帐户。

```
[root@workstation ~]# id jack
uid=501(jack) gid=501(jack) groups=501(jack)
[root@workstation ~]# vim nojack.pp
[root@workstation ~]# cat nojack.pp
user {'jack':
  ensure => 'absent',
}
[root@workstation ~]# puppet apply nojack.pp
Notice: Compiled catalog for workstation.lab.example.com in environment production
in 0.58 seconds
Notice: /Stage[main]/Main/User[jack]/ensure: removed
Notice: Finished catalog run in 1.02 seconds
[root@workstation ~]# id jack
id: jack: No such user
```

以下清单 katello-file.pp 声明应当存在名为 /var/tmp/katello-file 的文件。文件所有权、权限和内容在资源声明中指定。

```
[root@workstation ~]# vim katello-file.pp
[root@workstation ~]# cat katello-file.pp
file {'katello-file':
  path    => '/var/tmp/katello-file',
  ensure  => 'file',
  mode    => '640',
  owner   => 'katello',
  group   => 'katello',
  content => "I'm a test file.\n",
}
[root@workstation ~]# ls /var/tmp
scl0r3DGi  sclyCEuvj
[root@workstation ~]# puppet apply katello-file.pp
Notice: Compiled catalog for workstation.lab.example.com in environment production
in 0.37 seconds
Notice: /Stage[main]/Main/File[katello-file]/ensure: defined content as
'{md5}15735e435ae2af37745ce7f9f0e0fe94'
Notice: Finished catalog run in 0.32 seconds
[root@workstation ~]# ls /var/tmp
katello-file  scl0r3DGi  sclyCEuvj
[root@workstation ~]# ls -l /var/tmp/katello-file
-rw-r-----. 1 katello katello 17 Aug 28 08:52 /var/tmp/katello-file
[root@workstation ~]# cat /var/tmp/katello-file
I'm a test file.
```

> 用双引号括起包含撇号和指定为 “\n” 的换行符的字符串。 


### 练习：实施和应用 Puppet 清单

```
# vim test.pp
# My first Puppet DSL
notify { 'hello-world':
  message => 'Hello world!',
}
  
user { 'webmaster':
  ensure  => 'present',
  home    => '/home/webmaster',
  shell   => '/bin/bash',
}
file { '/home/webmaster':
  ensure   => 'directory',
  owner    => 'webmaster',
  group    => 'webmaster',
  mode     => '770',
  require  => User['webmaster'],
}
package { 'httpd':
  ensure  => 'present',
}

service { 'httpd':
  ensure  => 'running',
  enable  => true,
  require => Package['httpd'],
}
[root@servera ~]# puppet apply test.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 1.16 seconds
... Output omitted ...
Notice: Hello world!
Notice: /Stage[main]/Main/Notify[hello-world]/message: defined 'message'
 as 'Hello world!'
Notice: /Stage[main]/Main/Package[httpd]/ensure: created
Notice: /Stage[main]/Main/Service[httpd]/ensure: ensure changed 'stopped'
 to 'running'
Notice: Finished catalog run in 5.73 seconds
[root@servera ~]# rpm -q httpd
httpd-2.4.6-31.el7.x86_64
[root@servera ~]# systemctl is-active httpd
active
[root@servera ~]# systemctl is-enabled httpd
enabled
```


### 使用 Geppetto 实施 Puppet 清单

Puppet Labs 发布了一款 Puppet DSL 编辑器，称为 Geppetto。Geppetto 是一种集成开发环境 (IDE)，可用于创建 Puppet 清单和模块。Geppetto 可以由单一管理员使用，或者也可用于和他人协作。它可以和 git 或 Subversion 等修订控制系统集成。如果提供了身份验证凭据，Geppetto 还可将 Puppet 模块直接发布到 Puppet Forge。

###### 安装 Geppetto

目前可通过两种方式安装 Geppetto。一种方式是将 Geppetto 作为独立的产品下载和安装。另一种方式是在现有的 Eclipse IDE 安装基础上添加 Geppetto 功能。

1. 独立 Geppetto 安装

Puppet Labs Geppetto 主页上设有一个介绍如何安装 Geppetto 的区域。它包含指向 Geppetto 软件下载页面的超链接。

显示下载页面后，可以选择操作系统类型和架构。可用的操作系统包括 Linux、Macintosh OS X 和 Windows，系统架构可以选择 32 位或 64 位。选择完毕后，单击 Download 按钮即可开始下载 .zip 存档，其中含有 Geppetto 及所有必要组件。

.zip 存档应当提取到将要开发 Puppet 内容的用户的目录中。该存档将创建 geppetto 子目录，并为它填充超过 100 MB 的内容。子目录中将包含一个名为 geppetto 的可执行文件。执行该脚本将启动 Geppetto IDE。

2. 添加 Geppetto 功能到 Eclipse

安装 Geppetto 的另一种方式是将它的功能添加到现有的 Eclipse 环境中。在启动 Eclipse 后，选择 Help → Install New Software。这时会出现 Available Software 对话框。找到 Work with 框，再输入以下链接：https://geppetto-updates.puppetlabs.com/4.x。展开可用软件列表中出现的 Geppetto 条目，然后选中与安装 Geppetto 到 Eclipse 中对应的复选框。单击 Next 按钮，然后阅读并接受许可协议。安装结束时，单击 Finish。必须重新启动 Eclipse，然后才能在 IDE 中使用 Geppetto 功能。 

###### Geppetto 入门

通过运行 geppetto 脚本来启动 Geppetto。此脚本位于 ~/geppetto 目录中，由独立安装程序创建。

```
[student@workstation ~]$ ~/geppetto/geppetto
```

当 Geppetto 启动时，它会显示 Workspace Launcher 对话框。此处可以指定工具的工作空间。显示的默认选择是 ~/workspace。若有需要，可以通过 Browse... 按钮选择其他位置。可以通过选中复选框防止以后启动时显示此对话框。单击 OK 按钮以接受选择，Geppetto 便会继续运行。

Geppetto 项目通常包含有组成 Puppet 模块的所有文件。Geppetto 还可以创建和管理仅仅是清单的项目。若要创建 Geppetto 项目，请选择 File → New → Project...。这时会出现 New Project 对话框。展开 Puppet 文件夹，然后选择 Puppet Module Project 以开始构建 Puppet 模块，或选择 Puppet Project 以指定创建清单的目录。单击 Next 按钮，前进到指定所创建项目的名称的对话框。在 Project name 字段中输入名称，然后单击 Finish 按钮以创建该新项目。

创建的新项目将在屏幕左侧的 Project Explorer 窗格中处于选中状态。若要创建新清单，请单击 File 下拉菜单下方的新建图标。展开 Puppet 文件夹，再从文件类型列表中选择 New Puppet Manifest。单击 Next 按钮，然后指定要创建的清单的文件名。单击 Finish 按钮以创建该新项目。将创建新文件，并显示在右上方的窗格中。

这时，可以在编辑器中键入 Puppet DSL。Geppetto 将提供匹配的标点符号、右括号和引号，帮助创建 Puppet 清单。编辑器也会以各种颜色标出资源定义的不同元素，突出显示 Puppet 清单语法。当影响到所键入的内容时，框架的左边会显示语法错误和警告。鼠标悬停于符号上可显示气泡，带有更详细的说明消息。

Geppetto 也可以对 Puppet DSL 内容进行格式化，以方便阅读。若要这么做，可选择需要格式化的内容。按 Ctrl+A 可以快捷选择编辑器中的所有文本。若要格式化选中的文本，可按 Shift+Ctrl+F，或者右键单击选定内容，再从出现的菜单中选择 Source → Format。这是 Geppetto 的一项非常实用的功能。

键入了清单后，可以通过单击 Edit 下拉菜单下的 Save 图标来保存更改。按 Ctrl+S 或选择 File → Save，这两种方式都可以保存更改到新 Puppet 清单中。 


### 总结

在本章中，您学到了：

*    在 Puppet DSL 中指定 file、package、service 和 user 资源所需要的语法。
*    puppet 软件包在 Red Hat Enterprise Linux 上提供 Puppet 功能。
*    puppet apply 命令使主机符合作为参数专递的 Puppet 清单中指定的资源。 
