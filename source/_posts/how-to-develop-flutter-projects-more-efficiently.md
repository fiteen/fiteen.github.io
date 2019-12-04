---
title: 如何提升 Flutter 项目的开发效率
date: 2019-09-15 01:43:27
tags: Flutter
categories: 跨端
---

最近参与了一个 Flutter 项目的开发，总结了一些提升开发效率的工具和方法。

### UI 可视化工具

纯客户端开发者一开始可能会对写 Flutter 的界面布局会不太适应，那么这个 [https://flutterstudio.app](https://flutterstudio.app) 网站可以帮助你更快熟悉 Flutter 的常用组件，在这个工具上，你可以通过简单的拖拽直接实现布局。

<!--more-->

{% asset_img flutterstudio.png flutterstudio 网站 %}

### 代码模版

我们发现在开发时，IDE 自带的代码快捷提示都不太丰富，比如要创建一个包含所有生命周期相关方法的完整的 StatefulWidget，如果能一键导入就能快速很多，这时候就可以借助代码模版。我事先在网上找到一份比较全面的模版，有需要的可以参考 [code plugins](https://github.com/AweiLoveAndroid/Flutter-learning/blob/master/code_plugins/no_new_keywords/dart.json)，有时间我会按照自己习惯的风格再整理一份。

如果你使用的是 **VSCode**，打开路径：

> View → Command Palette... → 输入 >Preferences: Configure User Snippets

{% asset_img configure-user-snippets.png VSCode 上配置代码模版 %}

然后输入 `dart`，这时会打开一个 `dart.json` 文件，把上面的内容替换进来即可。

如果你使用的是 **Android Studio**，依次打开路径：

> Preferences → Editor → File and Code Templates

在 `Files` 下找到 `Dart File`,将 json 文件里的内容粘贴进去即可。

{% asset_img file-and-code-templates.png Android Studio 上配置代码模版 %}

这样我们只需要输入简单的前缀就能直接联想出整个代码块了。 

### 布局调试

在实现 UI 模块的时候，经常会出现布局错乱的情况，VSCode 也为此提供了界面调试工具，在 Flutter App **调试**过程中，打开路径：

> View → Command Palette... → 输入 >Flutter: Toggle Debug Painting

{% asset_img toggle-debug-painting.png 布局调试 %}

上面的辅助线可以帮助开发者检查布局。

> 注意：通过 `flutter run` 方式启动的模拟器/真机是没法开启布局调试的。

不过如果遇到难以定位的问题，建议还是使用 Androidio Studio 进行调试，它提供了下面这两个可视化工具：

[**Flutter Inspector**](https://flutterchina.club/inspector/)
  - 理解和查看现有布局
  - 诊断布局的问题

**Flutter Outline**
  - 视图预览
  - 调整 widget

{% asset_img flutter-outline-inspector.png Flutter Outline 和 Flutter Inspector %}

### 巧用快捷键

借助 IDE 中的快捷键也是我们提高开发效率的关键之一。以 Android Studio 为例：

- option+enter：对 widget 进行特定的操作

{% asset_img option+enter.png 快速修改布局 %}

- command+option+L：格式化代码，同时，也建议你在方法尾部尽量加逗号，这有助于自动格式化程序为 Flutter 样式代码插入适当的换行符

- control+option+O：一键清除多余的 imports

- ......

### 常用插件

常用的插件基本上都可以在[Dart 开源包的网站](https://pub.dartlang.org)里找到，选用合适的 package 可以帮你节省不少重复实现的时间。网上的 Coder 朋友们也总结了很多不错的插件，本文里就不一一记录了。

