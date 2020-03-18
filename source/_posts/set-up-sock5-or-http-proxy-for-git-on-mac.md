---
title: 在 Mac 上为 Git 和终端设置代理
date: 2018-09-02 18:59:02
tags:
    - Shadowsocks
    - V2Ray
    - GitHub
categories: Git
thumbnail: github.png
---

我们常会遇到从 GitHub 中 clone 代码或终端执行命令速度感人的情况，这时如果手上有不错的代理，可以借助代理来更快地下载资源。

<!--more-->
## 查看代理的监听地址和端口

先在本地 Shadowsocks/V2Ray 客户端中查看设置的本机 sock5/http 监听端口和 Host。例如：

![](view-port.png)

## Git

通常我们 clone 代码时有以下两种方式：

```bash
# HTTPS 协议
https://github.com/accountname/projectname.git
# SSH 协议
git@github.com:accountname/projectname.git
```

### 设置 HTTPS 协议的代理

以上面的配置为例，有如下两种方案：

设置全局 git 代理，注意这里不需要设置 `https.proxy`，[Git Documentation](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy) 中没有这个参数。

```bash
# 设置 socks5 代理
git config --global http.proxy socks5://127.0.0.1:1080
# 用 socks5h 速度更快
git config --global http.proxy socks5h://127.0.0.1:1080
# 设置 http 代理
git config --global http.proxy http://127.0.0.1:1087
```

> 注意：`ip` 和 `port` 根据自己本地实际配置修改。

取消代理：

```bash
git config --global --unset http.proxy
```

### 单独设置 SSH 协议的代理

修改用户目录下文件  `~/.ssh/config` 里的内容，对 GitHub 域名作单独处理：

```bash
Host github.com
    # 若使用的是默认端口，设置如下
    HostName           github.com
    # 如果想用443端口，设置如下
    # Hostname         ssh.github.com
    # Port             443
    User               git
    # 如果是 SOCKS5 代理，取消下面这行注释，并把 1080 改成自己 SOCKS5 代理的端口
    ProxyCommand     nc -x localhost:1080 %h %p
    # 如果是 HTTP 代理，取消下面这行注释，并把 1087 改成自己 HTTP 代理的端口
    # ProxyCommand     socat - PROXY:127.0.0.1:%h:%p,proxyport=1087
```

## Shell 终端

想让终端走代理那么只需在 `~/.bashrc` 或 `~/.zshrc` 文件中，直接写入以下内容并保存：

```bash
alias setproxy="export ALL_PROXY=socks5://127.0.0.1:1080"
alias unsetproxy="unset ALL_PROXY"
alias ip="curl -i http://ip.cn"
```

利用终端下载资源时，先执行  `setproxy` 命令，结束后执行  `unsetproxy` 命令如果终端提示 `command not found: setproxy`，说明配置没有生效，执行一下  `source ~/.bashrc` 或 `source ~/.zshrc` 即可。