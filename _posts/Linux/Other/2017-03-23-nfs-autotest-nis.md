layout: post
title:  "lookup_read_master: lookup(nisplus): couldn't locate nis+ table auto.master"
categories: Linux
tags: 
---


Remove nisplus from /etc/nsswitch.conf. For example:

```
# vim /etc/nsswitch.conf.
automount:  files nisplus
automount:  files
# systemctl restart autofs
```
