---
title: git随手瞎记D2～～不要点开看
date: 2020-09-26 23:04:04
categories:
- [Git]
tags:
---


<!--more-->

# Git

## 安装Git & 设置

请参考这个官方文档[点我](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

最小配置`user.name`和`user.email`
```shell
$ git config --global user.name 'your name'
$ git config --global user.email 'your email@domain.com'
```

若配置基本信息时未添加`--global`参数，则等同于local，此时的配置只针对当前仓库有效
```shell
# config的三个作用域

# 仅对当前仓库有效
$ git config --local 

# 对当前用户所有仓库生效
$ git config --global 

# 所有登陆系统用户都生效
$ git config --system 
```
若想查看config配置，使用`--list`参数
```shell
$ git config --list --local 
$ git config --list --global 
$ git config --list --system  
```

## Git使用入门

### 常用命令

```shell
#向仓库中添加file
$ git add readme.txt
$ git add a.html b.html

#将文件提交到仓库
$ git commit -m "wrote a readme file"

#查看git仓库某时刻状态
$ git status

# 为文件重命名
$ mv readme readme.md
$ git add readme.md
$ git rm readme
# git为文件重命名提供的简便操作
$ git mv readme readme.md

# git log常用用法
$ git log --pretty=oneline 
$ git log -n4 #显示前两次 
$ git log --all #显示仓库中所有分支的提交信息
$ git log --all --graph #以图形方式显示仓库中所有分支的提交信息

# git 查看所有分支&版本
$ git branch -av

# 分离头指针：此时HEAD指向一个提交而不是一个分支，我们正在工作在一个没有分支的状态下
# 此状态提交修改并切换分支后，修改可能会丢失，此时应该在分离头指针状态绑定一个分支
# 查看两次commit的差异
$ git diff HEAD HEAD^


```

常用场景
```shell
# 如何删除不使用的分支
$ git branch -d 9c6861f 


# 如何修改最新commit的meaasge信息 && 只能修改本地分支
$ git commit --amend


# 如何修改老旧commit的meaasge信息 && 只能修改本地分支
$ git rebase -i 429241f # hash号是目标提交的父提交号
r 429241f -> wq! 
修改信息 -> wq!


# 如何将多个连续的commit合并为一个commit
$ git rebase -i 429241f # hash号是目标提交的父提交号

pick 9c6861f info
s 4234213 info
s 8d32e83 info
s 2a4489d info -> wq!

添加新的合并信息
Create a complete page -> wq!


# 如何将多个不连续的commit合并为一个commit
$ git rebase -i 429241f # hash号是目标提交的父提交号,若目标提交已经是第一次提交没有父提交号，可以在此处填写目标提交号，再手动编辑交互式文件
# step 1 手动的加上目标提交Hash值
pick 4234213 #这个目标号是是手动添加的
# step 2 将需要合并的提交号都排列在一起，并设置为s
pick 4234213
s 4234213 info
s 2a4489d info -> wq!
# step 3
# 可能要添加这句话 git rebase --continue
添加新的合并信息
Create a complete page -> wq!

```












