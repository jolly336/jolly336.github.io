---
title: Github Tips之Git
date: 2018-08-04 13:34:06
tags: Github
categories: Github
---


> Github是为开发者提供Git仓库的托管服务，是一个让开发者与朋友、同事、同学及陌生人共享代码的完美场所。除了Git仓库的托管服务外，还为开发者和团队提供了一系列功能，帮助其提高效率、高品质地进行代码编写。本文将讲解一些关于Github Git的使用。

<img src="http://wanghaoxun.com/github_logo.png" width="60%" height="30%">

<!-- more -->


## Git
* git diff HEAD 查看本次提交与上次提交之间有什么差别
* git checkout - 切换回前一个分支
* git merge 合并分支，git merge --no-ff feature-A  为了在历史记录中明确记录下本次分支合并，我们需要创建合并提交
* git log --graph 以图表形式查看分支
* git reset --hard SHA-1 回退到某次提交
* git reflog 查看以当前状态为终点的历史日志，
* git commit -am "Add feature-C" -am命令来一次完成两步操作
* git rebase -i HEAD~1 选定当前分支中包含HEAD（最新提交）在内的2个最新历史记录为对象，并在编辑器中打开，在编辑器中修改命令参数，使用fixup即可，然后保存退出vim即可

![git rebase合并历史示例图](http://wanghaoxun.com/git_rebase.jpeg)

* git remote show origin 显示当前远端仓库地址
* git format-path 前一次commit hash -o ~/Downloads/nelson/script/
* git apply --reject ~/Downloads/nelson/script/path.patch

## Q & A

1. git add . -A -u 区别？

* git add -u：(保存修改和删除，但是不包括新建文件) - 将文件的修改、文件的删除，添加到暂存区。update
* git add .：(保存新的添加和修改，但是不包括删除) - 将文件的修改，文件的新建，添加到暂存区。
* git add -A：(保存所有的修改) - 将文件的修改，文件的删除，文件的新建，添加到暂存区。

