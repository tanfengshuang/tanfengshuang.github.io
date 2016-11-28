---
layout: post
title:  "交互工具和编辑器"
categories: Linux
tags: RHCSA 
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### 交互工具  

*    write <用户名>     指定一个在线用户发送短消息
*    wall <广播信息>    向所有在线用户发广播

### mesg  
mesg命令用于设置当前终端的写权限，即是否让其他用户向本终端发信息, 将mesg设置y时，其他用户可利用write命令将信息直接显示在您的屏幕上  

*    y 表示运行向当前终端写信息
*    n 表示禁止向当前终端写信息

```
# mesg y #允许系统用户将信息直接显示在你的屏幕上
# mesg n #不允许系统用户将信息直接显示在你的屏幕上
```

### 文本编辑器  

*    vi
*    vim
*    emacs
*    gedit
*    OpenOffice

#### vim

```
:set encoding           // 查看当前文件的编码格式
:set encoding=utf-8     // 设置编码格式
:r <文件名>             // 在当前位置插入文件内容
:r! <命令>              // 在当前位置插入命令执行结果
:set number             // 显示行号
:set nonumber           // 不显示行号
:set expandtab          // tab转换成空格
:set noexpandtab        // tab不转换成空格
:set ts=4               // tab为四个字符长度
:行号                   // 快速跳转到指定行
:s/a/b/g                // 在当前行替换
:2,19s/a/b/g            // 在2到19行替换
:%s/a/b/g               // 全文替换
```

