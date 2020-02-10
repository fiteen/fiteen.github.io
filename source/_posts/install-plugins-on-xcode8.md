---
title: 如何在 Xcode8上安装插件
date: 2016-11-30 12:02:20
tags:
    - Xcode
    - plugin
categories: iOS
---

正式推出 Xcode8 已有两个多月，也有不少朋友分享了安装插件的方法，笔者在这里整理了一个亲测有效的方法。

<!--more-->

1、更新 Xcode，目前最新版本是8.1；

2、由于安装插件会影响原来的 Xcode 打包上传，我们在应用程序里复制一个 Xcode，并重命名为 XcodeSigner；

![XcodeSigner](xcodesigner.png)

3、打开钥匙串，创建新证书，名称填 XcodeSigner，证书类型选择代码签名（Code Signing）；

![创建证书](create-a-certificate.png)

![填写证书信息](fill-in-the-certificate-information.png)

4、在终端命令中输入：`sudo codesign -f -s XcodeSigner /Applications/XcodeSigner.app`
耐心等待命令执行完毕；

5、获得 XcodeSigner 的 UUID；

通过在终端命令行输入：

```bash
defaults read /Applications/XcodeSigner.app/Contents/Info DVTPlugInCompatibilityUUID
``` 

6、在 GitHub 上下载好想安装的插件，以 ESJsonFormat 为例，打开方式选择 XcodeSigner；

![选择 XcodeSigner 作为打开方式](select-xcodesigner-as-the-open-method.png)

7、检查 info.plist 中是否已经添加了第5步获得的 UUID，未添加可能会造成 XcodeSigner 闪退。若文件中已经存在，直接运行项目即可；

![添加 UUID](add-an-uuid-to-Info-plist.png)

8、运行成功后，关闭 XcodeSigner ，重新启动，这时会弹出如下两类提示框，分别选择“ Load Bundle ”和“允许”（或“始终允许”），这时点击 Window 就能看到列表中多了 ESJsonFormat 了。

![加载 Bundle](load-bundle.png)

![选择“允许”](select-allow.png)
