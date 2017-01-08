---
layout: post
title:  "为Openshift应用添加附加应用"
categories: Openshift
tags: openshift
---

### 客户端配置Openshift支持的Cartridge

###### 查看cartridge命令

*    rhc cartridge list
*    rhc cartridges

###### 为现有应用添加附加Cartridge

*    rhc cartridge add cartridge-addon-type -a AppName

```
[student@server0-a] $ rhc cartridges

为现有应用添加mysql Cartridge
[student@server0-a] $ rhc app show firstphp
[student@server0-a] $ rhc cartridge add mysql-5.1 -a firstphp

登录应用firstphp
[student@server0-a] $ rhc ssh -a firstphp
在应用中使用mysql
[firstphp-ose.apps0.example.com ~] $ mysql
mysql> show databases;          -> 默认创建的有数据库firstphp

在应用的mysql数据库中创建表，并插入数据
mysql> use firstphp
mysql> create table users(user_id int not null auto_increment,
username varchar(200),
PRIMARY KEY(user_id))
mysql> desc users;
mysql> insert into users values(null, 'Pinkie Pie');
mysql> select * from users;

收集应用的环境参数
[firstphp-ose.apps0.example.com ~] $ env | grep OPENSHIFT

通过环境参数连接和使用Mysql数据库
[student@server0-a] $ vim ~student/ose/firstphp/dbtest.php
<?php
    $dbhost = getenv("OPENSHIFT_MYSQL_DB_HOST");
    $dbport = getenv("OPENSHIFT_MYSQL_DB_PORT");
    $dbuser = getenv("OPENSHIFT_MYSQL_DB_USERNAME");
    $dbpwd = getenv("OPENSHIFT_MYSQL_DB_PASSWORD");
    $dbname = getenv("OPENSHIFT_APP_NAME");

    $connection = mysql_connect($dbhost, $dbuser, $dbpwd);

    if(!$connection){
        echo "Could not connect to database";
    } else {
        echo "Connect to database <br />";
    }
    $dbconnection = mysql_select_db($dbname);
    $query = "SELECT * FROM user";
    $results = mysql_query(query);
    while($row=mysql_fetch_assoc($results)){
        echo $row['user_id']." ".$row['username']." \n";
    }
    mysql_close();
?>

[student@server0-a] $ firefox firstphp-ose.apps0.example.com/dbtest.php
Connected to database
1 Pinkie Pei
```

###### 通过rhc cartridge命令对Cartridge单独操作

*    重读Cartridge配置  -    rhc cartridge reload CartridgeName --app AppName
*    启动、关闭和重启Cartridge  

```
rhc cartridge start CartridgeName --app AppName
rhc cartridge stop CartridgeName --app AppName
rhc cartridge restart CartridgeName --app AppName
```
   
*    删除Cartridge  -   rhc cartridge remove CartridgeName --app AppName

```
[student@server0-a] $ rhc cartridge start mysql-5.1
Starting mysql-5.1 ...
done
[student@server0-a] $ rhc cartridge status mysql-5.1
RESULT:
MySQL is running
```

###### Hot Deploy

*    为了保证快速部署php应用代码，而不需要重新启动php Cartridge，需要设置Hot Deploy

```
[student@server0-a] $ touch ~student/ose/firstphp/.openshift/markers/hot_deploy
[student@server0-a] $ cd ~student/ose/firstphp/.openshift/
[student@server0-a] $ git add .
[student@server0-a] $ git commit -am '...'
[student@server0-a] $ git push
```

