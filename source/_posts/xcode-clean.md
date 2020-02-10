---
title: 释放你的内存——Xcode 缓存清理
date: 2017-03-27 23:15:12
tags: Xcode
categories: iOS
---

Xcode 使用久了经常会遇到系统内存不足的情况，我们来看看哪些缓存是可以清理的。

<!--more-->

## 设备版本支持文件

**路径**：`~/Library/Developer/Xcode/iOS DeviceSupport`。

存放的是所有真机调试过的设备版本支持文件，每个版本文件夹有几个G的大小，删除后可恢复，重新连接设备时可以重新生成相应版本的文件。

**结论**：文件较大，建议**删除不需要的版本文件夹**，比如一些旧版。

## 模拟器型号

**路径**：`~/Library/Developer/CoreSimulator/Devices`。

存放的是模拟器。每个模拟器标识符代表一台模拟器设备，具体见 `device.plist`。删除前先关闭所有模拟器，删除后不可恢复，但下次启动时可以重新创建模拟器。

**结论**：文件较大，可以选择**删除部分模拟器**，或者全删后，重新手动下载。

## 项目编译产生的缓存

**路径**：`~/Library/Developer/Xcode/DerivedData`。

存放的是 Xcode 编译项目产生的缓存，可重新生成。但如果删除了，下次编译时项目时会需要更多时间，可以保留当前需要维护的项目文件夹。

**结论**：文件大小看项目规模，建议**删除无需维护项目缓存**。

## 打包历史记录

**路径**：`~/Library/Developer/Xcode/Archives`。

存放的是按日期分类的 `.xcarchive` 文件。`.xcarchive` 是通过 Xcode 或者 xcodebuild archive 打包生成的文件，里面包括了.app文件、dSYM 符号文件等。它也对应的 Xcode-organizer 下的 Archives 列表，删除后**不可恢复**。

**结论**：文件大小看包体积，建议**删除多余的打包记录**。

## 打包产生的 App icon 历史版本

**路径**：`~/Library/Developer/Xcode/Products`。

**结论**：文件较小，建议**全部删除**。

## Playground 项目缓存

**路径**：`~/Library/Developer/XCPGDevices`。

存放的是建立/运行 Playground 项目时产生的缓存，删除后可恢复，运行时又会重新生成。

**结论**：文件大小看项目规模，建议**全部删除**。

## 描述文件

**路径**：`~/Library/MobileDevice/Provisioning Profiles`。

存放的是 App 签名时的描述文件，删除后可以再从苹果开发者账号中下载。

**结论**：文件较小，可以选择**删除无用的描述文件**。
