---
title: git常用命令
date: 2017-10-20T22:54:24+02:00
tags: 
- git
categories: git
---



总结下git通常情况下会使用的命令。



重新编辑提交信息：
```Shell
git commit --amend
```

<!--more-->

恢复工作现场到某个版本

```Shell
git reset --hard [HEAD^|commit版本号]
```

git reset命令根据–-soft –-mixed –-hard三种参数的不同进行commit层级的回滚方式，默认是–mixed方式。

git reset --soft   : 不会改变stage区，仅仅将commit回退到了指定的提交。

git reset --mixed   :  不会改变工作区，但是会用指定的commit覆盖stage 区。

git reset --hard    :    使用指定的commit的内容覆盖stage区和工作区。



git alias
而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中：

```Shell
$ cat .gitconfig

[alias]
	co = checkout
	ci = commit
	st = status
	pl = pull
	ps = push
	dt = difftool
	l = log --stat
	cp = cherry-pick
	ca = commit -a
	b = branch
[user]
    name = Your Name
    email = your@email.com
```



git revert

git revert相当于HEAD继续前进，提交一次新的特殊的commit，内容是与前面普通commit文本变化的反操作。比如前面commit是增加一行，那么revert就是删除一行。

git revert (commit)    ：  适用于非merge操作的普通commit。git revert操作可能会遇到冲突，可以通过git revert --abort终止此次revert操作，代码还原为revert命令前。

git revert (merge)     :     适用于merge操作的commit，需要加上-m 参数，该参数表述merge编号，按commit日志标记为1，2。如下图所示，一个merge操作的日志，意思是当前分支是9fa8667加上686fabf。执行git revert  -m 1表示撤销到9fa8667所代表的commit；git revert -2表示撤销到686fabf所代表的commit。执行命令如：git revert 1d8afa9632120bf1a7331e6b6cf38d86d761de96 -m 1

![image2](https://ws2.sinaimg.cn/large/006tKfTcgy1g05t2eirz9j31450jjgp5.jpg)



git stash
可以在切换分支的时候使用，在合并代码的时候使用，减少commit提交，使log看起来整洁。

```Shell

git stash     //存储未提交修改到缓存区

git stash save "work in progress for foo feature"

git stash pop   //pop出最新的一次修改,pop之后这次修改消失

git stash apply stash@{0}   //应用第一个修改，但不删除

'git stash list'命令可以查看你保存的'储藏'(stashes):

git stash drop stash@{0}    //删除

git stash clear             //清空
```




git pull = git fetch + git merge
```Shell
$ git fetch <远程主机名>

要更新所有分支，命令可以简写为：

$ git fetch

上面命令将某个远程主机的更新，全部取回本地。默认情况下，git fetch取回所有分支的更新。如果只想取回特定分支的更新，可以指定分支名,如下所示 -

$ git fetch <远程主机名> <分支名>

比如，取回origin主机的master分支。

$ git fetch origin master

所取回的更新，在本地主机上要用”远程主机名/分支名”的形式读取。比如origin主机的master分支，就可以用origin/master读取。
```
