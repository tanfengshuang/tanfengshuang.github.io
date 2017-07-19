---
layout: post
title:  "Implementing Git(405-5)"
categories: Linux
tags: RHCA 405
---

### 实施 Git 存储库

###### Git 简介

Git 是一种分布式修订控制系统，可供开发人员用于管理项目中的文件更改。修订控制系统具有许多益处，例如：

*    每一更改随同日志消息一起提交。这使得开发人员能够声明更改的原因。
*    文件可以回滚到以前的版本。如果工作中的文件受损并提交到了存储库中，此功能可以派上用场。
*    可以显示提交之间的文件更改。文件的历史更改记录显示出它在项目生命周期内的演变。
*    修订控制系统还可以记录谁执行了更改，以及提交的日期/时间。这样，其他项目成员可以辨别出应为指定更改担责（或享誉）的人。
*    多名贡献者可以提交更改到一个共享的项目。修订控制系统允许多名用户将更改合并到一起，并且一名贡献者的更改不会覆盖掉他人的更改。
*    包括 Git 在内的许多修订控制系统具有可促进工作流自动化的 hook。本节稍后将对此进行深入探讨。 


Git 管理文件的四个区域

1. Central Repository(upstream)
2. Local Repository
3. Staging Area
4. Working Tree

Git 将其元数据和项目数据库存放在存储库中。用户可以创建新的项目，或者也可从中央存储库克隆现有的共享项目。Git 拥有上文中列出所有修订控制系统的优点，但与众不同的地方是其分布式性质。项目从中央存储库克隆时，其本地副本称为本地存储库。这是原始上游存储库的完整副本，而不仅仅是项目文件的最新快照。Git 用户始终与本地存储库交互，偶尔会将更改推回到中央存储库。这可避免出现单一故障点，而且在网络不可用时用户仍然可以管理文件更改。

工作树是项目单一快照的一个签出版。这最初取自存储库的默认 master 分支的最新修订。工作树中的文件可用于审核和修改。

若要高效地使用 Git，用户必须清楚文件可以处于的三种状态：已提交、已修改和已暂存。当文件的数据存储在本地 Git 存储库后，该文件处于已提交状态。已修改状态表示文件已被添加、删除或更改，但尚未添加到将要提交到存储库的文件索引中。已暂存文件是已添加到下一次存储库提交中的已修改文件。

###### 创建 Git 存储库

可以使用 git init 命令创建 Git 存储库。它可用于创建私有存储库和共享存储库。以下命令为项目新建一个空的本地存储库：

```
[user@host ~]$ git init PROJECT
Initialized empty Git repository in /home/user/PROJECT/.git/
[user@host ~]$ ls -a PROJECT
.  ..  .git
```

创建了名为 PROJECT 的新目录，其中仅含有 .git 子目录。本地存储库的对象和元数据存储在 .git 中。PROJECT 目录也是工作树，最开始不含任何文件。Git 存储库还有一个暂存区域。

git init 也可以将现有的目录置于 Git 控制之下。文件的所有者更改到此目录，再执行下列命令：

```
[user@host projectdir]$ git init .
```

Git 将在当前目录下创建一个 .git 目录，使当前目录成为本地存储库。目录中现有的文件将保持不变，它将充当工作树。存储库最初为空。文件必须另行添加，因为 Git 不会推断哪些文件应当添加到存储库中。

下方的 git init 命令用于创建一个可供多位用户共享的空 Git 存储库：

```
[user@host projectdir]$ git init --bare --shared=true PROJECT
Initialized empty shared Git repository in /home/user/PROJECT/
[user@host ~]$ ls -a PROJECT
.  ..  branches  config  description  HEAD  hooks  info  objects  refs
```

--bare 选项创建不带工作树的存储库。这时存储库元数据不放到 .git 子目录中，所有的 Git 管理目录都在顶级目录中。--shared=true 选项创建设置了 setgid 位的可写入目录组。这使得子目录和文件继承该目录的组所有权，而不是由用户提交更改到存储库。 

###### 通过 Git hook 自动化工作流

Git hook 是执行 Git 期间在不同工作流阶段运行的程序。它们可以是 shell 脚本或二进制程序。唯一的要求是它们必须是具有特定文件名的可执行文件，具体取决于应当执行它们的 Git 工作流阶段（即 commit-msg、pre-commit、post-commit 和 post-receive）。

在创建新存储库时，Git 创建一个名为 hooks 的目录。由 git init 创建的本地存储库会创建一个 .git/hooks 子目录，该子目录用于创建 Git hook。在最初，Git 使用文件名后缀为 .sample 的示例脚本填充此目录。以下示例输出演示了默认情况下创建的示例 Git hook 脚本。

```
[user@host project]$ ls .git/hooks/
applypatch-msg.sample  pre-applypatch.sample      pre-push.sample
commit-msg.sample      pre-commit.sample          pre-rebase.sample
post-update.sample     prepare-commit-msg.sample  update.sample
```

hooks 目录位于通过 git init --bare --shared=true 命令创建的共享 Git 存储库的最顶层。它最初填充有同样的示例脚本。

######### 自动检查 Puppet 语法

语法检查是一项简单的 Puppet 工作流任务，可以通过 Git hook 进行自动化。当 Puppet 清单处于修订控制之下时，最好能够先检测（并修复）清单中的语法错误，然后再将清单提交到存储库中。最适合此任务的 Git hook 是 pre-commit hook。

以下 pre-commit hook 检查具有 .pp 后缀的所有文件的 Puppet 语法。当暂存的文件提交至存储库时会执行此 hook，然后创建或输入日志消息。Git 不通过标准输出传递命令行参数或信息到此脚本。第 5 行上的 git diff-index 命令生成要提交的所有已暂存文件的列表。

for 循环逐一处理各个文件。在第 8 行上，结尾为 .pp 后缀的每个文件作为参数传递到 puppet parser validate 命令。如果命令以非零退出状态退出（清单含有错误），则执行 if 语句的正文。向开发人员显示第 10 行上的错误消息，在第 11 行上设置一个标志变量。脚本不会在这个点上立即退出，因此一次提交中可以检查多个 Puppet 清单。

Git hook 在第 14 行上退出。它使用 exit_status 变量来确定脚本的退出状态。非零退出状态将使 Git 中止提交。所有文件都不会提交，直到问题解决为止。

```
     1  #!/bin/bash
     2
     3  exit_status=0
     4  # Validate Puppet syntax of manifests
     5  for file in $(git diff-index --name-only --cached HEAD --)
     6  do
     7    # Puppet manifest ends with .pp, so let Puppet parse it
     8    if [[ ${file} == *.pp ]] && ! puppet parser validate ${file}
     9    then
    10      echo "Commit failed: Puppet cannot parse ${file}"
    11      exit_status=1
    12    fi
    13  done
    14  exit ${exit_status}
```

在清单暂存之后，如果要提交的清单中含有可由 puppet parser validate 检测到的语法错误，git commit 将生成错误消息。

```
[user@host project]$ git commit
Error: Could not parse for environment production: Syntax error at '=>';
 expected '}' at /home/user/project/test.pp:5
Commit failed: Puppet cannot parse test.pp
```


### 使用 Git 管理代码

由于 Git 用户经常与多个贡献者一起修改项目，Git 会在每一次提交时发布用户的名称和电子邮件地址。这些值可以在项目级别上定义，但也可为用户设置全局默认值。git config 命令控制这些设置。将它与 --global 选项搭配时，可管理该用户参与的所有 Git 项目的默认设置。

```
[peter@host ~]$ git config --global user.name 'Peter (Starlord) Quill'
[peter@host ~]$ git config --global user.email peter@host.example.com
```

###### Git 工作流

在处理共享项目时，Git 用户使用 git clone 命令克隆现有的上游存储库。提供的路径名或 URL 决定了哪一个存储库被克隆到当前目录中。同时也会创建工作树，使得文件的目录也可准备好进行修改。由于工作树是未修改的，它最初是干净的状态。

> 另一种启动 Git 工作流的方式是使用 git init 命令创建一个新的私有项目。 

```
    |---    Central Repository(upstream)   <--- git init --bare --shared=true
    |--->    Local Repository              <---|
 git clone  Staging Area                    git init
    |--->    Working Tree                  <---|
```

在工作树中创建新的文件，并且修改现有的文件。这一更改会使工作树变为不干净状态。git status 命令显示工作树中已更改文件的相关信息。

git add 命令使文件从已修改状态变为已暂存状态。它被包含在暂存区域中，下一次提交时会保存到存储库中。当文件在项目其余部分中处于稳定且可用状态时，应当执行这一步。

> git rm 命令删除未修改的文件，并将它添加到已暂存文件的索引中。 

git commit 命令将对已暂存文件的更改提交到本地 Git 存储库。必须提供一条日志消息，说明保存当前一组已暂存文件的原因。日志消息不需要很长，但必须意思清楚才具用处。


```
             Central Repository(upstream)
    |--->    Local Repository              <---|
    |                                        commit
    |---     Staging                        ---|
    |        Area                          <---|
  commit -a                                   add
    |---     Working Tree                   ---|
```

git push 命令将对本地存储库的更改上传到原始上游存储库。这是 Git 用户将更改发布给同时处理项目的其他人的方式。在 Git 推送发挥作用之前，必须先定义默认的推送方式。以下命令将默认推送方式设置为 simple 方式。这是对新手而言最安全的选项，也是 Git 2.0 中的默认推送方式。

```
[peter@host ~]$ git config --global push.default simple
``` 

git pull 命令从上游存储库检索更新，并将它们保存到本地存储库中。它也会更新工作树中的文件。应当要经常执行此命令，从而获得其他人对项目作出的最新更改。git fetch 命令执行与 git pull 相同的操作，但不会更改工作树。 

```
    |---      |---  Central Repository(upstream)   <---|
    |        fetch                                    push
    |--->     |---  Local Repository                ---|
 git pull           Staging Area
    |--->           Working Tree
```

在编辑过程中，应当经常使用以下 Git 子命令来检查项目中文件的状况和最新状态：status、diff、log 和 show。git status 显示工作树中超出 Git 控制的已修改、已暂存和新文件的常规状态。git diff 显示工作树中的文件与其在存储库中的副本相比修改了什么。git log 命令显示文件的提交日志消息和关联的的 ID 哈希。可以将哈希用于 git show 命令，显示该文件提交至存储库的更改。

git reset 命令从暂存区域中删除之前为后续提交而添加的文件。此命令对文件的内容不起作用。git checkout -- 命令将文件内容恢复到已签入本地存储库中的最新状态。 

```
             Central Repository(upstream)
    |---     Local Repository
    |---     Staging Area                    ---|
 git checkout                                git reset
    |--->     Working Tree                   <---|
```

Git 快速参考

*    git clone 	将现有的 Git 项目克隆到当前的目录。
*    git status 	显示工作树中已修改和已暂存文件的状态。
*    git add 	暂存文件以准备下一次提交。
*    git commit 	将已暂存文件与描述性消息一起提交。
*    git diff 	显示工作树中的文件与本地存储库中的最新版本之间的差别。
*    git log 	查看存储库的文件提交历史记录。
*    git show 	与提交 ID 一起使用时，显示日志消息和文字差别。
*    git push 	将本地存储库推送到初始上游 Git 存储库。
*    git pull 	从上游 Git 存储库拉取更新到本地存储库，并在工作树中合并它们。
*    git fetch 	从上游 Git 存储库提取更新到本地存储库，但不改动工作树。
*    git checkout -- 	将工作树中已修改的文件恢复为最新的存储库状态。
*    git reset 	从下一次提交中取消暂存文件（与 git add 相反）。
*    git config 	管理 Git 配置值；例如，作者名称、电子邮件地址和默认推送模式。
*    git init 	在当前目录中创建一个新的 Git 存储库。 


### 练习：使用 Git 管理代码

1. 以 student 身份（密码 student）登录 servera。配置 Git 名称和电子邮件地址。

```
[student@servera ~]$ git config --global user.name 'Student User'
[student@servera ~]$ git config --global user.email student@servera.lab.example.com
[student@servera ~]$ cat ~/.gitconfig
[user]
        name = Student User
        email = student@servera.lab.example.com
```

2. Git 报告它没有执行推送，因为可以通过几种方式来执行推送。将 push.default 设置设为 simple，然后再次尝试推送。

```
[student@servera puppet]$ git push
warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:
 
  git config --global push.default matching
 
To squelch this message and adopt the new behavior now, use:
 
  git config --global push.default simple
 
See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)
 
No refs in common and none specified; doing nothing.
Perhaps you should specify a branch such as 'master'.
fatal: The remote end hung up unexpectedly
error: failed to push some refs to '/var/git/puppet.git'
 * [new branch]      master -> master

[student@servera puppet]$ git config --global push.default simple
[student@servera puppet]$ git push
Counting objects: 6, done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 2.41 KiB | 0 bytes/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To /var/git/puppet.git
 * [new branch]      master -> master
```

3. git diff 命令显示工作树中的已修改文件和签入到存储库中的最新版本之间的更改。

```
[terry@serverb puppet]$ git diff
diff --git a/manifests/motd.pp b/manifests/motd.pp
index f7cac14..0a55e77 100644
--- a/manifests/motd.pp
+++ b/manifests/motd.pp
@@ -2,6 +2,6 @@ file { '/etc/motd':
   ensure  => 'file',
   owner   => 'root',
   group   => 'root',
-  mode    => '444',
+  mode    => '644',
   content => "No trespassing.\n",
 }
```

4. git log 命令显示在提交对文件的更改时使用的日志消息列表。

```
[terry@serverb puppet]$ git log README
[terry@serverb puppet]$ git log manifests/motd.pp
commit 55c6f7161443102dca9cd4bf04d651c8439f7361
Author: Student User <student@servera.lab.example.com>
Date:   Sat Oct 10 20:42:05 2015 -0400
 
    Initial /etc/puppet content
```

5. 使用 git show 命令及文件的提交 ID，将显示该次提交中对文件进行的更改。初次提交时添加了 motd.pp 的所有内容。

```
[terry@serverb puppet]$ git show 55c6f7161443102dca9cd4bf04d651c8439f7361 manifests/motd.pp
commit 55c6f7161443102dca9cd4bf04d651c8439f7361
Author: Student User <student@servera.lab.example.com>
Date:   Sat Oct 10 20:42:05 2015 -0400
 
    Initial /etc/puppet content
 
diff --git a/manifests/motd.pp b/manifests/motd.pp
new file mode 100644
index 0000000..f7cac14
--- /dev/null
+++ b/manifests/motd.pp
@@ -0,0 +1,7 @@
+file { '/etc/motd': 
+  ensure  => 'file',
+  owner   => 'root',
+  group   => 'root',
+  mode    => '444',
+  content => "No trespassing.\n",
+}
```

6. 使用以下命令序列查看如何撤销尚未暂存的更改。git reset 对文件的内容没有影响。git checkout -- 命令将文件恢复到其原始状态。

```
[student@servera puppet]$ git reset README
Unstaged changes after reset:
M       README
[student@servera puppet]$ cat README 
This file was created by terry on serverb.
more stuff
[student@servera puppet]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   README
#
no changes added to commit (use "git add" and/or "git commit -a")
[student@servera puppet]$ git checkout -- README
[student@servera puppet]$ git status
# On branch master
nothing to commit, working directory clean
[student@servera puppet]$ cat README 
This file was created by terry on serverb.
```

7. 使用以下命令序列查看如何撤销已暂存的更改。git reset 会取消暂存文件。它与 git add 相反。

```
[student@servera puppet]$ echo 'another modification by student' >> README
[student@servera puppet]$ git add README
[student@servera puppet]$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README
#
[student@servera puppet]$ git reset README
Unstaged changes after reset:
M       README
[student@servera puppet]$ git status 
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   README
#
no changes added to commit (use "git add" and/or "git commit -a")
```


### 总结

在本章中，您学到了：

*    实施共享 Git 存储库时，首先要为 Git 用户创建一个专用的组，还可能要创建 git 用户。存储库必须是由该组所有的公共目录，并且设置有组 ID 权限集。
*    git init 命令可以在 Git 存储库中创建新项目。
*    在处理项目之前，用户必须使用 git config 命令定义要用于 Git 提交的名称和电子邮件地址。
*    在开始处理项目时，用户首先使用 git clone 命令从共享 Git 存储库克隆项目。
*    创建或修改了文件后，可使用 git add 命令将文件添加到暂存区域。
*    git commit 命令可将暂存的文件和日志消息一起保存到本地存储库中。
*    可以通过 git pull 命令从共享 Git 存储库获取其他用户进行的更改。
*    在本地 Git 存储库中进行的更改可以通过 git push 命令保存到原始共享存储库中。
