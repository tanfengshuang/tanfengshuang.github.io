---
layout: post
title:  "系统文件查找与文件管理"
categories: Linux
tags: RHCSA find tr cut tar uniq sort paste wc
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看


### 可执行文件的搜索

*    which
*    whereis


### find命令

*    从指定路径下递归向下搜索
*    支持按照各种条件方式搜索
*    支持对搜索得到的文件再进一步的使用指令操作(例如，复制 统计大小 删除等)

find <路径> <选项> [表达式]

*    -name      根据文件名查找文件
*    -user      根据文件拥有者查找文件
*    -group     根据文件所属组查找文件
*    -perm      根据文件权限查找文件
*    -size      根据文件大小查找文件[+size][-size]
*    -type      根据文件类型查找文件，常见类型有: f(普通文件), c(字符设备文件), b(块设备文件), l(链接文件), d(目录文件)
*    -o         表达式或
*    -and       表达式与
*    -not       表达式非

find <路径> <选项> [表达式] -exec 指令 {} \;

*    {} 代表find找到的文件
*    \  转意
*    ;  表示本行指令结束
*    例， find /etc/ -name "host" -exec du -h {} \;

```
# find /etc -size +1M
/etc/selinux/targeted/policy/policy.30
/etc/selinux/targeted/contexts/files/file_contexts.bin
/etc/udev/hwdb.bin

# find /etc/docker/ -type l
/etc/docker/certs.d/redhat.com/redhat-ca.crt
/etc/docker/certs.d/redhat.io/redhat-ca.crt

# ll /etc/docker/certs.d/redhat.com/redhat-ca.crt
lrwxrwxrwx. 1 root root 27 Oct 10 20:49 /etc/docker/certs.d/redhat.com/redhat-ca.crt -> /etc/rhsm/ca/redhat-uep.pem

# find / -type c -and -name tty*
/dev/ttyS4
/dev/ttyS31
/dev/ttyS30
/dev/ttyS29
...
```

### 常用的文件操作指令

*    head / tail / more / less     文件的查看
*    wc     统计文件的行 词 字数
*    grep   
*    sort
*    uniq
*    tr     转换字符
*    cut    显示文件中的某一列
*    paste  将文本按列拼接

##### wc

*    -c  只显示文件的字符数
*    -l  只显示行数
*    -w  只显示单词数

##### grep

*    -c  计算匹配关键字的行数
*    -i  忽略字符大小写的差别
*    -n  显示匹配的行及行号
*    -s  不显示不存在或不匹配文本的错误信息
*    -h  查询多个文件时，不显示文件名
*    -l  查询文件时只显示匹配字符所在的文件名
*    -v  取反
*    --color=auto   RHEL7中默认执行时已经包括了这个参数

##### sort

*    -o <输出文件>  将排序后的结果存入指定的文件
*    -r     以相反的顺序来排序
*    -t <分隔字符>  指定排序时所用的栏位分隔符
*    -u     忽略相同行, 或者使用uniq
*    -n     按照数字的大小进行排序
*    -k     是指定需要爱排序的栏位

-k参数    
FStart.CStart Modifie,FEnd.CEnd Modifier   
-------Start--------,-------End--------    
FStart.CStart 选项 , FEnd.CEnd 选项   

```
# cat sort.txt 
AAA:BB:CC 
aaa:30:1.6 
ccc:50:3.3 
ddd:20:4.2 
bbb:10:2.5 
eee:40:5.4 
eee:60:5.1 

#将BB列按照数字从小到大顺序排列： 
# sort -nk 2 -t: sort.txt 
AAA:BB:CC 
bbb:10:2.5 
ddd:20:4.2 
aaa:30:1.6 
eee:40:5.4 
ccc:50:3.3 
eee:60:5.1 

#将CC列数字从大到小顺序排列： 
# sort -nrk 3 -t: sort.txt 
eee:40:5.4 
eee:60:5.1 
ddd:20:4.2 
ccc:50:3.3 
bbb:10:2.5 
aaa:30:1.6 
AAA:BB:CC

从公司英文名称的第二个字母开始进行排序：
# sort -t ' ' -k 1.2 facebook.txt 
baidu 100 5000 
sohu 100 4500 
google 110 5000 
guge 50 3000

只针对公司英文名称的第二个字母进行排序，如果相同的按照员工工资进行降序排序： 
# sort -t ' ' -k 1.2,1.2 -nrk 3,3 facebook.txt 
baidu 100 5000 
google 110 5000 
sohu 100 4500 
guge 50 3000

```

##### uniq

*    -c 在每行旁边显示该行重复出现的次数
*    -d 仅显示重复出现的行列
*    -u 仅显示出一次的行列

```
删除重复行： 
# uniq file.txt 
# sort file.txt | uniq 
# sort -u file.txt 

只显示单一行： 
# uniq -u file.txt 
# sort file.txt | uniq -u 

统计各行在文件中出现的次数： 
# sort file.txt | uniq -c 

在文件中找出重复的行： 
# sort file.txt | uniq -d
```

##### tr

cat <文件> | tr [选项]

*    -s 把连续重复的字符以单独一个字符表示
*    -d 删除所有属于第一字符集的字符

```
将输入字符由大写转换为小写：
# echo "HELLO WORLD" | tr 'A-Z' 'a-z' 
hello world

用tr压缩字符，可以压缩输入中重复的字符：
# echo "thissss is         a text linnnnnnne." | tr -s ' sn'
this is a text line.

使用tr删除字符： 
# echo "hello 123 world 456" | tr -d '0-9' 
hello world

删除Windows文件“造成”的'^M'字符： 
# cat file | tr -s "\r" "\n" > new_file 
或 
# cat file | tr -d "\r" > new_file
```

##### cut

*    -d 指定分隔符
*    -f 指定显示某一列
*    -c 指定几个字符对应的列

```
# cat test.txt 
No Name Mark Percent 
01 tom 69 91 
02 jack 71 87 
03 alex 68 98 

使用 -f 选项提取指定字段： 
# cut -f 1 test.txt 
No 
01 
02 
03 

# cut -f2,3 test.txt 
Name Mark 
tom 69 
jack 71 
alex 68


# cat test2.txt 
No;Name;Mark;Percent 
01;tom;69;91 
02;jack;71;87 
03;alex;68;98 

指定字段分隔符
# cut -f2 -d";" test2.txt 
Name 
tom 
jack 
alex


指定字段的字符或者字节范围
# cut -f2 -d";" a | cut -c2,3
am
om
ac
le

# cut -f2 -d";" a | cut -c1
N
t
j
a

```


##### paste

paste [选项] [文件1] [文件2]

paste指令会把每个文件以列对列的方式，一列列的加以合并

*    -d 用指定的间隔字符取代跳格字符
*    -s 串列进行而非平行处理

### 文件的压缩和解压缩

#### 文件的压缩和解压缩

*    gzip gunzip    Linux标准压缩工具，对文本文件压缩可以达到75%的压缩率
*    bzip2 bunzip2  更新的Linux压缩工具，比gzip有更高的压缩率

#### 不解压显示压缩文件的内容

对于gzip压缩的文件
*    zcat   直接显示压缩文件的内容
*    zless  直接逐行显示压缩文件的内容

对于bzip压缩的文件
*    bzcat  直接显示压缩文件的内容
*    bzless 直接逐行显示压缩文件的内容

#### tar包

```
# tar czvf log-`date +%F`.tar.gz /var/log/*
# tar cjvf log-`date +%F`.tar.bz /var/log/*

# tar xzvf log-`date +%F`.tar.gz -C /tmp/
# tar xjvf log-`date +%F`.tar.bz -C /tmp/
```
