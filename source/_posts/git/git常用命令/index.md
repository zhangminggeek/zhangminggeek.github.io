---
title: git常用命令
date: 2019-02-14 22:42:21
categories: git
tags: [git]
---

## git init
在本地新建一个repo，进入一个项目目录，执行`git init`，会初始化一个repo，并在当前文件夹下创建一个.git文件夹。

## git clone
获取一个url对应的远程Git repo，创建一个local copy。
一般的格式是`git clone [url]`，clone下来的repo会以url最后一个斜线后面的名称命名，创建一个文件夹，如果想要指定特定的名称，可以`git clone [url] newname`指定。

## git status
查询repo的状态。

## git log
显示提交日志大纲。

## git add
在提交之前，Git有一个暂存区(staging area)，可以放入新添加的文件或者加入新的改动。commit时提交的改动是上一次加入到staging area中的改动，而不是我们disk上的改动。
`git add <filename>`添加单个文件
`git add .`会递归地添加当前工作目录中的所有文件
{% asset_img images/dafjdaldha.jpeg %}

## git diff
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

## git commit
提交已经被add进来的改动
`git commit -m “the commit message"`
{% asset_img images/dafadfasfe.jpeg %}

## git reset
git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本。
`git reset HEAD`把不小心add进去的文件从staged状态取出来,可以单独针对某一个文件操作：`git reset HEAD <filename>`
这里的HEAD关键字指的是当前分支最末梢最新的一个提交，也就是版本库中该分支上的最新版本。也可以把HEAD相应版本的版本号

## git revert
git revert的作用通过反做创建一个新的版本，这个版本的内容与我们要回退到的目标版本一样，但是HEAD指针是指向这个新生成的版本，而不是目标版本。
`git revert HEAD`撤销最近的一个提交
git revert会创建一个反向的新提交，可以通过参数-n来告诉Git先不要提交

## git stash
git stash将会把当前目录和index中的所有改动(但不包括未track的文件)压入一个栈，然后留给你一个clean的工作状态，即处于上一次最新提交处
`git stash list`会显示这个栈的list
`git stash apply`取出stash中的上一个项目(stash@{0})，并且应用于当前的工作目录，也可以指定别的项目，比如git stash apply stash@{1}.
`git stash pop`在应用stash中项目的同时删除它
`git stash clear`删除所有项目

## git branch
git branch可以用来列出分支，创建分支和删除分支。
`git branch`列出本地所有分支，当前分支会被星号标示出
`git branch [branchname]`创建一个新的分支
`git branch -D [branchname]`删除一个分支

## git checkout
`git checkout [branchname]`切换到一个分支
`git checkout -b [branchname]`创建并切换到新的分支
`git checkout <filename>`此命令会使用HEAD中的最新内容替换掉你的工作目录中的文件，已添加到暂存区的改动以及新文件都不会受到影响
注意：git checkout filename会删除该文件中所有没有暂存和提交的改动，这个操作是不可逆的

## git merge
`git merge [alias]/[branch]`把一个分支merge进当前的分支

## git tag
`git tag`列出现有标签
`git tag -a [tagname] -m"the tag message""`创建一个含附注类型的标签
`git show [tagname]`查看相应标签的版本信息，并连同显示打标签时的提交对象

## git fetch
把远程库的代码更新到本地库

## git pull
git pull会首先执行git fetch，然后执行git merge，把取来的分支的head merge到当前分支。这个merge操作会产生一个新的commit。

## git rebase
重新应用提交在另一个基本提示之上。rebase的过程中，也许会出现冲突，Git会停止rebase并让你解决冲突，在解决完冲突之后，用git add去更新这些内容，然后无需执行commit，只需要：
`git rebase --continue`就会继续打余下的补丁
`git rebase --abort`将会终止rebase,当前分支将会回到rebase之前的状态

## git push
`git push [remote-name] [branch-name]`将提交推送到远程分支

## 参考文章
[git](https://git-scm.com/docs/git)
[Git常用命令总结](https://www.cnblogs.com/my--sunshine/p/7093412.html)
[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
