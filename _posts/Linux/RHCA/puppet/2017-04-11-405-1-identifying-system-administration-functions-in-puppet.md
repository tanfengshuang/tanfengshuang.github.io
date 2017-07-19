---
layout: post
title:  "Identifying System Administration Functions in Puppet(405-1)"
categories: Linux
tags: RHCA 405
---


### 识别 Puppet 中的系统管理功能

###### 在红帽企业 Linux OpenStack 平台中查找 Puppet

Puppet 可以通过利用自定义声明性语言来定义系统应具备的面貌。系统管理员可以创建这些定义，更好地管理他们的类 Unix 系统和 Microsoft Windows 系统。借助 Puppet 提供的抽象，管理员可以在高层次术语中定义配置，不必关注具体的操作系统级命令。

红帽企业 Linux OpenStack 平台基于 OpenStack 基金会的 OpenStack，提供多个安装程序：

*    packstack—通过 ssh 将 OpenStack 服务部署到预安装的服务器上。
*    foreman—一款 Ruby on Rails 应用，不仅能部署 OpenStack 服务，也可为企业部署添加裸机调配。
*    director—基于 TripleO 配置 director，侧重于部署，但也能为企业配置管理添加升级和更新。


###### 演示：从 RHEL-OSP 安装 Puppet 代码

> 成果: 您应当能够描述如何安装 RHEL-OSP 安装程序（packstack、foreman 和 director），并且识别它们提供的 Puppet 代码

1. 列出现已可用且包含有 Puppet 代码的 OpenStack 软件包。

```
[root@workstation ~]# yum list openstack\*puppet\*
... Output omitted ...
Available Packages
openstack-packstack-puppet.noarch   2015.1-0.11.dev1589.g1d6372f.el7ost rhel7osp
openstack-puppet-modules.noarch     2015.1.8-8.el7ost                   rhel7osp
openstack-tripleo-puppet-elements.noarch
                                    0.0.1-4.el7ost                      rhel7osp
[root@workstation ~]# yum info openstack-packstack-puppet
Loaded plugins: langpacks
Available Packages
Name        : openstack-packstack-puppet
... Output omitted ...
Description : Puppet module used by Packstack to install OpenStack
 
[root@workstation ~]# yum info openstack-puppet-modules
Loaded plugins: langpacks
Available Packages
Name        : openstack-puppet-modules
... Output omitted ...
Description : A collection of Puppet modules which are required to install and
            : configure OpenStack via installers using Puppet configuration
            : tool.
 
[root@workstation ~]# yum info openstack-tripleo-puppet-elements
Loaded plugins: langpacks
Available Packages
Name        : openstack-tripleo-puppet-elements
... Output omitted ...
Description : OpenStack TripleO Puppet Elements is a collection of elements for
            : diskimage-builder that can be used to build OpenStack images
            : configured with Puppet for the TripleO program.
```

2. 安装所有可用的 OpenStack puppet 软件包。

```
[root@workstation ~]# yum -y install openstack\*puppet\*
... Output omitted ...
Transaction Summary
================================================================================
Install  3 Packages (+9 Dependent packages)
... Output omitted ...
Complete!
```

3. 注意这些软件包各自的 Puppet 代码存放位置。

```
[root@workstation ~]# rpm -ql openstack-packstack-puppet
/usr/share/openstack-puppet/modules/packstack
... Output omitted ...
[root@workstation ~]# rpm -ql openstack-puppet-modules
/usr/share/openstack-puppet/Puppetfile
/usr/share/openstack-puppet/modules/apache
... Output omitted ...
/usr/share/openstack-puppet/modules/aviator
... Output omitted ...
/usr/share/openstack-puppet/modules/ceilometer
... Output omitted ...
[root@workstation ~]# rpm -ql openstack-tripleo-puppet-elements
/usr/lib/python2.7/site-packages/tripleo_puppet_elements-0.0.1-py2.7.egg-info
... Output omitted ...
/usr/share/doc/openstack-tripleo-puppet-elements-0.0.1
... Output omitted ...
/usr/share/tripleo-puppet-elements
/usr/share/tripleo-puppet-elements/hiera
... Output omitted ...
/usr/share/tripleo-puppet-elements/puppet-modules
... Output omitted ...
```

4. 下一演示中将介绍用于配置 NTP 的 Puppet 模块，它由 openstack-puppet-modules 软件包提供。

```
[root@workstation ~]# rpm -ql openstack-packstack-puppet | grep ntp
/usr/share/openstack-puppet/modules/module-collectd/manifests/plugin/ntpd.pp
/usr/share/openstack-puppet/modules/module-collectd/templates/plugin/ntpd.conf.erb
/usr/share/openstack-puppet/modules/ntp
... Output omitted ...
/usr/share/openstack-puppet/modules/ntp/manifests
/usr/share/openstack-puppet/modules/ntp/manifests/config.pp
/usr/share/openstack-puppet/modules/ntp/manifests/init.pp
/usr/share/openstack-puppet/modules/ntp/manifests/install.pp
/usr/share/openstack-puppet/modules/ntp/manifests/params.pp
/usr/share/openstack-puppet/modules/ntp/manifests/service.pp
... Output omitted ...

```

###### 演示：查看 RHEL-OSP 中的 Puppet 代码

以下 Puppet 代码示例演示了 Puppet 可以执行的系统管理功能。这些 Puppet 规格展示 RHEL OpenStack 安装程序在安装时如何使用 Puppet 配置系统。 

1. 用于为 Red Hat Enterprise Linux OpenStack 平台配置 NTP 服务的 Puppet 清单可在 /usr/share/openstack-puppet/modules/ntp/manifests 目录中找到。首先更改到该目录；这样便能更加轻松地浏览其中找到的不同 Puppet 清单。

```
[root@workstation ~]# cd /usr/share/openstack-puppet/modules/ntp/manifests
```

2. 在以下屏幕截图中，Puppet 模块正在尝试确保已安装了 ntp 软件包。它使用变量来确定 Puppet 是否应当管理软件包可用性、软件包名称和状态。这些变量可以在下一示例中找到。

```
[root@workstation manifests]# cat install.pp
#
class ntp::install inherits ntp {
 
  if $ntp::package_manage {
 
    package { $ntp::package_name:
      ensure => $ntp::package_ensure,
    }
 
  }
 
}
```

3. 在以下屏幕截图中，Puppet 模块正在评估 ntp 服务的状态，它是否正在运行 (ensure) 以及是否在引导时启用 (enable)。同样，下一示例中将查看此代码使用的变量。

```
[root@workstation manifests]# cat service.pp
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

4. 在以下屏幕截图中，Puppet 模块正专注于 ntp 的特定配置文件的存在性、内容和安全性。如果启用了密钥，则确保存储这些密钥的目录存在、由 root 所有并具有 rwxr-xr-x 权限。接下来，确认配置文件存在、由 root 所有并具有 rw-r--r-- 权限，并且内容与提供的模板匹配。

```
[root@workstation manifests]# cat config.pp
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

5. 在以下屏幕截图中，Puppet 模块正在定义上例中应用的变量的值。注意使用 case 语句来分叉，并根据所支持的不同操作系统“系列”来定义不同的值（来自于文件最上方的默认值）。

```
[root@workstation manifests]# cat params.pp
class ntp::params {
 
  $autoupdate        = false
  $config_template   = 'ntp/ntp.conf.erb'
  $disable_monitor   = false
  $keys_enable       = false
  $keys_controlkey   = ''
  $keys_requestkey   = ''
  $keys_trusted      = []
... Output omitted ...
 
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
    'Solaris': {
      $config        = '/etc/inet/ntp.conf'
      $driftfile     = '/var/ntp/ntp.drift'
      $keys_file     = '/etc/inet/ntp.keys'
      $package_name  = $::operatingsystemrelease ? {
        /^(5\.10|10|10_u\d+)$/ => [ 'SUNWntpr', 'SUNWntpu' ],
        /^(5\.11|11|11\.\d+)$/ => [ 'service/network/ntp' ]
      }
      $restrict      = [
        'default kod nomodify notrap nopeer noquery',
        '-6 default kod nomodify notrap nopeer noquery',
        '127.0.0.1',
        '-6 ::1',
      ]
      $service_name  = 'network/ntp'
      $iburst_enable = false
      $servers       = [
        '0.pool.ntp.org',
        '1.pool.ntp.org',
        '2.pool.ntp.org',
        '3.pool.ntp.org',
      ]
      $maxpoll       = undef
    }
... Output omitted ...
```

6. 在以下屏幕截图中，Puppet 模块是 Puppet 类定义文件，它将引用前面的示例。RHEL-OSP 安装程序将“调用” init.pp 来执行 OpenStack 的配置和安装。注意末尾提到的 install、config 和最后的 service。

```
[root@workstation manifests]# cat init.pp
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
... Output omitted ...
) inherits ntp::params {
 
  validate_bool($broadcastclient)
  validate_absolute_path($config)
  validate_string($config_template)
  validate_bool($disable_auth)
  validate_bool($disable_monitor)
  validate_absolute_path($driftfile)
  if $logfile { validate_absolute_path($logfile) }
  if $leapfile { validate_absolute_path($leapfile) }
... Output omitted ...
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

### 总结

*    红帽企业 Linux OpenStack 平台安装程序使用 Puppet 代码来配置 OpenStack 服务。
*    阅读一些由 RHEL-OSP 使用的 Puppet 代码的基本示例，并且理解它们的用途。 


