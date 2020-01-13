---
title: Tmux 使用教程
date: 2019-12-30 23:31:55
tags: 工具
categories: 工具
---

相信很多人都在使用 tmux，但也可能有一些人不知道有 tmux 的存在，看看下图你就知道 tmux 是做什么的了。:）

<!-- more -->

![tmux-ohmyzsh](http://wanghaoxun.com/img/tmux-ohmyzsh.jpg)

## 前言

人最宝贵的是时间。要提高工作效率，就要熟练运用各种工具。tmux 是我日常工作中都要用到的，凡是看到朋友或公司同事用鼠标慢吞吞切换多个窗口，我都忍不住推荐并教会他们使用 tmux。好的工程师不在于年龄有多大，工作年限有多长，你会发现他们都有一个共同特点，就是很善于用工具，肯花时间去学习去配置自己每天用的工具（vim，tmux,shell 等）。他们能熟练地用快捷键在代码中游走，用快捷键熟练地切换各种窗口（比如本文要介绍的 tmux）。

## 一、Tmux 是什么？

![tmux-logo](http://wanghaoxun.com/img/tmux-logo.jpg)

Tmux 是一个非常有用的终端复用器（terminal multiplexer），一个用于在终端窗口中运行多个终端会话的工具。在 Tmux 中可以根据不同的工作任务创建不同的会话，每个会话又可以创建多个窗口来完成不同的工作，每个窗口又可以分割成很多小窗口。这些功能都是非常实用的。

## 二、基本用法

### 2.1 安装

MacOS 中安装

```
$ brew install tmux
```

开启 oh-my-zsh tmux 插件，编辑 vim ~/.zshrc 文件，搜索找到 plugins，添加如下即可。

```
plugins=(
  tmux
)
```
### 2.2 启动与退出

**安装完成后，键入 tmux 命令，就进入了 Tmux 窗口。**

```
$ tmux
```

上面命令会启动 tmux 窗口，底部有一个状态栏。状态栏的左侧是窗口信息（编号和名称），右侧是系统信息。

**按下 ctrl+d 或者显式输入 exit 命令，就可以退出 tmux 窗口。**

```
$ exit
```

### 2.3 前缀键

tmux 窗口有大量的快捷键。所有快捷键都要通过前缀键唤起。默认的前缀键是 ctrl+b ，即先按下 ctrl+b ，快捷键才会生效。

举例来说，帮助命令的快捷键是 ctrl+b ? 。它的用法是，在 tmux 窗口中，先按下 ctrl+b ，再按下 ? ，就会显示帮助信息。

然后，按下 ESC 键或 q 键，就可以退出帮助。

## 三、会话管理

### 3.1 新建会话

第一个启动的 tmux 窗口，编号是 0 ，第二个窗口的编号是 1 ，以此类推。这些窗口对应的会话，就是 0 号会话、1 号会话。

使用编号区分会话，不太直观，更好的方法是为会话起名。

```
$ tmux new -s <session-name>
```

上面命令新建一个指定名称的会话。

### 3.2 分离会话

在 tmux 窗口中，按下 ctrl+b d 或者输入 tmux detach 命令，就会将当前会话与窗口分离。

```
$ tmux detach
```

上面命令执行后，就会退出当前 tmux 窗口，但是会话和里面的进程仍然在后台运行。

tmux ls 命令可以查看当前所有的 Tmux 会话。

```
$ tmux ls
# or
$ tmux list-session
```
### 3.3 接入会话

tmux attach 命令用于重新接入某个已存在的会话。

```
# 使用会话编号
$ tmux attach -t 0

# 使用会话名称
$ tmux attach -t <session-name>
```
### 3.4 杀死会话

tmux kill-session 命令用于杀死某个会话。

```
# 使用会话编号
$ tmux kill-session -t 0

# 使用会话名称
$ tmux kill-session -t <session-name>
```

### 3.5 切换会话

tmux switch 命令用于切换会话。

```
# 使用会话编号
$ tmux switch -t 0

# 使用会话名称
$ tmux switch -t <session-name>
```

### 3.6 重命名会话

tmux rename-session 命令用于重命名会话。

```
$ tmux rename-session -t 0 <new-name>
```

上面命令将0号会话重命名。

### 3.7 会话快捷键

下面是一些会话相关的快捷键。

```
Ctrl+b d
Ctrl+b s
Ctrl+b $
```
## 四、窗格快捷键

下面是一些窗格操作的快捷键。

* ctrl+b % ：划分左右两个窗格。
* ctrl+b " ：划分上下两个窗格。
* ctrl+b <arrow key> ：光标切换到其他窗格。 <arrow key> 是指向要切换到的窗格的方向键，比如切换到下方窗格，就按方向键 ↓ 。
* ctrl+b ; ：光标切换到上一个窗格。
* ctrl+b o ：光标切换到下一个窗格。
* ctrl+b { ：当前窗格左移。
* ctrl+b } ：当前窗格右移。
* ctrl+b Ctrl+o ：当前窗格上移。
* ctrl+b Alt+o ：当前窗格下移。
* ctrl+b x ：关闭当前窗格。
* ctrl+b ! ：将当前窗格拆分为一个独立窗口。
* ctrl+b z ：当前窗格全屏显示，再使用一次会变回原来大小。
* ctrl+b Ctrl+<arrow key> ：按箭头方向调整窗格大小。
* ctrl+b q ：显示窗格编号。

## 五、窗口管理

除了将一个窗口划分成多个窗格，Tmux 也允许新建多个窗口。

### 5.1 新建窗口

tmux new-window 命令用来创建新窗口。

```
$ tmux new-window

# 新建一个指定名称的窗口
$ tmux new-window -n <window-name>
```

### 5.2 切换窗口

tmux select-window 命令用来切换窗口。

```
# 切换到指定编号的窗口
$ tmux select-window -t <window-number>

# 切换到指定名称的窗口
$ tmux select-window -t <window-name>
```

### 5.3 重命名窗口

tmux rename-window 命令用于为当前窗口起名（或重命名）。

```
$ tmux rename-window <new-name>
```

### 5.4 窗口快捷键

下面是一些窗口操作的快捷键。

* ctrl+b c ：创建一个新窗口，状态栏会显示多个窗口的信息。
* ctrl+b p ：切换到上一个窗口（按照状态栏上的顺序）。
* ctrl+b n ：切换到下一个窗口。
* ctrl+b <number> ：切换到指定编号的窗口，其中的 <number> 是状态栏上的窗口编号。
* ctrl+b w ：从列表中选择窗口。
* ctrl+b , ：窗口重命名。

## 六、其他命令

下面是一些其他命令。

```
# 列出所有快捷键，及其对应的 Tmux 命令
$ tmux list-keys

# 列出所有 Tmux 命令及其参数
$ tmux list-commands

# 列出当前所有 Tmux 会话的信息
$ tmux info

# 重新加载当前的 Tmux 配置
$ tmux source-file ~/.tmux.conf
```

## Q&A

* tmux 环境下 ls 目录颜色失效？

如果要显示目录颜色，就得配置 LS_COLORS 或者 LSCOLORS，Mac 这种 freeBSD 系统采用的是后者，需要拷贝如下代码到 ~/.zshrc 文件中，

```sh
# LSCOLORS
export LSCOLORS="exfxcxdxbxexexabagacad"
alias ls='ls -G'
```

source ~/.zshrc 重新刷新下配置，再执行就会发现已经出现颜色了！

## 参考

* [Tmux 使用笔记](https://www.tuicool.com/articles/FJvYJvv)
* [Tmux 使用教程](https://www.tuicool.com/articles/JvqMNbQ)
* [zsh中设置ls目录颜色](http://hushicai.github.io/2014/03/10/zsh-zhong-she-zhi-ls-mu-lu-yan-se.html)
* [wanqu/tmux](https://wanqu.co/a/115/tmux/)


