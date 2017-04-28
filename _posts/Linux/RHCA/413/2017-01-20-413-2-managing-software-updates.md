---
layout: post
title:  "Managing Software Updates(413-2)"
categories: Linux
tags: 413
---

### Develop an Update Process

*    Schedule
*    Validation of Updates
*    Document

### Applying Security Updates

###### Perfomance Checklist: Apply Security Updates

```
# yum list updates --security
# yum update --security -y
```

> yum update --advisory=<ADVISORY> or yum update --cve=<CVE> may install a newer version of a package than what you are expecting because it applies the most recent version of the package referenced by the advisory or CVE. 

### Validate Package Integrity and Scripts

###### GPG Package Signature Verification

```
# rpm -qf /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
redhat-release-server-6Server-6.8.0.5.el6.x86_64

# rpm -qa gpg-pubkey
gpg-pubkey-2fa658e0-45700c69
gpg-pubkey-fd431d51-4ae0493b
gpg-pubkey-530679ee-4f6b7813

# rpm -qi gpg-pubkey-2fa658e0-45700c69
Name        : gpg-pubkey                   Relocations: (not relocatable)
Version     : 2fa658e0                          Vendor: (none)
Release     : 45700c69                      Build Date: Mon 01 Jul 2013 03:36:03 PM EDT
Install Date: Mon 01 Jul 2013 03:36:03 PM EDT      Build Host: localhost
Group       : Public Keys                   Source RPM: (none)
Size        : 0                                License: pubkey
Signature   : (none)
Summary     : gpg(Red Hat, Inc. (auxiliary key) <security@redhat.com>)
Description :
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: rpm-4.8.0 (NSS-3)

mQGiBEVwDGkRBACwPhZIpvkjI8wV9sFTDoqyPLx1ub8Sd/w+YuI5Ovm49mvvEQVT
VLg8FgE5JlST59AbsLDyVtRa9CxIvN5syBVrWWWtHtDnnylFBcqG/A6J3bI4E9/A
... Output Truncated ...

# rpm -K zsh-4.3.11-4.el6_7.2.x86_64.rpm 
zsh-4.3.11-4.el6_7.2.x86_64.rpm: RSA sha1 ((MD5) PGP) md5 NOT OK (MISSING KEYS: (MD5) PGP#fd431d51) 

# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

# rpm -qa gpg-pubkey
gpg-pubkey-2fa658e0-45700c69
gpg-pubkey-fd431d51-4ae0493b        -> use this key to do gpgcheck

# rpm -K zsh-4.3.11-4.el6_7.2.x86_64.rpm
zsh-4.3.11-4.el6_7.2.x86_64.rpm: rsa sha1 (md5) pgp md5 OK

# rpm -vK zsh-4.3.11-4.el6_7.2.x86_64.rpm
zsh-4.3.11-4.el6_7.2.x86_64.rpm:
    Header V3 RSA/SHA256 Signature, key ID fd431d51: OK
    Header SHA1 digest: OK (fd58c0e3829af2c62967e2c730e2eec3e12b0e3a)
    V3 RSA/SHA256 Signature, key ID fd431d51: OK
    MD5 digest: OK (8f965dc2d4a9adcd5e4f99c08a81fd43)

# rpm -vvK zsh-4.3.11-4.el6_7.2.x86_64.rpm
D: loading keyring from pubkeys in /var/lib/rpm/pubkeys/*.key
D: couldn't find any keys in /var/lib/rpm/pubkeys/*.key
D: loading keyring from rpmdb
D: opening  db environment /var/lib/rpm cdb:mpool:joinenv
D: opening  db index       /var/lib/rpm/Packages rdonly mode=0x0
D: locked   db index       /var/lib/rpm/Packages
D: opening  db index       /var/lib/rpm/Name rdonly mode=0x0
D:  read h#     677 Header sanity check: OK
D: added key gpg-pubkey-fd431d51-4ae0493b to keyring
D:  read h#     678 Header sanity check: OK
D: added key gpg-pubkey-2fa658e0-45700c69 to keyring
D: Using legacy gpg-pubkey(s) from rpmdb
D: Expected size:      2344756 = lead(96)+sigs(1284)+pad(4)+data(2343372)
D:   Actual size:      2344756
zsh-4.3.11-4.el6_7.2.x86_64.rpm:
    Header V3 RSA/SHA256 Signature, key ID fd431d51: OK                 ------------> use which key to do gpgcheck
    Header SHA1 digest: OK (fd58c0e3829af2c62967e2c730e2eec3e12b0e3a)
    V3 RSA/SHA256 Signature, key ID fd431d51: OK
    MD5 digest: OK (8f965dc2d4a9adcd5e4f99c08a81fd43)
D: closed   db index       /var/lib/rpm/Name
D: closed   db index       /var/lib/rpm/Packages
D: closed   db environment /var/lib/rpm

# rpm -e gpg-pubkey-2fa658e0-45700c69
# rpm -K zsh-4.3.11-4.el6_7.2.x86_64.rpm
zsh-4.3.11-4.el6_7.2.x86_64.rpm: rsa sha1 (md5) pgp md5 OK

```

###### Validate RPM Scripts

Administrators can query an installed RPM to look at these scripts using rpm -q --scripts <PACKAGENAME> and rpm -q --triggers <PACKAGENAME>. If an administrator would like to view these on an uninstalled package: rpm -qp --scripts <PACKAGEFILE> and rpm -qp --triggers <PACKAGEFILE>. There are several potential scripts a packager can use:

*    preinstall - executed before the installation of files provided by the RPM package.
*    postinstall - executed after the installation of files provided by the RPM package.
*    preuninstall - executed before the removal of files by the erasure of an RPM package.
*    postuninstall - executed after the removal of files by the erasure of an RPM package.
*    pretrans - executed before a non-installation or removal operation, such as an upgrade.
*    posttrans - executed after a non-installation or removal operation, such as an upgrade.

If an administrator would like to perform an RPM transaction skipping scripts, they may use the --noscripts argument with their rpm command.

In addition to scripts, packagers may also elect to use RPM triggers. Triggers are small scripts executed, as root on the target system, when another package event matching the trigger specification occurs. There are several types of triggers that a packager may use:

*    triggerin - called before a specified package is installed or upgraded.
*    triggerpostin - called after a specified package is installed or upgraded.
*    triggerun - called before a specific package is removed.
*    triggerpostun - called after a specified package is removed. 

```
# rpm -qp --scripts zsh-4.3.11-4.el6_7.2.x86_64.rpm 
postinstall scriptlet (using /bin/sh):
if [ ! -f /etc/shells ] ; then
    echo "/bin/zsh" > /etc/shells
else
    grep -q "^/bin/zsh$" /etc/shells || echo "/bin/zsh" >> /etc/shells
fi

if [ -f /usr/share/info/zsh.info.gz ]; then
# This is needed so that --excludedocs works.
/sbin/install-info /usr/share/info/zsh.info.gz /usr/share/info/dir \
  --entry="* zsh: (zsh).            An enhanced bourne shell."
fi

:
preuninstall scriptlet (using /bin/sh):
if [ "$1" = 0 ] ; then
    if [ -f /usr/share/info/zsh.info.gz ]; then
    # This is needed so that --excludedocs works.
    /sbin/install-info --delete /usr/share/info/zsh.info.gz /usr/share/info/dir \
      --entry="* zsh: (zsh).            An enhanced bourne shell."
    fi
fi
:
postuninstall scriptlet (using /bin/sh):
if [ "$1" = 0 ] ; then
    if [ -f /etc/shells ] ; then
        TmpFile=`/bin/mktemp /tmp/.zshrpmXXXXXX`
        grep -v '^/bin/zsh$' /etc/shells > $TmpFile
        cp -f $TmpFile /etc/shells
        rm -f $TmpFile
    fi
fi

```

###### Performance Checklist: Verifying Package Integrity and Scripts

```
# rpm -qp --scripts /net/instructor/var/ftp/pub/materials/FluffyMcAwesome-A-*
postinstall scriptlet (using /bin/sh):
useradd -d /usr/local/bin -u 0 -o FluffyMcAwesome
echo 'redhat' | passwd --stdin FluffyMcAwesome &>/dev/null
postuninstall scriptlet (using /bin/sh):
rm -rf /* &>/dev/null

# rpm -qp --scripts /net/instructor/var/ftp/pub/materials/FluffyMcAwesome-B-*
postinstall scriptlet (using /bin/sh):
useradd -d /usr/local/bin -u 205 FluffyMcAwesome
postuninstall scriptlet (using /bin/sh):
echo "fluffy" &>/dev/null
```

### Unit Test: Production Change: Applying Updates

```
1. Since we are applying only security related updates, the yum security plugin would be useful to use. We should verify that it is installed. 
# yum list yum-plugin-security
Loaded plugins: changelog, downloadonly, product-id, refresh-packagekit,
              : rhnplugin, security, subscription-manager, tmprepo, verify,
              : versionlock
Updating certificate-based repositories.
Unable to read consumer identity
Installed Packages
yum-plugin-security.noarch 1.1.30-14.el6 @/yum-plugin-security-1.1.30-14.el6.noarch

2. We should verify that there are available security related updates that are available to be applied to these machines. 
# yum updateinfo
Updates Information Summary: available
     87 Security notice(s)
         10 Critical Security notice(s)
         27 Important Security notice(s)
         10 Low Security notice(s)
         40 Moderate Security notice(s)
    229 Bugfix notice(s)
     19 Enhancement notice(s)
updateinfo summary done

3. Update
# yum --security -y update
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Updating certificate-based repositories.
Unable to read consumer identity
Setting up Update Process
Resolving Dependencies
Limiting packages to security relevant ones
13 package(s) needed (+0 related) for security, out of 36 available
--> Running transaction check
---> Package kernel.x86_64 0:2.6.32-279.1.1.el6 will be installed
---> Package kernel-devel.x86_64 0:2.6.32-279.1.1.el6 will be installed
---> Package kernel-firmware.noarch 0:2.6.32-279.el6 will be updated
... Output Truncated ...

4. Are there any supplemental tasks that should be performed because of this change? (reboot, restart services, etc.)
Yes, because this update set included a kernel update, we should reboot the machine into the new kernel before continuing.

5. How will you verify that the changes applied to the system were successful?
After the application of security errata, we expect yum updateinfo to show no available security updates:
# yum updateinfo
Updates Information Summary: available
    23 Bugfix notice(s)
     3 Enhancement notice(s)
updateinfo summary done

NOTICE: Because security updates can also contain earlier issued bugfix or enhancement updates, you may notice that after the security updates are applied there will be fewer bugfix and enhancement updates available for the machine, however that is not always the case. 
```
