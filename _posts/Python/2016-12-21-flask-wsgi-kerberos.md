---
layout: post
title:  "Flask+WSGI+Kerberos"
categories: Python
tags: Flask WSGI Kerberos
---

### 配置Apache 

```
# yum install mod_wsgi mod_auth_kerb 

# vim **.conf
LoadModule wsgi_module modules/mod_wsgi.so
WSGIScriptAlias / /var/www/cgi-bin/test/application

LoadModule auth_kerb_module modules/mod_auth_kerb.so

<Location />
    #SSLRequireSSL
    AuthType Kerberos
    AuthName "Web UI login with kerberos"
    KrbMethodNegotiate on
    KrbMethodK5Passwd on
    KrbServiceName HTTP
    KrbAuthRealm EXAMPLE.COM
    Krb5Keytab /tmp/test.keytab
    KrbSaveCredentials off 
    KrbVerifyKDC on
    Require valid-user
</Location>

#<LocationMatch "(/auth/login|/json/|/admin)">
#</LocationMatch>
```


```
# chmod 644 /tmp/test.keytab        -> to fix the following error
[auth_kerb:error] [pid 1107] [client 10.73.62.27:60890] gss_acquire_cred() failed: Unspecified GSS failure.  Minor code may provide more information (, Permission denied)


# yum install -y krb5*
...
Dependencies Resolved
=========================================================================================================================================================================================
 Package                                       Arch                                Version                                         Repository                                       Size
=========================================================================================================================================================================================
Installing:
 krb5-pkinit                                   x86_64                              1.14.1-27.el7_3                                 rhel-7-server-rpms                              158 k
 krb5-server                                   x86_64                              1.14.1-27.el7_3                                 rhel-7-server-rpms                              977 k
 krb5-server-ldap                              x86_64                              1.14.1-27.el7_3                                 rhel-7-server-rpms                              186 k
 krb5-workstation                              x86_64                              1.14.1-27.el7_3                                 rhel-7-server-rpms                              772 k
Installing for dependencies:
 libverto-tevent                               x86_64                              0.2.5-4.el7                                     rhel-7-server-rpms                              9.0 k
 words                                         noarch                              3.0-22.el7                                      rhel-7-server-rpms                              1.4 M

Transaction Summary
=========================================================================================================================================================================================
Install  4 Packages (+2 Dependent packages)
...

# vim /etc/krb5.conf
[logging]
default = FILE:/var/log/krb5libs.log
kdc = FILE:/var/log/krb5kdc.log
admin_server = FILE:/var/log/kadmind.log

[libdefaults]
default_realm = EXAMPLE.COM
dns_lookup_realm = false
dns_lookup_kdc = false
ticket_lifetime = 24h
renew_lifetime = 7d
forwardable = yes
allow_weak_crypto = yes

[realms]
EXAMPLE.COM = {
 kdc = kerberos.corp.example.com:88
 admin_server = kerberos.corp.example.com:749
}

[domain_realm]
.example.com = EXAMPLE.COM
example.com = EXAMPLE.COM

# kinit $username
Password for $username@EXAMPLE.COM: 

# curl -I --negotiate -u : http://test.example.com/                     -> The --negotiate flag flips on SPNego for cURL, and the -u : forces cURL to pick up the authenticated user’s cached ticket from kinit’ing. You can see your cached ticket with klist.
HTTP/1.1 401 Unauthorized
Date: Fri, 30 Dec 2016 06:19:32 GMT
Server: Apache/2.4.6 (Red Hat Enterprise Linux) mod_auth_kerb/5.4 mod_wsgi/3.4 Python/2.7.5
WWW-Authenticate: Negotiate
WWW-Authenticate: Basic realm="Ethel kerberos login"
Content-Type: text/html; charset=iso-8859-1

HTTP/1.1 200 OK
Date: Fri, 30 Dec 2016 06:19:32 GMT
Server: Apache/2.4.6 (Red Hat Enterprise Linux) mod_auth_kerb/5.4 mod_wsgi/3.4 Python/2.7.5
WWW-Authenticate: Negotiate YIGZBgkqhkiG9xIBAgICAG+BiTCBhqADAgEFoQMCAQ+iejB4oAMCARKicQRvB2BCi9xx2Xh6at4YP3M5d8C6rGl/TmvnODXgBckKuX8DNkIJYrGYyKmvKgivJEvVGAlscEdVR/mFAtX2WsFcbGl88HAZjnJ637eToarilMd3SZwK8qsBKt7ZDbQwwJJbA1z7aDrsEAWgZ7alJusM
Content-Length: 58371
Content-Type: text/html; charset=utf-8
```

Kerberos/SASSL/OpenLDAP : GSSAPI Error: Unspecified GSS failure. Minor code may provide more information ()[http://stackoverflow.com/questions/23936099/kerberos-sassl-openldap-gssapi-error-unspecified-gss-failure-minor-code-may]
Single Sign On with Kerberos using Debian and Windows Server 2008 R2[http://blog.stefan-macke.com/2011/04/19/single-sign-on-with-kerberos-using-debian-and-windows-server-2008-r2/]
Part 2: Apache and Kerberos for Django Authentication + Authorization[http://www.roguelynn.com/words/apache-kerberos-for-django/]


### WSGI



### Flask


[How to access Apache Basic Authentication user in Flask](http://stackoverflow.com/questions/20940651/how-to-access-apache-basic-authentication-user-in-flask)
[http basic authentication “log out”](http://stackoverflow.com/questions/4163122/http-basic-authentication-log-out)
