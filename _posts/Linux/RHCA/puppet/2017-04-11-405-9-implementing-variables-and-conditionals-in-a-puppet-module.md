---
layout: post
title:  "Implementing Variables and Conditionals in a Puppet Module(405-9)"
categories: Linux
tags: RHCA 405
---


### 实施变量

###### 使用变量

与其他脚本语言中一样，Puppet 变量为值提供存储，以便未来检索使用。在 Puppet DSL 中，变量名称注有 $ 前缀。它们区分大小写，可以包含字母数字字符和下划线。变量值使用 = 运算符分配。

Puppet DSL 提供若干种不同的数据类型，包括布尔值、字符串、数字、数组、哈希和正则表达式。除正则表达式外，变量可以分配任何数据类型的值。

下列变量分配示例中涉及了字符串、数字、数组和哈希。

```
$boolean_var = true
$number_var = 5
$string_var = "Custom message.\n"
$array_var = [ 'a', 'b', 'c' ]
$hash_var = {
 'first'  => 'primary'
 'second' => 'secondary'
 'third'  => 'tertiary'
}
```

在变量分配了值后，可以通过变量名称来检索它的值。Puppet 将执行变量替换，将变量替换为它的值。当变量包含在双引号内时，Puppet 会将它们替换为其值。不过，包含在单引号内的变量视为文字，因而不会被解译。

从变量检索值时，一些上下文需要额外的说明。例如，某一变量名称可能比较模糊，因为没有空白字符将它与跟在后面的字符区隔开。在这样的情形中，变量应当包含在 {} 内，从而清晰地描述变量名称的边界。

```
my $foo = 'name'
my $foobar = 'birthday'

notify { "My $foobar": }
notice { "My ${foo}bar": }
```

在编程中，术语变量指的是其内容可以更改或可变的存储。术语常数指的是其内容保持不变的存储。在 Puppet 中，术语变量稍具误导性。一旦 Puppet 变量被分配了值，就无法在原始变量分配发生的同一范围内重新分配值。 

###### 理解变量范围

与其他编程语言中类似，范围指的是程序中一个变量保持有效的代码部分。变量在它被定义的范围内有效，也在任何子范围内有效。因此，父范围内定义的变量内容可以在子范围内访问，反之则不然。在子范围中定义的变量无法从其父范围内检索它的内容。

在 Puppet 中，顶级范围是所有范围的父级。在这一范围中定义的任何变量可以从所有范围访问，因为所有范围都是顶级范围的子级。

顶级范围内的下一级范围是节点范围。节点范围由节点定义创建，后者会在本课程的后续部分中更详细阐述。由于各个节点仅匹配一个节点定义，因此只能有一个节点范围。

```
# site.pp

$top_var = 'TOP'

node 'sandbox.example.com' {
  $node_var = "NODE"
  notify {"Node view of node: $node_var":}
  notify {"Node view of top: $top_var":}
}

notify {"Top view of node: $node_var":}

[student@sandbox] $ puppet apply site.pp
notice: Node view of node: NODE
notice: Node view of top: TOP
notice: Top view of node:
```

节点范围内包含一个或多个类范围，它们与定义的各个 Puppet 类相关。在每个类范围内，可以访问顶级范围和节点范围中的变量。但是，不能访问在其他类范围中定义的变量。

虽然变量只能在一个范围内分配一次，此限制不适用于不同的范围。某一范围从父范围继承的变量可以重新分配新的值。

###### 内置变量

上例围绕的是自定义变量。除了自定义变量外，Puppet 也提供一些预定义变量。这些变量称为内置变量，由 facter 创建，并且填充它所发现的事实。

这些变量在顶级范围中定义，因此能从 Puppet 清单内的任何位置访问。这些内置变量不仅包括 Puppet 默认附带的变量（称为核心事实），也包括用户定义的自定义事实。

内置变量的命名方式与其他变量没有区别。这使得它难以和用户定义的变量相区分。另外，由于内置变量在顶级范围内定义，它们有可能会在子范围中被覆盖，即子范围中定义了名称相同的变量时。若要避免这样的问题，一种办法是对内置变量名称使用全完限定的语法。使用变量名称 $::FACTNAME 而非 $FACTNAME；这样，就能从任何范围专门引用内置变量，并使它们与用户定义的变量区分开来。 

### 练习：实施变量

1. 首先在 Puppet 清单中定义一个变量，并且使用它一次。

    a. 创建名为 vars.pp 的 Puppet 清单。它应当定义变量 $dir，这是系统上需要存在的一个目录的路径。应当定义一个 file 资源，使用该 $dir 变量。该清单应当类似于下方所示内容：

```
    $dir = '/var/tmp/example'
     
    file { "$dir":
      ensure => 'directory',
      mode   => '755',
    }
```

    b. 应用 vars.pp 清单。创建的新目录按照 $dir 变量的值进行命名。

```
    [root@servera labwork]# puppet apply vars.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.19 seconds
    Notice: /Stage[main]/Main/File[/var/tmp/example]/ensure: created
    Notice: Finished catalog run in 0.05 seconds
    [root@servera labwork]# ls -ld /var/tmp/example
    drwxr-xr-x. 2 root root 6 Sep  8 15:23 /var/tmp/example
```

    c. 将 vars.pp 与描述性消息一起提交到本地 Git 存储库。

```
    [root@servera labwork]# git add vars.pp
    [root@servera labwork]# git commit -m 'Initial version.'
    ... Output omitted ...
```

2. Puppet 变量具有重要价值，因为它们只需定义一次就可多次用于同一清单内的多个相关对象。

    a. 再将几个 file 资源添加到 vars.pp，以管理一个名为 /var/tmp/example/public.txt 的全局可读文件和名为 /var/tmp/example/secret.txt 的私密文件。务必尽可能使用 $dir 变量。生成的清单应当类似于下方所示内容：

```
    ... Output omitted ...
     
    file { "$dir/public.txt":
      ensure  => 'file',
      mode    => '644',
      content => "Public content\n",
    }
     
    file { "$dir/secret.txt":
      ensure  => 'file',
      mode    => '600',
      content => "Secret content\n",
    }
```

    b. 应用该 Puppet 清单，并查看它对文件系统进行的调整。验证新文件的内容。

```
    [root@servera labwork]# puppet apply vars.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.20 seconds
    Notice: /Stage[main]/Main/File[/var/tmp/example/public.txt]/ensure: defined
     content as '{md5}5ebb85561c44250e747fc25bda1f94cd'
    Notice: /Stage[main]/Main/File[/var/tmp/example/secret.txt]/ensure: defined
     content as '{md5}9943fb5ded3b0a575b6c33e7ce304b9d'
    Notice: Finished catalog run in 0.08 seconds
    [root@servera labwork]# ls -l /var/tmp/example
    total 8
    -rw-r--r--. 1 root root 15 Sep  8 15:24 public.txt
    -rw-------. 1 root root 15 Sep  8 15:24 secret.txt
    [root@servera labwork]# more /var/tmp/example/*
    ::::::::::::::
    /var/tmp/example/public.txt
    ::::::::::::::
    Public content
    ::::::::::::::
    /var/tmp/example/secret.txt
    ::::::::::::::
    Secret content
```

    c. 将更改与描述性日志消息一起提交到本地 Git 存储库。

```
    [root@servera labwork]# git add vars.pp
    [root@servera labwork]# git commit -m 'Added file resources that use variables.'
    ... Output omitted ...
```

3. Puppet 变量只能分配一次值。

    a. 将另一行添加到 vars.pp，紧接在最后一个 file 资源的前面，它将尝试更改 $dir 的值。生成的清单应当类似于下方所示：

```
    ... Output omitted ...
     
    file { "$dir/public.txt":
      ensure  => 'file',
      mode    => '644',
      content => "Public content\n",
    }
     
    $dir = '/tmp/example'
     
    file { "$dir/secret.txt":
      ensure  => 'file',
      mode    => '600',
      content => "Secret content\n",
    }
```

    b. 尝试应用更新后的清单。

```
    [root@servera labwork]# puppet apply vars.pp
    Error: Cannot reassign variable dir at /root/labwork/vars.pp:14 on node
     servera.lab.example.com
    Error: Cannot reassign variable dir at /root/labwork/vars.pp:14 on node
     servera.lab.example.com
```

    puppet apply 命令会失败，因为存在第二个变量定义。

    c. 使用 Git 丢弃更改，并将清单恢复到原先正常工作的状态。

```
    [root@servera labwork]# git checkout -- vars.pp
```

4. Facter 提供的系统事实是可以被引用的变量。

    a. 修改 Puppet 清单，使它将 osfamily 系统事实的值放入私密文件 secret.txt 中。字符串应当用双引括起，以便能识别变量引用。


```
    ... Output omitted ...
     
    file { "$dir/secret.txt":
      ensure  => 'file',
      mode    => '600',
      content => "The osfamily for this system is $::osfamily.\n",
    }
```

    b. 使用 facter 命令显示 osfamily 系统事实的值。

```
    [root@servera labwork]# facter osfamily
    RedHat
```

    c. 应用该清单，并显示 /var/tmp/example/secret.txt 文件的新内容。

```
    [root@servera labwork]# puppet apply vars.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.20 seconds
    Notice: /Stage[main]/Main/File[/var/tmp/example/secret.txt]/content:
     content changed '{md5}9943fb5ded3b0a575b6c33e7ce304b9d' to
     '{md5}55636325d2aa3adc5644bb1a4785ec82'
    Notice: Finished catalog run in 0.10 seconds
    [root@servera labwork]# cat /var/tmp/example/secret.txt
    The osfamily for this system is RedHat.
```

5. 当变量前后紧接文本时，其名称必须用大括号 ({ }) 分隔开。

    a. 更改 Puppet 清单，使 public.txt 文件的内容在一连串文本中使用 $dir 变量。务必将变量名称括在大括号内。

```
    ... Output omitted ...
    file { "$dir/public.txt":
      ensure  => 'file',
      mode    => '644',
      content => "Here is a variable${dir}immediately enclosed in text.\n",
    }
    ... Output omitted ...
```

    b. 应用该清单，并观察它对 /var/tmp/example/public.txt 进行的更改。

```
    [root@servera labwork]# puppet apply vars.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.19 seconds
    Notice: /Stage[main]/Main/File[/var/tmp/example/public.txt]/content:
     content changed '{md5}5ebb85561c44250e747fc25bda1f94cd' to
     '{md5}bce844cb785b7ddb157e180bdb933515'
    Notice: Finished catalog run in 0.09 seconds
    [root@servera labwork]# cat /var/tmp/example/public.txt
    Here is a variable/var/tmp/exampleimmediately enclosed in text.
```

    c. 将更改与描述性日志消息一起提交到本地 Git 存储库。

```
    [root@servera labwork]# git add vars.pp
    [root@servera labwork]# git commit -m 'Use facts and {}s.'
    ... Output omitted ...
```


### 实施条件句

###### 使用条件句

虽然 Puppet 清单声明了所需的系统最终状态，但它们并不指明这些状态是如何达到的。其原因在于，为了让 Puppet 能够成为跨平台工具，它需要在不同系统上执行不同的操作来取得同样的结果。例如，虽然 RHEL 6 和 RHEL 7 系统上都可运行网络时间服务器，但前者利用 ntpd 服务来实现，而后者则可利用 chronyd 服务来实现。

在大型企业环境中，可能有许多不同的系统通过 Puppet 管理其配置。系统之间的差别可能需要使用稍有差异的 Puppet 代码来实现相同的配置目标。尽管可以为不同的系统实施不同的代码集合，但更可取的是将决策制定逻辑实施到代码中以容纳这些差别。这样，既可以实现代码重复利用，还可以在组织的 Puppet 基础架构壮大时简化 Puppet 代码管理。

与许多编程语言类似，Puppet DSL 为管理员提供在代码中添加条件语句的功能。顾名思义，条件语句可以针对不同的条件执行不同的代码。对于 Puppet 配置管理，这些条件通常基于从 Puppet 事实派生的系统信息。 

######### 条件运算符

条件表达式通过利用运算符和运算对象创建。最常见的运算符类型是比较运算符。比较运算符可以评估字符串或数值。

下表列出了常用于字符串比较的一些运算符。

*   运算符 	描述
*   == 	测试两个字符串是否相同。
*   != 	测试两个字符串是否不同。
*   in 	测试右侧运算对象是否包含左侧运算对象。最常用于比较左侧字符串运算对象和右侧数组运算对象，来查看字符串是否与数组中的某一元素匹配。

下表列出了常用于数字比较的一些运算符。

*   运算符 	描述
*   == 	测试两个数字是否相等。
*   != 	测试两个数字是否不等。
*   < 	测试左侧运算对象是否小于右侧运算对象。
*   <= 	测试左侧运算对象是否小于或等于右侧运算对象。
*   > 	测试左侧运算对象是否大于右侧运算对象。
*   >= 	测试左侧运算对象是否大于或等于右侧运算对象。

可以利用 and 和 or 运算符，将多个条件表达式组合在一起以创建复合表达式。and 运算符测试它评估的运算对象条件表达式是否都为 true。or 运算符测试它评估的运算对象条件表达式中是否有任何一个为 true。 

###### 使用 if 语句

Puppet 提供多种类型的条件语句，各自适合不同的场景。最基本的类型是经典的 if 语句。最简单形式的 if 语句在某一条件评估为 true 时执行一个代码块。

```
if $::is_virtual == 'true' {
... Output omitted ...
}
```

if 语句也可以通过 else 关键字扩展，来指定一个条件为 true 时的一组操作和该条件为 false 时的另一组操作。

```
if $::is_virtual == 'true' {
... Output omitted ...
} else {
... Output omitted ...
}
```

最后，还可以使用 elsif 关键字将 if 语句扩展为测试多个条件。这一结构允许针对每一条件执行不同的操作，并在这些条件中无一被证明为 true 时执行一个综合代码块。

```
if $::is_virtual == 'true' {
... Output omitted ...
} elsif $::is_virtual == 'false' {
... Output omitted ...
} else {
... Output omitted ...
}
```

###### 使用 unless 语句

if 语句主要用于在某一条件为 true 时执行一组操作，但经常也会有需要在相反情景（即条件为 false 时）执行代码的时候。对于这样的情形，Puppet 提供了 unless 语句。

如果测试 fasle 条件比使用 if 语句更为简单且更有效率，则应当使用 unless 语句。

下列示例演示了一种情景，其中使用 unless 语句比使用 if 语句在编程上更有效率，且更容易理解。

```
unless $memorysize >= 2048 {
... Output omitted ...
}
```


####### 使用 case 语句

虽然 if 语句可以通过一个或多个 elsif 关键字来扩展，从而允许对不同的条件执行不同的代码，但过度使用 elsif 关键字会使代码难以读懂。Puppet 提供了 case 语句，作为这些情形中的一种替代条件语句。

使用 case 语句时，针对控制表达式评估一组值。当某一个值针对控制表达式评估为 true 时，执行与它对应的代码块，然后退出 case 语句。

if 语句中的 else 关键字允许在这些条件中无一被评估为 true 时执行代码块。类似地，case 语句提供一个 default 值，如果前面所有值都没能针对控制表达式评估成功，它充当一个综合值。

```
case $operatingsystem {
  /Linux/:     { notify 'Powered by Linux.': }
  'Solaris':   { notify 'Powered by Solaris.': }
  default:     { notify 'Welcome.': }
}
```

###### 使用选择器语句

Puppet 提供的另一种条件语句是选择器语句。与 case 语句类似，针对选择器语句中的控制表达式评估一组值。不过，在某一个值针对控制表达式评估为 true 时，不执行代码块，而是返回一个值。

选择器语句由多个组件组成：控制变量、条件和值。控制变量针对各个条件进行评估。当条件针对控制变量评估为 true 时，返回其对应的值。类似于 case 语句，选择器语句也提供一个 default 条件，如果前面所有条件都没能针对控制变量评估成功，它充当一个综合条件。

```
$control_variable ? {
  'case_1' => 'return_value_1',
  'case_2' => 'return_value_2',
  default  => 'default_return_value',
}
```

鉴于其性质，选择器语句在变量分配逻辑根据特定条件而有变化的情形中使用效果最好。下例演示了如何将选择器语句用于变量分配。

```
$message = $operatingsystem ? {
  /Linux/      => 'Powered by Linux.',
  'Solaris'    => 'Powered by Solaris.',
  default      => 'Welcome.',
}
```


### 引导式练习：实施条件句

1. 首先观察如何使用选择器来基于表达式的值返回不同的值。

    a. 创建一个名为 condition.pp 的清单，它根据选择器设置变量 $disk 的值。选择器将使用 is_virtual 系统事实来决定要返回的值。

    使用 notify 资源来显示 $disk 的最终值。生成的清单应当类似于下方所示：

```
    $disk = $::is_virtual ? {
      'true'  => '/dev/vda',
      default => '/dev/sda',
    }
     
    notify { 'disk':
      message => "value is $disk.",
    }
```

    b. 使用 facter 命令显示 is_virtual 事实的值。

```
    [root@servera labwork]# facter is_virtual
    true
```

    c. 应用清单，并且确认选择器返回了 /dev/vda，这个值就是分配给 $disk 变量的值。

```
    [root@servera ~]# puppet apply condition.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: value is /dev/vda.
    Notice: /Stage[main]/Main/Notify[disk]/message: defined 'message' as
     'value is /dev/vda.'
    Notice: Finished catalog run in 0.05 seconds
```

2. 使用 if 条件句在清单中可选地应用 file 资源。

    a. 扩展 condition.pp，使其类似于下方所示：

```
    $disk = $::is_virtual ? {
      'true'  => '/dev/vda',
      default => '/dev/sda',
    }
     
    notify { 'disk':
      message => "value is $disk.",
    }
     
    if $::is_virtual {
      file { '/tmp/virtual.txt':
        ensure  => 'file',
        content => "This is a virtual system. Disk is ${disk}.\n",
      }
    }
```

    b. 使用 puppet apply 命令应用清单。由于 is_virtual 事实评估为 true，因此应用 if 条件中定义的资源并且创建文件。

```
    [root@servera ~]# puppet apply condition.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.20 seconds
    Notice: /Stage[main]/Main/File[/tmp/virtual.txt]/ensure: defined content
     as '{md5}815b89d5a2775cce3d5e9ba12e832544'
    Notice: value is /dev/vda.
    Notice: /Stage[main]/Main/Notify[disk]/message: defined 'message' as
     'value is /dev/vda.'
    Notice: Finished catalog run in 0.05 seconds
    [root@servera ~]# cat /tmp/virtual.txt
    This is a virtual system. Disk is /dev/vda.
```

3. 更改为 unless 条件句，以颠倒 if 条件句的逻辑。

    a. 将 condition.pp 修改为类似于下方所示：

```
    $disk = $::is_virtual ? {
      'true'  => '/dev/vda',
      default => '/dev/sda',
    }

    notify { 'disk':
      message => "value is $disk.",
    }

    unless $::is_virtual {
      file { '/tmp/physical.txt':
        ensure  => 'file',
        content => "This is a physical system. Disk is ${disk}.\n",
      }
    }
```
    
    注意文件的名称，并且其内容已有更改以反映逻辑的反向性质。

    b. 应用 Puppet 清单。没有创建 /tmp/physical.txt 文件，因为使用了 unless 条件的表达式为 true，因此跳过了相关的代码。

```
    [root@servera ~]# puppet apply condition.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: value is /dev/vda.
    Notice: /Stage[main]/Main/Notify[disk]/message: defined 'message' as
     'value is /dev/vda.'
    Notice: Finished catalog run in 0.05 seconds
    [root@servera ~]# ls /tmp/physical.txt
    ls: cannot access /tmp/physical.txt: No such file or directory
```

4. 观察 if 条件句如何与 else 子句搭配。

    a. 修改 condition.pp 清单。将 unless 条件句改回到 if 条件句，再添加一个 else 子句，使清单类似于下方所示：

```
    $disk = $::is_virtual ? {
      'true'  => '/dev/vda',
      default => '/dev/sda',
    }
     
    notify { 'disk':
      message => "value is $disk.",
    }
     
    if $::is_virtual {
      file { '/tmp/virtual.txt':
        ensure  => 'file',
        content => "This is a virtual system. Disk is ${disk}.\n",
      }
    }
    else {
      file { '/tmp/physical.txt':
        ensure  => 'file',
        content => "This is a physical system. Disk is ${disk}.\n",
      }
    }
```

    b. 删除 /tmp 中以 .txt 结尾的所有文件，清理前面的 Puppet 操作。应用清单，并确认已创建了 virtual.txt。

```
    [root@servera ~]# rm -f /tmp/*.txt
    [root@servera ~]# puppet apply condition.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.20 seconds
    Notice: /Stage[main]/Main/File[/tmp/virtual.txt]/ensure: defined content
     as '{md5}815b89d5a2775cce3d5e9ba12e832544'
    Notice: value is /dev/vda.
    Notice: /Stage[main]/Main/Notify[disk]/message: defined 'message' as
     'value is /dev/vda.'
    Notice: Finished catalog run in 0.05 seconds
```

    由于条件为 true，因此应用了第一个子句，并且跳过了 else 子句。

5. 实施根据 osfamily 系统事实的值进行分叉的 if-elsif-else 条件。

    a. 创建名为 os-check.pp 的清单，它实施下列 Puppet DSL：

```
    if $::osfamily == 'RedHat' or $::osfamily == 'CentOS' {
      notify { "This is a Red Hat family system." : }
    }
    elsif $::osfamily == 'Solaris' {
      notify { "This is a Solaris system." : }
    }
    else {
      notify { "Not sure what OS family this system is." : }
    }
```

    b. 使用 facter 命令显示 osfamily 事实的值，然后应用清单。由于第一个条件评估为 true，因此它将实施 notify 资源，并且跳过该条件的其余子句。

```
    [root@servera ~]# facter osfamily
    RedHat
    [root@servera ~]# puppet apply os-check.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: This is a Red Hat family system.
    Notice: /Stage[main]/Main/Notify[This is a Red Hat family system.]/
     message: defined 'message' as 'This is a Red Hat family system.'
    Notice: Finished catalog run in 0.05 seconds
```

6. 利用 case 语句而不使用 if-elsif-else 语句，实施上一条件句。

    a. 将 os-check.pp 清单修改为类似于下方所示：

```
    case $::osfamily {
      'RedHat', 'CentOS': {
        notify { "This is a Red Hat family system." : }
      }
      'Solaris': {
        notify { "This is a Solaris system." : }
      }
      default: {
        notify { "Not sure what OS family this system is." : }
      }
    }
```

    b. 应用该清单时，它将显示同样的通知，因为 case 条件句的逻辑和原先的代码一致。

```
    [root@servera ~]# puppet apply os-check.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: This is a Red Hat family system.
    Notice: /Stage[main]/Main/Notify[This is a Red Hat family system.]/
     message: defined 'message' as 'This is a Red Hat family system.'
    Notice: Finished catalog run in 0.05 seconds
```


### 实施正则表达式


正则表达式通常也被称为其缩写形式 regexp，是用于定义搜索模式的一系列字符。在编程中，正则表达式主要用于字符串模式匹配。

Puppet DSL 提供了正则表达式功能。用户通过这些功能，可以实施用于字符串比较的条件表达式，这与基本的等式评估相比，功能更强、灵活性更高。

###### 正则表达式匹配运算符

Puppet 提供 2 个用于正则表达式的运算符。

*    =~ 运算符用于评估字符串是否与正则表达式匹配。
*    !~ 运算符执行的刚好相反，它评估字符串是否与正则表达式不匹配。

=~ 和 !~ 正则表达式匹配运算符都针对正则表达式模式评估字符串。Puppet 正则表达式用 / 字符括起。这种用法的语法遵循以下格式：被评估的字符串放在运算符的左侧，而使用的正则表达式则放在运算符的右侧。

```
if $hostname =~ /myhost/ {
  notify { 'Matched myhost' : }
}

if $hostname !~ /myhost/ {
  notify { 'Did not match myhost' : }
}
```

在使用 =~ 运算符时，如果字符串与正则表式匹配，则字符串比较将评估为 true，反之亦然。在使用 !~ 运算符时，如果字符串与正则表式匹配，则字符串比较将评估为 false，反之亦然。

正则表达式匹配运算符通常用在 if 语句的条件表达式中。不过，它们也可在选择器和 case 语句中用于评估条件。

$message = $operatingsystem ? {
  /Linux/      => 'Powered by Linux.',
  'Solaris'    => 'Powered by Solaris.',
  default      => 'Welcome.',
}

case $operatingsystem {
  /Linux/:     { notify 'Powered by Linux.': }
  'Solaris':   { notify 'Powered by Solaris.': }
  default:     { notify 'Welcome.': }
}

###### 正则表达式语法

Puppet 采用标准的 Ruby 正则表达式。因此，正则表达式对某一 Puppet 安装是否有效取决于它所附带的 Ruby 版本。

与其他编程语言中使用的正则表达式类似，Puppet 的正则表达式也利用了一组元字符。这些元字符在正则表达式内具有特殊含义。因此，如果要在字面意义上匹配元字符，它必须使用反斜杠转义。

下表中列出了 Puppet 正则表达式中用于匹配单字符的元字符。

常用的字符匹配元字符

*    元字符 	描述
*    . 	    匹配单个字符。
*    [ ] 	匹配方括号内的单个字符。
*    [^ ] 	匹配不包含在方括号内的单个字符。
*    \w 	匹配单个字母数字字符。
*    \W 	匹配单个非字母数字字符。
*    \d 	匹配单个数字字符。
*    \D 	匹配单个非数字字符。
*    \s 	匹配单个空格字符。
*    \S 	匹配单个非空格字符。

在匹配字符串时，通常需要匹配一系列字符串。当字符序列由相同字符或同类字符组成时，可以使用重复修饰符来扩展正则表达式。

下表中列出了 Puppet 正则表达式中用于指示字符重复的常用修饰符。

常用重复修饰符

*    修饰符 	描述
*    * 	    将前一个元素匹配零次或多次。
*    ? 	    将前一个元素匹配零次或一次。
*    + 	    将前一个元素匹配一次或多次。
*    {x} 	将前一个元素匹配恰好 x 次。
*    {x,} 	将前一个元素匹配 x 次或更多次。
*    {,y} 	将前一个元素匹配 y 次或更少次。
*    {x,y} 	将前一个元素匹配至少 x 次，但不超过 y 次。

如果字符串中的任意位置上存在相应字符，字符匹配正则表达式将对字符串进行匹配。有时候，需要仅在正则表达式匹配字符串中指定位置时才将匹配评估为 true。

下表中列出了定位匹配 Puppet 正则表达式中常用的元字符列表。

常用定位元字符

*    元字符 	描述
*    ^ 	    匹配行首。
*    $ 	    匹配行尾。 


### 练习：实施正则表达式

1. 使用 facter 命令显示 ipaddress 系统事实的值。

```
[root@servera labwork]# facter ipaddress
172.25.250.10
```

我们将在条件表达式中用它查看可以利用哪些正则表达式来进行匹配。 

2. == 运算符检查字符完全匹配。

    a. 创建名为 regex.pp 的清单，它包含有下列 Puppet DSL：

```
    if $::ipaddress == '172.25.250.11' {
      notify { "Matched." : }
    }
```

    b. 应用该清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Finished catalog run in 0.05 seconds
```

    notify 资源没有应用，因为系统事实不完全匹配该字符串。

3. 使用逻辑运算符生成更复杂的表达式，它将匹配当前的主机以及应当要匹配的原始主机。

    a. 如果它参与的任一表达式为 true，则 or 逻辑运算符将评估为 true。添加与 172.25.250.10 的等效性比较将使表达式在 servera 上匹配。

```
    if $::ipaddress == '172.25.250.10' or $::ipaddress == '172.25.250.11' {
      notify { "Matched." : }
    }
```

    b. 应用 regex.pp 清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Matched.
    Notice: /Stage[main]/Main/Notify[Matched.]/message: defined 'message'
     as 'Matched.'
    Notice: Finished catalog run in 0.05 seconds
```

4. 将 regex.pp 修改为使用正则表达式而非逻辑表达式来匹配 IP 地址。

    a. =~ 运算符将对正则表达式进行匹配，它包括在斜杠 (/ /) 内。

```
    if $::ipaddress =~ /172\.25\.250\.1[01]/ {
      notify { "Matched." : }
    }
```

    b. 应用更新后的 regex.pp 清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Matched.
    Notice: /Stage[main]/Main/Notify[Matched.]/message: defined 'message'
     as 'Matched.'
    Notice: Finished catalog run in 0.05 seconds
```

    句点匹配正则表达式中的任意字符。它的前面必须加上反斜杠，才能被视文字字符。[01] 匹配一个字符，可以是零或一。 

5. 将 regex.pp 修改为使用不同的正则表达式来匹配 IP 地址。

    a. \d 表达式将匹配任何一位数字。加号是一个修饰符，它将匹配其前面的字符（本例中为数字）一次或多次。

```
    if $::ipaddress =~ /172\.25\.250.\d+/ {
      notify { "Matched." : }
    }
```

    b. 应用更新后的清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Matched.
    Notice: /Stage[main]/Main/Notify[Matched.]/message: defined 'message'
     as 'Matched.'
    Notice: Finished catalog run in 0.07 seconds
```

6. 在原始正则表达式中，插入另一个匹配字符串 ([a-z]*) 的正则表达式。

    a. 生成的清单应当类似于下方所示：

```
    if $::ipaddress =~ /172\.25\.250\.[a-z]*\d/ {
      notify { "Matched." : }
    }
```

    b. 应用更新后的清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Matched.
    Notice: /Stage[main]/Main/Notify[Matched.]/message: defined 'message'
     as 'Matched.'
    Notice: Finished catalog run in 0.05 seconds
```

    该正则表达式依然匹配 IP 地址。这是因为，星号代表零个或多个它前面的字符。在本例中，字母字符是可选的。 

7. 将星号更改为加号修饰符。

    a. 这将匹配一个或多个字母字符，使得 IP 地址不再匹配。将 regex.pp 修改为类似于下方所示。

```
    if $::ipaddress =~ /172\.25\.250\.[a-z]+\d/ {
      notify { "Matched." : }
    }
```

    b. 应用更新后的清单。

```
    [root@servera labwork]# puppet apply regex.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Finished catalog run in 0.05 seconds
```

8. 检查代表空白和行锚点的特殊字符在 Puppet 正则表达式中的工作方式。

    a. 创建一个名为 redhat.pp 的清单，以定义一个名为 $os_name 的变量。变量的原始值应当是字符串 “Red Hat”。文件的内容应当类似于下方所示。

```
    $os_name = 'Red Hat'
     
    if $os_name =~ /RedHat/ {
      notify { 'RedHat matched.' : }
    }
     
    if $os_name =~ /Red\sHat/ {
      notify { 'Red\sHat matched.' : }
    }
     
    if $os_name =~ /Red\s*Hat/ {
      notify { 'Red\s*Hat matched.' : }
    }
     
    if $os_name =~ /Hat/ {
      notify { 'Hat matched.' : }
    }
     
    if $os_name =~ /^Hat/ {
      notify { '^Hat matched.' : }
    }
     
    if $os_name =~ /Hat$/ {
      notify { 'Hat$ matched.' : }
    }
```

    b. 应用该清单，以查看匹配的正则表达式。

```
    [root@servera labwork]# puppet apply redhat.pp
    Notice: Compiled catalog for servera.lab.example.com in environment
     production in 0.04 seconds
    Notice: Hat$ matched.
    Notice: /Stage[main]/Main/Notify[Hat$ matched.]/message: defined 'message'
     as 'Hat$ matched.'
    Notice: Red\s*Hat matched.
    Notice: /Stage[main]/Main/Notify[Red\s*Hat matched.]/message: defined
     'message' as 'Red\s*Hat matched.'
    Notice: Hat matched.
    Notice: /Stage[main]/Main/Notify[Hat matched.]/message: defined 'message'
     as 'Hat matched.'
    Notice: Red\sHat matched.
    Notice: /Stage[main]/Main/Notify[Red\sHat matched.]/message: defined
     'message' as 'Red\sHat matched.'
    Notice: Finished catalog run in 0.05 seconds
```

9. 修改 redhat.pp，并且删除字符串常数中的空格。再次应用该清单，并查看与修改后字符串匹配的表达式。

```
$os_name = 'RedHat'
 
... Output omitted ...
[root@servera labwork]# puppet apply redhat.pp
Notice: Compiled catalog for servera.lab.example.com in environment
 production in 0.04 seconds
Notice: RedHat matched.
Notice: /Stage[main]/Main/Notify[RedHat matched.]/message: defined
 'message' as 'RedHat matched.'
Notice: Hat$ matched.
Notice: /Stage[main]/Main/Notify[Hat$ matched.]/message: defined 'message'
 as 'Hat$ matched.'
Notice: Red\s*Hat matched.
Notice: /Stage[main]/Main/Notify[Red\s*Hat matched.]/message: defined
 'message' as 'Red\s*Hat matched.'
Notice: Hat matched.
Notice: /Stage[main]/Main/Notify[Hat matched.]/message: defined 'message'
 as 'Hat matched.'
Notice: Finished catalog run in 0.05 seconds
```


### 实验：在 Puppet 模块中实施变量和条件句

1. 以 root 身份登录 servera，再创建一个名为 /etc/puppet/manifests/myenv.pp 的 Puppet 清单。

必要时，先创建 /etc/puppet/manifests 目录。

```
[root@servera ~]# mkdir -p /etc/puppet/manifests
[root@servera ~]# vi /etc/puppet/manifests/myenv.pp
```

2. 在清单中，定义用于指定用户的变量 $username。此用户将是由这一清单管理的文件的默认所有者和组所有者。将以下行添加到 myenv.pp：

```
$username = 'student'

File { 
  owner    => "$username",
  group    => "$username",
}
```

3. 添加一个条件语句来设置名为 $homedir 的变量，它用于存储 $username 的主目录。root 用户的主目录是 /root。其他用户的主目录是 /home/USERNAME. 将以下行添加到 myenv.pp。务必用双引号括起变量引用处。

```
if $username == 'root' {
 
  $homedir = "/$username"
 
} else {
 
  $homedir = "/home/$username"
}
```

4. 添加 file 资源，以管理用户主目录中的 .vimrc 文件和 .vim 目录。.vim 目录应当仅存在即可。.vimrc 文件应当至少包含以下指令：

```
set ai sw=2
```

将以下行添加到 myenv.pp。使用 $username 变量来统一引用用户的主目录。

```
file { "/$homedir/.vimrc":
  ensure   => 'file',
  content  => "set ai sw=2\n",
}
 
file { "/$homedir/.vim":
  ensure   => 'directory',
}
```

5. 添加一个条件语句，以管理非 root 用户的 ~/.ssh/authorized_keys。authorized_keys 文件应当包含 servera 上 student 用户的公共 SSH 密钥。

使用 ssh-copy-id 将公共密钥发布到 authorized_keys 文件。

```
[student@servera ~]$ ssh-copy-id localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is 11:92:7e:64:b5:4a:c7:94:1e:ea:c6:62:12:d7:2c:5a.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to
filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are
prompted now it is to install the new keys
student@localhost's password: student
 
Number of key(s) added: 1
 
Now try logging into the machine, with:   "ssh 'localhost'"
and check to make sure that only the key(s) you wanted were added.
```

显示 authorized_keys 文件，再创建 Puppet DSL，它会创建一个与其完全相同的副本到非 root 用户的 authorized_keys 文件中。

```
[student@servera ~]$ cat ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuzEAZkFuIP85vfeg8Izpu...
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDkXaKlM/8F+b7QIiyS3cRT...
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSBHmVyih/hBbjOcGUGw5y...
```

放置 file 资源的绝佳位置是您之前定义的条件语句中。务必使 Puppet 先创建 .ssh 目录。生成的 Puppet DSL 应当类似于下方所示：

```
if $username == 'root' {
 
  $homedir = "/$username"
 
  notify { 'No SSH authorized_keys for root': }
 
} else {
 
  $homedir = "/home/$username"
 
  file { "/$homedir/.ssh":
    ensure   => 'directory',
    mode     => '700',
  }
 
  file { "/$homedir/.ssh/authorized_keys":
    ensure   => 'file',
    content  => ' ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuzEAZkFuIP85vfeg8Izpu...
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDkXaKlM/8F+b7QIiyS3cRT...
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSBHmVyih/hBbjOcGUGw5y...',
    mode     => '600',
  }
}
```

6. 通过更改 $username 变量，使用两个不同的用户 root 和 student 测试此清单。

首先对 root 用户测试该清单。

```
[root@servera ~]# ls -ld ~/.vim ~/.vimrc
[root@servera ~]# 
ls: cannot access /root/.vim: No such file or directory
ls: cannot access /root/.vimrc: No such file or directory
[root@servera ~]# grep 'username = ' /etc/puppet/manifests/myenv.pp
$username = 'root',
[root@servera ~]# puppet apply /etc/puppet/manifests/myenv.pp
Notice: Compiled catalog for servera.lab.example.com in environment production in
0.18 seconds
Notice: No SSH authorized_keys for root
Notice: /Stage[main]/Main/Notify[No SSH authorized_keys for root]/message:
defined 'message' as 'No SSH authorized_keys for root'
Notice: /Stage[main]/Main/File[//root/.vimrc]/ensure: defined content as
'{md5}268b95d07e47f072f35cd5055cd5e8dc'
Notice: /Stage[main]/Main/File[//root/.vim]/ensure: created
Notice: Finished catalog run in 0.06 seconds
[root@servera ~]# ls -ld ~/.vim ~/.vimrc
drwxr-xr-x. 2 root root  6 Aug 28 20:40 /root/.vim
-rw-r--r--. 1 root root 12 Aug 28 20:40 /root/.vimrc
[root@servera ~]# cat ~/.vimrc
set ai sw=2
```

编辑清单，再将 $username 更改为 student。应用清单，确保它执行相应的步骤。

```
[root@servera ~]# ls -ld ~student/.vim ~student/.vimrc ~student/.ssh/authorized_keys
ls: cannot access /home/student/.vim: No such file or directory
ls: cannot access /home/student/.vimrc: No such file or directory
-rw-------. 1 student student 413 Aug 28 20:40 /home/student/.ssh/authorized_keys
[root@servera ~]# grep 'username = ' /etc/puppet/manifests/myenv.pp
$username = 'student'
[root@servera ~]# puppet apply /etc/puppet/manifests/myenv.pp
Notice: Compiled catalog for servera.lab.example.com in environment production in
0.20 seconds
Notice: /Stage[main]/Main/File[//home/student/.vim]/ensure: created
Notice: /Stage[main]/Main/File[//home/student/.vimrc]/ensure: defined
content as '{md5}268b95d07e47f072f35cd5055cd5e8dc'
Notice: /Stage[main]/Main/File[//home/student/.ssh/authorized_keys]/ensure:
defined content as '{md5}16bab87a7a3a8499c752fb20f13201a6'
Notice: Finished catalog run in 0.07 seconds
[root@servera ~]# ls -ld ~student/.vim ~student/.vimrc ~student/.ssh/authorized_keys
-rw-------. 1 student student 413 Aug 28 20:44 /home/student/.ssh/authorized_keys
drwxr-xr-x. 2 student student   6 Aug 28 20:44 /home/student/.vim
-rw-r--r--. 1 student student  12 Aug 28 20:44 /home/student/.vimrc
[root@servera ~]# cat /home/student/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuzEAZkFuIP85vfeg8Izpu
/Up7LQcuKTU8S4XjL/zuH9ZImWM2Z9otxmBqvw2p+Dp6MJpGbOOYPjWY3mUhyY8gvkTEvd
6wSOt2tv2htrGo3h0OpXyIJVs6l04lLRNVvJ/Eqfk3evt3CeiXMlBUtV6E855VVacwwgX+
PWI3uNN37XBJyi8gkyEV96plTPzohabnUKT4pS0uFK8DbP5WQcg9MSwIyRkA5P8gR4/SkY
/occM+RrymtGWs8JHg3qN9DWysz8epBQmRAKFadGG3okDlxhFhOe6U2qeVr0dpVY3Cwy6M
FQXzZcIjGQw6PkWkebEcp7MSo61ZKGnIdvzNTliB student@servera.lab.example.com
[root@servera ~]# cat /home/student/.vimrc
set ai sw=2
```

### 总结

在本章中，您学到了：

*    $var = value 向变量分配一个值。
*    变量可以通过以下任何方式引用：$var、$classname::var、${var} 和 ${classname::var}。
*    Facter 事实是提供系统相关信息的全局变量。
*    if-elsif-else、unless 和 case 语句可用于有条件地执行代码，并将资源应用到 Puppet 清单中。
*    Puppet 中的正则表达式包括在斜杠 (/ /) 内，并使用 =~ 运算符进行匹配。
*    在 Puppet 正则表达式中，\s 匹配空白，\d 则匹配数字。
*    星号和加号为修饰符，它们与紧接在其前面的字符表达式复用。
*    选择符 (? { }) 是根据问号前面的值来提供不同的值的表达式。 
