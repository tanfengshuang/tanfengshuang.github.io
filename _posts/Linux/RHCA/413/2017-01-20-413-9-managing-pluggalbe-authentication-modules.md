---
layout: post
title:  "Managing Pluggalbe Authentication Modules(413-9)"
categories: Linux
tags: 413 cracklib.so pam_tally2.so
---

### PAM Syntax And Configuration

AM（Pluggable Authentication Modules ）是由Sun提出的一种认证机制。它通过提供一些动态链接库和一套统一的API，将系统提供的服务和该服务的认证方式分开，使得系统管理员可以灵活地根据需要给不同的服务配置不同的认证方式而无需更改服务程序，同时也便于向系统中添加新的认证手段。 PAM模块是一种嵌入式模块，修改后即时生效。

###### Pluggable Authentication Modules

pam的重要文件：
*    /usr/lib64/libpam*         ## PAM核心库
*    /etc/pam.d/*               ## PAM各个模块的配置文件
*    /etc/security              ## Additional supplementary configuration files
*    /lib64/security/pam_*.so   ## 可动态加载的PAM模块

```
# rpm -qf `which sshd`
openssh-server-5.3p1-117.el6.x86_64
# rpm -ql openssh-server | grep pam
/etc/pam.d/ssh-keycat
/etc/pam.d/sshd

# ls /usr/lib64/libpam*
/usr/lib64/libpamc.so  /usr/lib64/libpam_misc.so  /usr/lib64/libpam.so

# ls /etc/pam.d/
abrt-cli-root  crond             fingerprint-auth-ac  passwd            reboot        runuser-l          smtp.postfix   subscription-manager  system-auth-ac
atd            cups              halt                 password-auth     remote        setup              smtp.sendmail  sudo                  system-config-network
chfn           cvs               login                password-auth-ac  rhn_register  smartcard-auth     sshd           sudo-i                system-config-network-cmd
chsh           eject             newrole              polkit-1          run_init      smartcard-auth-ac  ssh-keycat     su-l
config-util    fingerprint-auth  other                poweroff          runuser       smtp               su             system-auth

# ls /etc/security/
access.conf  console.apps      console.perms    group.conf   limits.d        namespace.d     opasswd       sepermit.conf
chroot.conf  console.handlers  console.perms.d  limits.conf  namespace.conf  namespace.init  pam_env.conf  time.conf

# ls /lib64/security/
pam_access.so        pam_deny.so       pam_filter.so   pam_limits.so     pam_namespace.so   pam_rootok.so          pam_succeed_if.so  pam_unix_auth.so     pam_xauth.so
pam_cap.so           pam_echo.so       pam_fprintd.so  pam_listfile.so   pam_nologin.so     pam_securetty.so       pam_tally2.so      pam_unix_passwd.so
pam_chroot.so        pam_env.so        pam_ftp.so      pam_localuser.so  pam_passwdqc.so    pam_selinux_permit.so  pam_time.so        pam_unix_session.so
pam_ck_connector.so  pam_exec.so       pam_group.so    pam_loginuid.so   pam_permit.so      pam_selinux.so         pam_timestamp.so   pam_unix.so
pam_console.so       pam_faildelay.so  pam_issue.so    pam_mail.so       pam_postgresok.so  pam_sepermit.so        pam_tty_audit.so   pam_userdb.so
pam_cracklib.so      pam_faillock.so   pam_keyinit.so  pam_mkhomedir.so  pam_pwhistory.so   pam_shells.so          pam_umask.so       pam_warn.so
pam_debug.so         pam_filter        pam_lastlog.so  pam_motd.so       pam_rhosts.so      pam_stress.so          pam_unix_acct.so   pam_wheel.so


# cat /etc/pam.d/sshd
#PAM-1.0
auth       required     pam_sepermit.so
auth       include      password-auth
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth

# locate pam_nologin.so
/lib64/security/pam_nologin.so

# vim /etc/pam.d/password-auth
```

###### /etc/pam.d/ Configuration File Syntax

>    PAM配置文件(/etc/pam.d/*)的每一行的格式：Module-type   Control-flag   Module-path   Arguments 

PAM Rule Types

*    auth: 表示鉴别类接口模块类型, 用于检查用户和密码，并分配权限; 这种类型的模块为用户验证提供两方面服务。让应用程序提示用户输入密码或者其他标记，确认用户合法性；通过他的凭证许可权限，设定组成员关系或者其他优先权。
*    account: 表示账户类接口，主要负责账户合法性检查，确认帐号是否过期，是否有权限登录系统等；这种模块执行的是基于非验证的帐号管理。他主要用于限制/允许用户对某个服务的访问时间，当前有效的系统资源（最多可以多少用户），限制用户位置（例如：root只能通过控制台登录）。
*    password: 口令类接口。控制用户更改密码的全过程。也就是有些资料所说的升级用户验证标记。
*    session: 会话类接口。实现从用户登录成功到退出的会话控制；处理为用户提供服务之前/后需要做的些事情。包括：开启/关闭交换数据的信息，监视目录等，设置用户会话环境等。也就是说这是在系统正式进行服务提供之前的最后一道关口。

Common PAM Controls

*    required： 所有带 required标记的模块全部成功后，该程序才能通过鉴别。如果出现了错误，PAM并不立刻将错误消息返回给应用程序，而是在所有模块都调用完毕后才将错误消息返回调用他的程序。这样做的目的就是不让用户知道自己被哪个模块拒绝，通过一种隐蔽的方式来保护系统服务。就像设置防火墙规则的时候将拒绝类的规则都设置为drop一样，以致于用户在访问网络不成功的时候无法准确判断到底是被拒绝还是目标网络不可达。
*    requisite： 只有带此标记的模块返回成功后，用户才能通过鉴别。一旦失败就不再执行堆中后面的其他模块，并且鉴别过程到此结束，同时也会立即返回错误信息
*    sufficient： 只要标记为sufficient的模块一旦验证成功，那么PAM便立即向应用程序返回成功结果而不必尝试任何其他模块。即便后面的层叠模块使用了requisite或者required控制标志也是一样。当标记为sufficient的模块失败时，sufficient模块会当做 optional对待。因此拥有sufficient标志位的配置项在执行验证出错的时候并不会导致整个验证失败
*    optional： 表示即便该行所涉及的模块验证失败用户仍能通过认证。在PAM体系中，带有该标记的模块失败后将继续处理下一模块。也就是说即使本行指定的模块验证失败，也允许用户享受应用程序提供的服务。使用该标志，PAM框架会忽略这个模块产生的验证错误，继续顺序执行下一个层叠模块。
*    include： 表示在验证过程中调用其他的PAM配置文件。在RHEL系统中有相当多的应用通过完整调用/etc/pam.d/system-auth来实现认证而不需要重新逐一去写配置项。

```
多数情况下auth和account会一起用来对用户登录和使用服务的情况进行限制。这样的限制会更加完整。比如下面是一个具体的例子：login是一个应用程序。Login要完成两件工作——首先查询用户，然后为用户提供所需的服务，例如提供一个shell程序。通常Login要求用户输入名称和密码进行验证。当用户名输入的时候，系统自然会去比对该用户是否是一个合法用户，是否在存在于本地或者远程的用户数据库中。如果该账号确实存在，那么是否过期。这些个工作是由account接口来负责。
如果用户满足上述登录的前提条件，那么他是否具有可登录系统的口令，口令是否过期等。这个工作就要由auth接口来负责了，他通常会将用户口令信息加密并提供给本地（/etc/shadow）或者远程的(ldap，kerberos等)口令验证方式进行验证。
如果用户能够登录成功，证明auth和account的工作已经完成。但整个验证过程并没有完全结束。因为还有一些其他的问题没有得到确认。例如，用户能够在服务器上同时开启多少个窗口登录，用户可以在登录之后使用多少终端多长时间，用户能够访问哪些资源和不能访问哪些资源等等。也就是说登录之后的后续验证和环境定义等还需要其他的接口。这就要用到session和password了

# which login
/bin/login
# rpm -qf /bin/login
util-linux-ng-2.17.2-12.24.el6.x86_64
# rpm -ql util-linux-ng | grep pam
/etc/pam.d/chfn
/etc/pam.d/chsh
/etc/pam.d/login
/etc/pam.d/remote

# cat /etc/pam.d/login
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
auth       include      system-auth
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      system-auth
-session   optional     pam_ck_connector.so

```

### PAM Documentation

*   /usr/share/doc/pam-*
*   man -k pam_
*   man -k keyword  - Display man pages containing keyword

```
# ls /usr/share/doc/pam-1.1.1/
Copyright  html  Linux-PAM_SAG.txt  rfc86.0.txt  txts

# man man
-k     Equivalent to apropos.

# man apropos
apropos(1)                                                          apropos(1)
NAME
       apropos - search the whatis database for strings
SYNOPSIS
       apropos keyword ...

# man -k pam_       -> 显示man手册中所有包含pam_的命令或说明文件,并列条显示
group.conf [group]   (5)  - configuration file for the pam_group module
limits.conf [limits] (5)  - configuration file for the pam_limits module
pam_access           (8)  - PAM module for logdaemon style login access control
pam_acct_mgmt        (3)  - PAM account validation management
pam_authenticate     (3)  - account authentication
pam_chauthtok        (3)  - updating authentication tokens
pam_ck_connector     (8)  - Register session with ConsoleKit
pam_close_session    (3)  - terminate PAM session management
pam_console          (8)  - determine user owning the system console
pam_console_apply    (8)  - set or revoke permissions for users at the system console
pam_conv             (3)  - PAM conversation function
pam_cracklib         (8)  - PAM module to check the password against dictionary words
pam_debug            (8)  - PAM module to debug the PAM stack
pam_deny             (8)  - The locking-out PAM module
pam_echo             (8)  - PAM module for printing text messages
pam_ecryptfs         (8)  - PAM module for eCryptfs
pam_end              (3)  - termination of PAM transaction
pam_env              (8)  - PAM module to set/unset environment variables
pam_env.conf [pam_env] (5)  - the environment variables config file
pam_env [environment] (5)  - PAM module to set/unset environment variables
pam_error            (3)  - display error messages to the user
pam_exec             (8)  - PAM module which calls an external command
pam_fail_delay       (3)  - request a delay on failure
pam_faildelay        (8)  - Change the delay on failure per-application
pam_faillock         (8)  - Module counting authentication failures during a specified interval
pam_filter           (8)  - PAM filter module
pam_ftp              (8)  - PAM module for anonymous access module
pam_get_authtok      (3)  - get authentication token
pam_get_data         (3)  - get module internal data
pam_getenv           (3)  - get a PAM environment variable
pam_getenvlist       (3)  - getting the PAM environment
pam_get_item         (3)  - getting PAM informations
pam_get_user         (3)  - get user name
pam_group            (8)  - PAM module for group access
pam_info             (3)  - display messages to the user
pam_issue            (8)  - PAM module to add issue file to user prompt
pam_keyinit          (8)  - Kernel session keyring initialiser module
pam_lastlog          (8)  - PAM module to display date of last login and perform inactive account lock out
pam_limits           (8)  - PAM module to limit resources
pam_listfile         (8)  - deny or allow services based on an arbitrary file
pam_localuser        (8)  - require users to be listed in /etc/passwd
pam_loginuid         (8)  - Record user's login uid to the process attribute
pam_mail             (8)  - Inform about available mail
pam_misc_drop_env    (3)  - liberating a locally saved environment
pam_misc_paste_env   (3)  - transcribing an environment to that of PAM
pam_misc_setenv      (3)  - BSD like PAM environment variable setting
pam_mkhomedir        (8)  - PAM module to create users home directory
pam_motd             (8)  - Display the motd file
pam_namespace        (8)  - PAM module for configuring namespace for a session
pam_nologin          (8)  - Prevent non-root users from login
pam_open_session     (3)  - start PAM session management
pam_passwdqc         (8)  - Password quality-control PAM module
pam_permit           (8)  - The promiscuous module
pam_postgresok       (8)  - simple check of real UID and corresponding account name
pam_prompt           (3)  - interface to conversation function
pam_putenv           (3)  - set or change PAM environment variable
pam_pwhistory        (8)  - PAM module to remember last passwords
pam_rhosts           (8)  - The rhosts PAM module
pam_rootok           (8)  - Gain only root access
pam_securetty        (8)  - Limit root login to special devices
pam_selinux          (8)  - PAM module to set the default security context
pam_sepermit         (8)  - PAM module to allow/deny login depending on SELinux enforcement state
pam_setcred          (3)  - establish / delete user credentials
pam_set_data         (3)  - set module internal data
pam_set_item         (3)  - set and update PAM informations
pam_shells           (8)  - PAM module to check for valid login shell
pam_sm_acct_mgmt     (3)  - PAM service function for account management
pam_sm_authenticate  (3)  - PAM service function for user authentication
pam_sm_chauthtok     (3)  - PAM service function for authentication token management
pam_sm_close_session (3)  - PAM service function to terminate session management
pam_sm_open_session  (3)  - PAM service function to start session management
pam_sm_setcred       (3)  - PAM service function to alter credentials
pam_start            (3)  - initialization of PAM transaction
pam_strerror         (3)  - return string describing PAM error code
pam_succeed_if       (8)  - test account characteristics
pam_syslog           (3)  - send messages to the system logger
pam_tally2           (8)  - The login counter (tallying) module
pam_time             (8)  - PAM module for time control access
pam_timestamp        (8)  - Authenticate using cached successful authentication attempts
pam_timestamp_check  (8)  - Check to see if the default timestamp is valid
pam_tty_audit        (8)  - Enable or disable TTY auditing for specified users
pam_umask            (8)  - PAM module to set the file mode creation mask
pam_unix             (8)  - Module for traditional password authentication
pam_userdb           (8)  - PAM module to authenticate against a db database
pam_verror [pam_error] (3)  - display error messages to the user
pam_vinfo [pam_info] (3)  - display messages to the user
pam_vprompt [pam_prompt] (3)  - interface to conversation function
pam_vsyslog [pam_syslog] (3)  - send messages to the system logger
pam_warn             (8)  - PAM module which logs all PAM items if called
pam_wheel            (8)  - Only permit root access to members of group wheel
pam_xauth            (8)  - PAM module to forward xauth keys between users
pam_xauth_data       (3)  - structure containing X authentication data
sepermit.conf [sepermit] (5)  - configuration file for the pam_sepermit module
time.conf [time]     (5)  - configuration file for the pam_time module

# man 8 pam_cracklib
# man 8 pam_deny
# man 8 pam_permit
```

### Configure Password Requirement through PAM

###### Open Lab: Configure a Password Length and Complexity Policy

PAM Module: pam_cracklib.so

pam_cracklib是一个PAM模块，用来检查密码是否违反密码字典，这个验证模块可以通过插入password堆栈，为特殊的应用提供可插入式密码强度性检测。 它的工作方式就是先提示用户输入密码，然后使用一个系统字典和一套规则来检测输入的密码是否不能满足强壮性要求。密码的强度检测分二次进行，第一次只是检测密码是否是提供的对比字典中的一部分，如果检测结果是否定的，那么就会提供一些附加的检测来进一步检测其强度，例如检测新密码中的字符占旧密码字符的比例，密码的长度，所用字符大小写状况，以及是否使用了特殊字符等等。(libpam-cracklib)

*    debug：将debug信息写入syslog
*    type=XXX：提示输入密码的文本内容。默认是"New UNIX password: " and "Retype UNIX password: "，可自定
*    retry=N：用户最多可以几次输入密码后报错。默认是1次。
*    difok=N：新密码有几个字符不能和旧密码相同，默认是5个。另外如果新密码有1/2的字符于旧不同，也会被接受。
*    diginore=N：默认当新密码有23个字符时，difok选项会被忽略。
*    minlen=N：最小密码长度。
*    dcredit=N：当N>=0时，N代表新密码最多可以有多少个阿拉伯数字。当N<0时，N代表新密码最少要有多少个阿拉伯数字。
*    ucredit=N：和dcredit差不多，但是这里说的是大写字母。
*    lcredit=N：和dcredit差不多，但是这里说的是小写字母。
*    ocredit=N：和dcredit差不多，但是这里说的是特殊字符。
*    use_authtok：在某个与密码相关的验证模块后使用此选项，例如pam_unix.so验证模块 

```
# grep ^password /etc/pam.d/sshd
password   include      password-auth
# grep ^password /etc/pam.d/login
password   include      system-auth
# grep ^password /etc/pam.d/gdm
password   include      system-auth

# grep ^password /etc/pam.d/system-auth
password    requisite     pam_cracklib.so try_first_pass retry=3 type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so

# grep ^password /etc/pam.d/password-auth
password    requisite     pam_cracklib.so try_first_pass retry=3 type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so

# vim /etc/pam.d/system-auth
password    requisite     pam_cracklib.so try_first_pass retry=3 type=TTT ocredit=-1 dcredit=-1 ucredit==-1 minlen=8

# vim /etc/pam.d/password-auth
password    requisite     pam_cracklib.so try_first_pass retry=3 type=TTT ocredit=-1 dcredit=-1 ucredit==-1 minlen=8

# passwd u1                 -> 123.comT
Changing password for user u1.
New TTT password: 
Retype new TTT password: 
passwd: all authentication tokens updated successfully.
```

### Supplementary PAM Configuration

*    pam_time.so use /etc/security/time.conf
*    pam_limits.so use /etc/security/limits.conf

```
# ls /etc/security
access.conf  console.apps      console.perms    group.conf   limits.d        namespace.d     opasswd       sepermit.conf
chroot.conf  console.handlers  console.perms.d  limits.conf  namespace.conf  namespace.init  pam_env.conf  time.conf
```

###### Proformance Checklist: Apply Limits to Users

```
# grep session /etc/pam.d/sshd 
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth

# grep session /etc/pam.d/password-auth
session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so


# vim /etc/security/limits.conf
#<domain>      <type>  <item>         <value>
test            hard   maxlogins        2

# ps -aux |grep test
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      3500  0.0  0.0 103320   824 pts/2    S+   07:01   0:00 grep test
# ssh test@...
Last login: Sun Jan 22 06:59:43 2017 from 10.66.129.74

# ps -aux |grep test
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      3501  0.8  0.2 102092  4032 ?        Ss   07:01   0:00 sshd: test [priv]
test      3505  0.2  0.0 102092  1848 ?        S    07:01   0:00 sshd: test@pts/3 
test      3506  0.5  0.0 108356  1732 pts/3    Ss+  07:01   0:00 -bash
root      3528  0.0  0.0 103320   824 pts/2    S+   07:01   0:00 grep test
# ssh test@...
Last login: Sun Jan 22 07:01:26 2017 from 10.66.129.74

# ps -aux |grep test
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      3501  0.4  0.2 102092  4032 ?        Ss   07:01   0:00 sshd: test [priv]
test      3505  0.0  0.0 102092  1848 ?        S    07:01   0:00 sshd: test@pts/3 
test      3506  0.1  0.0 108356  1732 pts/3    Ss+  07:01   0:00 -bash
root      3529  0.9  0.2 102092  4028 ?        Ss   07:01   0:00 sshd: test [priv]
test      3533  0.4  0.0 102092  1844 ?        S    07:01   0:00 sshd: test@pts/4 
test      3534  0.4  0.0 108356  1724 pts/4    Ss+  07:01   0:00 -bash
root      3556  0.0  0.0 103320   820 pts/2    S+   07:01   0:00 grep test

# ssh test@...
Connection to cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com closed.

# tail -f /var/log/secure
Jan 22 07:01:25 cloud-qe-16-vm-02 sshd[3501]: pam_unix(sshd:session): session opened for user test by (uid=0)
Jan 22 07:01:51 cloud-qe-16-vm-02 sshd[3529]: Accepted password for test from 10.66.129.74 port 34624 ssh2
Jan 22 07:01:51 cloud-qe-16-vm-02 sshd[3529]: pam_unix(sshd:session): session opened for user test by (uid=0)
Jan 22 07:02:15 cloud-qe-16-vm-02 sshd[3557]: Accepted password for test from 10.66.129.74 port 34630 ssh2
Jan 22 07:02:15 cloud-qe-16-vm-02 sshd[3557]: pam_limits(sshd:session): Too many logins (max 2) for test
Jan 22 07:02:15 cloud-qe-16-vm-02 sshd[3557]: pam_unix(sshd:session): session opened for user test by (uid=0)
Jan 22 07:02:15 cloud-qe-16-vm-02 sshd[3557]: error: PAM: pam_open_session(): Permission denied
Jan 22 07:02:16 cloud-qe-16-vm-02 sshd[3561]: Received disconnect from 10.66.129.74: 11: disconnected by user
```

```
# vim /etc/security/limits.conf
#<domain>      <type>  <item>         <value>
test            hard    nproc           5

# ps -aux | grep ^test
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
test      3505  0.0  0.0 102092  1848 ?        S    07:01   0:00 sshd: test@pts/3 
test      3506  0.0  0.0 108356  1732 pts/3    Ss+  07:01   0:00 -bash
test      3533  0.0  0.0 102092  1848 ?        S    07:01   0:00 sshd: test@pts/4 
test      3534  0.0  0.0 108356  1732 pts/4    Ss+  07:01   0:00 -bash

# ssh test@...
shell request failed on channel 0

# tail -f /var/log/secure
Jan 22 07:09:51 cloud-qe-16-vm-02 sshd[3579]: Accepted password for test from 10.66.129.74 port 34704 ssh2
Jan 22 07:09:51 cloud-qe-16-vm-02 sshd[3579]: pam_unix(sshd:session): session opened for user test by (uid=0)
Jan 22 07:09:52 cloud-qe-16-vm-02 sshd[3583]: error: do_exec_pty: fork: Resource temporarily unavailable
Jan 22 07:09:52 cloud-qe-16-vm-02 sshd[3579]: pam_unix(sshd:session): session closed for user test
```

###### ulimit

*    -H 设置硬资源限制.
*    -S 设置软资源限制.
*    -a 显示当前所有的资源限制.
*    -c size:设置core文件的最大值.单位:blocks
*    -d size:设置数据段的最大值.单位:kbytes
*    -f size:设置创建文件的最大值.单位:blocks
*    -l size:设置在内存中锁定进程的最大值.单位:kbytes
*    -m size:设置可以使用的常驻内存的最大值.单位:kbytes
*    -n size:设置内核可以同时打开的文件描述符的最大值.单位:n
*    -p size:设置管道缓冲区的最大值.单位:kbytes
*    -s size:设置堆栈的最大值.单位:kbytes
*    -t size:设置CPU使用时间的最大上限.单位:seconds
*    -v size:设置虚拟内存的最大值.单位:kbytes
*    -u <程序数目> 　用户最多可开启的程序数目

```
# ulimit 
unlimited

# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7397
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7397
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```


### Lock Accounts With Multiple Failed Logins

```
# man pam_tally2
NAME
       pam_tally2 - The login counter (tallying) module

SYNOPSIS
       pam_tally2.so [file=/path/to/counter] [onerr=[fail|succeed]] [magic_root] [even_deny_root] [deny=n] [lock_time=n] [unlock_time=n] [root_unlock_time=n]
                     [serialize] [audit] [silent] [no_log_info] [debug]

       pam_tally2 [--file /path/to/counter] [--user username] [--reset[=n]] [--quiet]
```

全局参数

*    file       用于指定统计次数存放的位置，默认保存在/var/log/tallylog文件中；
*    onerr   当意外发生时，返加PAM_SUCCESS或pam错误代码，一般该项不进行配置；
*    audit   如果登录的用户不存在，则将访问信息写入系统日志；
*    silent   静默模式，不输出任何日志信息；
*    no_log_info 不打印日志信息通过syslog

认证选项

*    deny  指定最大几次认证错误，如果超出此错误，将执行后面的策略。如锁定N秒，如果后面没有其他策略指定时，默认永远锁定，除非手动解锁。
*    lock_time  锁定多长时间，按秒为单位；
*    unlock_time 指定认证被锁后，多长时间自动解锁用户；
*    magic_root 如果用户uid＝0（即root账户或相当于root的帐户）在帐户认证时调用该模块发现失败时，不计入统计；
*    no_lock_time 不使用.fail_locktime项在/var/log/faillog 中记录用户 －－－按英文直译不太明白，个人理解即不进行用户锁定；
*    even_deny_root root用户在认证出错时，一样被锁定（该功能慎用，搞不好就要单用户时解锁了）
*    root_unlock_time  root用户在失败时，锁定多长时间。该选项一般是配合even_deny_root 一起使用的。

解锁与查看失败

*    pam_tally2 --user test               ## 查看test用户登录的错误次数及详细信息
*    pam_tally2 --user test --reset       ## 清空test用户的错误登录次数，即手动解锁, 使用faillog -r命令也可以进行解锁

*    /etc/pam.d/login中配置只在本地文本终端上做限制；
*    /etc/pam.d/kde在配置时在kde图形界面调用时限制；
*    /etc/pam.d/sshd中配置时在通过ssh连接时做限制；
*    /etc/pam.d/system-auth中配置凡是调用 system-auth 文件的服务，都会生效。

> pam_tally2与pam_tally模块的区别是前者增加了自动解锁时间的功能，后者没有。所以在老的发行版中，如果使用了pam_tally模块时，可以使用pam_tally 、faillog配合crontab 进行自动解锁。

```
# vim /etc/pam.d/system-auth
auth        required      pam_tally2.so deny=3 unlock_time=180
account     required      pam_tally2.so

# vim /etc/pam.d/password-auth
auth        required      pam_tally2.so deny=3 unlock_time=180
account     required      pam_tally2.so

# pam_tally2 --user test 
Login           Failures Latest failure     From
test                0

# ssh test@...          -> 输入错误密码3次后，以后再输入正确密码也不能成功登录了

# pam_tally2 --user test 
Login           Failures Latest failure     From
test                5    01/22/17 07:33:10  10.66.129.74

# pam_tally2 --user test --reset
Login           Failures Latest failure     From
test                5    01/22/17 07:33:02  10.66.129.74

# tail -f /var/log/secure
Jan 22 07:30:52 cloud-qe-16-vm-02 sshd[3650]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.66.129.74  user=test
Jan 22 07:30:54 cloud-qe-16-vm-02 sshd[3650]: Failed password for test from 10.66.129.74 port 35104 ssh2
Jan 22 07:31:05 cloud-qe-16-vm-02 unix_chkpwd[3656]: password check failed for user (test)
Jan 22 07:31:07 cloud-qe-16-vm-02 sshd[3650]: Failed password for test from 10.66.129.74 port 35104 ssh2
Jan 22 07:31:14 cloud-qe-16-vm-02 unix_chkpwd[3658]: password check failed for user (test)
Jan 22 07:31:15 cloud-qe-16-vm-02 sshd[3650]: Failed password for test from 10.66.129.74 port 35104 ssh2
Jan 22 07:31:16 cloud-qe-16-vm-02 sshd[3651]: Connection closed by 10.66.129.74
Jan 22 07:31:16 cloud-qe-16-vm-02 sshd[3650]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.66.129.74  user=test
Jan 22 07:32:41 cloud-qe-16-vm-02 sshd[3660]: pam_tally2(sshd:auth): user test (500) tally 4, deny 3
Jan 22 07:32:41 cloud-qe-16-vm-02 unix_chkpwd[3662]: password check failed for user (test)
Jan 22 07:32:41 cloud-qe-16-vm-02 sshd[3660]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.66.129.74  user=test
Jan 22 07:32:43 cloud-qe-16-vm-02 sshd[3660]: Failed password for test from 10.66.129.74 port 35122 ssh2
Jan 22 07:33:02 cloud-qe-16-vm-02 sshd[3660]: pam_tally2(sshd:auth): user test (500) tally 5, deny 3
Jan 22 07:33:03 cloud-qe-16-vm-02 sshd[3660]: Failed password for test from 10.66.129.74 port 35122 ssh2

# pam_tally2 --user test
Login           Failures Latest failure     From
test                0
```

###### Workshop: Lock Accounts with Failed Logins

### Uinit Test: Obey My Security Policy

*    PAM Module: pam_cracklib.so
*    PAM Module: pam_tally2.so



### PAM的工作原理与流程

当某一个有认证需求的应用程序需要验证的时候，一般在应用程序中就会定义负责对其认证的PAM配置文件。以vsftpd为例，在它的配置文件/etc/vsftpd/vsftpd.conf中就有这样一行定义：

```
pam_service_name=vsftpd 
```

表示登录FTP服务器的时候进行认证是根据/etc/pam.d/vsftpd文件定义的内容进行。

那么，当程序需要认证的时候已经找到相关的pam配置文件，认证过程是如何进行的？下面我们将通过解读/etc/pam.d/system-auth文件予以说明。

首先要声明一点的是：system-auth是一个非常重要的pam配置文件，主要负责用户登录系统的认证工作。而且该文件不仅仅只是负责用户登录系统认证，其它的程序和服务通过include接口也可以调用到它，从而节省了很多重新自定义配置的工作。所以应该说该文件是系统安全的总开关和核心的pam配置文件。

下面是/etc/pam.d/system-auth文件的全部内容：

```
# cat /etc/pam.d/system-auth | grep -v "^#"
auth        required      pam_env.so
auth        sufficient    pam_fprintd.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 500 quiet
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 500 quiet
account     required      pam_permit.so

password    requisite     pam_cracklib.so try_first_pass retry=3 type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so

```

第一部分表示，当用户登录的时候，首先会通过auth类接口对用户身份进行识别和密码认证。所以在该过程中验证会经过几个带auth的配置项。

其中的第一步是通过pam_env.so模块来定义用户登录之后的环境变量， pam_env.so允许设置和更改用户登录时候的环境变量，默认情况下，若没有特别指定配置文件，将依据/etc/security/pam_env.conf进行用户登录之后环境变量的设置。

然后通过pam_unix.so模块来提示用户输入密码，并将用户密码与/etc/shadow中记录的密码信息进行对比，如果密码比对结果正确则允许用户登录，而且该配置项的使用的是“sufficient”控制位，即表示只要该配置项的验证通过，用户即可完全通过认证而不用再去走下面的认证项。不过在特殊情况下，用户允许使用空密码登录系统，例如当将某个用户在/etc/shadow中的密码字段删除之后，该用户可以只输入用户名直接登录系统。

下面的配置项中，通过pam_succeed_if.so对用户的登录条件做一些限制，表示允许uid大于500的用户在通过密码验证的情况下登录，在Linux系统中，一般系统用户的uid都在500之内，所以该项即表示允许使用useradd命令以及默认选项建立的普通用户直接由本地控制台登录系统。

最后通过pam_deny.so模块对所有不满足上述任意条件的登录请求直接拒绝，pam_deny.so是一个特殊的模块，该模块返回值永远为否，类似于大多数安全机制的配置准则，在所有认证规则走完之后，对不匹配任何规则的请求直接拒绝。

第二部分的三个配置项主要表示通过account账户类接口来识别账户的合法性以及登录权限。

第一行仍然使用pam_unix.so模块来声明用户需要通过密码认证。第二行承认了系统中uid小于500的系统用户的合法性。之后对所有类型的用户登录请求都开放控制台。

第三部分会通过password口另类接口来确认用户使用的密码或者口令的合法性。第一行配置项表示需要的情况下将调用pam_cracklib来验证用户密码复杂度。如果用户输入密码不满足复杂度要求或者密码错，最多将在三次这种错误之后直接返回密码错误的提示，否则期间任何一次正确的密码验证都允许登录。需要指出的是，pam_cracklib.so是一个常用的控制密码复杂度的pam模块，关于其用法举例我们会在之后详细介绍。之后带pam_unix.so和pam_deny.so的两行配置项的意思与之前类似。都表示需要通过密码认证并对不符合上述任何配置项要求的登录请求直接予以拒绝。不过用户如果执行的操作是单纯的登录，则这部分配置是不起作用的。

第四部分主要将通过session会话类接口为用户初始化会话连接。其中几个比较重要的地方包括，使用pam_keyinit.so表示当用户登录的时候为其建立相应的密钥环，并在用户登出的时候予以撤销。不过该行配置的控制位使用的是optional，表示这并非必要条件。之后通过pam_limits.so限制用户登录时的会话连接资源，相关pam_limit.so配置文件是/etc/security/limits.conf，默认情况下对每个登录用户都没有限制。关于该模块的配置方法在后面也会详细介绍。

可见，不同应用程序通过配置文件在认证过程中调用不同的pam模块来定制具体的认证流程。其中我们不难看出，其实可以根据实际的需要对pam的配置文件进行修改以满足不同的认证需求，例如下面的例子：

```
auth    required    pam_env.so
auth    required    pam_tally.so onerr=fail deny=5
auth    sufficient  pam_unix.so nullok try_first_pass
auth    requisite   pam_succeed_if.so uid >= 500 quiet
auth    required    pam_deny.so

account required    pam_unix.so
account sufficient  pam_succeed_if.so uid < 500 quiet
account required    pam_permit.so

password    requisite pam_cracklib.so try_first_pass retry=3 minlen=10 lcredit=-1 ucredit=-1 dcredit=-1 ocredit=-1 difok=6
password    requisite pam_passwdqc.so use_first_pass enforce=everyone
password    sufficient pam_unix.so md5 remember=6 shadow nullok try_first_pass use_authtok
password    required pam_deny.so

session     optional pam_keyinit.so revoke
session     required pam_limits.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required pam_unix.so
```

在其中就增加了对用户密码修改时复杂度的限制，用户多次错误输入密码之后的锁定限制以及用户使用密码历史等限制选项。

所以我们通过对上述system-auth配置文件的修改，模块的增加和选项的变化，从很大的程度上增加了用户登录验证的安全性要求。我们会在之后的文章中对该配置进行详细说明。

另外也一定需要注意，在整个的PAM配置文件当中，配置项以及模块调用的逻辑顺序非常关键。因为PAM是按照配置项的先后顺序来进行验证。错误的模块调用顺序很可能导致严重的安全问题甚至系统错误。所以对PAM配置进行修改的时候务必要考虑这一点。

[Linux可插拔认证模块的基本概念与架构](http://www.infoq.com/cn/articles/wjl-linux-pluggable-authentication-module)
[Linux可插拔认证模块（PAM）的配置文件、工作原理与流程](http://www.infoq.com/cn/articles/linux-pam-one)

