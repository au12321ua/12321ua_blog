---
title: git命令行的使用
date: 2024-09-06 21:46:56
tags: ["git"]
excerpt : "git命令行的基本用法，待补充..."
categories: ["tools"]
---


{% folding green::写文的目的%}
虽然一直在使用github desktop，但感觉还是有必要了解一下git命令行的基本用法。以前也整理过一次，这次拿出来复习一下，并做了一些补充。
另外由于github desktop非常“傻瓜”，所以这里也不过多说明。
{% endfolding %}

## git命令行的基础用法

### 添加文件

1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
  
  这一命令将文件添加到暂存区
  
1. 使用命令`git commit -m <message>`，完成。
  
  这一命令将暂存区的文件提交到仓库
  
  `-m`后添加的是本次提交的说明，可以是任意内容，但要有意义
  

- 解释：git的版本库有两个很重要的板块。一个是stage（或者叫index），称作暂存区。另一个是master，是自动创建的。git add就是将文件提交到stage，然后通过git commit，将stage中的内容全部提交到master中，并清空stage。
  
- git保存的是修改而不是文件
  

### 查看修改

1. `git status`查看仓库当前的状态
  
2. `git diff <file>`比较文件的不同（若没有<file>则比较当前目录全部）
  
  - 尚未缓存的改动：`git diff`
  - 查看已缓存的改动： `git diff --cached`或`git diff --staged`
  - 查看已缓存的与未缓存的所有改动：`git diff HEAD`
  - 显示摘要而非整个 diff：`git diff --stat`
  - 显示不同分支之间的差异：`git diff [first-branch]...[second-branch]`

### 版本回退

三个概念：

- HEAD：分支的指针，指向当前branch最顶端的一个commit，该分支上一次commit后的节点
  
- stage（暂存区）：是一堆将在下一次commit中提交的文件，提交之后它就是 HEAD的父节点. 一般指git add添加的文件
  
- Working Copy（工作副本）：指当前工作目录下的文件，一般指，有修改，没有 git add，没有 git commit的文件
  

相关命令：

1. `git log`显示从最近到最远的提交日志
  
  `--oneline`开启简洁模式
  
  `--graph`可以将版本显示图形化（可以展现不同的分支情况）
  
2. `git reset --hard/soft/mixed <版本号>`回退版本
  
  示例：`git reset --hard HEAD^` 回退到上一个版本（包括本地文件）
  
  **补充说明**
  
  1. 命令参数：(github desktop因为没有暂存区的概念，所以回退时只回退了HEAD)
    
    - `--hard`：HEAD & stage & working copy同时改变到你要回退到的版本；
      
    - `--mixed`（默认）：改变HEAD和stage到回退的版本，本地文件不改变；
      
    - `--soft`：只将HEAD回退到指定版本
      
  2. 版本号简写：
    
    `HEAD`指向当前版本，可以用`HEAD^`表示上个版本，`HEAD^^`表示上上个版本，以此类推，也可以用`HEAD~n`表示往上n个版本
    
  3. 每一个版本都有专门的版本号`commit id`，这是由一串很长的数字表示（哈希值），表示时可以不用写全，只需要一部分（但也不要太少，不然可能会有多个符合目标）
    
3. `HEAD`回退后，后续版本号无法通过`git log`查询，但可以通过`git reflog`查看之前的每一个命令，该命令可以显示每一个命令所处版本的版本号
  

### 撤销修改

1. 情形1：修改但还没有提交到暂存区
  
  `git checkout -- <file>`返回到上一次版本修改
  
2. 情形2：修改提交到暂存区，但还没有上传到master
  
  `git reset HEAD <file>`清空暂存区（即是使用默认的`mixed`模式）
  
3. 情形3：修改上传
  
  回退版本（但如果上传到远程库，则无法修改了...）
  

### 删除文件

命令`git rm`用于删除一个文件。
如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

### 新建分支

在git中HEAD指向master，master指向版本，master是默认的一个指针，在操作时，创建一个新的指针，最后再合并，可以使操作更加安全

1. 查看全部分支：`git branch``-r`：显示远端；`-a`显示本地和远端
2. 创建分支：`git branch <new branch name>`
3. 切换分支：`git switch <name>`
4. 创建+切换分支：`git switch -c <name>`
5. 合并某分支到当前分支：`git merge <name>`
6. 删除分支：`git branch -d <name>``-D`则是强制删除

补充：

当切换分支时，想保存当前分支尚未提交的内容，可以用`git stash`命令，重新切换回当前分支后，再用`git stash pop`命令恢复。

github desktop也有这个功能，在侧栏右键即可看见这一选项。

### 分支合并

1. 冲突分支的合并：必须手动在文本中解决冲突，才能继续执行合并提交
  
2. 关闭`fast forward`模式的合并：`git merge -no-ff <name>`
  
  也可以通过`-m "注释"`添加说明
  
3. 查看分支合并图：`git log --gragh`
  
4. 分支策略
  
  master：主分支，稳定的版本
  dev：开发分支，修改的版本
  自定义分支：个人工作的版本
  
  每个人将自己的分支合并到dev，dev再合并到master

  ![branches](/img/gituse/branches.png)

### 标签管理：

- 命令`git tag <tagname> <commit id>`用于新建一个标签，默认为HEAD，也可以指定一个commit id(tagname比如v1.0)；
- 命令`git tag -a <tagname> -m "..."`可以指定标签信息；
- 命令`git tag`可以查看所有标签;
- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签


