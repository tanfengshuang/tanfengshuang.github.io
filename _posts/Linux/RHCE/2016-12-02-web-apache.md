---
layout: post
title:  "Web服务的配置与应用"
categories: Linux
tags: RHCE
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### HTTP协议

###### HTTP协议

*    WWW的目的就是使信息更易于获取，而不管它们的地理位置在哪里，当时用超文本作为WWW文档的标准格式后，人们开发了可以快速获取这些超文本文档的协议-HTTP协议，即超文本传输协议
*    HTTP是应用级的协议，主要用于分布式 协作的信息系统。HTTP协议是通用的 无状态的，其系统的建设和传输与数据无关。HTTP也是面向对象的协议，可以用于各种任务，包括名字服务 分布式对象管理 请求方法的扩展 命令等。
*    在Internet上，HTTP通信往往发生在TCP/IP连接上，其默认端口为80，也可以通过其他端口

###### Web服务工作原理

1. Web浏览器使用HTTP命令向一个特定的服务器发出Web页面请求
2. 若该服务器在特定端口（通常是TCP 80端口）处接收到Web页面请求后，就发送一个应答并在客户和服务器之间建立连接
3. 服务器Web查找客户端所需文档，若Web服务器查找到所请求的文档，就会将所有请求的文档传送给Web浏览器。若该文档不存在，则服务器会发送一个相应的错误提示文档给客户端，Web浏览器接收到文档后，就将他显示出来
4. 当客户端浏览完成后，服务器会等待客户端一会儿，超过一定时间客户端没有新请求的话，就服务器就会断开连接


CCNA router and switch
huawei


### Apache服务

*    后台进程：httpd
*    脚本：/usr/lib/systemd/system/httpd.service
*    使用端口：80(http),443(https)	(可以通过/etc/services文件查看所有Liux常用端口)
*    所需PRM包：httpd
*    配置路径：/etc/httpd/conf/httpd.conf
*    默认网站存放地址：/var/www/html/

/etc/httpd/conf/httpd.conf
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
ServerName www.example.com:80
<Directory />
    AllowOverride none
    Require all denied
</Directory>
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
IncludeOptional conf.d/*.conf



```
# 安装apache，并提供服务
# 在qin1上
# yum install -y httpd
# echo basictest > /var/www/html/index.html
# systemctl restart httpd
# systemctl enable httpd
# netstat -antupl | grep httpd
# firewall-cmd --permanent --add-service=http
# firewall-cmd --reload
## 访问http://192.168.100.1


# 软链接网站
# mkdir /local
# echo qintest > /local/index.html
# semanage fcontext -a -t httpd_sys_content '/local(/.*)?'
# restorecon -vvFR /local
# ln -s /local/ /var/www/html/soft
## 访问http://192.168.100.1/soft


# 基于域名的虚拟主机
# mkdir /var/www/qin
# mkdir /var/www/bing
# echo qin > /var/www/qin/index.html
# echo bing > /var/www/bing/index.html

# cp /etc/unbound/local.d/qin.com.conf /etc/unbound/local.d/bing.com.conf
# vim /etc/unbound/local.d/bing.com.conf
:%s/qin/bing/g
# systemctl restart unbound

# vim /usr/share/doc/httpd/httpd-vhosts.conf	-> 模板文件
# vim /etc/httpd/conf.d/0.conf
<VirtualHost 192.168.100.1:80>
    ServerAdmin admin@XXX.com
    DocumentRoot "/var/www/html"
    ServerName 192.168.100.1
    ErrorLog "/var/log/httpd/192.168.100.1-error_log"
    CustomLog "/var/log/httpd/192.168.100.1-access_log" common
</VirtualHost>

# vim /etc/httpd/conf.d/qin.conf
<VirtualHost www.qin.com:80>
    ServerAdmin admin@qin.com
    DocumentRoot "/var/www/qin"
    ServerName www.qin.com
    ErrorLog "/var/log/httpd/www.qin.com-error_log"
    CustomLog "/var/log/httpd/www.qin.com-access_log" common
</VirtualHost>

# /etc/httpd/conf.d/bing.conf
<VirtualHost www.bing.com:80>
    ServerAdmin admin@bing.com
    DocumentRoot "/var/www/bing"
    ServerName www.bing.com
    ErrorLog "/var/log/httpd/www.bing.com-error_log"
    CustomLog "/var/log/httpd/www.bing.com-access_log" common
</VirtualHost>

# systemctl restart httpd
## http://www.qin.com
## http://www.bing.com

# ls /etc/httpd/logs/ 		-> httpd自动生成的log
access_log error_log www.qin.com-error_log www.qin.com-access_log www.bing.com-error_log www.bing.com-access_log

# ls /var/log/httpd/		-> 自己配置生成的log
access_log error_log www.qin.com-error_log www.qin.com-access_log www.bing.com-error_log www.bing.com-access_log

```

### 调用脚本

```
# vim /var/www/cgi-bin/shell.sh
#! /bin/sh
echo -en "Content-Type: text/html;charset=UTF-8\n\n"
date +%c

# vim /var/www/cgi-bin/perl.sh
#! /usr/bin/perl
print "Content-Type: text/html; charset=UTF-8\n\n";
$now=localtime();
print "$now\n";

# vim /var/www/cgi-bin/python.sh
#! /usr/bin/env python
import time
def application(environ, start_response):
    response_body = 'UNIX EPOCH time is now: %s\n" % time.time()
    status = '200 OK'
    response_headers = [('Content-Type', 'text/plain'),
                        ('Content-Length', '1'),
                        ('Content-Length', str(len(response_body)))]
    start_response(status, response_headers)
    return [response_body]

# 给脚本赋予可执行权限，python的脚本不需要，因为它是通过模块调用的
# chmod a+x /var/www/cgi-bin/shell.sh
# chmod a+x /var/www/cgi-bin/perl.sh

# 对于shell和perl的配置
# vim /etc/httpd/conf.d/0.conf
<virtualhost 192.168.100.1:80>
    servername 192.168.100.1
    documentroot /var/www/html
    <ifmodule alias_module>
        scriptalias /jiaoben/ "/var/www/cgi-bin/"
    </ifmodule>
<virtualhost>

# 对于python脚本，需要先安装模块mod_wsgi
# yum install mod_wsgi
# vim /etc/httpd/conf.d/0.conf
<virtualhost 192.168.100.1:80>
    servername 192.168.100.1
    documentroot /var/www/html
    wsgiscriptalias /python /var/www/cig-bin/
<virtualhost>
```

### allow,deny

```
# 全部可以通行
Order deny,allow
all from all
deny from 219.204.253.8

# 全部可以通行
Order deny, allow
deny from 219.204.253.8
allow from all

# 只有219.204.253.8不能通行
Order allow,deny
deny from 219.204.253.8
allow from all

# 只有219.204.253.8不能通行
Order allow,deny
allow from all
deny from 219.204.253.8

```

### SSL加密 - Secure Socket Layer

```
# yum install mod_ssl
# cd /etc/pki/tls/certs/
# make qin.cert
# ls
qin.crt qin.key
# pwd
/etc/pki/tls/certs/
# mv qin.key ../private/
# vim /etc/httpd/conf.d/ssl.conf
Listen 443 https		-> https的监听端口443在ssl.conf文件中已经配置好，不用单独配置了
SSLEngine off			-> 将ssl.conf配置文件中的SSLEngine改为off
# cat /etc/httpd/conf.d/ssl.conf | grep ^SSL
SSLPassPhraseDialog exec:/usr/libexec/httpd-ssl-pass-dialog
SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
SSLSessionCacheTimeout  300
SSLRandomSeed startup file:/dev/urandom  256
SSLRandomSeed connect builtin
SSLCryptoDevice builtin
SSLEngine off
SSLProtocol all -SSLv3
SSLProxyProtocol all -SSLv3
SSLHonorCipherOrder on
SSLCipherSuite PROFILE=SYSTEM
SSLProxyCipherSuite PROFILE=SYSTEM
SSLCertificateFile /etc/pki/tls/certs/localhost.crt
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
# cat /etc/httpd/conf.d/ssl.conf | grep ^SSL | tail -n8 | grep -v Proxy
SSLEngine off
SSLProtocol all -SSLv3
SSLCipherSuite PROFILE=SYSTEM
SSLCertificateFile /etc/pki/tls/certs/localhost.crt
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
# vim /etc/httpd/conf.d/0.conf
<virtualhost 192.168.100.1:80>
    servername 192.168.100.1
    documentroot /var/www/html
    SSLEngine on
    SSLProtocol all -SSLv3
    SSLCipherSuite PROFILE=SYSTEM
    SSLCertificateFile /etc/pki/tls/certs/qin.crt
    SSLCertificateKeyFile /etc/pki/tls/private/qin.key
<virtualhost>
# systemctl restart httpd    -> 需要输入刚才设置的密码
```


### LAMP架构

*    dede织梦CMS - www.dedecms.com
*    phpwind官方论坛
*    discuz官方站


```
# yum install -y php* mariadb*
# unzip Discuz.zip
# rm -rf /var/www/qin/*
# cp -rf upload/* /var/www/qin/
# semanage fcontext -l | grep http | grep rw
# chcon -R -t httpd_sys_rw_content_t /var/www/qin/
# chown apache:apche /var/www/qin/
# systemctl restart mariadb
# systemctl enable mariadb
# mysqladmin -u root password '123456'
# systemctl restart httpd

## http://www.qin.com	-> 需要输入数据库密码123456，更改表前缀
```

```
# 安装php后，在下面配置文件中有这样的配置，所以可以自动识别index.php
# vim /etc/httpd/conf.d/php.conf
DirectoryIndex index.php
```




