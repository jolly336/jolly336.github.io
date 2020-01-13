---
title: Github Tips之pull request(PR)
date: 2018-08-04 12:16:25
tags: Github
categories: Github
---

> Github是为开发者提供Git仓库的托管服务，是一个让开发者与朋友、同事、同学及陌生人共享代码的完美场所。除了Git仓库的托管服务外，还为开发者和团队提供了一系列功能，帮助其提高效率、高品质地进行代码编写。本文将讲解一些关于Github PR(pull request)的使用和技巧。

<img src="http://wanghaoxun.com/github_logo.png" width="60%" height="30%">

<!-- more -->

## 前言

Pull Request 是用户修改代码后向对方仓库发送采纳请求的功能，也是GitHub的核心功能正因为有了这个功能，才会让众多开发者轻松地加入到开源开发的队伍中来。

在 Pull Request页面能够列表查看当前处于Open状态的 Pull Request。通过点击页面左部和上部的选项可以进行筛选和重新排列。
在列表中点击特定的 Pull Request 就会进入详细页面。页 面上方显示着这次是从谁的哪个分支向谁的哪个分支发送了 Pull Request。下面，我们以[GlideTest](https://github.com/haoxunwang/GlideTest)仓库为例进行讲解。注：下面haoxunwang是[GlideTest](https://github.com/haoxunwang/GlideTest)仓库的原作者，MogaoCaves是提交PR的开发者，MogaoCaves的[GlideTest](https://github.com/MogaoCaves/GlideTest)仓库是fork来自原作者haoxunwang的仓库。

图1【PR首页，显示了[GlideTest](https://github.com/haoxunwang/GlideTest)仓库的pr所有的信息】
![github_pr_home](http://wanghaoxun.com/github_pr_home.jpg)

图2【PR详情，可以看到提交者从哪个分支提交到仓库的哪个分支】
![github_pr_detail](http://wanghaoxun.com/github_pr_detail.png)

## 发送PR流程

![PR概念图](http://wanghaoxun.com/pr%E6%A6%82%E5%BF%B5%E5%9B%BE.jpg)

### 拉取代码

* 首先fork目标仓库（以[GlideTest](https://github.com/haoxunwang/GlideTest)为例）
* git clone git@github.com:MogaoCaves/GlideTest.git
* git checkout -b work master创建特性分支并自动切换
* 添加代码
* git diff 查看修改是否正确
* git push origin work 创建远程分支
* git branch -a 查看分支，origin/work已经被创建

### 发送PR

* 登录GitHub并切换到work分支，点击分支名左侧的绿色按钮，查看修改是否正确
* 确认无误之后，点击create pull request 
* send pull request
* 该仓库的管理者会接到通知
* 代码修改合理，我们此次提交Pull Request会被仓库拥有者采纳

### 想让PR更加有效的方法？  

* 不必等代码最终开发完成，在PR中附带一段简单代码让大家有个大体印象，能获取不少反馈
* 使用Tasklist语法，反映出哪些功能已经实现，将来要做哪些工作，能加快审查者的工作效率，还能作为自己的备忘录使用
* 不要在PR中添加无关的修改，处理与主题无关的作业请另外创建分支，不然会让原本清晰的讨论变得一团糟
* 明确标出“正在开发过程中”，在标题前加上“[WIP]”字样，WIP是work in progress的简写，等所有功能完成之后，再消去这个前缀
![WIP示意图](http://wanghaoxun.com/WIP%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

### 仓库的维护

> Fork或clone来的仓库，一旦放置不管就会离最新的源代码越来越远。如果不以最新的源代码为基础开发，劳神费力地编写代码也很可能是白费力气。所以需要我们学习如何让仓库保持最新状态。

![将仓库更新至最新状态](http://wanghaoxun.com/UpdateToNew.jpg)

* git clone git@github.com:MogaoCaves/GlideTest.git 克隆fork来的仓库
* git remote add upstream git@github.com:haoxunwang/GlideTest.git 给原仓库设置upstream名称，将其作为远程仓库。今后，这个仓库以upstream作为原仓库的标识符，只需要设定一次
* git fetch upstream 获取最新数据，让仓库维持最新状态
* git merge upstream/master 将upstream/master分支与当前分支(master)合并，

## 接受PR流程

> 采纳PR前的准备：代码审查

![PR接受](http://wanghaoxun.com/PR%E6%8E%A5%E5%8F%97.jpg)

* 在本地开发环境中反映PR的内容，例如，PR接受方的用户名为haoxunwang，发送方的用户名为“PR 发送者”
* git clone git@github.com:haoxunwang/GlideTest.git 克隆接受方仓库到本地
* git remote add mogao(PR发送者) git@github.com:(PR发送者)MogaoCaves/GlideTest.git 
* git fetch PR发送者 获取发送方的远程仓库 git fetch mogao
* git checkout -b pr1 创建用于检查的分支 git checkout -b work
![PR_checkout](http://wanghaoxun.com/PR_checkout.jpg)

* git merge PR发送者/work 合并发送方分支代码 git merge mogao/work
* git branch -D PR1 检查无误之后，删除分支 git branch -D work
* git checkout master 切换至我们master分支
* git merge PR发送者/work 合并PR发送者/work分支的内容 git merge mogao/work
* git push 推送至远端

## Q & A

* 为何要在特性分支（例如，本文的work分支）中进行作业再提交PR？
    当前Git的主流开发模式都会使用特性分支。特性分支顾名思义，是集中实现单一特性（主题），除此之外不进行任何作业的分支。在日常开发中，往往会创建数个特性分支，同时在此之外再保留一个随时可以发布软件的稳定分支。稳定分支的角色通常由master分支担当。所以请养成创建特性分支后再修改代码的好习惯吧。在Github上发送Pull Request时，一般都是发送特性分支。这样一来，PR就拥有了更明确的特性（主题）。为了让对方了解自己修改代码的意图，有助于提高代码审查的效率。


