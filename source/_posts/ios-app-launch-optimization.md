---
title: 如何实现 iOS App 的冷启动优化
date: 2020-02-29 12:19:22
tags: 
    - 冷启动
    - Instruments
    - MachOView
categories: iOS
thumbnail: app-launch.png
---

当 App 中的业务模块越来越多、越来越复杂，集成了更多的三方库，App 启动也会越来越慢，因此我们希望能在业务扩张的同时，保持较优的启动速度，给用户带来良好的使用体验。

<!--more-->

## 热启动与冷启动

当用户按下 home 键，iOS App 不会立刻被 kill，而是存活一段时间，这段时间里用户再打开 App，App 基本上不需要做什么，就能还原到退到后台前的状态。我们把 App 进程还在系统中，无需开启新进程的启动过程称为**热启动**。

而**冷启动**则是指 App 不在系统进程中，比如设备重启后，或是手动杀死 App 进程，又或是 App 长时间未打开过，用户再点击启动 App 的过程，这时需要创建一个新进程分配给 App。我们可以将冷启动看作一次完整的 App 启动过程，本文讨论的就是冷启动的优化。

## 冷启动概要

WWDC 2016 中首次出现了 App 启动优化的话题，其中提到：

- App 启动最佳速度是 400ms 以内，因为从点击 App 图标启动，然后 Launch Screen 出现再消失的时间就是 400ms；
- App 启动最慢不得大于 20s，否则进程会被系统杀死；（启动时间最好以 App 所支持的最低配置设备为准。）

冷启动的整个过程是指从用户唤起 App 开始到 AppDelegate 中的 `didFinishLaunchingWithOptions` 方法执行完毕为止，并以执行 `main()` 函数的时机为分界点，分为 `pre-main` 和 `main()` 两个阶段。

也有一种说法是将整个冷启动阶段以主 UI 框架的 `viewDidAppear` 函数执行完毕才算结束。这两种说法都可以，前者的界定范围是 App 启动和初始化完毕，后者的界定范围是用户视角的启动完毕，也就是首屏已经被加载出来。

> **注意**：这里很多文章都会把第二个阶段描述为 **main 函数之后**，个人认为这种说法不是很好，容易让人误解。要知道 main 函数在 App 运行过程中是不会退出的，无论是 AppDelegate 中的 `didFinishLaunchingWithOptions` 方法还是 ViewController 中的`viewDidAppear` 方法，都还是在 main 函数内部执行的。


## pre-main 阶段

`pre-main` 阶段指的是从用户唤起 App 到 `main()` 函数执行之前的过程。

### 查看阶段耗时

我们可以在 Xcode 中配置环境变量 `DYLD_PRINT_STATISTICS` 为 1（`Edit Scheme → Run → Arguments → Environment Variables → +`）。

![设置环境变量](set-environment-variables.png)

这时在 iOS 10 以上系统中运行一个 TestDemo，`pre-main` 阶段的启动时间会在控制台中打印出来。

```bash
Total pre-main time: 354.21 milliseconds (100.0%)
         dylib loading time:  25.52 milliseconds (7.2%)
        rebase/binding time:  12.70 milliseconds (3.5%)
            ObjC setup time: 152.74 milliseconds (43.1%)
           initializer time: 163.24 milliseconds (46.0%)
           slowest intializers :
             libSystem.B.dylib :   7.98 milliseconds (2.2%)
   libBacktraceRecording.dylib :  13.53 milliseconds (3.8%)
    libMainThreadChecker.dylib :  41.11 milliseconds (11.6%)
                      TestDemo :  88.76 milliseconds (25.0%)

```

如果要更详细的信息，就设置 `DYLD_PRINT_STATISTICS_DETAILS` 为 1。

```bash
  total time: 1.6 seconds (100.0%)
  total images loaded:  388 (381 from dyld shared cache)
  total segments mapped: 23, into 413 pages
  total images loading time: 805.78 milliseconds (48.6%)
  total load time in ObjC: 152.74 milliseconds (9.2%)
  total debugger pause time: 780.26 milliseconds (47.1%)
  total dtrace DOF registration time:   0.00 milliseconds (0.0%)
  total rebase fixups:  54,265
  total rebase fixups time:  20.77 milliseconds (1.2%)
  total binding fixups: 527,211
  total binding fixups time: 513.54 milliseconds (31.0%)
  total weak binding fixups time:   0.31 milliseconds (0.0%)
  total redo shared cached bindings time: 521.93 milliseconds (31.5%)
  total bindings lazily fixed up: 0 of 0
  total time in initializers and ObjC +load: 163.24 milliseconds (9.8%)
                         libSystem.B.dylib :   7.98 milliseconds (0.4%)
               libBacktraceRecording.dylib :  13.53 milliseconds (0.8%)
                libMainThreadChecker.dylib :  41.11 milliseconds (2.4%)
              libViewDebuggerSupport.dylib :   6.68 milliseconds (0.4%)
                                  TestDemo :  88.76 milliseconds (5.3%)
total symbol trie searches:    1306942
total symbol table binary searches:    0
total images defining weak symbols:  41
total images using weak symbols:  105
```

这里统计到的启动耗时出现一定波动是正常的，无须过分在意。

### 理论知识

为了更准确地了解 App 启动的流程，我们先熟悉一下几个概念。

#### Mach-O

[Mach-O](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/MachOOverview.html)（Mach Object File Format)是一种用于记录可执行文件、对象代码、共享库、动态加载代码和内存转储的文件格式。App 编译生成的二进制**可执行文件**就是 Mach-O 格式的，iOS 工程所有的类编译后会生成对应的目标文件 `.o` 文件，而这个可执行文件就是这些 `.o` 文件的集合。

在 Xcode 的控制台输入以下命令，可以打印出运行时所有加载进应用程序的 Mach-O 文件。

```bash
image list -o -f
```

Mach-O 文件主要由三部分组成：
  - Mach header：描述 Mach-O 的 CPU 架构、文件类型以及加载命令等；
  - Load commands：描述了文件中数据的具体组织结构，不同的数据类型使用不同的加载命令；
  - Data：Data 中的每个段（segment）的数据都保存在这里，每个段都有一个或多个 Section，它们存放了具体的数据与代码，主要包含这三种类型：
    - `__TEXT` 包含 Mach header，被执行的代码和只读常量（如 C 字符串）。只读可执行（r-x）。
    - `__DATA` 包含全局变量，静态变量等。可读写（rw-）。
    - `__LINKEDIT` 包含了加载程序的**元数据**，比如函数的名称和地址。只读（r–-）。

#### dylib

dylib 也是一种 Mach-O 格式的文件，后缀名为 `.dylib` 的文件就是动态库（也叫动态链接库）。动态库是运行时加载的，可以被多个 App 的进程共用。

如果想知道 TestDemo 中依赖的所有动态库，可以通过下面的指令实现：

```bash
otool -L /TestDemo.app/TestDemo
```

动态链接库分为**系统 dylib** 和**内嵌 dylib**（embed dylib，即开发者手动引入的动态库）。系统 dylib 有：
  - iOS 中用到的所有系统 framework，比如 UIKit、Foundation；
  - 系统级别的 libSystem（如 libdispatch(GCD)、libsystem_c(C 语言库)、libsystem_blocks(Block)、libCommonCrypto(加密库，比如常用的 md5)）；
  - 加载 OC runtime 方法的 libobjc；
  - ……

#### dyld

[dyld](https://opensource.apple.com/tarballs/dyld/)（Dynamic Link Editor）：动态链接器，其本质也是 Mach-O 文件，一个专门用来加载 dylib 文件的库。 dyld 位于 `/usr/lib/dyld`，可以在 mac 和越狱机中找到。dyld 会将 App 依赖的动态库和 App 文件加载到内存后执行。

#### dyld shared cache

dyld shared cache 就是动态库共享缓存。当需要加载的动态库非常多时，相互依赖的符号也更多了，为了节省解析处理符号的时间，OS X 和 iOS 上的动态链接器使用了共享缓存。OS X 的共享缓存位于 `/private/var/db/dyld/`，iOS 的则在 `/System/Library/Caches/com.apple.dyld/`。

当加载一个 Mach-O 文件时，dyld 首先会检查是否存在于共享缓存，存在就直接取出使用。每一个进程都会把这个共享缓存映射到了自己的地址空间中。这种方法大大优化了 OS X 和 iOS 上程序的启动时间。

#### images

images 在这里不是指图片，而是**镜像**。每个 App 都是以 images 为单位进行加载的。images 类型包括：
  - executable：应用的二进制可执行文件；
  - dylib：动态链接库；
  - bundle：资源文件，属于不能被链接的 dylib，只能在运行时通过 `dlopen()` 加载。

#### framework

framework 可以是动态库，也是静态库，是一个包含 dylib、bundle 和头文件的文件夹。

### 启动过程分析与优化

启动一个应用时，系统会通过 `fork()` 方法来新创建一个进程，然后执行镜像通过 `exec()` 来替换为另一个可执行程序，然后执行如下操作：

1. 把可执行文件加载到内存空间，从可执行文件中能够分析出 dyld 的路径；
2. 把 dyld 加载到内存；
3. dyld 从可执行文件的依赖开始，递归加载所有的依赖动态链接库 dylib 并进行相应的初始化操作。

结合上面 `pre-main` 打印的结果，我们可以大致了解整个启动过程如下图所示：

![](pre-main-launch-step.png)

#### Load Dylibs

这一步，指的是**动态库加载**。在此阶段，dyld 会：

1. 分析 App 依赖的所有 dylib；
2. 找到 dylib 对应的 Mach-O 文件；
3. 打开、读取这些 Mach-O 文件，并验证其有效性；
4. 在系统内核中注册代码签名；
5. 对 dylib 的每一个 segment 调用 `mmap()`。

一般情况下，iOS App 需要加载 100-400 个 dylibs。这些动态库包括系统的，也包括开发者手动引入的。其中大部分 dylib 都是系统库，系统已经做了优化，因此开发者更应关心自己手动集成的内嵌 dylib，加载它们时性能开销较大。

App 中依赖的 dylib 越少越好，Apple 官方建议尽量将内嵌 dylib 的个数维持在 6 个以内。

**优化方案**：

- 尽量不使用内嵌 dylib；
- 合并已有内嵌 dylib；
- 检查 framework 的 `optional` 和 `required` 设置，如果 framework 在当前的 App 支持的 iOS 系统版本中都存在，就设为 `required`，因为设为 `optional` 会有额外的检查导致加载变慢；
- 使用静态库作为代替；（不过静态库会在编译期被打进可执行文件，造成可执行文件体积增大，两者各有利弊，开发者自行权衡。）
- 懒加载 dylib。（但使用 `dlopen()` 对性能会产生影响，因为 App 启动时是原本是单线程运行，系统会取消加锁，但 `dlopen()` 开启了多线程，系统不得不加锁，这样不仅会使性能降低，可能还会造成死锁及未知的后果，不是很推荐这种做法。）

#### Rebase/Binding

这一步，做的是**指针重定位**。

在 dylib 的加载过程中，系统为了安全考虑，引入了 ASLR（Address Space Layout Randomization）技术和代码签名。由于 ASLR 的存在，镜像会在新的随机地址（actual_address）上加载，和之前指针指向的地址（preferred_address）会有一个偏差（slide，slide=actual_address-preferred_address），因此 dyld 需要修正这个偏差，指向正确的地址。具体通过这两步实现：

第一步：**Rebase**，在 image 内部调整指针的指向。将 image 读入内存，并以 page 为单位进行加密验证，保证不会被篡改，性能消耗主要在 IO。

第二步：**Binding**，符号绑定。将指针指向 image 外部的内容。查询符号表，设置指向镜像外部的指针，性能消耗主要在 CPU 计算。

通过以下命令可以查看 rebase 和 bind 等信息：

```shell
xcrun dyldinfo -rebase -bind -lazy_bind TestDemo.app/TestDemo
```

通过 LC_DYLD_INFO_ONLY 可以查看各种信息的偏移量和大小。如果想要更方便直观地查看，推荐使用 [MachOView](https://github.com/fiteen/fiteen.github.io/releases/tag/v0.1.2) 工具。

指针数量越少，指针修复的耗时也就越少。所以，优化该阶段的关键就是减少 `__DATA` 段中的指针数量。

**优化方案**：

- 减少 ObjC 类（class）、方法（selector）、分类（category）的数量，比如合并一些功能，删除无效的类、方法和分类等（可以借助 [AppCode](/resources) 的 Inspect Code 功能进行代码瘦身）；
- 减少 C++ 虚函数；（虚函数会创建 vtable，这也会在 `__DATA` 段中创建结构。）
- 多用 Swift Structs。（因为 Swift Structs 是静态分发的，它的结构内部做了优化，符号数量更少。）

#### ObjC Setup

完成 Rebase 和 Bind 之后，通知 runtime 去做一些代码运行时需要做的事情：

- dyld 会注册所有声明过的 ObjC 类；
- 将分类插入到类的方法列表中；
- 检查每个 selector 的唯一性。

**优化方案**：

Rebase/Binding 阶段优化好了，这一步的耗时也会相应减少。

#### Initializers

Rebase 和 Binding 属于静态调整（fix-up），修改的是 `__DATA` 段中的内容，而这里则开始动态调整，往堆和栈中写入内容。具体工作有：

- 调用每个 Objc 类和分类中的 `+load` 方法；
- 调用 C/C++ 中的构造器函数（用 `attribute((constructor))` 修饰的函数）；
- 创建非基本类型的 C++ 静态全局变量。

**优化方案**：

- 尽量避免在类的 `+load` 方法中初始化，可以推迟到 `+initiailize` 中进行；（因为在一个 `+load` 方法中进行运行时方法替换操作会带来 4ms 的消耗）
- 避免使用 `__atribute__((constructor))` 将方法显式标记为初始化器，而是让初始化方法调用时再执行。比如用 `dispatch_once()`、`pthread_once()` 或 `std::once()`，相当于在第一次使用时才初始化，推迟了一部分工作耗时。：
- 减少非基本类型的 C++ 静态全局变量的个数。（因为这类全局变量通常是类或者结构体，如果在构造函数中有繁重的工作，就会拖慢启动速度）

总结一下 `pre-main` 阶段可行的优化方案：

- 重新梳理架构，减少不必要的内置动态库数量；
- 进行代码瘦身，合并或删除无效的 ObjC 类、Category、方法、C++ 静态全局变量等；
- 将不必须在 `+load` 方法中执行的任务延迟到 `+initialize` 中；
- 减少 C++ 虚函数。

## main() 阶段

对于 `main()` 阶段，主要测量的就是从 `main()` 函数开始执行到 `didFinishLaunchingWithOptions` 方法执行结束的耗时。

### 查看阶段耗时

这里介绍两种查看 `main()` 阶段耗时的方法。

**方法一：手动插入代码，进行耗时计算。**

```objc
// 第一步：在 main() 函数里用变量 MainStartTime 记录当前时间
CFAbsoluteTime MainStartTime;
int main(int argc, char * argv[]) {
    MainStartTime = CFAbsoluteTimeGetCurrent();
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

// 第二步：在 AppDelegate.m 文件中用 extern 声明全局变量 MainStartTime
extern CFAbsoluteTime MainStartTime;

// 第三步：在 didFinishLaunchingWithOptions 方法结束前，再获取一下当前时间，与 MainStartTime 的差值就是 main() 函数阶段的耗时
double mainLaunchTime = (CFAbsoluteTimeGetCurrent() - MainStartTime);
NSLog(@"main() 阶段耗时：%.2fms", mainLaunchTime * 1000);
```

**方法二：借助 Instruments 的 Time Profiler 工具查看耗时。**

打开方式为：`Xcode → Open Developer Tool → Instruments → Time Profiler`。

![Time Profiler](time-profiler.png)

操作步骤：

1. 配置 Scheme。点击 `Edit Scheme` 找到 `Profile` 下的 `Build Configuration`，设置为 `Debug`。

2. 配置 PROJECT。点击 PROJECT，在 `Build Settings` 中找到 `Build Options` 选项里的 `Debug Information Format`，把 `Debug` 对应的值改为 `DWARF with dSYM File`。

3. 启动 Time Profiler，点击左上角红色圆形按钮开始检测，然后就可以看到执行代码的完整路径和对应的耗时。

为了方面查看应用程序中实际代码的执行耗时和代码路径实际所在的位置，可以勾选上 `Call Tree` 中的 `Separate Thread` 和 `Hide System Libraries`。

![](time-profiler-tree.png)

### 启动优化

`main()` 被调用之后，`didFinishLaunchingWithOptions` 阶段，App 会进行必要的初始化操作，而 `viewDidAppear` 执行结束之前则是做了首页内容的加载和显示。

关于 **App 的初始化**，除了统计、日志这种须要在 App 一启动就配置的事件，有一些配置也可以考虑延迟加载。如果你在 `didFinishLaunchingWithOptions` 中同时也涉及到了**首屏的加载**，那么可以考虑从这些角度优化：

- 用纯代码的方式，而不是 xib/Storyboard，来加载首页视图
- 延迟暂时不需要的二方/三方库加载；
- 延迟执行部分业务逻辑和 UI 配置；
- 延迟加载/懒加载部分视图；
- 避免首屏加载时大量的本地/网络数据读取；
- 在 release 包中移除 NSLog 打印；
- 在视觉可接受的范围内，压缩页面中的图片大小；
- ……

如果首屏为 H5 页面，针对它的优化，参考 [VasSonic](https://github.com/fiteen/fiteen.github.io/releases/tag/v0.1.5) 的原理，可以从这几个角度入手：

- 终端耗时
  - webView 预加载：在 App 启动时期预先加载了一次 webView，通过创建空的 webView，预先启动 Web 线程，完成一些全局性的初始化工作，对二次创建 webView 能有数百毫秒的提升。

- 页面耗时（静态页面）
  - 静态直出：服务端拉取数据后通过 Node.js 进行渲染，生成包含首屏数据的 HTML 文件，发布到 CDN 上，webView 直接从 CDN 上获取；
  - 离线预推：使用离线包。

- 页面耗时（经常需要动态更新的页面）
  - 并行加载：WebView 的打开和资源的请求并行；
  - 动态缓存：动态页面缓存在客户端，用户下次打开的时候先打开缓存页面，然后再刷新；
  - 动静分离：将页面分为静态模板和动态数据，根据不同的启动场景进行不同的刷新方案；
  - 预加载：提前拉取需要的增量更新数据。

## 小结

随着业务的增长，App 中的模块越来越多，冷启动的时间也必不可少地增加。冷启动本就是一个比较复杂的流程，它的优化没有固定的公式，我们需要结合业务，配合一些性能分析工具和线上监控日志，有耐心、多维度地进行分析和解决。

---

参考链接：

[WWDC2016: Optimizing App Startup Time](https://developer.apple.com/videos/play/wwdc2016/406/)
[WWDC2017: App Startup Time: Past, Present, and Future](https://developer.apple.com/videos/play/wwdc2017/413/)
[优化 App 的启动时间](http://yulingtianxia.com/blog/2016/10/30/Optimizing-App-Startup-Time/)
[今日头条 iOS 客户端启动速度优化](https://juejin.im/entry/5b6061bef265da0f574dfd21)
[VasSonic 源码](https://github.com/Tencent/VasSonic)
