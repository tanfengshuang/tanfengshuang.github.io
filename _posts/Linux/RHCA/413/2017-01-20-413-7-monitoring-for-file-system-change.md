---
layout: post
title:  "Monitoring for File System Change(413-7)"
categories: Linux
tags: 413
---

### Using Intrusion Detection Software to Monitor Changes

###### Install AIDE

```
# yum install -y aide
```

###### Configure AIDE

```
# vim /etc/aide.conf
param = value(param is not a built-in AIDE setting)

PERMS = p+i+u+g+acl+selinux

/dir1 group
=/dir2 group
!/dir3 group

man 5 aide.conf
```

###### Initialize the AIDE Database

```
# aide --init
# mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

This DB is where AIDE expects the database to be when performing file system checks.
```
