---
title: Git 手册之 Mac 上给 Git 设置 SOCKS5/HTTP 代理
date: 2018-09-02 18:59:02
tags: 科学上网
categories: Git
---

我们常会遇到从 GitHub 上 clone 代码的时候龟速的情况，这时如果手上有不错的代理，可以借助代理来获取更快下载/上传资源的速度。

<!--more-->

通常我们 clone 代码时有以下两种方式：

```
// HTTPS 方式
https://github.com/accountname/projectname.git
// SSH 方式
git@github.com:accountname/projectname.git
```

#### 设置 HTTP 方式的代理

由于 Shadowsocks 客户端就提供一个本地的 SOCKS5 代理，代理地址是 127.0.0.1:1080。在终端输入以下配置：

```bash
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"
```

取消代理则：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

也可以直接修改用户主目录下的  `.gitconfig` 文件，插入如下内容：

```
[http]
        proxy = socks5://127.0.0.1:1080
[https]
        proxy = socks5://127.0.0.1:1080
```

如果你用的不是 SOCKS5，而是 HTTP 代理，就把上面命令中的 `socks5` 换成 `http` ，同时改成正确的端口号。

#### 设置 SSH 方式的代理

修改用户目录下文件  `~/.ssh/config ` 里的内容，对 GitHub 域名作单独处理：

```
Host github.com
    # 若使用的是默认端口，设置如下
    HostName           github.com
    # 如果想用443端口，设置如下
    # Hostname         ssh.github.com
    # Port             443
    User               git
    # 如果是 SOCKS5 代理，取消下面这行注释，并把 1080 改成自己 SOCKS5 代理的端口
    # ProxyCommand     nc -x localhost:1080 %h %p
    # 如果是 HTTP 代理，取消下面这行注释，并把 6666 改成自己 HTTP 代理的端口
    # ProxyCommand     socat - PROXY:127.0.0.1:%h:%p,proxyport=6666
```

#### 直接在终端设置临时代理

或者我们可以在 `~/.bashrc`文件中，直接写入以下内容并保存：

```
alias setproxy="export ALL_PROXY=socks5://127.0.0.1:1080"
alias unsetproxy="unset ALL_PROXY"
alias ip="curl -i http://ip.cn"
```

clone 之前先在终端执行  `setproxy` 命令，结束后执行  `unsetproxy` 命令如果终端提示 `command not found: setproxy`，说明配置没有生效，执行一下  `source ~/.bashrc` 即可。