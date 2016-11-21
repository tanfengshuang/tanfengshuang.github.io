---
layout: post
title:  "修改workspace目录"
categories: Eclipse
tags: Eclipse
---


Source Link: http://blog.csdn.net/haolyj98/article/details/9222089

Eclipse是一款很强的Java IDE，我们在开始的时候，往往设定了默认的workspace，当用久在之后，我们可能要去更改一下workspace的位置。下面有几种方法可以更改workspace的目录。
1. 进入 Window > Preferences > General > Startup and Shutdown 选中 Prompt for workspace on startup。
2. 进入Eclipse的安装目录，找到configuration 目录下的 .settings 文件夹，里面有一个 org.eclipse.ui.ide.prefs， 用Ultra Edit等打开，也可以用写字板打开，找到RECENT_WORKSPACES，按照它的格式>修改一下。
3. 先打开Eclipse，进入之后，再去打开一次，会提示 Workspace in use or cannot be created, choose a different one 。 这时候就会提示你更改workspace的目录了。

这三种方法都可以更改，选一种适合的就可以。
