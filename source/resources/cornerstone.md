---
title: Cornerstone 破解
---

> ⚠️⚠️⚠️ **请支持正版，仅供技术交流。**

[官方网站](https://cornerstone.assembla.com)

## 网上直接搜索破解版

例如：吾爱破解提供的[百度网盘下载地址](https://pan.baidu.com/s/19UY-fokUDgEcX85O69dSng)  密码:3ycy

## 自行破解

#### 方案一（未安装软件）

如果之前未安装，直接把当前系统时间改成未来的某个时间，再安装，成功运行一次后可以退出程序，将系统时间还原。

#### 方案二（已安装软件）

如果安装前未修改系统日期，需要通过修改 plist 文件的方式破解。步骤如下：

1. 显示系统隐藏文件。

  在终端中输入以下命令：

  ```
  defaults write    ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool true
  ```
  
  这里 `true` 改成 `false` 就可以不再显示隐藏文件。

2. 重启 Finder。

   按住 `Command+Option+esc` 打开对话框，选中Finder，点击“重新开启”。

3. 删除安装记录。

   前往目录 `~/Library/Preferences/ByHost`，找到 `GlobalPreferences.XXXX.plist` 文件，`XXXX` 是一串数字，删掉包含 `com.zennaware.Cornerstone:2.0` 字样的一行，然后保存。

4. 重新安装。

   卸载应用，按照方法一的步骤再次安装，就能得到永久破解版。
