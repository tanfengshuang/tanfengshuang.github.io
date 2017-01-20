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

### Validate Package Integrity and Scripts

###### GPG Package Signature Verification

```
# rpm -qf /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
redhat-release-server-6Server-6.8.0.5.el6.x86_64

# rpm -qa gpg-pubkey

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

### Unit Test: Production Change: Applying Updates

