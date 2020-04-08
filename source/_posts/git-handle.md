---
title: 常用的Git命令行操作
date: 2020-04-08 17:03:59
tags:
---

git 代码提交
1. git status	查看那些文件需要提交
2. git add .	添加文件到本地
3. git commit -m '备注'	添加文件到git仓库
4. git push	提交到git仓库

git 切换分支
1. git pull
2. git checkout (分支名称)
3. git status	查看分支状态

git 合并分支
首先把代码先提交到分支上——查看git 代码提交
然后如下步骤：
1. git checkout master	切换到master主分支上
2. git merge origin/index-swiper	把线上的分支代码合并到刚切换下来的master主分支上
3. git push	提交到线上master主分支

*********************************************************************************
git 其他指令

git branch				查看git上所有的分支
git branch newBranch 		创建新分支
git branch -d newBranch 	删除分支

git checkout branchName 	切换到新分支 
git checkout -b branchName 	新建且切换到新分支 
git checkout --merge newB 	切换分支的时候，将当前分支修改的内容一起打包带走，同步到切换的分支下

***
.gitignore		这个文件中可以设置一些文件默认不提交到线上及本地的git仓库中

Git工具下载：[Git](https://git-scm.com/)
