---
title: Git 手册之用 .gitignore 忽略文件
date: 2016-11-22 01:34:57
tags: .gitignore
categories: Git
---

提交代码后我们经常发现，即使没有任何代码修改，有一些文件也会提示更新，例如：`UserInterfaceState.xcuserstate`、`.DS_Store` 等。

这种情况可以通过添加 `.gitignore` 文件解决。

<!--more-->

## 如何在项目中添加 `.gitignore`

具体**步骤**如下：

步骤一：打开终端 进入项目中 `.git` 同目录下
```bash
cd /path/to/file
```

步骤二：创建 `.gitignore` 文件

```bash
touch .gitignore
```

步骤三：打开 `.gitignore` 文件

```bash
open .gitignore
```

步骤四：参照 [.gitignore 模版](https://github.com/github/gitignore)，找到对应的开发语言，将模版文本粘贴到自己的 `.gitignore` 中

步骤五：更新项目

```bash
git add .gitignore
git commit -m "feat: add .gitignore file"
git push
```

**注意**：

1、`.gitignore` 只能作用于未跟踪的文件，也就是未被 git 记录的文件；

2、已经 `push` 过的文件，如果想要在本地保留，但想要从远程仓库中删除，并在以后的提交中忽略，执行命令：

```bash
git rm --cached /path/to/file
```

3、已经 `push` 过的文件，想在以后的提交时忽略跟踪，也就是说即使本地已经修改过，但不修改也不删除远程仓库中相应文件，执行命令：

```bash
git update-index --assume-unchanged /path/to/file
```
如果要取消忽略，执行命令：

```bash
git update-index –no-assume-unchanged /path/to/file
```
  

## 删除 .DS_Store

`.DS_Store` 是Mac OS 保存文件夹的自定义属性的隐藏文件。如果项目中还没有自动生成 `.DS_Store`，把它加入到 `.gitignore` 中即可；但如果项目中已经有了，先从项目中将其删除，再把它加入到 `.gitignore` 里。步骤如下：

步骤一：删除项目中的所有 `.DS_Store`

```bash
find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch
```

步骤二：将 `.DS_Store` 加入到 `.gitignore` 文件中

```bash
echo .DS_Store >> ~/.gitignore
```

步骤三：更新项目

```bash
git add --all
git commit -m "feat: ignore .DS_Store"
git push
```

如果只需要删除磁盘上的 `.DS_Store`，用下面的命令来删除当前目录及其子目录下的所有 `.DS_Store` 文件：

```bash
find . -name '*.DS_Store' -type f -delete
```

你也可以通过输入这串命令直接禁止生成 `.DS_store`，重启Mac即可生效：

```bash
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

恢复 `.DS_store` 生成的命令为：

```bash
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```