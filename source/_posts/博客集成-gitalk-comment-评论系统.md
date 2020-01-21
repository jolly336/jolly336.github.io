---
title: 博客集成 gitalk comment 评论系统
date: 2020-01-15 23:36:10
categories: [博客]
tags: [博客, gitalk]
---

本文记录如何在基于 Hexo 的博客里添加 gitalk comment 评论系统。

<!-- more -->

[GITALK](https://gitalk.github.io/) ：一个基于 Github Issue 和 Preact 开发的评论插件。

## 步骤

> gitalk 是基于 GitHub Issue 来操作评论的，那就需要我们在 GitHub 上注册指定的 Application 来申请操作权限！

### 1. Register Application

点击 https://github.com/settings/applications/new 注册链接去自己 GitHub 上注册新应用。

<img desc="Github 注册 App" src="github_register_app.jpg" width="60%"/>

说明：

* Application name # 随便写，这里就起个 Gitalk 吧
* Homepage URL # 网站 URL，写自己的博客地址，类似：https://jolly336.github.io 或 https://nelsonblog.me 我自己做了博客 CANME
* Authorization callback URL # 当评论系统出错时，跳转地址，这里和 homepage URL 保持一致就行
* Application description，可选，随便写点啥

填写完成后点击注册，会生成一个 GitHub Application，[点击这里](https://github.com/settings/developers) 的 OAuth Apps 列表会看到创建好的 App，如果后面评论调试出错了，还需要回这里修改的。

注册完记录下生成的 Client ID 和 Client Secret 两个 key，待会修改主题配置需要用到。

![Gitalk App](github_app_gitalk_keys.jpg)

### 2. 修改 next 主题

> 提示：**以下修改都在自己本地博客 next 主题目录！** 找到 next 主题目录，我自己博客对应的是 jolly336.github.io/themes/next 目录。

1. **gitalk.swig**

新建 /layout/_third-party/comments/gitalk.swig 文件，并添加内容：

```
{% if page.comments && theme.gitalk.enable %}
  <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
  <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
  <script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js"></script>
   <script type="text/javascript">
        var gitalk = new Gitalk({
          clientID: '{{ theme.gitalk.ClientID }}',
          clientSecret: '{{ theme.gitalk.ClientSecret }}',
          repo: '{{ theme.gitalk.repo }}',
          owner: '{{ theme.gitalk.githubID }}',
          admin: ['{{ theme.gitalk.adminUser }}'],
          id: md5(window.location.pathname),
          distractionFreeMode: '{{ theme.gitalk.distractionFreeMode }}'
        })
        gitalk.render('gitalk-container')
       </script>
{% endif %}
```

2. **comments.swig**

修改 /layout/_partials/comments.swig，添加内容如下：

**打开文件找到这个代码块，在里面添加 gitalk 元素，注意别写错位置了！**，博主就是一开始写错位置，导致 gitalk 评论死活不显示，最后通过 console 才定位到是这的问题！

* 添加之前：

```
{% elseif theme.valine.appid and theme.valine.appkey %}
    <div class="comments" id="comments"></div>
{% endif %}
```

* 要添加的 gitalk 元素代码块：

```
{% elseif theme.gitalk.enable %}
<div id="gitalk-container"></div>
```

* 添加之后：

```
{% elseif theme.valine.appid and theme.valine.appkey %}
    <div class="comments" id="comments"></div>
    {% elseif theme.gitalk.enable %}    //  将复制的两行代码贴在这个位置
    <div id="gitalk-container"></div>
{% endif %}
```

3. **index.swig**

修改 layout/_third-party/comments/index.swig，在最后一行添加内容：

```
{% include 'gitalk.swig' %}
```

4. **gitalk.styl**

新建 /source/css/_common/components/third-party/gitalk.styl 文件，添加内容：

```
.gt-header a, .gt-comments a, .gt-popup a
  border-bottom: none;
.gt-container .gt-popup .gt-action.is--active:before
  top: 0.7em;
```

5. **third-party.styl**

修改 /source/css/_common/components/third-party/third-party.styl，在最后一行上添加内容，引入样式：

```
@import "gitalk";
```

6. **_config.yml**

在主题配置文件 next/_config.yml 中添加如下内容：

```
gitalk:
  enable: true # 是否开启Gitalk
  githubID: jolly336  # 你的github ID，用来说明你是个人还是某个组织的，一定需要；
  repo: blog-comments # 你要新建一个 repo 来保存这些 comments，这里 repo 就随便新建一个就行；博主自己在 Github 上创建了一个名为 blog-comments 的仓库专门存储评论了
  ClientID: 30b4a5db3a17c04cb39d # 新建的 GitHub App，把第一步申请的 Client ID 复制过来就行。请注意，这个一定要和你部署的网站的对应起来，一个网站对应一个这个client；
  ClientSecret: 88e3b0d57f6fe55f9bdb09513bad26b7ae314081
  adminUser: jolly336 # 你的admin 用户名，通常就是你自己
  distractionFreeMode: true
```

以上就是 NexT 中添加 gitalk 评论的配置，博客上传到 GitHub 上后，打开页面进入某一博客内容下，就可看到我们增加的 Gitalk 评论了。

看见自己创建好的 Gitalk 评论系统，是不是有点小激动呢？：）

![创建好的 Gitalk 评论系统](gitalk_comment.jpg)

下一节分享如何统计博客阅读量，敬请期待~

## 参考

* [终于！！！记录如何在hexo next主题下配置gitalk评论系统](https://jinfagang.github.io/2018/10/07/%E7%BB%88%E4%BA%8E%EF%BC%81%EF%BC%81%EF%BC%81%E8%AE%B0%E5%BD%95%E5%A6%82%E4%BD%95%E5%9C%A8hexo-next%E4%B8%BB%E9%A2%98%E4%B8%8B%E9%85%8D%E7%BD%AEgitalk%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F/)
* [Hexo NexT主题中集成gitalk评论系统](https://asdfv1929.github.io/2018/01/20/gitalk/)
* [hexo 使用 gitalk 评论组件的几个注意点！！！](https://www.jianshu.com/p/b4ca8e7c7980)

