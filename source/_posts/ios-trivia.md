---
title: 【持续更新】这些 iOS 冷知识，你知道吗？
date: 2020-03-14 13:02:08
tags: 
     - 内存管理
     - 应用安全
     - block
categories: iOS
thumbnail: trivia.png
---

笔者最近在准备面试时候，回顾了一些过去写的项目和知识点，从底层和原理的角度重新去看代码和问题，发现了几个有意思的地方。

<!--more-->

## 单例对象的内存管理

### 问题背景

在解决 App 防止抓包问题的时候，有一种常见的解决方案就是：**检测是否存在代理服务器**。其实现为：

```objc
+ (BOOL)getProxyStatus {
    CFDictionaryRef dicRef = CFNetworkCopySystemProxySettings();
    const CFStringRef proxyCFstr = CFDictionaryGetValue(dicRef, (const void*)kCFNetworkProxiesHTTPProxy);
    CFRelease(dicRef);
    NSString *proxy = (__bridge NSString*)(proxyCFstr);
    if(proxy) {
        return YES;
    }
    return NO;
}
```

在我前面的一篇文章《[iOS 内存泄漏场景与解决方案](https://blog.fiteen.top/2020/ios-memory-leak)》中，有提到非 OC 对象在使用完毕后，需要我们手动释放。

那么上面这段代码中，在执行 `CFRelease(dicRef);` 之后，`dicRef` 是不是应该就被释放了呢？

### 问题探讨

让我们来写一段测试代码试试看：

```objc
CFDictionaryRef dicRef = CFNetworkCopySystemProxySettings();
NSLog(@"%ld, %p", CFGetRetainCount(dicRef), dicRef);
CFRelease(dicRef);
NSLog(@"%ld, %p", CFGetRetainCount(dicRef), dicRef);
CFRelease(dicRef);
NSLog(@"%ld, %p", CFGetRetainCount(dicRef), dicRef);
```

打印结果为：

```
2, 0x6000004b9720
1, 0x6000004b9720
(lldb) 
```

程序在运行到第三次 `NSLog` 的时候才崩溃，说明对 `dicRef` 对象 release 两次才能将他彻底释放。

这很奇怪，按照以往的经验，第一次打印 `dicRef` 的引用计数值不应该是 1 才对吗？

修改一下代码，继续测试：

```objc
CFDictionaryRef dicRef = CFNetworkCopySystemProxySettings();
CFRelease(dicRef);
CFRelease(dicRef);
NSLog(@"%p", CFNetworkCopySystemProxySettings());
```

这次运行到最后一行代码的时候，居然还是崩溃了。连 `CFNetworkCopySystemProxySettings()` 对象都直接从内存里被销毁了？难道 `dicRef` 没有重新创建对象，而是指向了真正的地址？

为了验证猜想，我们定义两份 `dicRef` 对象，并打印出他们的地址和引用计数。

```objc
CFDictionaryRef dicRef = CFNetworkCopySystemProxySettings();
NSLog(@"%p, %ld,", dicRef, CFGetRetainCount(dicRef));
CFDictionaryRef dicRef1 = CFNetworkCopySystemProxySettings();
NSLog(@"%p, %p, %ld, %ld", dicRef, dicRef1, CFGetRetainCount(dicRef), CFGetRetainCount(dicRef1));
```

打印结果为：

```
0x600003bd2040, 2,
0x600003bd2040, 0x600003bd2040, 3, 3
```

果然如此。`dicRef` 和 `dicRef1` 的地址是一样的，而且第二次打印时，在没有对 `dicRef` 对象执行任何操作的情况下，它的引用计数居然又加了 1。

那么我们可以大胆猜测：

实际上，**每次调用 `CFNetworkCopySystemProxySettings()` 返回的地址一直是同一个，未调用时它的引用计数就为 1，而且每调用一次，引用计数都会加 1**。

如此看来，`CFNetworkCopySystemProxySettings()` 返回的对象在引用计数上的表现和其它**系统单例**十分相似，比如 `[UIApplication sharedApplication]`、`[UIPasteboard generalPasteboard]`、`[NSNotificationCenter defaultCenter]` 等。

单例对象一旦建立，对象指针会保存在静态区，单例对象在堆中分配的内存空间，只在应用程序终止后才会被释放。

![](singleton-memory.png)

因此对于这类单例对象，调用一次就需要释放一次（ARC 下 OC 对象无需手动释放），保持它的引用计数为 1（而不是 0），保证其不被系统回收，下次调用时，依然能正常访问。


## block 属性用什么修饰

### 问题背景

这个问题来源于一道司空见惯的面试题：

> iOS 种 `block` 属性用什么修饰？（`copy` 还是 `strong`？）

Stack Overflow 上也有相关的问题：[Cocoa blocks as strong pointers vs copy](https://stackoverflow.com/questions/27152580/cocoa-blocks-as-strong-pointers-vs-copy)。

### 问题探讨

先来回顾一些概念。

iOS 内存分区为：栈区、堆区、全局区、常量区、代码区（地址从高到低）。常见的 block 有三种：

- NSGlobalBlock：存在全局区的 block；
- NSStackBlock：存在栈区的 block；
- NSMallocBlock：存在堆区的 block。

block 有**自动捕获变量**的特性。当 block 内部没有引入外部变量的时候，不管它用什么类型修饰，block 都会存在全局区，但如果引入了外部变量呢？

这个问题要在 ARC 和 MRC 两种环境下讨论。

> Xcode 中设置 MRC 的开关：
> 1. 全局设置：TARGETS → `Build Settings` → `Apple Clang - Language - Objective-C` → `Objective-C Automatic Reference Counting` 设为 `No`；（ARC 对应的是 `Yes`）
> 2. 局部设置：TARGETS → `Build Phases` → `Compile Sources` → 找到需要设置的文件 → 在对应的 `Compiler Flags` 中设置 `-fno-objc-arc`。（ARC 对应的是 `-fobjc-arc`）

针对这个问题，网上有一种答案：

- MRC 环境下，只能用 `copy` 修饰。使用 `copy` 修饰，会将栈区的 block 拷贝到堆区，但 `strong` 不行；
- ARC 环境下，用 `copy` 和 `strong` 都可以。

看似没什么问题，于是我在 MRC 环境执行了如下代码：

```objc
// 分别用 copy 和 strong 修饰 block 属性
@property (nonatomic, copy) void (^copyBlock)(void);
@property (nonatomic, strong) void (^strongBlock)(void);

int x = 0;
    
// 打印 normalBlock 所在的内存地址
void(^normalBlock)(void) = ^{
    NSLog(@"%d", x);
};
NSLog(@"normalBlock: %@", normalBlock);

// 打印 copyBlock 所在的内存地址
self.copyBlock = ^(void) {
    NSLog(@"%d", x);
};
NSLog(@"copyBlock: %@", self.copyBlock);

// 打印 strongBlock 所在的内存地址
self.strongBlock = ^(void) {
    NSLog(@"%d", x);
};
NSLog(@"strongBlock: %@", self.strongBlock);
```
打印结果为：

```
normalBlock: <__NSStackBlock__: 0x7ffeee29b138>
copyBlock: <__NSMallocBlock__: 0x6000021ac360>
strongBlock: <__NSMallocBlock__: 0x600002198240>
```

从 normalBlock 的位置，我们可以看出，默认是存在栈区的，但是很奇怪的是，为什么 `strongBlock` 位于堆区？难道 MRC 时期用 `strong` 修饰就是可以的？

其实不然，要知道 MRC 时期，只有 `assign`、`retain` 和 `copy` 修饰符，`strong` 和 `weak` 是 ARC 时期才引入的。

`strong` 在 MRC 中对应的是 `retain`，我们来看一下在 MRC 下用这两个属性修饰 block 的区别。

```objc
// MRC 下分别用 copy 和 retain 修饰 block 属性
@property (nonatomic, copy) void (^copyBlock)(void);
@property (nonatomic, retain) void (^retainBlock)(void);

// 打印 copyBlock 所在的内存地址
int x = 0;
self.copyBlock = ^(void) {
    NSLog(@"%d", x);
};
NSLog(@"copyBlock: %@", self.copyBlock);

// 打印 retainBlock 所在的内存地址
self.retainBlock = ^(void) {
    NSLog(@"%d", x);
};
NSLog(@"retainBlock: %@", self.retainBlock);
```

打印结果为：

```obj
copyBlock: <__NSMallocBlock__: 0x6000038f96b0>
retainBlock: <__NSStackBlock__: 0x7ffeed0a90e0>
```

我们可以看到用 `copy` 修饰的 block 存在堆区，而 `retain` 修饰的 block 存在栈区。

那么**修饰符**的作用在哪里，为什么会出现不同的结果，我们通过反汇编来探究一下。

把断点打在 `self.copyBlock` 的声明函数这一行（在上述引用代码的第7行，不是 block 内部）。然后开启 `Debug` → `Debug Workflow` → `Always show Disassembly` 查看汇编代码，点击 Step into。

![](copy-block-callq.png)

在 `callq` 指令中可以看到声明的 copyBlock 属性具有 `copy` 的特性。

然后断点打在 `self.retainBlock` 的声明函数这一行，再进入查看，可以注意到 retainBlock 不具有`copy` 的特性。

![](retain-block-callq.png)

再在 ARC 下试一试。把断点打在 `self.strongBlock` 的声明函数这一行，进入查看，可以发现，用 `strong` 修饰的属性，也具有 `copy` 的特性。

![](strong-block-callq.png)

这也就很好解释了为什么 MRC 下用 `retain` 修饰的属性位于栈区，而用 `copy`、`strong` 修饰的属性存在堆区。

MRC 下，在定义 block 属性时，使用 `copy` 是为了把 block 从栈区拷贝到堆区，因为栈区中的变量出了作用域之后就会被销毁，无法在全局使用，而把栈区的属性拷贝到堆区中全局共享，就不会被销毁了。

ARC 下，不需要使用 `copy` 修饰，因为 ARC 下的 block 属性本来就在堆区。

> 那为什么开发者基本上都只用 `copy` 呢？
>
> 这是 MRC 的历史遗留问题，上面也说到了，`strong` 是 ARC 时期引入的，开发者早已习惯了用 `copy` 来修饰 block 罢了。

最后再补充一个小知识点。

```objc
// ARC 下定义 normalBlock 后再打印其所在的内存地址
void(^normalBlock)(void) = ^{
    NSLog(@"%d", x);
};
NSLog(@"normalBlock: %@", normalBlock);

// 直接打印某个 block 的内存地址
NSLog(@"block: %@", ^{
    NSLog(@"%d", x);
});
```

打印结果为：

```
normalBlock: <__NSMallocBlock__: 0x600001ebe670>
block: <__NSStackBlock__: 0x7ffee8752110>
```

block 的实现是相同的，为什么一个在堆区，一个在栈区？

这个现象叫做**运算符重载**。定义 normalBlock 的时候 `=` 实际上执行了一次 `copy`，为了管理 `normalBlock` 的内存，它被转移到了堆区。

---

暂时先总结到这里，后续如果有新的发现，我也会在此文中继续补充，欢迎订阅、收藏～

