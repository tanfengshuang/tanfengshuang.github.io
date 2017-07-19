---
layout: post
title:  "Implementing Puppet Modules(405-7)"
categories: Linux
tags: RHCA 405
---

### 构建 Puppet 模块

###### 编写 Puppet 模块

通过创建 Puppet 模块，可以将 Puppet 清单和相关的文件打包到一个文件中。模块是一个 tar 存档，具有确定的目录层次结构。puppet module 可以帮助在系统中开发、构建、安装和管理 Puppet 模块。

######### 研究模块目录结构

Puppet 模块为其中包含的 Puppet 代码和相关文件提供标准的可预测目录结构。在模块开发期间，Puppet 模块的顶级目录由模块作者名和模块名称组合而成。模块构建并分发后，其顶级目录还包含模块的版本号。

下方列出了模块层次结构中一些基本或最常使用的目录：

*    manifests 目录包含模块中定义的所有 Puppet 清单。它始终包含有 init.pp 清单，这是模块的“主”清单。
*    files 目录包含静态文件。它们通常是供模块清单所创建的用户或服务使用的配置文件。
*    lib/facter 目录包含自定义事实定义。它们应当是供模块中清单使用的事实。
*    tests 目录包含用于测试模块所提供的功能的清单。它还包含有 init.pp 清单，用于测试模块的“主”清单。

1. metadata.json 文件: 每个模块的顶级目录中含有一个 metadata.json 信息文件。此文件是用于定义元数据名称/值对的 JSON 格式文件，元数据描述模块的版本和用途，并指向提供有更多文档和联系信息的额外 URL。以下是一个最小的 metadata.json 文件,示例还演示了如何定义模块依赖项。这一个模块依赖于 puppetlabs-stdlib 模块提供的函数库。

```
[user@host ~]$ cat rht-modname/metadata.json
{
  "name": "rht-modname",
  "version": "0.1.0",
  "author": "rht",
  "summary": null,
  "license": "Apache 2.0",
  "source": "",
  "project_page": null,
  "issues_url": null,
  "dependencies": [
    {
      "name": "puppetlabs-stdlib",
      "version_range": ">= 1.0.0"
    }
  ]
}
```

2. puppet module generate: 此命令创建用于编写模块的工作目录的框架。该命令的语法为：

```
      [user@host ~]$ puppet module generate author-modulename
```

模块名称的 author 指定模块作者，而 modulename 则指定所写入的模块的名称。此命令在当前目录下创建一个用于编写模块的目录结构，名为 author-modulename。

```
[user@host ~]$ puppet module generate rht-modname
We need to create a metadata.json file for this module.  Please answer
the following questions; if the question is not applicable to this module,
feel free to leave it blank.
 
Puppet uses Semantic Versioning (semver.org) to version modules.
What version is this module?  [0.1.0]
--> Enter
 
Who wrote this module?  [rht]
--> Enter
 
... Output omitted ...
```

puppet module generate 询问与模块相关的几个问题（版本、许可、简短说明和联系人电子邮件地址等）。系统提供了默认值，因此在任何提示符处输入 Enter 将接受提议的默认值。如果使用 --skip-interview 选项调用 puppet module generate，可以跳过这些问题。 


###### 构建 Puppet 模块

puppet module build 命令取一个 Puppet 模块工作目录，并将它打包为 tar 存档。其顶级目录作为参数传递到此命令。命令会构建模块，并将它打包到其工作目录的 pkg 目录中。

```
[root@host ~]# puppet module build rht-modname
Notice: Building /root/rht-modname for release
Module built: /root/rht-modname/pkg/rht-modname-0.1.0.tar.gz
[root@host ~]# ls -F rht-modname/pkg
rht-modname-0.1.0/  rht-modname-0.1.0.tar.gz
```

puppet module install 命令将 Puppet 模块安装到系统的 /etc/puppet/modules 目录中。它必须以 root 身份执行。如果 /etc/puppet/modules 目录尚未存在，Puppet 会创建此目录。

```
[root@host ~]# puppet module install rht-modname/pkg/rht-modname-0.1.0.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└─┬ rht-modname (v0.1.0)
  └── puppetlabs-stdlib (v4.9.0)
```

安装了模块后，puppet module install 还会显示该模块具有的任何依赖项。示例中的模块依赖于 puppetlabs-stdlib，后者已在系统上安装。


### 练习：构建 Puppet 模块

1. 以 student 身份登录 workstation，再从 http:///materials.example.com/modules/thoraxe-motd-0.1.1.tar.gz 下载一个简单的模块。

```
[student@workstation ~]$ curl -O http://materials.example.com/modules/thoraxe-motd-0.1.1.tar.gz
... Output omitted ...
[student@workstation ~]$ file thoraxe-motd-0.1.1.tar.gz
thoraxe-motd-0.1.1.tar.gz: gzip compressed data, from Unix, last modified:
 Fri Mar 21 16:18:31 2014
```

2. 提取存档中的文件，并更改到模块的最上层目录。

```
[student@workstation ~]$ tar xvf thoraxe-motd-0.1.1.tar.gz
thoraxe-motd-0.1.1/
thoraxe-motd-0.1.1/Modulefile
thoraxe-motd-0.1.1/tests/
thoraxe-motd-0.1.1/tests/init.pp
thoraxe-motd-0.1.1/manifests/
thoraxe-motd-0.1.1/manifests/init.pp
thoraxe-motd-0.1.1/README
thoraxe-motd-0.1.1/metadata.json
thoraxe-motd-0.1.1/spec/
thoraxe-motd-0.1.1/spec/spec_helper.rb
[student@workstation ~]$ cd thoraxe-motd-0.1.1
```

3. 检查 manifests/init.pp 文件。它包含 Puppet 模块的主要 Puppet DSL。

```
[student@workstation thoraxe-motd-0.1.1]$ cat manifests/init.pp
# == Class: motd
#
# Full description of class motd here.
#
... Output omitted ...
# === Copyright
#
# Copyright 2014 Your name here, unless otherwise noted.
#
class motd ($message = 'This is the default message') {

  file { '/etc/motd':
    content => "$message \n",
  }

}
```

4. 以 student 身份（密码 student）登录 servera。为避免重复键入密码，可生成 SSH 密钥对，并将公钥上传到 workstation 上 student 的帐户。

```
[student@servera ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/student/.ssh/id_rsa): Enter
Enter passphrase (empty for no passphrase): Enter
Enter same passphrase again: Enter
Your identification has been saved in /home/student/.ssh/id_rsa.
Your public key has been saved in /home/student/.ssh/id_rsa.pub.
The key fingerprint is:
a4:fb:38:bb:95:f1:f8:4d:4d:67:0f:54:53:17:59:00 student@servera.lab.example.com
... Output omitted ...
[student@servera ~]$ ssh-copy-id workstation
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to
 filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are
 prompted now it is to install the new keys
student@workstation's password: student
... Output omitted ...
```

5. 为 student 定义全局 Git 配置设置。

```
[student@servera ~]$ git config --global user.name 'Student User'
[student@servera ~]$ git config --global user.email student@servera.lab.example.com
[student@servera ~]$ git config --global push.default simple
```

6. 克隆 ssh://student/var/git/labwork.git Git 存储库。更改到该目录，因为所有开发工作都将在该目录中完成。

```
[student@servera ~]$ git clone ssh://workstation/var/git/labwork.git
Cloning into 'labwork'...
remote: Counting objects: 13, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 13 (delta 2), reused 0 (delta 0)
Receiving objects: 100% (13/13), done.
Resolving deltas: 100% (2/2), done.
[student@servera ~]$ cd labwork
```

7. 利用 puppet module generate 命令创建一个新的工作目录，用于您要创建的模块。以空答案回答所有问题，从而接受工具提供的默认值。在提示时，为模块提供一个简单的描述。

```
[student@servera labwork]$ puppet module generate rht-webtest
We need to create a metadata.json file for this module.  Please answer the
following questions; if the question is not applicable to this module, feel free
to leave it blank.

Puppet uses Semantic Versioning (semver.org) to version modules.
What version is this module?  [0.1.0]
--> Enter

Who wrote this module?  [rht]
--> Enter

What license does this module code fall under?  [Apache 2.0]
--> Enter

How would you describe this module in a single sentence?
--> Red Hat Training test web server.

Where is this module's source code repository?
--> Enter

Where can others go to learn more about this module?
--> Enter

Where can others go to file issues about this module?
--> Enter

----------------------------------------
{
  "name": "rht-webtest",
  "version": "0.1.0",
  "author": "rht",
  "summary": "Red Hat Training test web server.",
  "license": "Apache 2.0",
  "source": "",
  "project_page": null,
  "issues_url": null,
  "dependencies": [
    {
      "name": "puppetlabs-stdlib",
      "version_range": ">= 1.0.0"
    }
  ]
}
----------------------------------------

About to generate this metadata; continue? [n/Y]
--> y

Notice: Generating module at /home/student/labwork/rht-webtest...
Notice: Populating ERB templates...
Finished; module generated in rht-webtest.
rht-webtest/Rakefile
rht-webtest/manifests
rht-webtest/manifests/init.pp
rht-webtest/spec
rht-webtest/spec/classes
rht-webtest/spec/classes/init_spec.rb
rht-webtest/spec/spec_helper.rb
rht-webtest/tests
rht-webtest/tests/init.pp
rht-webtest/README.md
rht-webtest/metadata.json
```

8. 将新的模块目录 rht-webtest 置于 Git 控制之下。与描述性日志消息一起提交到本地存储库。

```
[student@servera labwork]$ git add rht-webtest
[student@servera labwork]$ git commit -m 'Initial version of rht-webtest.'
[master 90d6d34] Initial version of rht-webtest.
 7 files changed, 191 insertions(+)
 create mode 100644 rht-webtest/README.md
 create mode 100644 rht-webtest/Rakefile
 create mode 100644 rht-webtest/manifests/init.pp
 create mode 100644 rht-webtest/metadata.json
 create mode 100644 rht-webtest/spec/classes/init_spec.rb
 create mode 100644 rht-webtest/spec/spec_helper.rb
 create mode 100644 rht-webtest/tests/init.pp
```

9. 更改到由 puppet module generate 创建的模块工作目录，并进行研究。

```
[student@servera labwork]$ cd rht-webtest/
[student@servera rht-webtest]$ ls -F
manifests/  metadata.json  Rakefile  README.md  spec/  tests/
```

检查为主要 Puppet 清单 manifests/init.pp 生成的内容。

```
[student@servera rht-webtest]$ ls manifests
init.pp
[student@servera rht-webtest]$ cat manifests/init.pp
# == Class: webtest
#
# Full description of class webtest here.
#
# === Parameters
#
# Document parameters here.
#
# [*sample_parameter*]
#   Explanation of what this parameter affects and what it defaults to.
#   e.g. "Specify one or more upstream ntp servers as an array."
#
... Output omitted ...
# === Copyright
#
# Copyright 2015 Your name here, unless otherwise noted.
#
class webtest {

}
```

10. 使现有 test.pp Puppet 清单的资源成为模块的主要 Puppet 清单。删除或注释掉 manifests/init.pp 中的样板 Puppet DSL，再添加 Web 服务器 Puppet 资源代替。test.pp 清单可以在 labwork 目录中找到，因为它之前已签入到 Git 中。删除 notify 资源，因为不会再用到它。如果无法访问现有的清单，下方列出了必要 Puppet DSL。

```
# Copyright 2015 Your name here, unless otherwise noted.
#

user { 'webmaster':
  ensure  => 'present',
  home    => '/home/webmaster',
  shell   => '/bin/bash',
}

file { '/home/webmaster':
  ensure  => 'directory',
  owner   => 'webmaster',
  group   => 'webmaster',
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
```

将更新后的 init.pp 清单添加到 Git 暂存区域。

```
[student@servera rht-webtest]$ git add manifests/init.pp
```

11. 许多模块作者使用来自 puppetlabs-stdlib Puppet 模块的库函数。puppet manifest generate 命令假定是这种情形，所以它会默认在 metadata.json 中创建包含有此模块的 JSON 列表。

由于我们在示例中使用的 Puppet 清单是自包含型的，因此请删除 puppetlabs-stdlib 引用，再使 dependencies 值变为空列表。生成的 metadata.json 文件内容应当如下所示。

```
{
  "name": "rht-webtest",
  "version": "0.1.0",
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

将更新后的 metadata.json 文件添加到 Git 暂存区域。

```
[student@servera rht-webtest]$ git add metadata.json
```

12. 更改到模块工作目录的父目录，再使用 puppet module build 命令将工作目录中的 Puppet 组件汇编成模块。

```
[student@servera rht-webtest]$ cd ..
[student@servera labwork]$ puppet module build rht-webtest
Notice: Building /home/student/labwork/rht-webtest for release
Module built: /home/student/labwork/rht-webtest/pkg/rht-webtest-0.1.0.tar.gz
```

工作目录中出现一个名为 pkg 的新目录。它包含有从构建生成的模块的 tar 存档。

```
[student@servera labwork]$ ls -l rht-webtest/pkg
total 8
drwxrwxr-x. 5 student student 4096 Sep 15 19:36 rht-webtest-0.1.0
-rw-rw-r--. 1 student student 3631 Sep 15 19:36 rht-webtest-0.1.0.tar.gz
[student@servera labwork]$ tar tzf rht-webtest/pkg/rht-webtest-0.1.0.tar.gz 
rht-webtest-0.1.0/
rht-webtest-0.1.0/Rakefile
rht-webtest-0.1.0/manifests/
rht-webtest-0.1.0/manifests/init.pp
rht-webtest-0.1.0/spec/
rht-webtest-0.1.0/spec/classes/
rht-webtest-0.1.0/spec/classes/init_spec.rb
rht-webtest-0.1.0/spec/spec_helper.rb
rht-webtest-0.1.0/tests/
rht-webtest-0.1.0/tests/init.pp
rht-webtest-0.1.0/README.md
rht-webtest-0.1.0/metadata.json
rht-webtest-0.1.0/checksums.json
```

13. 将更改与有意义的日志消息一起提交到本地 Git 存储库。rht-webtest Puppet 模块构建成功后，将它们推送到上游存储库。

```
[student@servera labwork]$ git commit -m 'First edition of rht-webtest module.'
[master 79fa396] First edition of rht-webtest module.
 2 files changed, 20 insertions(+), 5 deletions(-)
[student@servera labwork]$ git push
Counting objects: 21, done.
Compressing objects: 100% (16/16), done.
Writing objects: 100% (20/20), 4.46 KiB | 0 bytes/s, done.
Total 20 (delta 4), reused 0 (delta 0)
To ssh://workstation/var/git/labwork.git
   d401e79..79fa396  master -> master
```

14. 
```
[student@servera labwork]$ puppet install rht-webtest/pkg/rht-webtest-0.1.0.tar.gz
[student@servera labwork]$ puppet modules list

[student@servera labwork]$ puppet help module
[student@servera labwork]$ man puppet-module
```


### 实施类

###### 通过类重新利用 Puppet 代码

Puppet 清单指定应当如何配置某一特定的系统；但是，有一种方式能够让 Puppet 资源定义变得更加通用并可重新利用，从而应用到多个主机上，那就是 Puppet 类。

Puppet 类定义 Puppet 资源定义的命名块，可供以后使用。在某种程度上，它们类似于编程语言中的函数定义。Puppet 类通常定义实施一项服务或运行某一应用（用户、软件包、配置文件和服务）所需的全部资源。Puppet 类可以合并成更高级别的类来定义系统角色，如 FTP 服务器。

下例中演示了 Puppet 类定义的最基本语法：

```
class class_name {
  resource definitions...
}
```

class 关键字后面是类的名称。此外，也可选择在类名称后加上类参数，并用括号括起。用花括号括起定义 Puppet 类的代码。通常，这是资源定义的列表，比如定义提供网络服务需要的所有资源。下方演示了如何定义取用参数的类：

```
class class_name ($param = 'value') {
  resource definitions...
}
```

include 命令用于要使用类的清单中。它类似于在编程语言中调用前面定义的函数。

在模块中，Puppet 类在位于 manifests 目录下的文件中定义，每个文件包含一个类。每个文件的名称应当与所定义的类相同。init.pp 文件应当定义名称与模块名称相同的类。


### 练习：实施类

1. 在 servera 上以 student 身份（密码 student）登录。编辑 rht-webtest 模块的主要清单。修改 manifests/init.pp，将现有的 Puppet DSL 更改成名为 webtest 的 Puppet 类。类定义应当类似于下方所示。

```
#
# Copyright 2015 Your name here, unless otherwise noted.
#

class webtest {
  user { 'webmaster':
    ensure  => 'present',
    home    => '/home/webmaster',
    shell   => '/bin/bash',
  }

  file { '/home/webmaster':
    ensure  => 'directory',
    owner   => 'webmaster',
    group   => 'webmaster',
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
}
```

2. 由于 rht-webtest 模块已被更新，因此也要更新模块的元数据信息。编辑 metadata.json 文件，将 version 字段递增到 0.1.1。生成的文件应当类似于下方所示。

```
{
  "name": "rht-webtest",
  "version": "0.1.1",
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

3. 使用 puppet module build 构建模块的更新版本。

```
[student@servera rht-webtest]$ cd ..
[student@servera labwork]$ puppet module build rht-webtest
Notice: Building /home/student/labwork/rht-webtest for release
Module built: /home/student/labwork/rht-webtest/pkg/rht-webtest-0.1.1.tar.gz
[student@servera labwork]$ ls rht-webtest/pkg
rht-webtest-0.1.0         rht-webtest-0.1.1
rht-webtest-0.1.0.tar.gz  rht-webtest-0.1.1.tar.gz
```

4. 将您更改的文件添加到暂存区域，然后将它们与帮助性日志消息一起提交到本地 Git 存储库。将它们推送到中央 Git 存储库。

```
[student@servera labwork]$ git add rht-webtest/manifests/init.pp
[student@servera labwork]$ git add rht-webtest/metadata.json
[student@servera labwork]$ git commit -m 'Created webtest Puppet class.'
[master 2982983] Created webtest Puppet class.
 2 files changed, 23 insertions(+), 19 deletions(-)
[student@servera labwork]$ git push
Counting objects: 11, done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 775 bytes | 0 bytes/s, done.
Total 6 (delta 3), reused 0 (delta 0)
To ssh://workstation/var/git/labwork.git
   79fa396..2982983  master -> master
```


### 利用烟雾测试部署 Puppet 模块

模块层次结构中的 tests 目录存放用于测试模块中 Puppet 清单所提供功能的清单。至少，它应当包含与 manifests/init.pp 清单对应的 init.pp 烟雾测试清单。由于 manifests/init.pp 定义模块的主类，因此 tests/init.pp 应当仅仅是通过将该类包含在 include 语句中来使用它。

puppet module generate 命令基于所定义的模块/类的名称来创建正常工作的烟雾测试清单。比较下方由工具创建的模板文件：

```
[root@host ~]# puppet module generate --skip-interview rht-modulename

Notice: Generating module at /root/rht-modulename...
Notice: Populating ERB templates...
Finished; module generated in rht-modulename.
... Output omitted ...
rht-modulename/manifests/init.pp
... Output omitted ...
rht-modulename/tests/init.pp
... Output omitted ...
[root@host ~]# tail rht-modulename/manifests/init.pp
# Author Name <author@domain.com>
#
# === Copyright
#
# Copyright 2015 Your name here, unless otherwise noted.
#
class modulename {
}
[root@host ~]# tail rht-modulename/tests/init.pp
# type.
#
# Tests are then run by using puppet apply --noop (to check for compilation
# errors and view a log of events) or by fully applying the test in a virtual
# environment (to compare the resulting system state to the desired state).
#
# Learn more about module testing here:
# http://docs.puppetlabs.com/guides/tests_smoke.html
#
include modulename
```

如果 Puppet 模块创建新的资源类型或定义可由其他模块使用的库函数，则可以向该目录添加其他文件来测试更为具体的功能。对于带有参数的 Puppet 类，可以定义其他清单来使用不同的参数值测试该类。

烟雾测试清单也可以为管理员演示如何使用 Puppet 模块提供的类。从这一角度上看，它们可以充当最基本形式的文档。

###### 执行烟雾测试

将 puppet apply --noop 命令用于 tests/init.pp 清单，可以显示在使用模块所定义的类时 Puppet 将执行哪些命令和操作。应当要仔细研究列出的 Notice 消息，从而确保执行正确系统配置需要的所有步骤。

在将新模块发布到生产环境之前，可以对 tests/init.pp 清单使用 puppet apply 命令（但不带 --noop 选项）来执行更加详尽的烟雾测试。这应当在全新的虚拟机上执行，从而能够仔细研究对系统配置进行的更改。 


### 练习：利用烟雾测试部署 Puppet 模块

1. 以 student 身份登录 workstation，再检查 thoraxe-motd Puppet 模块的测试清单。检查 tests/init.pp 文件。

```
[student@workstation ~]$ cat thoraxe-motd-0.1.1/tests/init.pp
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
include motd
```

由于此模块定义名为 motd 的 Puppet 类，因此测试清单仅包含该类以应用它。

2. 以 root 身份登录 servera，再检查该模块的测试清单。它应当包含在 manifests/init.pp 中定义的 Puppet 类。

```
[root@servera ~]# cd ~student/labwork/rht-webtest
[root@servera rht-webtest]# cat tests/init.pp
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
include webtest
```

不需要执行任何操作，因为烟雾测试已经就位。puppet module generate 命令创建了 tests/init.pp，它引用了以模块名称取名的模块类。

3. 由于该模块中的烟雾测试已经就位，因此不需要通过更改元数据信息来递增版本。上一实践练习中已构建了模块，因此可使用 puppet module install 命令安装它。

```
[root@servera rht-webtest]# ls -F /etc/puppet
auth.conf  modules/  puppet.conf
[root@servera rht-webtest]# puppet module install pkg/rht-webtest-0.1.1.tar.gz
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Created target directory /etc/puppet/modules
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── rht-webtest (v0.1.1)
[root@servera rht-webtest]# ls -F /etc/puppet
auth.conf  manifests/  modules/  puppet.conf
```

4. puppet module list 命令将显示已安装模块的列表。

```
[root@servera rht-webtest]# puppet module list
/etc/puppet/modules
└── rht-webtest (v0.1.1)
/usr/share/puppet/modules (no modules installed)
```

5. 对测试清单使用 puppet apply --noop，从而对模块提供的 Puppet 类型进行烟雾测试。虽然测试清单已在模块开发目录中，但请使用 /etc/puppet/modules/webtest/tests/init.pp 中由已安装模块提供的测试清单。

```
[root@servera rht-webtest]# puppet apply --noop /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment production
 in 0.52 seconds
Warning: The package type's allow_virtual parameter will be changing its
 default value from false to true in a future release. If you do not want
 to allow virtual packages, please explicitly set allow_virtual to false.
   (at /usr/share/ruby/vendor_ruby/puppet/type.rb:816:in `set_default')
Notice: Finished catalog run in 0.11 seconds
```

6. 该模块不会对系统进行任何更改，因为所需状态的所有条件都已满足。删除 httpd 软件包，然后使用 --noop 选项再次应用烟雾测试。

```
[root@servera rht-webtest]# yum -y erase httpd
... Output omittted ...
[root@servera rht-webtest]# puppet apply --noop /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment production
 in 0.53 seconds
Warning: The package type's allow_virtual parameter will be changing its
 default value from false to true in a future release. If you do not want
 to allow virtual packages, please explicitly set allow_virtual to false.
   (at /usr/share/ruby/vendor_ruby/puppet/type.rb:816:in `set_default')
Notice: /Stage[main]/Webtest/Package[httpd]/ensure: current_value absent, should be present (noop)
Notice: /Stage[main]/Webtest/Service[httpd]/ensure: current_value stopped, should be running (noop)
Notice: Class[Webtest]: Would have triggered 'refresh' from 2 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.21 seconds
```

7. --noop 选项阻止 puppet apply 命令更改系统。更为广泛的测试可以省略该选项，并且应用更改来确认 Puppet 类正常工作。

```
[root@servera rht-webtest]# puppet apply /etc/puppet/modules/webtest/tests/init.pp
Notice: Compiled catalog for servera.lab.example.com in environment production
 in 0.55 seconds
Warning: The package type's allow_virtual parameter will be changing its
 default value from false to true in a future release. If you do not want
 to allow virtual packages, please explicitly set allow_virtual to false.
   (at /usr/share/ruby/vendor_ruby/puppet/type.rb:816:in `set_default')
Notice: /Stage[main]/Webtest/Package[httpd]/ensure: created
Notice: /Stage[main]/Webtest/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
Notice: Finished catalog run in 2.58 seconds
```


### 总结

在本章中，您学到了：

*    puppet module generate 命令将创建用于开发 Puppet 模块的模板目录结构。
*    puppet module build 命令取一个模块开发目录，并将它汇编为模块。
*    puppet module install 命令将 Puppet 模块提取到 /etc/puppet/modules 下的目录中。
*    Puppet 模块应当按照每一清单一个类来定义 Puppet 类，manifests/init.pp 中定义的主类应当具有与模块相同的名称。
*    每一 Puppet 模块应当在 tests/init.pp 中定义烟雾测试，它应包含由模块定义的主类。 
