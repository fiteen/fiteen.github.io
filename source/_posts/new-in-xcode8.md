---
title: Xcode 8 新特性
date: 2016-09-14 10:10:20
tags: Xcode
categories: iOS
---

依照苹果一贯的风格，今天，发布 iOS 10 的同时，开发者们期待已久的 Xcode 8 正式版也上线了。它更新了哪些大家感兴趣的部分呢，翻译一下 App Store 里的更新内容：

![](new-in-xcode8.png)

<!--more-->

Xcode 8 支持 Swift 3、iOS 10、watchOS 3、tvOS 10 以及 macOS Sierra。

**Xcode 8 的新特性**：

- 编辑文件时，会高亮当前行代码，swift 中支持彩色和图像文字，添加了补全图片名功能（这个很赞，又可以少用一个插件了）；

- 应用程序拓展可以使第三方能够添加新功能到源编辑器中；

- 可以自动管理或自定义设置代码签名；

- 关于 runtime 运行时，对内存泄漏将会发出警告，针对 UI 对齐以及资源竞争问题，将会通过线程检查工具来解决；

- 在运行时机制中，内存调试器给出数据和对象关系图的可视化和操作形式；

- 优化界面像素控制，可以预览每一种目标设备，同时可以调整缩放级别；

- 对默认字体 San Francisco Mono 进行了字体加大和加粗处理（这点貌似被很多苹果用户吐槽了）；

- 文档查看采用模糊匹配方法，在一个统一的参考库内搜索，即使在脱机时也可以使用；

- Interface  Builder和整个 IDE的优化提升；

- 对Siri功能进行拓展，iMessage也加入新玩法，表情包和贴纸包更多了；


**Swift 3 的新特性**：

- 为 GCD 和 Core Graphics 提供增强版的 Swift API；

- 在 Swift 3 中贯穿统一的 API 风格，甚至包括在平台 SDK 框架里亦然；

- Playgrounds 为开源工具链提供支持；

- Xcode 会帮助你将原来既有的 Swift 代码移植成 Swift 3 语法；

- Swift 2.3 可以直接过渡到 3.0，并提供相应最新的SDK。


以上为翻译内容，如有不到欢迎指出。

---

今天笔者也更新了 Xcode 8，有一些感受和经验分享一下：

1. 选择模拟器的时候已经看不到 iPhone 4s，这是因为它无法升级至 iOS 10，已经默认被苹果抛弃了。所以如果要调试旧机型，记得手动添加进去。

2. 如果你发现了自己注释快捷键失效了，可以试试关掉 Xcode，重启电脑。

3. 运行程序时会发现控制台打印一大堆东西，简直逼死强迫症，要去除它们，只要进行如下操作：打开 `Product → Scheme → Edit Scheme`，并在弹出的窗口中选择 `Run → Arguments`，往 `Environment Variables` 中添加一条 `OS_ACTIVITY_MODE`, `value` 设置为 `disable`。