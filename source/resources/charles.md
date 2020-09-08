---
title: Charles Mac 版破解
---

> ⚠️⚠️⚠️ **请支持[正版](https://www.charlesproxy.com)，仅供技术交流。**

步骤：

1、从[官网下载](https://www.charlesproxy.com/download/latest-release/)最新版的安装包，如果下载速度太慢，也可以[在此](https://lemon.qq.com/lab/app/Charles.html)下载。

2、在[Charles 在线破解工具](https://www.zzzmode.com/mytools/charles/)网站上生成破解后的 charles.jar 文件

3、替换本地`/Applications/Charles.app/Contents/Java` 目录下的 charles.jar 文件

如果安装时出现这样的报错信息：

![](/resources/images/damaged.png)

只需要在命令行中输入：

```bash
sudo spctl --master-disable
```

设置隐私中， 允许任何来源软件即可。

