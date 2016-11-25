---
layout: post
title:  "fedora上解压rar文件"
categories: Linux
tags: Fedora unrar
---

unrar不是在fedora官方的源，需要安装rpmfusion

```
# rpm -Uvh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm
# rpm -Uvh http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm

# yum install unrar

# unrar e file_name.rar
```

