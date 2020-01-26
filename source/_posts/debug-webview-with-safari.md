---
title: 借助 Safari 调试苹果手机上的 webView
date: 2018-07-25 15:21:07
tags: webView
categories: iOS
---

## iPhone 真机/模拟器设置

需要如下图所示，点击“设置” → 点击 “Safari 浏览器” → 点击“高级” → 打开“Web 检查器”。

<!--more-->

![打开 Web 检查器](open-the-web-inspector.jpg)

若模拟器中无“Web 检查器”选项，无需设置。

## Safari 设置

打开 Mac 电脑中的 Safari 浏览器，打开偏好设置，点击菜单中的“高级”选项卡，勾选“在菜单栏中显示“开发”菜单”

![Safari-偏好设置-高级](safari-perferences-advanced.jpg)

## 进入检查器

在手机/模拟器中打开浏览器/App中的某个网页，在 Mac 中打开 Safari，在“开发”中找到目标设备。

![选择目标设备](select-target-device.jpg)

> 如果你选择的是模拟器，但是开发列表中未出现，重启 Safari 即可。因为必须确保先打开模拟器，再打开 Safari。


点击目标设备中需要查看的网址，便会弹出这个页面对应的检查器。

![网页检查器](web-inspector.jpg)