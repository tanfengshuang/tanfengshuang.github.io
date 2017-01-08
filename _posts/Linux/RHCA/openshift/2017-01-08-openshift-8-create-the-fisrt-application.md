---
layout: post
title:  "Openshift创建并管理第一个OpenShift应用"
categories: Openshift
tags: openshift
---

### 创建第一个应用

*    创建命令 - rhc app create -a AppName -t Cartridgetype
*    创建SSH密钥或将已创建的密钥同步到OpenShift服务器
*    创建本地AppName同名目录
*    申请更新AppName-Namespace.CLOUD-DOMAIN域名

###### 更新第一个应用

*    更新使用git命令    - git add/commit/push

###### AppName同名目录结构

*    在rhc setup时，于命令当前路径创建AppName同名目录
*    目录中特有目录：.git(项目git版本控制目录), .openshift(Openshift应用及附加应用控制目录)

###### 创建第一个OpenShift应用实验

*    使用student用户登录server0-a
*    创建应用保存目录ose
*    创建第一个Openshift应用firstphp,类型为php-5.3
*    使用git命令提交更新
*    测试更新
*    测试firstphp目录和firstphp-ose.apps0.example.com域名是否创建更新正常
*    修改index.php文件，更新内容


```
[student@server0-a] $ mkdir ose && cd ose
[student@server0-a] $ rhc app create -a firstphp -t php-5.3 

[student@server0-a] $ cd /home/student/ose/firstphp
[student@server0-a] $ git commit -am ".."
[student@server0-a] $ git push
[student@server0-a] $ 

[student@server0-a] $ firefox http://firstphp-ose.apps0.example.com

[student@server0-a] $ ls -ld /home/student/ose/firstphp
[student@server0-a] $ dig firstphp-ose.apps0.example.com

[student@server0-a] $ vim /home/student/ose/firstphp/index.php



```


###### 使用git命令提交新文件

```
创建新文件time.php
[student@server0-a] $ cat /home/student/ose/firstphp/time.php
<?php
    echo 'UTC   ', date('G:i');
?>


添加文件到更新列表
[student@server0-a] $ cd /home/student/ose/firstphp/
[student@server0-a] $ git add .
[student@server0-a] $ git commit -am '...'
[student@server0-a] $ git push

```

###### 开启和关闭应用

```
开启应用
[student@server0-a] $ rhc start -a firstphp     -> -a AppName
RESULT:
firstphp started

关闭应用
[student@server0-a] $  rhc stop -a firstphp     -> -a AppName
RESULT:
firstphp stopped

```

###### 查看应用信息


```
查看详细信息
[student@server0-a] $ rhc show -a firstphp


查看运行状态
[student@server0-a] $ rhc show --state -a firstphp
Cartridge php-5.3 is started

```


###### 远程在应用上执行命令

```
查看磁盘限额
[student@server0-a] $ rhc ssh -a firstphp "quota -a"

[student@server0-a] $ rhc ssh -a firstphp "pwd"
[student@server0-a] $ rhc ssh -a firstphp "ls -l"

清除日志、临时目录内容和整理git提交
[student@server0-a] $ rhc app tidy -a firstphp

查看应用日志
[student@server0-a] $ rhc tail -a firstphp
```


###### 为应用创建域名别名

*    设置DNS - 创建一条指向应用原有域名的CNAME记录并生效
*    添加应用域名 - rhc alias add AppName DNS-CNAME

###### 为应用创建备份和还原备份

*    创建备份 - rhc snapshot save AppName
*    还原备份 - rhc snapshot restore -a AppName -b backupfile.tar.gz

```
[student@server0-a] $ firefox http://server0-a.example.com        -> 打开web console，看到上面新建的application firstphp，点击firstphp，进入，显示firstphp-ose.apps0.example.com, 并有Cartridge PHP 5.3
[student@server0-a] $ rhc alias add firstphp firstphp0.example.com
Alias 'firstphp0.example.com' has been added
[student@server0-a] $ firefox http://server0-a.example.com        -> 打开web console，点击firstphp，进入，看到firstphp firstphp0.example.com，并有Cartridge PHP 5.3

[student@server0-a] $ rhc snapshot save firstphp
Pulling down a snapshot of appliction 'firstphp' to firstphp.tar.gz
...
done

[student@server0-a] $ ls
firstphp.tar.gz

[student@server0-a] $ tar -tf firstphp.tar.gz | grep kevin
./XXX/app-deployment/2016-12-25_10-00-00/repo/keven.php

[student@server0-a] $ rhc snapshot restore -a firstapp -f firstphp.tar.gz
Restoring from snapshot firstphp.tar.gz to application 'firstphp' ...
done

```










