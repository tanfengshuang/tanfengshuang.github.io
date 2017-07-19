---
layout: post
title:  "Identifying Advanced System Administration Functions in Puppet(405-10)"
categories: Linux
tags: RHCA 405
---

### 识别 Puppet 中的高级系统管理功能

###### 回顾红帽企业 Linux OpenStack 平台中的 Puppet

如前文中所述，红帽企业 Linux OpenStack 平台提供了几个安装程序：packstack、foreman 和 director。它们都使用 Puppet 来安装和配置 OpenStack 服务。

###### 演示：确认从 RHEL-OSP 进行的 Puppet 代码安装

1. 验证这三个软件包显示在 “Installed Packages” 标题之下。

```
[root@workstation ~]# yum list openstack\*puppet\*
... Output omitted ...
Installed Packages
openstack-packstack-puppet.noarch  2015.1-0.11.dev1589.g1d6372f.el7ost @rhel7osp
openstack-puppet-modules.noarch    2015.1.8-8.el7ost                   @rhel7osp
openstack-tripleo-puppet-elements.noarch
                                   0.0.1-4.el7ost                      @rhel7osp
```

2. 如果因任何原因当前没有安装这些软件包，可通过下面这一对命令改正：

```
[root@workstation ~]# yum -y install openstack\*puppet\*
```

###### 回顾 RHEL-OSP 中的 Puppet 代码： config.pp

以下 Puppet 代码负责创建配置 NTP 服务所需的资源。它创建一个或两个文件资源：一个配置文件，以及一个可选的目录。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/manifests/config.pp
#
class ntp::config inherits ntp {

  if $ntp::keys_enable {
    $directory = ntp_dirname($ntp::keys_file)
    file { $directory:
      ensure => directory,
      owner  => 0,
      group  => 0,
      mode   => '0755',
    }
  }

  file { $ntp::config:
    ensure  => file,
    owner   => 0,
    group   => 0,
    mode    => '0644',
    content => template($ntp::config_template),
  }
}
```

此代码创建的第一个资源是用于存储 NTP 加密密钥的目录。只有在 $ntp::keys_enable 变量已设为 true 时才会创建。定义了一个新变量 $directory，以包含要创建的目录的名称。它派生自 $ntp::keys_file 变量的值。调用了 ntp_dirname() 函数，它返回作为其参数传递的路径名的目录部分。

此代码创建的第二个资源是 NTP 配置文件。它始终都会创建，其名称由 $ntp::config 变量的值决定。由 $ntp::config_template 变量指定的模板文件的内容转换为 NTP 配置文件。

ntp::config Puppet 类中所引用的变量从什么地方获取值？继承的 ntp 类可能存放着此问题的答案。 

###### 回顾 RHEL-OSP 中的 Puppet 代码： init.pp

以下 Puppet 代码是定义 ntp 模块的主类。该 Puppet 类的名称与模块相同，在 init.pp 文件中定义。它定义 Puppet 类的参数，并且调用 stdlib 库提供的函数来验证传递到该类的参数。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/manifests/init.pp
class ntp (
  $autoupdate        = $ntp::params::autoupdate,
  $broadcastclient   = $ntp::params::broadcastclient,
  $config            = $ntp::params::config,
  $config_template   = $ntp::params::config_template,
  $disable_auth      = $ntp::params::disable_auth,
  $disable_monitor   = $ntp::params::disable_monitor,
  $fudge             = $ntp::params::fudge,
  $driftfile         = $ntp::params::driftfile,
  $leapfile          = $ntp::params::leapfile,
  $logfile           = $ntp::params::logfile,
  $iburst_enable     = $ntp::params::iburst_enable,
  $keys_enable       = $ntp::params::keys_enable,
  $keys_file         = $ntp::params::keys_file,
  $keys_controlkey   = $ntp::params::keys_controlkey,
  $keys_requestkey   = $ntp::params::keys_requestkey,
  $keys_trusted      = $ntp::params::keys_trusted,
  $minpoll           = $ntp::params::minpoll,
  $maxpoll           = $ntp::params::maxpoll,
  $package_ensure    = $ntp::params::package_ensure,
  $package_manage    = $ntp::params::package_manage,
  $package_name      = $ntp::params::package_name,
  $panic             = $ntp::params::panic,
  $peers             = $ntp::params::peers,
  $preferred_servers = $ntp::params::preferred_servers,
  $restrict          = $ntp::params::restrict,
  $interfaces        = $ntp::params::interfaces,
  $servers           = $ntp::params::servers,
  $service_enable    = $ntp::params::service_enable,
  $service_ensure    = $ntp::params::service_ensure,
  $service_manage    = $ntp::params::service_manage,
  $service_name      = $ntp::params::service_name,
  $stepout           = $ntp::params::stepout,
  $tinker            = $ntp::params::tinker,
  $udlc              = $ntp::params::udlc,
  $udlc_stratum      = $ntp::params::udlc_stratum,
) inherits ntp::params {

  validate_bool($broadcastclient)
  validate_absolute_path($config)
  validate_string($config_template)
  validate_bool($disable_auth)
  validate_bool($disable_monitor)
  validate_absolute_path($driftfile)
  if $logfile { validate_absolute_path($logfile) }
  if $leapfile { validate_absolute_path($leapfile) }
  validate_bool($iburst_enable)
  validate_bool($keys_enable)
  validate_re($keys_controlkey, ['^\d+$', ''])
  validate_re($keys_requestkey, ['^\d+$', ''])
  validate_array($keys_trusted)
  if $minpoll { validate_numeric($minpoll, 16, 3) }
  if $maxpoll { validate_numeric($maxpoll, 16, 3) }
  validate_string($package_ensure)
  validate_bool($package_manage)
  validate_array($package_name)
  if $panic { validate_numeric($panic, 65535, 0) }
  validate_array($preferred_servers)
  validate_array($restrict)
  validate_array($interfaces)
  validate_array($servers)
  validate_array($fudge)
  validate_bool($service_enable)
  validate_string($service_ensure)
  validate_bool($service_manage)
  validate_string($service_name)
  if $stepout { validate_numeric($stepout, 65535, 0) }
  validate_bool($tinker)
  validate_bool($udlc)
  validate_array($peers)

  if $autoupdate {
    notice('autoupdate parameter has been deprecated and replaced with package_ensure.  Set this to latest for the same behavior as autoupdate => true.')
  }

  # Anchor this as per #8040 - this ensures that classes won't float off and
  # mess everything up.  You can read about this at:
  # http://docs.puppetlabs.com/puppet/2.7/reference/lang_containment.html#known-issues
  anchor { 'ntp::begin': } ->
  class { '::ntp::install': } ->
  class { '::ntp::config': } ~>
  class { '::ntp::service': } ->
  anchor { 'ntp::end': }

}
```

ntp 类声明并验证供 ntp::config 类使用的变量。在引用的四个变量中，除 $keys_file（在 config.pp 中引用为 $ntp::keys_file）外，其余都得到验证。

此类调用 validate_absolute_path()、validate_string() 和 validate_bool stdlib 库函数来执行验证。

在被引用时，主要的 ntp 类可以具有类参数分配的值。默认值由类定义中分配的 $ntp::params 值提供。继承的 ntp::params 类定义被引用的变量的默认值。

### 回顾 RHEL-OSP 中的 Puppet 代码： params.pp

以下 Puppet 代码定义 ntp::params 类。此类定义 ntp 类的参数的默认值。此类使用基于系统事实的条件句来确定某些默认值。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/manifests/params.pp
class ntp::params {

  $autoupdate        = false
  $config_template   = 'ntp/ntp.conf.erb'
  $disable_monitor   = false
  $keys_enable       = false
  $keys_controlkey   = ''
  $keys_requestkey   = ''
  $keys_trusted      = []
  $logfile           = undef
  $minpoll           = undef
  $leapfile          = undef
  $package_ensure    = 'present'
  $peers             = []
  $preferred_servers = []
  $service_enable    = true
  $service_ensure    = 'running'
  $service_manage    = true
  $stepout           = undef
  $udlc              = false
  $udlc_stratum      = '10'
  $interfaces        = []
  $disable_auth      = false
  $broadcastclient   = false

  # Allow a list of fudge options
  $fudge             = []

  $default_config       = '/etc/ntp.conf'
  $default_keys_file    = '/etc/ntp/keys'
  $default_driftfile    = '/var/lib/ntp/drift'
  $default_package_name = ['ntp']
  $default_service_name = 'ntpd'

  $package_manage = $::osfamily ? {
    'FreeBSD' => false,
    default   => true,
  }

  if str2bool($::is_virtual) {
    $tinker = true
    $panic  = 0
  }
  else {
    $tinker = false
    $panic  = undef
  }

  case $::osfamily {
    'AIX': {
      $config        = $default_config
      $keys_file     = '/etc/ntp.keys'
      $driftfile     = '/etc/ntp.drift'
      $package_name  = [ 'bos.net.tcp.client' ]
      $restrict      = [
        'default nomodify notrap nopeer noquery',
        '127.0.0.1',
      ]
      $service_name  = 'xntpd'
      $iburst_enable = true
      $servers       = [
        '0.debian.pool.ntp.org',
        '1.debian.pool.ntp.org',
        '2.debian.pool.ntp.org',
        '3.debian.pool.ntp.org',
      ]
      $maxpoll       = undef
    }
... Output omitted ...
    'RedHat': {
      $config          = $default_config
      $keys_file       = $default_keys_file
      $driftfile       = $default_driftfile
      $package_name    = $default_package_name
      $service_name    = $default_service_name
      $restrict        = [
        'default kod nomodify notrap nopeer noquery',
        '-6 default kod nomodify notrap nopeer noquery',
        '127.0.0.1',
        '-6 ::1',
      ]
      $iburst_enable   = false
      $servers         = [
        '0.centos.pool.ntp.org',
        '1.centos.pool.ntp.org',
        '2.centos.pool.ntp.org',
      ]
      $maxpoll         = undef
    }
... Output omitted ...
    default: {
      fail("The ${module_name} module is not supported on an ${::osfamily} based system.")
    }
  }
}
```

$config_template 和 $keys_enable 的默认值先在 Puppet 类中设置。它们在 ntp 类中分别引用为 $ntp::params::config_template 和 $ntp::params::keys_enable。$keys_enable 定义的默认值是 false，因此必须将它覆盖才能使用 NTP 加密密钥。

计算 $ntp::params::config 的值。最初，分配此变量的默认值到 $default_config 变量。类中声明的 case 语句的各个子句中分配它的最终值。case 语句基于 facter 事实 $::osfamily 的值来分叉。

同时也计算 $ntp::params::keys_file 的值。在 AIX 上安装该模块时，它会分配默认值 /etc/ntp.keys。在 Red Hat Enterprise Linux 上，它会分配到 $default_keys_file 变量的值，后者定义为 /etc/ntp/keys。

大部分默认值不会在 Puppet 类定义的资源中引用。相反，它们中的许多会在从模板文件创建 NTP 配置资源时扩展。请思考模板文件的以下部分。

```
[root@workstation ~]# cat /usr/share/openstack-puppet/modules/ntp/templates/ntp.conf.erb
# ntp.conf: Managed by puppet.
#
<% if @tinker == true and (@panic or @stepout) -%>
# Enable next tinker options:
# panic - keep ntpd from panicking in the event of a large clock skew
# when a VM guest is suspended and resumed;
# stepout - allow ntpd change offset faster
tinker<% if @panic -%> panic <%= @panic %><% end %><% if @stepout -%> stepout <%= @stepout %><% end %>
<% end -%>

... Output omitted ...

# Set up servers for ntpd with next options:
# server - IP address or DNS name of upstream NTP server
# iburst - allow send sync packages faster if upstream unavailable
# prefer - select preferrable server
# minpoll - set minimal update frequency
# maxpoll - set maximal update frequency
<% [@servers].flatten.each do |server| -%>
server <%= server %><% if @iburst_enable == true -%> iburst<% end %><% if @preferred_servers.include?(server) -%> prefer<% end %><% if @minpoll -%> minpoll <%= @minpoll %><% end %><% if @maxpoll -%> maxpoll <%= @maxpoll %><% end %>
<% end -%>

... Output omitted ...

<% if @keys_enable -%>
keys <%= @keys_file %>
<% unless @keys_trusted.empty? -%>
trustedkey <%= @keys_trusted.join(' ') %>
<% end -%>
<% if @keys_requestkey != '' -%>
requestkey <%= @keys_requestkey %>
<% end -%>
<% if @keys_controlkey != '' -%>
controlkey <%= @keys_controlkey %>
<% end -%>

<% end -%>
... Output omitted ...
```

如果定义了 $ntp::keys_enable，则配置文件中包含有 keys 指令，指向 $ntp::keys_file 中定义的路径名。$ntp::servers 列表中指定的主机被扩展，从而在生成的 NTP 配置文件中分隔多个 server 行。 


### 总结 

在本章中，您学到了：

*    ntp::config 类定义所需的资源，从而创建由 RHEL-OSP 所用 ntp 服务使用的配置文件。
*    ntp 类在 init.pp 中声明，是 ntp Puppet 模块的入口点；它声明所有的类参数。
*    ntp 类也调用 stdlib 库函数来验证传递到该模块中的参数的数据类型。
*    ntp::params 类基于系统事实定义 ntp 类参数的默认值。 
