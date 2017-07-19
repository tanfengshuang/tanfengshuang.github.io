---
layout: post
title:  "Implementing Relationships in a Puppet Module(405-8)"
categories: Linux
tags: RHCA 405
---

### 创建命名空间

Puppet 模块中的主清单是 manifests/init.pp。此清单定义名称与模块相同的类。有时候，当这一清单变得非常大时，将类按照功能分割成较小的类可有帮助，例如 file 资源、user 资源和 service 资源。可选资源可以划分到单独的类中。可以根据资源在部署服务过程中的功能（如安装、配置和服务），将资源分组到不同的类中。

当一个类分割成相关的类时，类的名称和定义它们的清单很重要。它们划分成不同的区段，称为命名空间。命名空间告知 Puppet 自动加载程序如何查找模块中定义的各个类。

###### 在模块中映射命名空间

在开始接触命名空间时，最好是在正常工作的示例中查看它们的用法。ssh 模块为 OpenStack 安装程序提供的清单中，许多都定义了类。这些相关的清单位于 /usr/share/openstack-puppet/modules/ssh/manifests 目录中。下方的 tree 命令输出中显示了清单列表。

```
[root@host manifests]# tree
.
├── client
│   ├── config
│   │   └── user.pp
│   ├── config.pp
│   └── install.pp
├── client.pp
├── hostkeys.pp
├── init.pp
├── knownhosts.pp
├── params.pp
├── server
│   ├── config.pp
│   ├── host_key.pp
│   ├── install.pp
│   ├── match_block.pp
│   └── service.pp
└── server.pp
 
3 directories, 14 files
```

重复搜索字符串 “class”，可以识别哪些清单定义了模块的命名空间中的另一个类。模块的命名空间是 ssh，init.pp 清单定义 ssh 类，如预期一样。

```
[root@host manifests]# grep -R 'class' .
./client/config.pp:class ssh::client::config
./client/install.pp:class ssh::client::install {
./client.pp:class ssh::client(
./hostkeys.pp:class ssh::hostkeys {
./init.pp:class ssh (
./init.pp:  class { 'ssh::server':
./init.pp:  class { 'ssh::client':
./knownhosts.pp:class ssh::knownhosts {
./params.pp:class ssh::params {
./server/config.pp:class ssh::server::config {
./server/install.pp:class ssh::server::install {
./server/service.pp:class ssh::server::service {
./server.pp:class ssh::server(
```

注意 client.pp 清单如何定义 ssh::client 类。名称的第一个元素 ssh 是模块的命名空间，而 client 则是在该命名空间内定义的类的名称。init.pp 的 ssh 类定义中引用了这个类。ssh::server 类同样如此，它在 server.pp 文件中定义。

位于 server 子目录中的 install.pp 清单定义了一个名为 ssh::server::install 的类。与前文中一样，ssh 是模块的名称，但 install 是要在 ssh::server 命名空间中定义的类的名称。该命名空间告知 Puppet 自动加载程序定义对象的模块，同时也告知从模块目录结构的什么位置查找其定义。

双冒号 (::) 分隔对象名称中的命名空间元素。它类似于 Puppet 对象的完整路径名。如果使用简单名称，即单一元素，则不需要双冒号分隔符；其引用应用到在当前清单中定义的 Puppet 对象。

###### 创建额外的烟雾测试

对于更加复杂的模块，可以编写额外的烟雾测试来测试新类的功能。可以采用的一个好做法是在 tests 中创建清单，它们映射到 manifests 中所要操作的对应清单。

OpenStack SSH Puppet 模块的 tests 目录中有两个烟雾测试清单：init.pp 和 server.pp。如下是 server.pp 的内容。

```
include ssh::server
```

不需要为模块中的每一个类定义烟雾测试，但可以利用一些额外的烟雾测试来测试模块的一部分功能。 


### 练习：创建命名空间

1. 在 servera 上以 student 身份登录，再更改到 labwork/rht-webtest 目录并检查其模块。它最初应当在 manifests 目录中有一个清单，init.pp。

```
[student@servera ~]$ cd labwork/rht-webtest
[student@servera rht-webtest]$ ls
manifests  metadata.json  pkg  Rakefile  README.md  spec  tests
[student@servera rht-webtest]$ ls manifests
init.pp
```

2. 将任何上游更改拉取到本地 Git 存储库和工作树。

```
[student@servera rht-webtest]$ git pull
remote: Counting objects: 16, done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 14 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (14/14), done.
From ssh://workstation/var/git/labwork
   2982983..689b34d  master     -> origin/master
... Output omitted ...
```

3. 创建一个新类 rht-webtest::files，它用于管理由 rht-webtest 类管理的文件。

    a. 将原始的 manifests/init.pp 文件复制到 manifests/files.pp，以重复利用其中定义的 file 资源。

```
    [student@servera rht-webtest]$ cp manifests/init.pp manifests/files.pp
    [student@servera rht-webtest]$ ls manifests
    files.pp  init.pp
```

    b. 编辑 manifests/files.pp，使它定义一个名为 webtest::files 的类。完成后，它应当类似如下。

```
    # == Class: webtest::files
    #
     
    class webtest::files {
     
      file { '/var/www/html/index.html':
        ensure  => 'file',
        mode    => '0644',
        content => "<h1>This web site managed by Puppet.</h1>\n",
        require => Package['httpd'],
      }
    }
```

4. 创建两个新类，名为 webtest::users::alice 和 webtest::users::bob，它们用于管理 Web 管理员的用户帐户。

    a. 在模块层次结构中创建名为 manifests/users 的目录。这将包含用于定义 webtest::users 命名空间中类的清单。

```
    [student@servera rht-webtest]$ mkdir manifests/users
```
   
    b. 创建一个清单，它将定义管理 alice 用户帐户的类。该清单应当名为 manifests/users/alice.pp，并且应当定义名为 webtest::users::alice 的类。完成后，它应当类似如下：

```
    # == Class: webtest::users::alice
    #
     
    class webtest::users::alice {
     
      user { 'alice':
        ensure  => 'present',
        groups  => 'webmaster',
        home    => '/home/webmaster',
        shell   => '/bin/bash',
        require => File['/home/webmaster'],
      }
    }
```

    c. 对用于管理 bob 用户帐户的类执行相同的操作。完成后，名为 manifests/users/bob.pp 的清单应当类似如下：

```
    # == Class: webtest::users::bob
    #
     
    class webtest::users::bob {
     
      user { 'bob':
        ensure  => 'present',
        groups  => 'webmaster',
        home    => '/home/webmaster',
        shell   => '/bin/bash',
        require => File['/home/webmaster'],
      }
    }
```

5. 编辑主模块清单 init.pp。删除任何由定义的新类管理的资源定义。在主类中包含新类。完成后，init.pp 应当类似如下。

```
# == Class: webtest
#
... Output omitted ...
 
class webtest {
 
  user { 'webmaster':
    ensure  => 'present',
    home    => '/home/webmaster',
    shell   => '/bin/bash',
  }
 
  file { '/home/webmaster':
    ensure  => 'directory',
    owner   => 'webmaster',
    mode    => '0770',
    require => User['webmaster'],
  }
 
  package { 'httpd':
    ensure  => 'present',
  }
 
  service { 'httpd':
    ensure  => 'running',
    enable  => true,
    require => Package['httpd'],
  }
 
  include 'webtest::files'
  include 'webtest::users::alice'
  include 'webtest::users::bob'
}
```

6. 更新 metadata.json 文件，并递增模块版本。

```
{
  "name": "rht-webtest",
  "version": "0.1.2",
  "author": "rht",
  "summary": "Red Hat Training test web server.",
  "license": "Apache 2.0",
  "source": "",
  "project_page": null,
  "issues_url": null,
  "dependencies": [
  ]
}
```

7. 将清单和 metadata.json 更改添加到 Git 暂存区域。将更改与描述性日志消息一起提交到本地 Git 存储库。

```
[student@servera rht-webtest]$ git add manifests metadata.json
[student@servera rht-webtest]$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   manifests/files.pp
#       modified:   manifests/init.pp
#       new file:   manifests/users/alice.pp
#       new file:   manifests/users/bob.pp
#       modified:   metadata.json
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       pkg/
[student@servera rht-webtest]$ git commit -m 'Created more namespaces.'
[master 0e5e0ac] Created more namespaces.
 5 files changed, 46 insertions(+), 1 deletion(-)
 create mode 100644 rht-webtest/manifests/files.pp
 create mode 100644 rht-webtest/manifests/users/alice.pp
 create mode 100644 rht-webtest/manifests/users/bob.pp
```

8. 构建 rht-webtest 模块。

```
[student@servera rht-webtest]$ cd ..
[student@servera labwork]$ puppet module build rht-webtest
Notice: Building /home/student/labwork/rht-webtest for release
Module built: /home/student/labwork/rht-webtest/pkg/rht-webtest-0.1.2.tar.gz
```

9. 在 servera 上以 root 身份（密码 redhat）登录。安装 rht-webtest 模块。

```
[root@servera ~]# puppet module install --force \
      ~student/labwork/rht-webtest/pkg/rht-webtest-0.1.2.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── rht-webtest (v0.1.2)
```

10. 使用 --noop 选项应用已安装模块的烟雾测试，以确认它应当要执行的操作。

```
[root@servera ~]# puppet apply --noop /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 1.03 seconds
... Output omitted ...
Notice: /Stage[main]/Webtest::Users::Bob/User[bob]/ensure: current_value
 absent, should be present (noop)
Notice: Class[Webtest::Users::Bob]: Would have triggered 'refresh'
 from 1 events
Notice: /Stage[main]/Webtest::Files/File[/var/www/html/index.html]/ensure:
 current_value absent, should be file (noop)
Notice: Class[Webtest::Files]: Would have triggered 'refresh' from 1 events
Notice: /Stage[main]/Webtest::Users::Alice/User[alice]/ensure:
 current_value absent, should be present (noop)
Notice: Class[Webtest::Users::Alice]: Would have triggered 'refresh'
 from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 3 events
Notice: Finished catalog run in 0.39 seconds
```

11. 应用模块烟雾测试，并确认文件和用户都存在。 

        a. 首先，应用已安装模块的烟雾测试。       

```
        [root@servera ~]# puppet apply /etc/puppet/modules/webtest/tests/init.pp
        Notice: Compiled catalog for servera.lab.example.com in environment
         production in 1.01 seconds
        ... Output omitted ...
        Notice: /Stage[main]/Webtest::Users::Bob/User[bob]/ensure: created
        Notice: /Stage[main]/Webtest::Files/File[/var/www/html/index.html]/ensure:
         defined content as '{md5}6e9ca937d95e30d6a4b04a2053860cf6'
        Notice: /Stage[main]/Webtest::Users::Alice/User[alice]/ensure: created
        Notice: Finished catalog run in 0.58 seconds
```

        b. alice 和 bob 用户应当存在。

```
        [root@servera ~]# id alice
        uid=1013(alice) gid=1013(alice) groups=1013(alice),1001(webmaster)
        [root@servera ~]# id bob
        uid=1012(bob) gid=1012(bob) groups=1012(bob),1001(webmaster)

        index.html 文件应当已创建并具有正确的内容。

        [root@servera ~]# ls -l /var/www/html
        total 4
        -rw-r- -r- -. 1 root root 42 Sep 23 16:27 index.html
        [root@servera ~]# cat /var/www/html/index.html
        <h1>This web site managed by Puppet.</h1>
```

12. 以 student 身份，将您进行的任何其他更改提交到本地 Git 存储库。成功完成了练习时，将您提交的更改推送到上游 Git 存储库。

```
    [student@servera labwork]$ git push
    ... Output omitted ...
```


### 创建关系和依赖项

###### 定义 Puppet 对象之间的关系

Puppet 清单描述系统所需的最终状态。它们不是自上而下有序渐进的程序。Puppet 可以按照它认为最佳的顺序，自由处理清单中的资源。资源的次序或类型都不暗示它们应当要以怎样的顺序来完成。如果需要以特定的顺序实施一组资源，就必须显式声明它们之间的关系。

######### 利用关系元参数

声明资源之间关系的一种方式是使用关系元参数。Puppet 语言提供四种关系元参数，它们可用于任意资源类型。它们是 before、require、notify 和 subscribe 元参数。这些元参数在资源定义的正文中使用。其值引用其他 Puppet 对象，采用以下形式：

> Resource_type[ resource_title ]

Resource_type 以这种方式使用时，必须大写。通常在方括号内指定一个资源标题，但也可在需要引用多个资源时使用以逗号分隔的标题列表。

1. require

必须先应用用于定义 webmaster 用户的资源，然后创建用于管理 /home/webmaster 目录的 file 资源。

```
file { '/home/webmaster':
  ensure  => 'directory',
  owner   => 'webmaster',
  mode    => '0770',
  require => User['webmaster'],
}
```

2. before

指定资源之间这种关系的另一种方式是在 user 资源中使用 before 元参数。生成的 user 资源应当类似于下方所示：

```
user { 'webmaster':
  ensure  => 'present',
  home    => '/home/webmaster',
  shell   => '/bin/bash',
  before  => File['/home/webmaster'],
}
```

> 可以使用 require 和 before 元参数中的任何一个，但不需要同时指定它们。

3. subscribe 和 notify 元参数类似于 require 和 before 元参数，区别是通过向从属的资源发送通知来“刷新”该资源。例如，修改服务的配置文件时就会出现这种关系。

```
service { 'webserver':
  ensure  => 'running',
  name    => 'httpd',
  enable  => true,
}

file { '/etc/httpd/conf/httpd.conf':
  ensure  => 'file',
  owner   => 'root',
  group   => 'root',
  notify  => Service['webserver'],
  ... Output omitted ...
}
```

在上例中，Puppet 先创建 httpd.conf 文件，然后启动 httpd 服务。从资源排序角度上看，notify 元参数与 before 类似。它的区别在于，配置文件修改时会刷新 httpd 服务。通常，这会导致主机上重新启动该服务。

下例与上例相当，但它使用的是 subscribe 元参数，而非 notify。

```
service { 'webserver':
  ensure    => 'running',
  name      => 'httpd',
  enable    => true,
  subscribe => File['/etc/httpd/conf/httpd.conf'],
}

file { '/etc/httpd/conf/httpd.conf':
  ensure  => 'file',
  owner   => 'root',
  group   => 'root',
  ... Output omitted ...
}
```

######### 引用资源：标题和名变量

每一资源必须指定有标题。标题是资源声明中第一个字符串，其后紧跟一个冒号。每一资源类型具有一个指定资源本地名称的参数；这称为名变量 (namevar)，因为它通常是 name 参数。

name 参数是 service 资源的名变量。它指定要管理的服务的名称。file 资源的名变量是 path 参数。

省略了资源的名变量时，Puppet 将资源的标题用作隐式名变量的值。下例演示了这一情形。虽然省略了 path 参数，但所要管理的目录的路径名是 /var/www/html/examples，即它的标题。 

```
file { '/var/www/html/examples':
  ensure  => 'directory',
  owner   => 'root',
  mode    => '0755',
}
```

Puppet 对象的标题用于在 Puppet DSL 中引用它们。这对 Puppet 对象很有用处，如服务；根据所使用的操作系统，可能指代不同的服务名称。例如，Red Hat Enterprise Linux 6 上按下方所示定义 NTP 服务：

```
service { 'ntp':
  name    => 'ntpd',
  ensure  => 'running',
  enable  => true,
}
```

而在 Red Hat Enterprise Linux 7 系统上，此资源应当如下方所示：

```
service { 'ntp':
  name    => 'chrony',
  ensure  => 'running',
  enable  => true,
}
```

由于在引用中使用了标题，前两个示例都能作为 Service['ntp'] 来正确指代。标题 ntp 是 Puppet 解析器用于解析引用的名称。 

######### 利用时间表来表达关系

时间表是表达 Puppet 对象间关系的另一种方式。它们通过 -> 和 ~> 运算符定义。在使用时间表时，资源定义的顺序很重要，它们按照从左到右的顺序处理。

*    -> 运算符声明排序关系。必须先管理运算符左边的资源，再处理右边的资源。
*    ~> 定义相同的次序，但同时也建立通知关系。如果此运算符左边的资源有更改，则右边的资源会得到刷新。

下方为摘自 ntp/manifests/init.pp 清单的代码片段，该清单包含在 OpenStack 安装程序中。它演示了如何使用时间表来定义依赖项，即本例中的 Puppet 类。

```
anchor { 'ntp::begin': } ->
class { '::ntp::install': } ->
class { '::ntp::config': } ~>
class { '::ntp::service': } ->
anchor { 'ntp::end': }
```

首先应用 ::ntp::install 类的资源。处理完它们后，再应用 ::ntp::config 类的资源。最后，应用 ::ntp::service 类的资源。由于 ::ntp::config 和 ::ntp::service 类之间使用了 ~> 时间表运算符，对配置文件的任何资源更新将导致刷新第二个类中的服务。

class 关系包含在 anchor 模式之中，从而向后兼容 Puppet 3.4 及更早版本。此构造使得在定位符内定义的三个类包含在要使用它们的类中，它们就变成供该类专用，无法被其他类引用。 

###### 失败的依赖项故障排除

当引用指向不存在的资源时，缺少的依赖项会导致编译错误。出现这种情形时，Puppet 不会应用清单中的任何资源。

如果 Puppet 无法应用前提资源，它就不会继续应用依赖于该前提的资源。无关的资源则继续处理。出现这种情况时，会发出以下警告：

> warning: RESOURCE: Skipping because of failed dependencies

要留心的另一种依赖项问题是循环依赖。当两个或多个资源之间的依赖项形成循环时，会出现此问题。Puppet 将编译清单，但因为无法应用它，而会显示以下错误消息：

```
err: Could not apply complete catalog: Found 1 dependency cycle:
(RESOURCE => OTHER RESOURCE => RESOURCE)
```

### 练习：创建关系和依赖项

1. 创建名为 relate-do.pp 的清单，它将安装 httpd 软件包。它还应当启动 httpd 服务，并将它配置为在引导时启动。生成的清单应当类似于下方所示。

```
package { 'httpd':
  ensure  => 'present',
}
 
service { 'httpd':
  ensure  => 'running',
  enable  => true,
}
```

2. 应用 relate-do.pp 清单，以安装 httpd 软件包并启动 httpd 服务。如果它尝试在安装软件包之前启动服务，则可能会显示 systemctl 错误。

```
[root@servera ~]# puppet apply relate-do.pp 
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 0.86 seconds
... Output omitted ...
Error: Could not start Service[httpd]: Execution of '/usr/bin/systemctl
 start httpd' returned 6: Failed to issue method call: Unit httpd.service
 failed to load: No such file or directory.
Wrapped exception:
Execution of '/usr/bin/systemctl start httpd' returned 6: Failed to issue
 method call: Unit httpd.service failed to load: No such file or directory.
Error: /Stage[main]/Main/Service[httpd]/ensure: change from stopped
 to running failed: Could not start Service[httpd]: Execution of
 '/usr/bin/systemctl start httpd' returned 6: Failed to issue method call:
 Unit httpd.service failed to load: No such file or directory.
Notice: /Stage[main]/Main/Package[httpd]/ensure: created
Notice: Finished catalog run in 5.22 seconds
```

3. 编辑 relate-do.pp 清单，再添加 require 元参数到 service 资源。这样，它会要求先安装 httpd 软件包。

```
package { 'httpd':
  ensure  => 'present',
}
 
service { 'httpd':
  ensure  => 'running',
  enable  => true,
  require => Package['httpd'],
}
```

4. 表达 package 和 service 资源之间关系的另一种方式是必须先安装 httpd 软件包后才能管理 httpd 服务器。从 service 资源删除 require 元参数，再将 before 元参数添加到 package 资源。 

```
package { 'httpd':
  ensure  => 'present',
  before  => Service['httpd'],
}
 
service { 'httpd':
  ensure  => 'running',
  enable  => true,
}
```


### 引用文件


###### 从 Puppet 模块提供文件内容

Puppet 开发人员利用 file 资源定义文件系统上文件的状态。此资源的 content 参数可以取自决定文件内容的字符串。如果有许多文字材料，它会让清单充斥大量内容而混乱不堪。本节展示几种 Puppet 备选方法，通过模块来提供丰富的文件内容。

######### 提供静态文件内容

如果必须要提供无需更改的静态文件，可以在 file 资源中使用 source 参数。注意它和 content 参数互斥，给定 file 资源中只能使用两者之一。

source 参数的值应当是模块提供的文件的路径名。下方演示了 source 参数的用法：

```
source => 'puppet:///modules/MODULENAME/FILENAME',
```

MODULENAME 是提供文件的 Puppet 模块的名称。FILENAME 是所提供的原始文件的名称。在编写 Puppet 模块时，将文件放入 files 目录即可发布该文件。此目录必须与 manifests 目录处于 Puppet 模块开发树中的同一级上。

若要从名为 acme 的 Puppet 模块发布名为 widget.conf 的文件，原始的 widget.conf 应复制到模块的 rht-acme/files 目录。模块清单中相关的 file 资源应当如下方所示：

source => 'puppet:///modules/acme/widget.conf',

######### 提供可变文件内容

模板是向 file 资源提供大量文件内容的另一种方式。使用模板的清单通过 content 参数定义 file 资源，但它的值是对内置的 template() 函数的调用。

```
content => template('MODULENAME/FILENAME.erb'),
```

Puppet 在所引用的模块的 templates 目录的 FILENAME.erb 文件内查找模板，本例中为 MODULENAME。

Puppet 用于模板的语言是嵌入式 Ruby (ERB)。其表达式包含在 <% %> 标签内。下方是几个最简单的 ERB 构造：

```
<%# COMMENT %>
<%= EXPRESSION %>
```

第一种形式是用于记录模板的 ERB 注释。它不会以任何形式改动文件内容。第二种形式修改模板文件的内容。它将包含的 EXPRESSION 的值插入到文本中，该值通常是系统事实。

以下模板 hosts.erb 可以和 Puppet template() 函数一起用于在 Linux 中创建 /etc/hosts 的内容。

```
127.0.0.1   localhost localhost.localdomain
::1         localhost localhost.localdomain

<%= @ipaddress %>   <%= @fqdn %>
```

模板中的第一个表达式展开到客户端主机的 IP 地址，第二个表达式则展开到完全限定的主机名。@ 前缀告知 ERB 使用外部变量或事实，而不使用其自己的某一变量。

下方是一个实用的 Bash 函数，可以检查 ERB 模板的语法。Puppet 开发人员可以在 .bashrc 中定义此实用函数，或者将它的代码用在 Git hook 中，以在提交模板到存储库之前先检查其语法。

```
validate_erb() {
  erb -P -x -T - $1 | ruby -c
}
```


### 练习：引用文件

1. 以 student 用户身份登录 servera。更改到 labwork 目录，再从中央 Git 存储库拉取任何更新。

```
[student@servera ~]$ cd labwork
[student@servera labwork]$ git pull
... Output omitted ...
```

2. 更改到 rht-webtest 的工作目录，再创建要部署的静态文件，名为 index.html。您将需要在模块目录层次结构中创建 files 目录。

```
[student@servera labwork]$ cd rht-webtest
[student@servera rht-webtest]$ mkdir files
[student@servera rht-webtest]$ vim files/index.html
[student@servera rht-webtest]$ cat files/index.html
<h1>Puppet provided index.html</h1>
<p>
  This is a file provided in the files/ directory.
</p>
```

3. 修改 files.pp 清单，使 index.html file 资源的内容来自由 Puppet 模块提供的文件，而不是硬编码到资源 Puppet DSL 中。

```
[student@servera rht-webtest]$ vi manifests/files.pp
[student@servera rht-webtest]$ cat manifests/files.pp
# == Class: webtest::files
#
 
class webtest::files {
 
  file { '/var/www/html/index.html':
    ensure  => 'file',
    mode    => '0644',
    source  => 'puppet:///modules/webtest/index.html',
    require => Package['httpd'],
  }
 
}
```

4. 这需要使用 source 参数，而非 content 参数。其值告知 Puppet 从何处获取文件，即 webtest Puppet 模块名为 index.html 的文件。

在元数据文件中递增模块版本。

```
[student@servera rht-webtest]$ vim metadata.json
[student@servera rht-webtest]$ grep version metadata.json
  "version": "0.1.3",
```

5. 将 files 目录、清单和 metadata.json 更改添加到 Git 暂存区域。将更改与描述性日志消息一起提交到本地 Git 存储库。

```
[student@servera rht-webtest]$ git add files manifests/files.pp metadata.json
[student@servera rht-webtest]$ git commit -m 'Put index.html content in a file.'
... Output omitted ...
```

6. 构建模块的新版本。

```
[student@servera rht-webtest]$ cd ..
[student@servera labwork]$ puppet module build rht-webtest
Notice: Building /home/student/labwork/rht-webtest for release
Module built: /home/student/labwork/rht-webtest/pkg/rht-webtest-0.1.3.tar.gz
```

7. 以 root 身份在 servera 上登录，再安装模块的新版本。

```
[root@servera ~]# puppet module uninstall --force rht-webtest
Notice: Preparing to uninstall 'rht-webtest' ...
Removed 'rht-webtest' (v0.1.2) from /etc/puppet/modules
[root@servera ~]# puppet module install \
      ~student/labwork/rht-webtest/pkg/rht-webtest-0.1.3.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── rht-webtest (v0.1.3)
```

8. 应用已安装模块的烟雾测试清单。确认 /var/www/html/index.html 包含有 Puppet 模块提供的文件内容。

```
[root@servera ~]# puppet apply /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 1.18 seconds
... Output omitted ...
Notice: /Stage[main]/Webtest::Files/File[/var/www/html/index.html]/ensure:
 defined content as '{md5}7e69ac6558d1991aedd02348cbee33b5'
Notice: Finished catalog run in 0.74 seconds
[root@servera ~]# cat /var/www/html/index.html
<h1>Puppet provided index.html</h1>
<p>
  This is a file provided in the files/ directory.
</p>
```

9. 当 Puppet 需要基于主机相关事实自定义文件内容时，必须使用模板，而不使用固定内容文件。

重新以 student 身份登录 servera。定义名为 validate_erb 的函数，它将使用 Puppet 的模板语言 Puppet ERB 检查文件的语法。在 ~/.bashrc 文件中定义，使它永久存在。

```
[student@servera labwork]$ cat ~/.bashrc
# .bashrc

... Output omitted ...

# User specific aliases and functions

validate_erb() {
  erb -P -x -T - $1 | ruby -c
}
[student@servera labwork]$ source ~/.bashrc
```

10. 将固定格式的 index.html 文件转换为模板，Puppet 将利用系统的某一事实对它自定义。

    a. 创建 templates 目录，并将固定内容 index.html 复制到该目录。添加 .erb 扩展名，以告知 Puppet 这是以 Puppet ERB 语言编写的模板。

```
    [student@servera labwork]$ cd rht-webtest
    [student@servera rht-webtest]$ mkdir templates
    [student@servera rht-webtest]$ cp files/index.html templates/index.html.erb
```

    b. 修改 index.html.erb 模板，使它包含以下内容：

```
    <h1>Puppet provided index.html</h1>
    <p>
      This is a template provided in the templates/ directory.
    </p>
    <p>
      It is being hosted on <%= @fqdn %>.
    </p>

    <%= @fqdn %> 表达式将替换为 fqdn 系统事实。
```

    c. 通过 validate_erb 函数检查模板的语法。

```
     [student@servera rht-webtest]$ validate_erb templates/index.html.erb
    Syntax OK
```

11. 修改 files.pp 清单，使它使用模板而非 files 中的固定内容文件。生成的文件应当类似于下方所示。

```
# == Class: webtest::files
#
 
class webtest::files {
 
  file { '/var/www/html/index.html':
    ensure  => 'file',
    mode    => '0644',
    content => template('webtest/index.html.erb'),
    require => Package['httpd'],
  }
 
}
```

将 source 参数改回到 content，并调用 template() 函数。

12. 递增模块版本

```
[student@servera rht-webtest]$ grep version metadata.json
  "version": "0.1.4",
```

13. 将 templates 目录、清单和 metadata.json 更改添加到 Git 暂存区域。将更改与描述性日志消息一起提交到本地 Git 存储库。

```
[student@servera rht-webtest]$ git add templates manifests/files.pp metadata.json
[student@servera rht-webtest]$ git commit -m 'Put index.html content in a template.'
... Output omitted ...
```

14. 构建模块的新版本。

```
[student@servera rht-webtest]$ cd ..
[student@servera labwork]$ puppet module build rht-webtest
Notice: Building /root/rht-webtest for release
Module built: /root/rht-webtest/pkg/rht-webtest-0.1.4.tar.gz
```

15. 重新以 root 身份在 servera 上登录，再安装刚才构建的模块最新版本。

```
[root@servera ~]# puppet module install --force \
      ~student/labwork/rht-webtest/pkg/rht-webtest-0.1.4.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── rht-webtest (v0.1.4)
```

16. 应用已安装模块的烟雾测试清单。确认 /var/www/html/index.html 包含有模板的内容，其变量引用处已替换为 servera 的完全限定主机名。

```
[root@servera ~]# puppet apply /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 1.06 seconds
... Output omitted ...
Notice: /Stage[main]/Webtest::Files/File[/var/www/html/index.html]/content:
 content changed '{md5}7e69ac6558d1991aedd02348cbee33b5' to
 '{md5}6e9ca937d95e30d6a4b04a2053860cf6'
Notice: Finished catalog run in 0.43 seconds
[root@servera ~]# cat /var/www/html/index.html
<h1>Puppet provided index.html</h1>
<p>
  This is a template provided in the templates/ directory.
</p>
<p>
  It is being hosted on servera.lab.example.com.
</p>
```

17. 以 student 身份，将您进行的任何其他更改提交到本地 Git 存储库。成功完成了练习时，将您提交的更改推送到上游 Git 存储库。

```
[student@servera labwork]$ git push
... Output omitted ...
```


### 实验：在 Puppet 模块中实施关系

1. 以 student 身份登录 workstation，再运行实验设置脚本。它将提供的配置文件和登录旗帜复制到 servera 上的 /var/tmp 目录。

```
[student@workstation ~]$ lab puppet-relationships setup
```

2. 以 root 身份在 servera 上登录，再将 servera 上的防火墙配置为允许传入的 FTP 连接。

```
[root@servera ~]# firewall-cmd --add-service=ftp
success
[root@servera ~]# firewall-cmd --permanent --add-service=ftp
success
```

3. 以 servera 身份在 student 上登录，再为名为 rht-ftpcustom 的 Puppet 模块构建一个新的目录层次结构。将它创建在 labwork 目录中，从而让 Git 来管理更改。

使用 puppet module generate 命令创建目录层次结构和 Puppet 模块元数据。

```
[student@servera ~]$ cd labwork
[student@servera labwork]$ puppet module generate --skip-interview rht-ftpcustom
```

4. 确保提供的配置文件和登录旗帜文件已打包在 Puppet 模块中。

在模块开发目录中创建 files 目录，再将提供的配置文件复制到其中。不需要模板，因为不需要以任何方式修改文件。

```
[student@servera labwork]$ mkdir -p rht-ftpcustom/files
[student@servera labwork]$ cp /var/tmp/{custom-banner,vsftpd.conf} rht-ftpcustom/files
```

5. 编写下列用途的 Puppet 类：安装 vsftpd 软件包、部署提供的文件，以及管理 vsftpd 服务。Puppet 关系应当定义为按照下列顺序创建资源：安装软件，再配置软件，然后启用并启动服务。对服务配置进行的任何更正应当致使刷新服务。

    a. 创建一个 Puppet 类，它用于安装所需的 vsftpd 软件包。将它保存到以 Puppet 类命名的文件内，本例中为 manifests/install.pp。

```
    class ftpcustom::install {
     
      package { 'vsftpd':
        ensure  => 'present',
      }
     
    }
```

    b. 创建一个 Puppet 类，它将通过安装提供的 custom-banner 和 vsftpd.conf 文件来配置 FTP 服务。将它保存到以 Puppet 类命名的文件内，本例中为 manifests/config.pp。在 file 资源中利用 source 参数，以确保文件内容保持不变。

```
    class ftpcustom::config {
     
      file { '/etc/vsftpd/vsftpd.conf':
        ensure   => 'file',
        mode     => '600',
        source   => 'puppet:///modules/ftpcustom/vsftpd.conf',
      }
     
      file { '/etc/vsftpd/custom-banner':
        ensure   => 'file',
        mode     => '644',
        source   => 'puppet:///modules/ftpcustom/custom-banner',
      }
    }
```

    c. 创建一个 Puppet 类，它用于启动并启用 vsftpd 服务。将它保存到以 Puppet 类命名的文件内，本例中为 manifests/service.pp。

```
    class ftpcustom::service {
     
      service { 'vsftpd':
        ensure  => 'running',
        enable  => true,
        require => Package['vsftpd'],
      }
     
    }
```

    d. 修改在 manifests/init.pp 中定义的 ftpcustom Puppet 类。它应当包含为此模块定义的其他 Puppet 类。依赖项可以通过利用 before、require、notify 和 subscribe 元参数来定义；但下方用的是时间表。

```
    [student@servera labwork]$ vim rht-ftpcustom/manifests/init.pp
    [student@servera labwork]$ tail rht-ftpcustom/manifests/init.pp
    #
    class ftpcustom {
     
      class { '::ftpcustom::install': } ->
      class { '::ftpcustom::config': } ~>
      class { '::ftpcustom::service': }
     
    }
```

6. 定义一个主要烟雾测试清单，它将部署 FTP 服务。

默认情况下，可通过 puppet module generate 命令生成适当的烟雾测试。如果 Puppet 模块是手动创建的，请确保烟雾测试清单包含有 ftpcustom 类。

```
# The baseline for module testing used by Puppet Labs is that each manifest
# should have a corresponding test manifest that declares that class or defined
# type.
#
# Tests are then run by using puppet apply --noop (to check for compilation
# errors and view a log of events) or by fully applying the test in a virtual
# environment (to compare the resulting system state to the desired state).
#
# Learn more about module testing here:
# http://docs.puppetlabs.com/guides/tests_smoke.html
#
include ftpcustom
```

7. 更新模块元数据，使它不含有依赖于其他模块的依赖项。

更新 metadata.json 文件中的 dependencies 字段。

```
{
  "name": "rht-ftpcustom",
  "version": "0.1.0",
  "author": "rht",
  "summary": null,
  "license": "Apache 2.0",
  "source": "",
  "project_page": null,
  "issues_url": null,
  "dependencies": [
  ]
}
```

8. 将模块源代码目录添加到 Git 暂存区域。将更改与描述性日志消息一起提交。

```
[student@servera labwork]$ git add rht-ftpcustom
[student@servera labwork]$ git commit -m 'Initial version of rht-ftpcustom.'
... Output omitted ...
```

9. 构建并安装 rht-ftpcustom 模块。

    a. 可通过非特权用户构建模块。

```
    [student@servera labwork]$ puppet module build rht-ftpcustom
    Notice: Building /home/student/labwork/rht-ftpcustom for release
    Module built: /home/student/labwork/rht-ftpcustom/pkg/rht-ftpcustom-0.1.0.tar.gz
```

    b. 安装模块需要 root 特权。

```
    [root@servera ~]# puppet module install ~student/labwork/rht-ftpcustom/pkg/rht-ftpcustom-0.1.0.tar.gz
    Notice: Preparing to install into /etc/puppet/modules ...
    Notice: Downloading from https://forgeapi.puppetlabs.com ...
    Notice: Installing -- do not interrupt ...
    /etc/puppet/modules
    └── rht-ftpcustom (v0.1.0)
```

    c. 对模块进行烟雾测试也需要 root 特权。

```
    [root@servera ~]# puppet apply /etc/puppet/modules/ftpcustom/tests/init.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.96 seconds
    ... Output omitted ...
    Notice: /Stage[main]/Ftpcustom::Install/Package[vsftpd]/ensure: created
    Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/content:
     content changed '{md5}c4072ca90053a6e86cf86850c343346d' to
     '{md5}81ee930dab4087a96ea4f55b699702d2'
    Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/custom-banner]/ensure:
     defined content as '{md5}61c72b9a1e7bf54059d3b680dd1a0929'
    Notice: /Stage[main]/Ftpcustom::Service/Service[vsftpd]/ensure: ensure
     changed 'stopped' to 'running'
    Notice: Finished catalog run in 7.41 seconds
```

10. 测试 FTP 服务，确保其配置正确。连接成功时会显示以下消息：

```
[student@workstation ~]$ ftp servera.lab.example.com
Connected to servera.lab.example.com (172.25.250.10).
220-############################################################
220-#################### Private FTP Server ####################
220-############# Only authorized users permitted. #############
220-############################################################
220 
Name (servera.lab.example.com:student): 
```

最好也测试配置文件和服务之间的关系。如果 Puppet 恢复了配置文件，服务应当要刷新。

```
[root@servera ~]# echo > /etc/vsftpd/vsftpd.conf
[root@servera ~]# puppet apply /etc/puppet/modules/ftpcustom/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 1.01 seconds
... Output omitted ...
Notice: /Stage[main]/Ftpcustom::Config/File[/etc/vsftpd/vsftpd.conf]/content:
 content changed '{md5}68b329da9893e34099c7d8ad5cb9c940' to
'{md5}81ee930dab4087a96ea4f55b699702d2'
Notice: /Stage[main]/Ftpcustom::Service/Service[vsftpd]: Triggered
 'refresh' from 1 events
Notice: Finished catalog run in 0.68 seconds
```


### 总结

在本章中，您学到了：

*    命名空间定义模块中文件的文件路径，供 Puppet 自动加载程序搜索 Puppet 对象的定义。其第一个元素是要搜索的模块，其余的元素是用于搜索对象的目录。
*    require 和 before 元参数指定 Puppet 资源之间的有序关系。
*    subscribe 和 notify 元参数也指定资源之间的有序关系，但在依赖项改变状态时从属的资源将会刷新。
*    引用 Puppet 对象时，首先以大写字母开头，再将要引用的资源的标题放在方括号内。
*    Puppet 模块可以在 files 目录中发布静态文件，供其资源使用。可在 file 资源中通过 source 参数引用它们。
*    模块的 templates 目录中可以发布动态文件内容。可通过调用 template() 函数将它们包含在 file 资源的 content 参数中。
