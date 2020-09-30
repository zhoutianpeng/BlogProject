---
title: git随手瞎记D1～～不要点开看
date: 2020-09-26 20:25:17
categories:
- [Git]
tags:
---

该笔记是看廖雪峰大佬的笔记整理的，比较乱～
<!--more-->
# 人人都会用的几个命令

```shell
#创建repository
$ git init

#向仓库中添加file
$ git add readme.txt

#将文件提交到仓库
$ git commit -m "wrote a readme file"

#查看git仓库某时刻状态
$ git status

#查看git仓库此时与上次提交的变化,git diff本身只显示尚未暂存的改动
$ git diff readme.txt 
#若也需要查看已暂存将要添加到下次提交的内容，使用如下命令
$ git diff --cached

#查看git提交历史记录
$ git log
$ git log --pretty=oneline 

#假设此时打印的结果如下
# 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
# e475afc93c209a690c39c13a46716e8fa000c366 add distributed
# eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file

# 在git中用 HEAD 表示当前版本，使用 HEAD^ 表示上个版本，同理 HEAD^^ 表示上上个版本

#回退git版本
$ git reset --hard HEAD^

# 注意此时通过git log命令无法查询到 1094adb 这个提交信息
# 如果需要返回 1094adb 提交，通过以下命令返回：
$ git reset --head 1094adb

# 若你找不到 1094adb 的commit id可以通过如下操作找回commit id，这个命令记录了你的每一次命令；
$ git reflog

```

# 人人都知道的几个概念

## 工作区、版本库、暂存区
- 工作区: 执行`git init`命令的这个目录
- 版本库: 工作区中有一个`.git`隐藏目录，这个目录是版本库，版本库中有git为我们创建的第一个分之`master`，以及指向`master`的指针叫`HEAD` // HEAD是文件吗？？？
- 暂存区: 版本库中`index`目录或叫`stage`目录，可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

## Git的文件管理

Git跟踪并管理的是修改，而非文件
```
工作区 -> 暂存区 -> 仓库

git add把文件从工作区>>>>暂存区，git commit把文件从暂存区>>>>仓库，

git diff 查看工作区和暂存区差异(修改了还未暂存的变化)

git diff --cached 查看暂存区和仓库差异(所有暂存了还未提交的变化)

git diff HEAD -- readme.txt 查看工作区和仓库的差异

git add的反向命令git checkout，撤销工作区修改，即把暂存区最新版本转移到工作区，

git commit的反向命令git reset HEAD，就是把仓库最新版本转移到暂存区。
```

### 撤销修改
```shell
# 让文件回到最近一次git commit或 git add时的状态
# 若文件修改后还没有执行add，撤销修改就回到和版本库一模一样的状态；
# 若文件已经add，又作了修改，撤销修改就回到add时的状态。
$ git checkout -- readme.txt

# 若文件已经add，但未commit
$ git reset HEAD <file>
# 此时文件依旧被修改了，但是未add；撤销add使用checkout命令
```

### 删除文件
若一个文件存在与仓库中，你将文件在工作区删除后
- 如果你是想提交操作，即删除仓库中的这个文件，使用这个命令
    ```
    git rm test.txt
    git commit -m "delete test.txt"
    ```

- 如果你删错了，想在工作区复原文件，使用这个命令
    ```
    git checkout -- test.txt
    ```

## 远程仓库

现在的情景是，你已经在本地创建了一个Git仓库，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的库既可以作为备份，又可以让其他人通过该仓库来协作，真是一举多得。

- 在github创建仓库

- 把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库

    `git remote add origin git@github.com:XXXXXX/XXXXX.git`

- 将本地库的所有内容推送到远程分支，此时是第一次推送需要加`-u`命令

    `git push -u origin master`

- 未来需要将本地的提交推送至github使用命令

    `git push origin master`

现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆

首先，登陆GitHub，创建一个新的仓库，名字叫gitskills

我们勾选Initialize this repository with a README，这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件

执行命令clone工程下来
```
$ git clone git@github.com:michaelliao/gitskills.git
```

## 分支管理

### 创建与合并分支
```shell
# git创建新分支，并切换到分支中
$ git checkout -b dev

# 等价于如下两条命令
$ git branch dev
$ git checkout dev

# 查看当前分支
$ git branch

# 合并分支, merge命令用于合并指定分支到当前分支
$ git merge dev

# 删除分支
$ git branch -d dev

```

### 处理冲突
```shell
# 在某次合并操作中
$ git merge feature1

# Git告诉我们文件存在冲突，必须手动解决冲突后再提交。git status可以告诉我们冲突的文件：
$ git status

#可以直接查看冲突文件，git会用特殊符号标记处冲突位置
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1

#需要你确认好冲突问题后，决定留下哪段代码，最后合并提交，删除开发分支
$ git add readme.txt 
$ git commit -m "conflict fixed"

# 用带参数的git log查看分支的合并情况
$ git log --graph --pretty=oneline --abbrev-commit

```

### BUG分支
现在的情景是，当你需要创建一个分支issue来修复BUG，但是当前正在dev上进行的工作还没有提交，而且也不能提交时，Git提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作；

- 使用 `git stash`命令暂存

- 使用 `git status`命令查看当前工作区是否干净

- 创建BUG修复分支完成任务，比如此时是在master分支做修改
    ```shell
    $ git checkout master
    
    $ git checkout -b issue-101

    # fixed bug ...

    $ git commit -m "fix bug 101"

    $ git checkout master

    $ git merge --no-ff -m "merged bug fix 101" issue-101
    ```
- 切换回dev分支继续之前的任务
    ```shell
    $ git checkout dev
    
    $ git status
    ```

- 恢复stash区域的环境，使用`git stash list`命令看看
    ```shell
    $ git stash list
    stash@{0}: WIP on dev: f52c633 add merge
    ```
    - 使用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；
    - 使用`git stash pop`，恢复的同时把stash内容也删了

    你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令`$ git stash apply stash@{0}`


如果刚刚修复的BUG在此dev分支也存在，可以不合并分支，使用`cherry-pick`命令复制一个特定的提交到当前分支
```shell
$ git branch
* dev
  master

$ git cherry-pick 4c805e2
```

### 多人协作
- 使用`git remote`命令查看远程库信息。或者使用`git remote -v`命令查看等详细的信息

- 使用`git push origin master`推送分支到远程库

- 获取远程库其他分支(dev)到本地`git checkout -b dev origin/dev`，本地完成开发推送到远程分支
    ```
    $ git add env.txt

    $ git commit -m "add env"

    $ git push origin dev
    ```

```
因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin <branch-name>推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！

如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。
```

## 标签管理

### 创建标签 

使用`git tag <name>`就可以在当前分支创建一个新标签
```shell
$ git tag v1.0
```

使用`git tag`命令查看所有标签，标签是按照字母排序显示
```shell
$ git tag
v1.0
```

为历史提交创建一个tag
```shell
$ git tag v0.9 f52c633
```

### 维护标签
命令`git push origin <tagname>`可以推送一个本地标签；

命令`git push origin --tags`可以推送全部未推送过的本地标签；

命令`git tag -d <tagname>`可以删除一个本地标签；

命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

## 自定义git

### .gitignore



