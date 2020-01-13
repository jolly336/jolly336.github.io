---
title: Mac Terminal 终端代理配置
date: 2019-12-03 00:04:19
tags: Mac
categories: Mac
---

本文讲解如何给 Mac 系统终端配置 Http 代理设置，代理我们使用 Shadowsocks。经常在命令行终端下工作的码农们，SS无法正常工作。因为在终端下不支持socks5代理，只支持http代理，这就很尴尬了。像常用的 wget、curl、git、brew 等命令行工具执行会很慢。

<!-- more -->

## 配置过程

1. ~/.bash_profile 文件配置代理开关

在 Mac 终端用 vim 编辑器打开 `~/.bash_profile`环境变量配置文件，添加如下配置代码

```
# ------terminal http proxy ----------
function proxy_on() {
    export http_proxy = http://127.0.0.1:1087
    export https_proxy = http://127.0.0.1:1087
    echo - e "已开启代理"
}
function proxy_off() {
    unset http_proxy
    unset https_proxy
    echo - e "已关闭代理"
}
```

注意：`http://127.0.0.1:1087`表示使用本地回环 ip 地址，端口是 Shadowsocks 开启的 HTTP 代理端口。具体可在 Shadowsocks 偏好设置里查看，如果没有开启 HTTP 代理，需要开启一下！

<img desc="Shadowsocks-Preferences" src="shadowsocks_preferences.jpg" width="60%" height="60%" />

2. 在终端直接运行 `proxy_on`即可打开终端 http 代理，`proxy_off` 关闭代理

运行命令`curl ip.gs`来检查是否开启成功！

<img desc="检查代理效果" src="terminal_ping.jpg" width="80%" height="50%" />

## 其它

* wikipedia 关于[ Shadowsocks](https://zh.wikipedia.org/zh-hans/Shadowsocks)的介绍如下：

> Shadowsocks（简称SS）是一种基于Socks5代理方式的加密传输协议，Shadowsocks分为服务器端和客户端，在使用之前，需要先将服务器端程序部署到服务器上面，然后通过客户端连接并创建本地代理。     
Shadowsocks的运行原理与其他代理工具基本相同，使用特定的中转服务器完成数据传输。例如，用户无法直接访问Google，但代理服务器可以访问，且用户可以直接连接代理服务器，那么用户就可以通过特定软件连接代理服务器，然后由代理服务器获取网站内容并回传给用户，从而实现代理上网的效果。另外用户在成功连接到服务器后，客户端会在本机构建一个本地Socks5代理（或VPN、透明代理等）。浏览网络时，网络流量需要先通过本地代理传递到客户端软件，然后才能发送到服务器端，反之亦然。

* 关于如何配置客户端和服务端代理，可以查看博主的另一篇博客  [科学上网的姿势，帮你解决翻墙上网的烦恼](http://nelsonblog.me/2018/03/25/%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%9A%84%E5%A7%BF%E5%8A%BF%EF%BC%8C%E5%B8%AE%E4%BD%A0%E8%A7%A3%E5%86%B3%E7%BF%BB%E5%A2%99%E4%B8%8A%E7%BD%91%E7%9A%84%E7%83%A6%E6%81%BC/)

## Q&A

* 终端代理已经显示国外的 ip，但是还是 ping 不通，为什么？

ss 代理是基于 tcp 或者 udp 协议，而 ping 是走的 icmp 协议因此在 ss 下不能 ping 通 google！

## 参考

* [mac 终端实现翻墙](https://kerminate.me/2018/10/22/mac-%E7%BB%88%E7%AB%AF%E5%AE%9E%E7%8E%B0%E7%BF%BB%E5%A2%99/)


