---
title: Mac 下避免 rm 引发的血案
date: 2018-04-12 21:53:11
tags: rm
categories: 技巧
---

习惯使用终端的用户，常会用 `rm -fr` 命令执行删除操作，但是这种删除的方式不会出现在废纸篓中，一旦误删，要想找回就比较麻烦。近期听说的此类血案也比较多，为了避免造成悲剧，推荐使用 trash 命令来执行删除。

<!--more-->

### 安装 trash

通过 Homebrew 安装 [Trash](https://github.com/ali-rantakari/trash)

```bash
$ brew install trash
```

安装成功后，可以通过 `trash -fr filename `命令删除文件，且文件会移到废纸篓中。

### 用 trash 替换 rm 命令

打开 ~/.bash_profile 文件

```bash
$ vim ~/.bash_profile
```

在文件中加入以下代码后保存文件：

```
alias rm="trash"
```

使命令生效：

```bash
$ source ~/.bash_profile
```

这时执行 rm 命令，被删除的文件就会存放在废纸篓中了，废纸篓里的文件虽无法执行“放回原处”的方法，但可以通过鼠标拖拽恢复。
