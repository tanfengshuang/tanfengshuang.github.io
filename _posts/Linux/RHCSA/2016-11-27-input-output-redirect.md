---
layout: post
title:  "输入 输出和重定向"
categories: Linux
tags: RHCSA tee
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看


### 错误输出重定向

```
# find / -user qin &> all
# find / -user qin > all 2>&1
```

### 输入结束符

```
# cat > qin << EOF
> 1
> 2
> 3
> 4
> 5
> 6
> EOF
```


```
# cat > file < quit
> hello 
> kitty
> quit
```

### 管道

```
# cat /boot/grub2/grub.cfg | tee file1 | grep -v ^# | tee file2 | grep -v ^$ | tee file3 > newgrub
```

### tee
tee命令用于将数据重定向到文件，另一方面还可以提供一份重定向数据的副本作为后续命令的stdin。简单的说就是把数据重定向到给定文件和屏幕上。
