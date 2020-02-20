---
title: Jetbrains Mac 版全系列 IDE 永久破解教程
---

> ⚠️⚠️⚠️ **请支持[正版](https://www.jetbrains.com/idea/)，仅供技术交流。**
**学生**凭学生证可**免费**申请正版授权。
**创业公司**可**五折**购买正版授权。

### 注意事项

现在 http://idea.lanyus.com/ 里的激活码已经不能用的，如果你曾经在 `/private/etc/hosts` 文件里 Jetbrains 相关的项⽬，请先删除。

以下教程**适用于 Jetbrains 全系列产品2019.3.1及以下版本。**，下文以 IntelliJ IDEA 的破解为例。

### 激活步骤

1、下载安装包。前往[官网](https://www.jetbrains.com/idea/)下载正版的 IDEA。

2、点击[下载（提取码：412x）](https://pan.baidu.com/s/1sosqRASs1WmKJMV8j8qHPQ)补丁文件，可以将补丁文件 `jetbrains-agent.jar` 放在 IDEA 安装目录的 bin 文件夹（即 `/Applications/IntelliJ IDEA.app/Contents/bin`）下。

3、启动 IDE，点击激活窗口的 `Evalutate for free`，开启试用。

4、修改配置。

进入欢迎页，在下图中的 `Configure` 中选择 `Edit Custom VM Options …`。

![](/resources/images/jetbrains-configure.png)

如下图所示，在弹出的 `idea.vmoptions` 文件的最后一行加入 `-javaagent:/Applications/IntelliJ IDEA.app/Contents/bin/jetbrains-agent.jar`。

![](/resources/images/jetbrains-vmoptions.png)

5、重启 IDE。（这一步切记**不要遗漏**。）

6、在欢迎页 `Configure` 中选择 `Manage License...`，选择 License server ⽅式，点击 `Discover Server` 按钮应该会⾃动填充上地址：`http://jetbrains-license-server`。然后点击 `Activate` 按钮，看到如下 `Licensed to XXX` 则激活完毕。

![](/resources/images/jetbrains-licensed-license-server.png)
   
如果不成功，尝试一下 Activation code 方式离线激活。将 `ACTIVATION_CODE.txt` 中的激活码粘贴到内容框中。你也可以选择[自定义激活码](https://zhile.io/custom-license.html)。
   
看到如下页面则激活完毕。

![](/resources/images/jetbrains-licensed-activation-code.png)
   
---
参考链接：[Jetbrains 系列产品2019.3.1最新激活方法[持续更新]](https://zhile.io/2018/08/25/jetbrains-license-server-crack.html)

