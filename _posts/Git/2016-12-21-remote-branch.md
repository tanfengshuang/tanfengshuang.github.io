---
layout: post
title:  "远程分支git remote"
categories: Git
tags: Git 远程分支
---


### 远程分支的创建

从远程分支检出的本地分支，称为跟踪分支(tracking branch)。跟踪分支是一种和远程分支有直接联系的本地分支。在跟踪分支里输入git push，Git 会自行推断应该向哪个服务器的哪个分支推送数据。反过来，在这些分支里运行git pull 会获取所有远程索引，并把它们的数据都合并到本地分支中来.

```
$ git push ssh://git@dev.remote.com/test.git master // 把本地仓库提交到远程仓库的master分支中

$ git remote add test ssh://git@dev.remote.com/test.git
$ git push test master
```

这两个操作是等价的，第二个操作的第一行的意思是添加一个标记，让origin指向ssh://git@dev.remote.com/test.git，也就是说你操作origin的时候，实际上就是在操作ssh://git@dev.remote.com/test.git。origin在这里完全可以理解为后者的别名。


### 远程分支的删除

```
# git remote rm test
```

### 远程分支的查看

*    git remote 不带参数，列出已经存在的远程分支
*    git remote -v | --verbose 列出详细信息，在每一个名字后面列出其远程url

```
# git remote
origin
production

# git remote -v
origin  ssh://ftan@code.engineering.redhat.com:22/account-management-tool (fetch)
origin  ssh://ftan@code.engineering.redhat.com:22/account-management-tool (push)
production  ssh://58064e12ecdd5c08370001c7@ethel-contentskuqe.itos.redhat.com/~/git/ethel.git (fetch)
production  ssh://58064e12ecdd5c08370001c7@ethel-contentskuqe.itos.redhat.com/~/git/ethel.git (push)

# git remote show origin
* remote origin
  Fetch URL: ssh://ftan@code.engineering.redhat.com:22/account-management-tool
  Push  URL: ssh://ftan@code.engineering.redhat.com:22/account-management-tool
  HEAD branch: master
  Remote branches:
    master      tracked
    testing     tracked
    version_1.0 tracked
    version_2.0 tracked
  Local branches configured for 'git pull':
    master      merges with remote master
    version_1.0 merges with remote version_1.0
  Local refs configured for 'git push':
    master      pushes to master      (up to date)
    version_1.0 pushes to version_1.0 (up to date)

# git remote show production
* remote production
  Fetch URL: ssh://58064e12ecdd5c08370001c7@ethel-contentskuqe.itos.redhat.com/~/git/ethel.git
  Push  URL: ssh://58064e12ecdd5c08370001c7@ethel-contentskuqe.itos.redhat.com/~/git/ethel.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local ref configured for 'git push':
    master pushes to master (up to date)

```

