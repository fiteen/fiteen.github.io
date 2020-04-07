---
title: 一个 iOS 开发者的两年大前端经历小结
date: 2020-3-27 19:30:49
tags: Flutter
categories: 
   - iOS
   - Android
   - 跨端
   - 前端
---

iOS 开发其实是我的本职工作，入行前两年的工作都是以原生应用开发为主，但是由于后来公司的业务方向都是轻原生重 H5，加上这两年行业大前端的概念非常火爆，使得很多原生应用开发者的日子并不那么好过，需要不断学习，才不会被淘汰。

相信每一位 iOS 开发的求职者，投递简历时，都会看到这样的信息：

- 熟悉 Android / 前端开发优先；
- 有 hybrid 应用开发、前端开发经验优先；
- 有跨端（RN、Weex、Flutter）应用的开发经验优先。

这两年，我也逐渐涉猎到了 Android、Web、小程序、Flutter 的开发。下面会根据我个人的大前端开发经历，做一个简单的回顾和小结，如果有大牛看到文章中错误观点，可以在评论区鞭策我。

## 客户端开发

iOS 和 Android 在客户端开发领域这几年一直处于分庭抗礼的阶段，尽管 Android 设备的市场占有率相较于 iOS 来说算是遥遥领先，加上这几年国内以华为、OPPO 为首的 Android 系统厂商的进步之大也是非常明显。但不可否认的是，苹果依然有一大批坚挺的拥护者，苹果对自家语言的持续升级和优化也是在稳固进行，因此，Android 始终无法一统江湖。

### iOS 与 Android

很多入门的移动端开发者，可能也会犹豫如何选择自己的研究学习方向，那么我就先从大方向上来谈一谈我视角中这两个平台的异同。

|    异同点    |                             iOS                              |                           Android                            |
| :----------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   核心语言   |            Objective-C（部分开源）、Swift（开源）            |                     Java、Kotlin（开源）                     |
|   开发公司   |                            Apple                             |             Google（Java）、JetBrains（Kotlin）              |
|   手机厂商   |                      Apple（唯一厂商）                       |               华为、OPPO、三星、小米、vivo 等                |
|    安全性    |                             较好                             |                             较差                             |
|   视觉风格   |                         Apple Design                         |                       Material Design                        |
|   开发工具   |                           Xcode 等                           |                      Android Studio 等                       |
|    安装包    |                           ipa 文件                           |                           apk 文件                           |
|   下载途径   | 1. 官方途径（正式版：AppStore、内测版：TestFlight）；<br>2. 第三方分发平台（通过信任企业证书） | 1. 各大应用商店（应用宝、豌豆荚、华为应用市场等）；<br>2. apk 直接安装 |
| 最新正式版本 |                            iOS 13                            |                   Android Q（Android 10）                    |

从开发的角度来看：

|    异同点    |                             iOS                              |                           Android                            |
| :----------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   页面布局   |               1. 纯代码；2. xib；3. Storyboard               |                      1. 纯代码；2. xml                       |
| 消息处理机制 |                           RunLoop                            |                            Looper                            |
| 垃圾回收机制 |                  ARC（~~MRC（过时方案）~~）                  |                           GC 机制                            |
|    多继承    |             不支持，但可以用分类和协议代替多继承             |                             支持                             |
| 数据存储方式 | 1. NSUserDefaults；2. 归档和解档；3. plist；4. Sqlite3；5. Core Data |  1. Shared Preferences；2. 内部存储与外部存储；《3. Sqlite3  |
|     推送     | [APNs](https://developer.apple.com/documentation/usernotifications/registering_your_app_with_apns?language=objc) |                GCM（国外）、自行实现（国内）                 |
|    多线程    |   1. NSThread；2. GCD；3. NSOperation 和 NSOperationQueue    | 1. Runnable/Thread；2. ExecutorServie；3. IntentService；4. AsyncTask；5. Service |
|              |                                                              |                                                              |
|              |                                                              |                                                              |
|              |                                                              |                                                              |
|              |                                                              |                                                              |
|              |                                                              |                                                              |

### Objective-C 与 Java

由于目前个人对 Swift 和 Kotlin 不甚熟悉，就不误人子弟，只总结一下在编写 Objective-C 与 Java 代码时感受到的一些差别。

|  差异点  |   Objective-C    |    Java     |
| :------: | :--------------: | :---------: |
|   语法   |     相对复杂       |    简洁    |
|  顶层类  | NSObject/NSProxy |   Object    |
| 类结构 | 一个 .h 头文件和一个 .m 实现文件 | 一个 java 文件 |
| 创建对象 |    alloc init    |     new     |
| 打印对象 |    description    | toString() |
| 方法重载 | 不支持 | 支持 |
|  |  |  |
|  |  |  |

## Web 的几种热门框架

### React

### Vue

### Koa2

## 小程序

### 微信小程序

### 支付宝小程序

### 字节跳动小程序

## 跨平台方案

跨平台一直是这几年比较热门的话题。

### React Native、Weex、Flutter 的比较

近来，沉淀数年的豪门 React Native 接连被 Airbnb、Udacity 放弃，阿里系的大量产品也在去 Weex 化，而 Flutter 在闲鱼团队的努力推广下强势崛起，成为跨端界的香饽饽。


|  差异点  |   React Native    |    Weex     | Flutter |
| :------: | :--------------: | :---------: | :------: |
|   核心语言   |     React       |    Vue    | Dart |
| 引擎 | JSCore | JS V8 | Flutter engine |
| 发布时间 | 2015 | 2016 | 2017 |
| 维护者 | Facebook | 阿里巴巴出品，目前托管 Apache | Google |
| 平台实现 | JavaScript | JavaScript | 无桥接，原生编码 |
| 上手难度 | 较难 | 容易 | 一般 |
| 支持系统 | iOS、Android | iOS、Android、Web | iOS、Android、Web |
| 框架程度 | 较重 | 较轻 | 重 |
| 社区 | 活跃 | emmm、 | 热门、备受瞩目和吹捧 |
| 使用代表 | Facebook、QQ | 天猫、优酷 | 闲鱼、美团 B 端 |
| 热更新 | 支持 | 支持 | 支持 |
| 前景 |  |  |  |

### Flutter 优势

（这段文字后面重新整理一下。）

Flutter 和 RN/Weex 的差异，核心在于渲染的基础由自己实现，简单来说：

- Flutter 的代码经过 Flutter 引擎直接就渲染到了屏幕上；
- 而 RN/Weex 的代码需要先跑到 Native 层处理一下，然后经过 Native 层渲染到屏幕。

很显然前者效率会更高。由于 Native 组件可能会随着系统的升级跟着一起升级（API 增、删或变化），RN/Weex 需要写很多胶水层代码来适配不同版本、不同平台的 Native 组件，而 Flutter 就不存在这个问题，但 Flutter 却不能像 RN/Weex 那般可以直接使用 Native 提供的丰富组件和属性，它需要使用 Flutter 引擎暴露出来的底层 API 做封装，

- 要具备 Flex 布局能力，就需要写一个 Flex 引擎来识别上层的 Flex 语法
- 想使用 React 的 DSL，上层就必须实现一个类 React 框架来对接 Flutter 引擎提供的渲染 API
- 想使用圆角、投影等等，就必须增加一种渲染策略来实现圆角效果和阴影效果等等

好在 Flutter 社区针对 Android 和 iOS 分别实现了一套适合各自系统风格的组件，长得跟 Native 一样。如果这些组件不能满足开发者的需求，开发者也可以很轻松地定义一种新的组件，这对开发者显然是十分友好的，我们可以拿到非常底层的 API 做各种想实现的效果，而且性能还特别高。

Flutter 引擎之上有一层是 Dart，事实上它就提供了上面我们所说的 Flex 布局能力、类 React 的 DSL 能力、各种动画、CSS rule 等，其实现方式就利用 Flutter 引擎提供的比较底层的可以直接在 GPU 上渲染的 API 能力。