---
layout: post
title:  "通过QuickStarts快速部署应用"
categories: Openshift
tags: openshift
---

### OpenShift QuickStarts

*    设计QuickStarts用以提高系统管理人员和开发人员的时间，同时提高效率
*    QuickStarts是应用代码、数据集成和配置脚本的集合
*    QuickStarts可以通过Git方便的完成本地项目的实例化
*    大量的QuickStarts保存在http://github.com/openshift上，包括WordPress, OwnCloud, EtherPad和MediaWiki

### 使用QuickStarts方式安装WordPress应用

```
创建wordpress应用
# ssh student@server0-a
[student@server0-a] $ cd ose
[student@server0-a] $ rhc app create wordpress -t php-5.3

添加mysql Cartidge到wordpress应用
[student@server0-a] $ rhc cartridge add mysql-5.1 wordpress

下载WordPress QuickStarts文件
[student@server0-a] $ cd ~
[student@server0-a] $ wget http://classroom.example.com/materials/wordpress.git.tar.gz

解开wordpress.git.tar.gz,并在本地初始化
[student@server0-a] $ tar zxvf wordpress.git.tar.gz
[student@server0-a] $ cd wordpress.git
[student@server0-a] $ git init --bare

将本地wordpress目录对应更新到wordpress.git
[student@server0-a] $ cd ~/ose/wordpress
[student@server0-a] $ git remote add upstream -m master ../../wordpress.git

添加git用户信息
[student@server0-a] $ git config --global user.email student@server0-a.example.com
[student@server0-a] $ git config --global user.name student.example.com

同步wordpress应用
1. 将wordpress.git同步到wordpress
[student@server0-a] $ cd ~/ose/wordpress
[student@server0-a] $ git pull -s recursive -X upstream master
2. 将wordpress同步到线上OpenShift应用
[student@server0-a] $ git push

测试wordpress部署情况
[student@server0-a] $ rhc app restart wordpress
[student@server0-a] $ firefox http://wordpress-ose.apps0.example.com
```

