---
layout: post
title:  "文件管理"
categories: Linux
tags: RHCSA chmod chattr setfacl ln
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

文件管理系统中，将可操作文件的用户分为三类：

*    文件的创建者 (u)
*    文件所属组的成员 (g)
*    其他用户 (o)

对每一类用户，权限系统分别给予他们三种权限：

*    读(r): 用户是否有权力读取文件内容
*    写(w): 用户是否有权力改变文件的内容
*    执行(x): 用户是否有权力执行文件
    
### chmod

*    通过 + - 为某类用户添加或去掉某种权限
    chmod u+x $file
*    通过 = 为某类用户赋予某权限
    chmod g=rx $file
*    通过三个数字为三种用户分别赋予权限 
    chmod 755 $file

### chattr lsattr

*    -a: 即append, 设定该参数后，只能向文件中添加数据，而不能删除，多用于服务器日志文件安全，之用root才能设定这个参数
*    -i: 设定文件不能被删除，改名，设定链接关系，同时不能写入或新增内容. i参数对于文件系统的安全设置有很大帮助
*    -s: 保密性的删除文件或目录，即磁盘空间被全部收回
*    -u: 与s相反，当设定为u时，数据内容其实还存在于磁盘中，可以用于undeletion

### 文件访问控制列表ACL - setfacl/getfacl
ACL可以为某个文件单独设置该文件具体的某用户或组的权限(对于不属于u g o的用户特别好用)

*    -m: 修改用户或组对文件的权限
*    -x: 取消用户或组对文件的权限

```
# setfacl -m u:boss:r $file		//设置用户boss对文件的访问权限 
# setfacl -m g:leader:r $file 		//设置用户组leader对文件的访问权限
# setfacl -x u:boss:r $file
# setfacl -x g:leader:r file 

# setfacl -m d:u:boss:r -R caiwubu/	//设置用户boss对目录caiwubu的默认访问权限，这样子，用户boss对于目录caiwubu下面的新增加的内容都有访问权限
```


### ln

*    软链接 ln -s <源文件名> <目标链接文件名>
*    硬链接 ln <源文件名> <目标链接文件名>
