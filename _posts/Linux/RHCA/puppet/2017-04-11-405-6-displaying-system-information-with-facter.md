---
layout: post
title:  "Displaying System Information with Facter(405-6)"
categories: Linux
tags: RHCA 405
---

### 使用 Facter 显示系统事实

###### 识别系统信息

Puppet 使用客户端系统的相关系统事实来决定配置它们的方式。在启动 Puppet 运行时，Puppet 代理首先识别它在其上运行的节点的系统事实。当代理与 Puppet 宿主联系时，它将这些事实发送到宿主。Puppet 宿主使用这些事实编译目录，目录则决定了将要在 Puppet 客户端上应用的清单。

Puppet 收集的与节点相关的一些事实包括：它所运行的操作系统、硬件信息和网络信息，如主机名和 IP 地址。这些事实可供 Puppet 用于在配置文件中生成特定于主机的信息。它们也供 Puppet 用于做出决策。

可以在 shell 提示符使用 facter 命令显示系统事实。不带参数时，facter 将列出描述系统的所有键/值对的列表。

```
[root@host ~]# facter | head
architecture => x86_64
augeasversion => 1.0.0
bios_release_date => 01/01/2011
bios_vendor => Bochs
bios_version => Bochs
blockdevice_vda_size => 107374182400
blockdevice_vda_vendor => 6900
blockdevices => vda
domain => example.com
facterversion => 1.7.6
```

给定单一参数时，facter 将显示对应的系统事实的值。如果指定的事实不存在，则不显示任何内容。

```
[root@host ~]# facter memorysize
3.74 GB
[root@host ~]# facter cpuspeed
[root@host ~]# 
```

###### 在 Puppet DSL 中使用系统事实

系统事实通常在 Puppet DSL 中引用。下列 Puppet DSL 片段具有一个取决于系统事实的条件：

```
[root@host ~]# cat facter-example.pp
if $::osfamily == 'redhat' {
  $fpath = '/tmp/example'
}
else {
  $fpath = '/var/tmp/example'
}
file { 'example':
  path    => $fpath,
  ensure  => file,
  mode    => 0444,
  owner   => 'root',
  group   => 'root',
  content => "\nIMPORTANT DATA.\n\n",
}
```

$::osfamily 是对系统事实的引用。facter osfamily 命令显示它的值。

```
[root@host ~]# facter osfamily
RedHat
```

虽然值中混合了大小写字母，但 Puppet DSL 中的比较不区分大小写。请思考应用了 facter-example.pp 清单时会出现什么情况。

```
[root@host ~]# puppet apply facter-example.pp
Notice: Compiled catalog for host.example.com in environment production
 in 0.23 seconds
Notice: /Stage[main]/Main/File[example]/ensure: defined content as
 '{md5}1d8b8c9d7e971f61687a3a552afc0cae'
Notice: Finished catalog run in 0.06 seconds
[root@host ~]# ls /tmp/example 
/tmp/example
[root@host ~]# ls /var/tmp/example
ls: cannot access /var/tmp/example: No such file or directory
[root@host ~]# cat /tmp/example

IMPORTANT DATA.
```

条件语句中的第一个子句将 $::osfamily 的值与 'redhat' 匹配，所以文件名被设为 /tmp/example。这是应用清单时所创建的文件。

###### 实用系统事实

以下是 facter 识别的一些实用系统事实。

可以获取硬件架构信息。

```
[root@host ~]# facter architecture
x86_64
```

可以运行 facter 命令识别用户相关的信息。

```
[root@host ~]# facter id
root
```

可以获取网络信息。这包括主机的完全限定域名。还可以识别网络接口的信息及它们对应的 IP 地址。

```
[root@host ~]# facter fqdn
host.example.com
[root@host ~]# facter | grep interfaces
interfaces => eth0,lo
[root@host ~]# facter | grep ipaddress
ipaddress => 172.25.250.10
ipaddress_eth0 => 172.25.250.10
ipaddress_lo => 127.0.0.1
```

可以识别操作系统版本相关的信息。

```
[root@host ~]# facter | grep operating
operatingsystem => RedHat
operatingsystemmajrelease => 7
operatingsystemrelease => 7.1
[root@host ~]# facter osfamily
RedHat
```

facter 可以确定它是否在虚拟主机上运行；而且，若为是，还可确定所用的虚拟化类型。

```
[root@host ~]# facter | grep virtual
is_virtual => true
virtual => kvm
```


### 练习：使用 Facter 显示系统事实

1. 在 servera 上以 root 身份登录，再确认系统上已安装了提供 facter 命令的软件包。

```
[root@servera ~]# yum -y install facter
```

2. 使用 facter 命令显示系统信息值列表。

```
[root@servera ~]# facter
architecture => x86_64
augeasversion => 1.1.0
bios_release_date => 01/01/2011
bios_vendor => Bochs
bios_version => Bochs
blockdevice_fd0_size => 0
blockdevice_sr0_model => QEMU DVD-ROM
blockdevice_sr0_size => 1073741312
blockdevice_sr0_vendor => QEMU
blockdevice_vda_size => 10737418240
... Output omitted ...
```

2. 什么 facter 命令将显示系统的主机名？

```
[root@servera ~]# facter hostname
servera
[root@servera ~]# facter fqdn
servera.lab.example.com
```

3. 您要使用哪一 facter 命令来显示系统的 IP 地址？

```
[root@servera ~]# facter ipaddress
172.25.250.10
```

4. facter 命令在由不同的用户使用时，会显示不同的信息。

a. 以 root 身份且不带参数执行 facter，再将输出保存到文件。

```
    [root@servera ~]# facter > /tmp/root.facts
```

b. 以 student 身份且不带参数执行 facter，再将输出保存到文件。

```
    [root@servera ~]# su - student
    [student@servera ~]$ facter > /tmp/student.facts
```

c. 使用 diff 命令比较这两个用户可以显示的系统事实。

```
    [student@servera ~]$ diff /tmp/root.facts /tmp/student.facts
    3,5d2
    < bios_release_date => 01/01/2011
    < bios_vendor => Bochs
    < bios_version => Bochs
    20c17
    < id => root
    ---
    > id => student
    25c22
    < is_virtual => true
    ---
    > is_virtual => false
    32d28
    < manufacturer => Bochs
    34c30
    < memoryfree_mb => 1671.92
    ---
    > memoryfree_mb => 1670.11
    47c43
    < path => /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
    ---
    > path => /usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/student/.local/bin:/home/student/bin
    51d46
    < productname => Bochs
    63d57
    < serialnumber => Not Specified
    75d68
    < type => Other
    80,82c73,74
    < uptime_seconds => 30624
    < uuid => Not Settable
    < virtual => kvm
    ---
    > uptime_seconds => 30637
    > virtual => physical
```

一些区别源自于系统变化（如 memoryfree_mb 和 uptime_seconds），而另一些区别的原因则在于系统层面上访问权限的不同（bios_* 和 is_virtual 事实）。 


### 添加新事实到 Facter

在 Linux 环境中添加新事实

*    Facter 具有提供额外自定义事实的功能。它可以是简单地创建结构化文件或脚本，或者也可像编写自定义 Ruby 代码那样复杂。本节探讨如何利用结构化文件和脚本创建额外的系统事实。
*    Facter 在以下两个目录中寻找额外的事实定义：/etc/facter/facts.d 和 /etc/puppetlabs/facter/facts.d/。定义新事实的结构化文件和脚本应当放入这两个目录中。这些目录并非由 facter 软件包创建的。

###### 结构化文件

Facter 当前支持三种结构化数据文件，即文本、YAML 和 JSON 文件。这些文件的文件名很重要，因为 Facter 使用文件名后缀（分别是 .txt、.yaml 和 .json）来判断文件中包含的结构化数据类型。

最简单的结构化文件是文本文件。它必须以 .txt 扩展名结尾。文件中的每一行包含一个键/值对，格式类似于下方所示：

```
key1=value1
key2=value2
key3=value3
...
keyN=valueN
```

YAML 结构化文件必须以 .yaml 扩展名结尾，并且必须包含有效的 YAML 语法。字符串值包括在单引号或双引号内。以下是 YAML 结构化文件示例：

```
---
key1: 'value1'
key2: 'value2'
key3: 'value3'
...
keyN: 'valueN'
```

JSON 结构化文件必须以 .json 扩展名结尾，并且必须包含有效的 JSON 语法。字符串值包括在双引号内。以下是 JSON 结构化文件示例：

```
{
  "key1": "value1",
  "key2": "value2",
  "key3": "value3",
  ...
  "keyN": "valueN"
}
```

###### 可执行程序

可执行程序能够为 Facter 提供动态计算或收集的动态事实。它们可以任何编程语言编写。facts.d 中的文件必须为可执行，Facter 才能执行并使用它。

通过可执行程序生成的事实必须列出为名称/值对，其格式与文本结构化文件格式一致。这一上下文中不支持 JSON 和 YAML 输出。

下例使用 Bash shell 脚本获取系统事实。本例使用 who 和 wc 命令计算已登录的用户数。定义了一个名为 custom_num_users 的系统事实。

```
[root@host ~]# cat /etc/facter/facts.d/custom
#!/bin/bash
PATH=/bin:/usr/bin
echo "custom_num_users=$(who | wc -l)"
[root@host ~]# chmod a+x /etc/facter/facts.d/custom
[root@host ~]# /etc/facter/facts.d/custom
custom_num_users=1
[root@host ~]# facter custom_num_users
1
```

如下是一个 C 程序，可显示一个静态的名称/值对。编译后生成的可执行文件放在 /etc/facter/facts.d 目录中。虽然示例程序中列出的事实不会更改，但它可以计算并返回运行时值。

```
[root@host ~]# cat custom-hello.c
#include <stdio.h>
 
void main (void)
{
    printf("custom_hello=Hello world!\n");
}
[root@host ~]# gcc -o /etc/facter/facts.d/custom-hello custom-hello.c
[root@host ~]# facter custom_hello
Hello world!
```

###### 故障排除

如果定义的外部事实没有显示出来，可利用 Facter 的调试模式列出有助于诊断问题的参考性消息。使用 --debug 选项调用 facter 可显示这些消息。

下例中显示了结构化文件中没有正确的键/值对定义时发生的情况：

```
[root@host ~]# facter --debug test
Fact file /etc/facter/facts.d/test.txt was parsed but returned an empty
data set
value for lsbdistid is still nil
Not an EC2 host
[root@host ~]# cat /etc/facter/facts.d/test.txt
test-This is a test
```

facter 的 --debug 选项也可帮助识别在输出中生成无效键/值对的可执行程序：

```
[root@host ~]# facter --debug test
Fact file /etc/facter/facts.d/test was parsed but returned an empty
data set
value for lsbdistid is still nil
Not an EC2 host
[root@host ~]# ls -l /etc/facter/facts.d/test
-rwxr-xr-x. 1 root root 37 Aug 24 09:53 /etc/facter/facts.d/test
[root@host ~]# cat /etc/facter/facts.d/test
#!/bin/bash
echo test-This is a test
```


### 练习：添加新事实到 Facter

1. 在 servera 上以 root 身份登录。创建 facter 在其中寻找自定义事实的目录 /etc/facter/facts.d。

```
[root@servera ~]# mkdir -p /etc/facter/facts.d
```

2. 可以利用以 .txt 扩展名结尾的文件来添加自定义事实。

    a. 使用编辑器创建文件 custom.txt，它含有两个新的自定义事实。其中应包含以下内容。

```
    custom_fact1=value one
    custom_fact2=value two
```

    b. 确认新事实可供 facter 使用。

```
    [root@servera ~]# facter | grep custom
    custom_fact1 => value one
    custom_fact2 => value two
```

3. 可以利用以 .yaml 扩展名结尾的文件来添加自定义事实。

    a. 使用编辑器创建文件 custom.yaml，它含有两个新的自定义事实。该文件应当包含以下内容。

```
    ---
    custom_fact3: 'value three'
    custom_fact4: 'value four'
```

    b. 确认新事实可供 facter 使用。

```
    [root@servera ~]# facter | grep custom
    ... Output omitted ...
    custom_fact3 => value three
    custom_fact4 => value four
```

4. 可以利用以 .json 扩展名结尾的文件来添加自定义事实。

    a. 使用编辑器创建文件 custom.json，它含有两个新的自定义事实。该文件应当包含以下内容：

```
    {"custom_fact5": "value five", "custom_fact6": "value six"}
```

    b. 确认新事实可供 facter 使用。

```
    [root@servera ~]# facter | grep custom
    ... Output omitted ...
    custom_fact5 => value five
    custom_fact6 => value six
```

5. 可以利用 shell 程序来添加自定义事实。它们是位于 /etc/facter/facts.d 目录下的可执行文件。

    a. 使用编辑器创建文件 custom，它包含一个新的自定义事实。该事实应当取名为 custom_realtime_seconds，包含以秒为单位表示的当前系统时间。以下 shell 脚本将产生所需的输出。

```
    #!/bin/bash
    echo "custom_realtime_seconds=$(date '+%s')"
```

    b. 使 shell 脚本成为可执行文件。

```
    [root@servera ~]# chmod 755 /etc/facter/facts.d/custom
```

    c. 确认新事实可供 facter 使用。

```
    [root@servera ~]# facter | grep custom
    ... Output omitted ...
    custom_realtime_seconds => 1439819414
```

    d. 等待几秒钟，再显示 custom_realtime_seconds 的值。它应当在每次检查了相关事实时进行计算。

```
    [root@servera ~]# facter | grep realtime
    custom_realtime_seconds => 1439819466
```

### 总结

在本章中，您学到了：

*    facter 命令显示关于主机系统的一组有限事实。
*    可以在 /etc/facter/facts.d 内的结构化文本、YAML 和 JSON 文件中定义额外的系统键/值对。
*    生成键/值对的可执行程序也可以放在 /etc/facter/facts.d 中。 

