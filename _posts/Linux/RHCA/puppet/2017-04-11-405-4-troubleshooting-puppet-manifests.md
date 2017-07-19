---
layout: post
title:  "Troubleshooting Puppet Manifests(405-4)"
categories: Linux
tags: RHCA 405
---

### 查找 Puppet DSL 文档

###### 利用本地 Puppet 文档

管理员可以从哪里找到关于 Puppet 资源及可用资源参数的更多信息？在 Red Hat Enterprise Linux 系统上安装 Puppet 时，系统上也会同时安装相应的文档。使用 puppet resource --types 命令可查看可用 Puppet 资源类型的列表。

```
[root@host ~]# puppet resource --types
augeas
computer
cron
exec
file
filebucket
group
host
interface
... Output omitted ...
```

列出 Puppet 资源类型仅提供资源的名称。若要查看资源用法的详细信息，可以生成本地文档或使用 puppet describe 命令。

###### 生成 Puppet 文档

puppet doc 命令将生成所有 Puppet 类型的参考资料。默认情况下，这会发送超过 6,000 行的文档内容到屏幕中。使用 less 命令可搜索特定资源或功能的文档。每一资源部分以 "###" 开头，后跟资源的名称。下例中使用 /### yumrepo 跳转到 yumrepo 资源定义。

```
[root@host ~]# puppet doc | less
### yumrepo
 
The client-side description of a yum repository. Repository
configurations are found by parsing `/etc/yum.conf` and
the files indicated by the `reposdir` option in that file
(see `yum.conf(5)` for details).
 
Most parameters are identical to the ones documented
in the `yum.conf(5)` man page.
 
Continuation lines that yum supports (for the `baseurl`, for example)
are not supported. This type does not attempt to read or verify the
existence of files listed in the `include` attribute.
 
#### Parameters
baseurl
: The URL for this repository. Set this to `absent` to remove it from the file completely.
 
  Valid values are `absent`. Values can match `/.*/`.
```

如示例中所示，资源的常规描述后面是 Parameters 部分。一些资源的 Parameters 部分前还有 Features 部分。某些功能可能不受部分提供商支持。


###### 描述 Puppet 资源

查看资源的功能的另一种方式是使用 puppet describe 命令。

```
[root@host ~]# puppet describe yumrepo
yumrepo
=======
The client-side description of a yum repository. Repository
configurations are found by parsing `/etc/yum.conf` and
the files indicated by the `reposdir` option in that file
(see `yum.conf(5)` for details).
Most parameters are identical to the ones documented
in the `yum.conf(5)` man page.
Continuation lines that yum supports (for the `baseurl`, for example)
are not supported. This type does not attempt to read or verify the
exinstence of files listed in the `include` attribute.
 
Parameters
----------
 
- **baseurl**
    The URL for this repository. Set this to `absent` to remove it from the
    file completely.
 
... Output omitted ...
```

与 puppet doc 命令不同，puppet describe 显示作为参数指定的单一 Puppet 资源的文档。其输出更容易阅读，因此它能够帮助 Puppet 用户找到关于资源参数和可能的值的更多信息，只要他们知道自己要关注的资源类型是什么。
利用 Puppet Labs 参考文档

Puppet Labs 网站上提供了额外的文档，网址为 https://docs.puppetlabs.com/。如需资源说明，可以查看 Open Source Puppet 列下方，再选择 Type Reference 链接。这时会出现各种可用资源类型的超链接。

> 注意一开始会显示文档的最新社区版本。可通过左侧导航区域中的链接选择其他版本。

Puppet Labs 文档页面井然有序。它们提供各种 Puppet 主题的参考信息和教程。 


### Puppet 清单故障排除

###### 使用 puppet parser validate 检查语法

Puppet 新用户在创建清单时，经常会犯一些简单的语法错误。可以在命令行中通过 puppet parser validate 命令调用 Puppet 解析器。命令的参数是要检查的清单的文件名。以下是一个正常工作的 Puppet 清单，没有任何语法错误。

```
[root@host ~]# cat -n sample.pp
     1  yumrepo { 'rhel7osp':
     2    baseurl  => 'http://content.example.com/puppet3.6/x86_64/dvd/rhel7osp/',
     3    enabled  => '1',
     4    gpgcheck => '0',
     5    descr    => 'Red Hat Enterprise Linux OpenStack Platform 7',
     6  }
     7
     8  package { 'openstack-puppet-modules':
     9    ensure  => 'present',
    10    require => Yumrepo['rhel7osp'],
    11  }
```

puppet parser validate 没有可报告的语法错误时，不会生成任何输出。这时它也会以零退出状态退出，因此它可以在 shell 程序中使用。

```
[root@host ~]# puppet parser validate sample.pp
[root@host ~]# echo $?
0
```

如果存在语法错误，解析器将返回错误以及和不良语法相关的信息。以下 Puppet DSL 片段来自一个第 6 行上缺少花括号的清单。

```
     4   gpgcheck => '0',
     5   descr => 'Red Hat Enterprise Linux OpenStack Platform 7',
     6
     7
     8  package { 'openstack-puppet-modules':
     9   ensure => 'present',
```

缺少花括号时，puppet parser validate 命令生成指引清单中下一行的错误，并生成 “expected '}'” 消息。错误发生在解析器到达不应在开放区域中的项目时，这可能是缺失花括号后的一行或多行。

```
[root@host ~]# puppet parser validate sample.pp
Error: Could not parse for environment production: Syntax error at '{';
 expected '}' at /root/sample.pp:8
```

其他拼写错误导致在行位置和花括号处的类似错误。以下清单在第 3 行上缺少分隔资源参数的逗号。

```
[root@host ~]# cat -n sample.pp
     1  yumrepo { 'rhel7osp':
     2    baseurl  => 'http://content.example.com/puppet3.6/x86_64/dvd/rhel7osp/',
     3    enabled  => '1'
     4    gpgcheck => '0',
     5    descr    => 'Red Hat Enterprise Linux OpenStack Platform 7',
     6  }
[root@host ~]# puppet parser validate sample.pp
Error: Could not parse for environment production: Syntax error at 'gpgcheck';
 expected '}' at /root/sample.pp:4
```

同样，解析器会生成指引下一行的错误，并且指出应当有右花括号。Puppet 解析器认为 gpgcheck 可能是一个新的资源快。

###### 利用 puppet apply --noop 进行故障排除

某些 Puppet 清单错误的语法是正确的，所以 puppet parser validate 无法检测到它们并生成输出。可以给出任何资源参数，但解析器不知道它是否正确或有效，直到清单被应用为止。

以下 Puppet 清单的第 4 行上有一个拼写错误的参数名。

```
[root@host ~]# cat -n sample.pp
     1  yumrepo { 'rhel7osp':
     2    baseurl  => 'http://content.example.com/puppet3.6/x86_64/dvd/rhel7osp/',
     3    enabled  => '1',
     4    pgcheck => '0',
     5    descr    => 'Red Hat Enterprise Linux OpenStack Platform 7',
     6  }
```

该清单的语法是正确的。所有逗号、花括号、引号和等号都在正确的位置上。利用 puppet parser validate 验证该清单将返回零退出状态。

```
[root@host ~]# puppet parser validate sample.pp
[root@host ~]# echo $?
0
```

在应用清单到系统之前，可使用 puppet apply --noop 进行试运行，不对系统进行任何更改。此命令会执行额外的步骤，解译资源参数并检查其分配的值。

```
[root@host ~]# puppet apply --noop sample.pp
Error: Invalid parameter pgcheck on Yumrepo[rhel7osp] at /root/sample.pp:6
 on node host.example.com
Wrapped exception:
Invalid parameter pgcheck
Error: Invalid parameter pgcheck on Yumrepo[rhel7osp] at /root/sample.pp:6
 on node host.example.com
```

生成的错误消息指向无效的参数。使用 puppet describe yumrepo 验证该资源正确且有效的参数名。修复了清单并且无错通过 --noop 测试后，就可将清单应用到系统。 


### 总结

在本章中，您学到了：

*    可以选择通过 puppet doc 命令为清单生成 Puppet 文档和参考资料。
*    puppet describe 命令显示关于 Puppet 资源的用法和参数的信息。
*    Puppet Labs 发布的 Puppet 3.6 参考手册是非常实用的 Puppet 文档。
*    puppet parser validate 命令可以识别简单的 Puppet DSL 语法错误。
*    puppet apply --noop 命令将识别清单中的其他错误，如无效的参数。 
