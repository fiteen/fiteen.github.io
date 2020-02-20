---
title: MWeb 破解安装
---

> ⚠️⚠️⚠️ **请支持[正版](https://zh.mweb.im)，仅供技术交流。**

### 安装步骤

1、点击[下载（提取码：yzz8）](https://pan.baidu.com/s/1NYRwJKTVJegd9c0E1vwXCQ) MWeb v3.3.4 破解版。

2、安装前先检查`系统偏好设置 → 安全性与隐私 → 通用`中是否已经选择`任何来源`选项，如果没有，在终端输入：

```
sudo spctl --master-disable
```

3、安装打开时，如果还是出现 `“MWeb”已损坏，无法打开。 您应该将它移到废纸篓。` 的提示，检查一下自己的 macOS 系统。如果是 macOS 10.15 Catalina，通过以下命令绕过苹果的公证 Gatekeeper：

```
sudo xattr -rd com.apple.quarantine /Applications/MWeb.app
```

