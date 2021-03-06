---
title: 你应该了解的 IDFA
date: 2017-02-07 14:26:10
tags: IDFA
categories: iOS
---

## 何为 IDFA

[IDFA](https://developer.apple.com/documentation/adsupport/asidentifiermanager)，苹果 iOS6 开始新增的广告标识符， 全称 Identifier For Advertising，是每台 iOS 设备的唯一 ID，是投放定向广告的唯一方法。

<!--more-->

在苹果禁用 UDID 后，IDFA 成为了标识 iPhone 用户的标准。通常用于广告追踪，在同一设备的不同 App 间进行信息共享。

IDFA 是一段 16 进制的 32 位字符串，例如`D7DFA3F1-0E1C-49CD-AFBC-75601390FEA2`。可以通过以下代码获取：

```objc
#import <AdSupport/ASIdentifierManager.h>

NSString *idfa = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
```

这个标识符虽然是唯一的，但并不是固定不变的，用户可以通过以下两种方式进行重置：

- 设置→隐私→广告→还原广告标识符
- 设置→通用→还原→还原所有设置/还原位置与信息

iOS10 之后，还新增了“**限制广告追踪**” 的设置，所以在获取 IDFA 之前，最好优先判断一下 `[[ASIdentifierManager sharedManager] isAdvertisingTrackingEnabled]` 返回的 BOOL 值，假如返回的是 YES，则能获取正确的 IDFA，否则获取到的字符串就会变成 `00000000-0000-0000-0000-000000000000`。

因此，IDFA 并不能成为精确标识用户唯一性的符号。如果要确保唯一且固定，建议采用 UUID+Keychain 的方式，或者借助 iOS 系统可以获取的参数自定义一套算法去生成标志符。


## 检查是否使用 IDFA

当 App 提交应用市场审核的时候，苹果会询问“*此 App 是否使用广告标识符号（IDFA）*”。这里除了本地代码以外，还需要鉴别导入的任何第三方库中，是否使用了 IDFA。检查的方法很简单：

1. 打开终端 cd 到要检查的文件根目录
2. 执行语句 `grep -r advertisingIdentifier . `

以含 IDFA 的友盟 SDK 为例，会出现 matches 的记录。

![检查命令](grep-command.png)


## 审核时关于 IDFA 选项的选择

那么如果选择了 “是”，就会提示你选择勾选 4 个选项框：

1. **在 App 内投放广告**

   服务应用中的广告。如果你的应用中集成了广告的时候，你需要勾选这一项。

2. **标明此 App 安装来自先前投放的特定广告**

   跟踪广告带来的安装。如果你使用了第三方的工具来跟踪广告带来的激活以及一些其他事件，但是应用里并没有展示广告你需要勾选这一项。

3. **标明此 App 中发生的操作来自先前投放的广告**

   跟踪广告带来的用户的后续行为。如果你使用了第三方的工具来跟踪广告带来的激活以及一些其他事件，你需要勾选这一项。

4. **iOS 中的“限制广告跟踪”设置**

   对您的应用使用 IDFA 的目的做下确认，只要您获取了 IDFA，那么这一项都是需要勾选的。

> **总结**
>
> 1. 如果你的应用里只是集成了广告，不追踪广告带来的激活行为，那么选择 1 和 4；
> 2. 如果你的应用没有广告，而又获取了 IDFA，选择 2 和 4；
> 3. 如果你的应用没有广告，但是需要追踪广告带来的激活行为，那么选择 2、3 和 4；
> 4. 如果你的应用里集成了广告，而且使用了 SDK 等用来追踪广告带来的激活行为，需要选择 1、2、3 和 4 。

如果还是无法确定如何选择，可以参考第三方的官方文档，基本上都会在开发文档中体现。