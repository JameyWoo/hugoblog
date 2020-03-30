---
title: "Git设置http及ssh代理 Win10与ubuntu18"
date: 2020-03-30T19:36:47+08:00
draft: false

categories: ["开发工具"]
tag: ["代理"]
---

## https 代理

之前在网上一直用的是如下设置.

```
// 设置 http 代理
git config --global http.proxy "http://127.0.0.1:8080"
git config --global https.proxy "http://127.0.0.1:8080"

// 或者 socks5 代理
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"

// 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

这种方式可以成功代理 http 方式的 git clone, 但是用 ssh 方式就还是 10+/20+ KiB的速度. 我之前一般都是用 ssh 的方式 clone.


## ssh 代理

我以前并不知道github中git clone对ssh和https链接的区别, 所以一直为这个困惑. 今天了解到, 原来走 ssh 的方式需要经过设置专门的代理方式.

### win10

在 win10 中, 需要在 `~/.ssh/config` 文件中添加以下内容
```
Host github.com
  User git
  Port 22
  Hostname github.com
  # socks5 代理用这个
  ProxyCommand connect -S 127.0.0.1:10808 -a none %h %p
  # http 代理用下面这个
  # ProxyCommand connect -H 127.0.0.1:10809 -a none %h %p
```
其中, `127.0.0.1:10808` 设置成自己的代理配置. 我用的是 socks5 代理. 

connect 是一个exe可执行程序, 在我的电脑中是在下面的目录
```
C:\Program Files\Git\mingw64\bin\connect.exe
```
要使用这个命令, 需要将目录 `C:\Program Files\Git\mingw64\bin` 添加到环境变量

执行 `connect -h` 可获取相关帮助
```
$ connect -h
connect --- simple relaying command via proxy.
Version 1.105
usage: C:\Program Files\Git\mingw64\bin\connect.exe [-dnhst45] [-p local-port]
          [-H proxy-server[:port]] [-S [user@]socks-server[:port]]
          [-T proxy-server[:port]]
          [-c telnet-proxy-command]
          host port
```

### Ubuntu-18.04

在ubuntu-18.04中也是类似的配置, 不过我不是用`connect.exe`, 而是用`nc`命令. 

同样的, 在 `~/.ssh/config` 中写入以下配置

```
Host github.com
    HostName github.com
    User git
    # http 代理方法
    # ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=6667
    # socks5 代理方法
    ProxyCommand nc -v -x 127.0.0.1:10808 %h %p
```

设置成功后, 执行 `git clone`, 速度达到了 `4MB/s`.

![](/images/20200330-1.png)