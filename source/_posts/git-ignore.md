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

具体步骤如下：

1. 打开终端 进入项目中 `.git` 同目录下

   ```
   cd <path>
   ```

2. 创建 `.gitignore` 文件

   ```
   touch .gitignore
   ```

3. 打开 `.gitignore` 文件

   ```
   open .gitignore
   ```

4. 参照 [.gitignore 模版](https://github.com/github/gitignore)，找到对应的开发语言，将模版文本粘贴到自己的 `.gitignore` 中

5. 更新项目

   ```
   git add .gitignore
   git commit -m "feat: add .gitignore file"
   git push
   ```
## 删除 .DS_Store

`.DS_Store` 是Mac OS 保存文件夹的自定义属性的隐藏文件。如果项目中还没有自动生成 `.DS_Store`，把它加入到 `.gitignore` 中即可；但如果项目中已经有了，先从项目中将其删除，再把它加入到 `.gitignore` 里。步骤如下：

1. 删除项目中的所有 `.DS_Store`

   ```
   find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch
   ```

2. 将 `.DS_Store` 加入到 `.gitignore` 文件中

   ```
   echo .DS_Store >> ~/.gitignore
   ```

3. 更新项目

   ```
   git add --all
   git commit -m "feat: ignore .DS_Store"
   git push
   ```

如果只需要删除磁盘上的 `.DS_Store`，用下面的命令来删除当前目录及其子目录下的所有 `.DS_Store` 文件：

```
find . -name '*.DS_Store' -type f -delete
```

你也可以通过输入这串命令直接禁止生成 `.DS_store`，重启Mac即可生效：

```
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

恢复 `.DS_store` 生成的命令为：

```
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```