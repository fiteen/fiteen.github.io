---
title: iOS 内存泄漏场景与解决方案
date: 2020-02-16 14:07:00
tag:
     - 内存管理
     - NSTimer
     - GCD
     - block
     - 委托模式
     - Instruments
categories: iOS
thumbnail: apple.png
---

**内存泄漏**指的是程序中已动态分配的**堆内存**（程序员自己管理的空间）由于某些原因未能释放或无法释放，造成系统内存的浪费，导致程序运行速度变慢甚至系统崩溃。

<!--more-->

在 iOS 开发中会遇到的内存泄漏场景可以分为几类：

## 循环引用

当对象 A 强引用对象 B，而对象 B 又强引用对象 A，或者多个对象互相强引用形成一个闭环，这就是**循环引用**。

### Block

Block 会对其内部的对象强引用，因此使用的时候需要确保不会形成循环引用。

举个例子，看下面这段代码：

```objc
self.block = ^{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@", self.name);
    });
};
self.block();
```

`block` 是 `self` 的属性，因此 `self` 强引用了 `block`，而 `block` 内部又调用了 `self`，因此 `block` 也强引用了 `self`。要解决这个循环引用的问题，有两种思路。

#### 使用 Weak-Strong Dance

先用 `__weak` 将 `self` 置为弱引用，打破“循环”关系，但是 `weakSelf` 在 `block` 中可能被提前释放，因此还需要在 `block` 内部，用 `__strong` 对 `weakSelf` 进行强引用，这样可以确保 `strongSelf` 在 `block` 结束后才会被释放。

```objc
__weak typeof(self) weakSelf = self;
self.block = ^{
    __strong typeof(self) strongSelf = weakSelf;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@", strongSelf.name);
    });
};
self.block();
```

#### 断开持有关系

使用 `__block` 关键字设置一个指针 `vc` 指向 `self`，重新形成一个 `self → block → vc → self` 的循环持有链。在调用结束后，将 `vc` 置为 `nil`，就能断开循环持有链，从而令 `self` 正常释放。

```objc
__block UIViewController *vc = self;
self.block = ^{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@", vc.name);
        vc = nil;
    });
};
self.block();
```

这里还要补充一个问题，为什么要用 `__block` 修饰 `vc`？

首先，`block` 本身不允许修改外部变量的值。但被 `__block` 修饰的变量会被存在了一个栈的结构体当中，成为结构体指针。当这个对象被 `block` 持有，就将“外部变量”在栈中的内存地址放到堆中，进而可以在 `block` 内部修改外部变量的值。

总结一下，没有 `__block` 就是**值传递**，有 `__block` 就是**指针传递**。

还有一种方式可以断开持有关系。就是将 `self` 以传参的形式传入 `block` 内部，这样 `self` 就不会被 `block` 持用，也就不会形成循环持有链。

```objc
self.block = ^(UIViewController *vc){
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@", vc.name);
    });
};
self.block(self);
```

### NSTimer

我们知道 `NSTimer` 对象是采用 `target-action` 方式创建的，通常 `target` 就是类本身，而我们为了方便又常把 `NSTimer` 声明为属性，像这样：

```objc
// 第一种创建方式，timer 默认添加进 runloop
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:self selector:@selector(timeFire) userInfo:nil repeats:YES];
// 第二种创建方式，需要手动将 timer 添加进 runloop
self.timer = [NSTimer timerWithTimeInterval:1.0f target:self selector:@selector(timeFire) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```

这就形成了 `self → timer → self(target)` 的循环持有链。只要 `self` 不释放，`dealloc` 就不会执行，`timer` 就无法在 `dealloc` 中销毁，`self` 始终被强引用，永远得不到释放，循环矛盾，最终造成内存泄漏。

那么如果只把 `timer` 作为局部变量，而不是属性呢？

```objc
NSTimer *timer = [NSTimer timerWithTimeInterval:1.0f target:self selector:@selector(timeFire) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

`self` 同样释放不了。

因为在加入 runloop 的操作中，`timer` 被强引用，这就形成了一条 `runloop → timer → self(target)` 的持有链。而 `timer` 作为局部变量，无法执行 `invalidate`，所以在 `timer` 被销毁之前，`self` 也不会被释放。

所以只要申请了 `timer`，加入了 runloop，并且 `target` 是 `self`，就算不是循环引用，也会造成内存泄漏，因为 `self` 没有释放的时机。

解决这个问题有好几种方式，开发者可以自行选择。

#### 在合适的时机销毁 NSTimer

当 `NSTimer` 初始化之后，加入 runloop 会导致被当前的页面强引用，因此不会执行 `dealloc`。所以需要在合适的时机销毁 `_timer`，断开 `_timer`、runloop 和当前页面之间的强引用关系。

```objc
[_timer invalidate];
_timer = nil;
```

`ViewController` 中的时机可以选择 `didMoveToParentViewController`、`viewDidDisappear`，`View` 中可以选择 `removeFromSuperview` 等，但这种方案并一定是正确可行的。

比如在注册页面中加了一个倒计时，如果在 `viewDidDisappear` 中销毁了 `_timer`，当用户点击跳转到用户协议页面时，倒计时就会被提前销毁，这是不合逻辑的。因此需要结合具体业务的需求场景来考虑。

#### 使用 GCD 的定时器

GCD 不基于 runloop，可以用 GCD 的计时器代替 NSTimer 实现计时任务。但需要注意的是，GCD 内部 block 中的循环引用问题还是需要解决的。

```objc
__weak typeof(self) weakSelf = self;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
dispatch_source_set_timer(_timer, DISPATCH_TIME_NOW, 1.0 * NSEC_PER_SEC, 0);
dispatch_source_set_event_handler(_timer, ^{
    [weakSelf timeFire];
});
// 开启计时器
dispatch_resume(_timer);
// 销毁计时器
// dispatch_source_cancel(_timer);
```

#### 借助中介者销毁

**中介者**指的是用别的对象代替 `target` 里的 `self`，中介者绑定 `selector` 之后，再在 `dealloc` 中释放 `timer`。

这里介绍两种中介者，一种是 NSObject 对象，一种是 NSProxy 的子类。它们的存在是为了断开对 `self` 的强引用，使之可以被释放。

##### 以一个 NSObject 对象作为中介者

新建一个 NSObject 对象 `_target`，为它**动态添加**一个方法，方法的地址指向 `self` 方法列表中的 `timeFire` 的 IMP。这样 `_target` 与 `self` 之间没有直接的引用关系，又能引用 `self` 里的方法，就不会出现循环引用。

```objc
_target = [NSObject new];
class_addMethod([_target class], @selector(timeFire), class_getMethodImplementation([self class], @selector(timeFire)), "v@:");
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_target selector:@selector(timeFire) userInfo:nil repeats:YES];
```

##### 以 NSProxy 的子类作为中介者

创建一个继承自 `NSProxy` 的子类 `WeakProxy`，将 `timer` 的 `target` 设置为 `WeakProxy` 实例，利用**完整的消息转发机制**实现执行 `self` 中的计时方法，解决循环引用。

{% codeblock WeakProxy.h lang:objc %}
@property (nonatomic, weak, readonly) id weakTarget;

+ (instancetype)proxyWithTarget:(id)target;
- (instancetype)initWithTarget:(id)target;
{% endcodeblock %}

{% codeblock WeakProxy.m lang:objc %}
@implementation WeakProxy

+ (instancetype)proxyWithTarget:(id)target {
    return [[self alloc] initWithTarget:target];
}

- (instancetype)initWithTarget:(id)target {
    _weakTarget = target;
    return self;
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    SEL sel = [invocation selector];
    if ([self.weakTarget respondsToSelector:sel]) {
        [invocation invokeWithTarget:self.weakTarget];
    }
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    return [self.weakTarget methodSignatureForSelector:sel];
}

- (BOOL)respondsToSelector:(SEL)aSelector {
    return [self.weakTarget respondsToSelector:aSelector];
}
@end
{% endcodeblock %}

然后这样创建 `timer`：

```objc
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:[WeakProxy proxyWithTarget:self] selector:@selector(timeFire) userInfo:nil repeats:YES];
```

这时候的循环持有链是这样的：

![](timer-weak-proxy.png)

由于 `WeakProxy` 与 `self` 之间是弱引用关系，`self` 最终是可以被销毁的。

#### 带 block 的 timer

iOS 10 之后，Apple 提供了一种 block 的方式来解决循环引用的问题。

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));
```

为了兼容 iOS 10 之前的方法，可以写成 NSTimer 分类的形式，将 block 作为 SEL 传入初始化方法中，统一以 block 的形式处理回调。

{% codeblock NSTimer+WeakTimer.m lang:objc %}
#import "NSTimer+WeakTimer.h"
 
@implementation NSTimer (WeakTimer)
 
+ (NSTimer *)ht_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                       repeats:(BOOL)repeats
                                         block:(void(^)(void))block {
    return [self scheduledTimerWithTimeInterval:interval
                                         target:self
                                       selector:@selector(ht_blockInvoke:)
                                       userInfo:[block copy]
                                        repeats:repeats];
}
 
+ (void)ht_blockInvoke:(NSTimer *)timer {
    void (^block)(void) = timer.userInfo;
    if(block) {
        block();
    }
}

@end
{% endcodeblock %}

然后在需要的类中创建 `timer`。

```objc
__weak typeof(self) weakSelf = self;
self.timer = [NSTimer ht_scheduledTimerWithTimeInterval:1.0f repeats:YES block:^{
    [weakSelf timeFire];
}];
```

### 委托模式

委托模式，是对象之间通信的一种设计模式。该模式的主旨是：定义一套接口，某对象若想接受另一个对象的委托，则需遵从此接口，以便成为其“委托对象”。

#### UITableView 的 delegate

我们常用的 `tableView` 与 `ViewController` 就是**委托方**和**代理方**的关系。

需要在控制器中加入列表时，通常我们会将 `tableView` 设为 `ViewController` 中 `view` 的子视图，`UIViewController` 的源码是这样定义 `view` 的：

{% codeblock UIViewController.h lang:objc %}
@property(null_resettable, nonatomic, strong) UIView *view;
{% endcodeblock %}

因此 `ViewController` 强引用了 `tableView`。而 `tableView` 又要委托 `ViewController` 帮它实现几个代理方法和数据源方法。如果此时 `dataSource` 和 `delegate` 属性用 `strong` 来修饰，就会出现 `UITableView` 与 `ViewController` 互相强引用，形成**循环引用**。

那么看一下 `UITableView` 的实现源码，我们会发现其中定义 `dataSource` 和 `delegate` 属性时是用 `weak` 修饰的。

{% codeblock UITableView.h lang:objc %}
@property (nonatomic, weak, nullable) id <UITableViewDataSource> dataSource;
@property (nonatomic, weak, nullable) id <UITableViewDelegate> delegate;
{% endcodeblock %}

所以 `tableView` 的 `dataSource` 和 `delegate` 只是 `weak` 指针，指向了 `ViewController`，它们之间的关系是这样的：

![](tableview-vc-relationship.png)

这也就避免了循环引用的发生。

#### NSURLSession 的 delegate

那么 `delegate` 一定被 `weak` 修饰吗？

也不一定，需要看具体的场景。比如 `NSURLSession` 类中的 `delegate` 就是用 `retain` 修饰的。

{% codeblock NSURLSession.h lang:objc %}
@property (nullable, readonly, retain) id <NSURLSessionDelegate> delegate;
{% endcodeblock %}

它这么做，是因为了确保网络请求回调之前，`delegate` 不被释放。

这也间接引起了 `AFNetworking` 中**循环引用**的出现。我们看 `AFURLSessionManager` 类中声明的 `session` 是 `strong` 类型的。

{% codeblock AFURLSessionManager.h lang:objc %}
/**
 The managed session.
 */
@property (readonly, nonatomic, strong) NSURLSession *session;
{% endcodeblock %}

在构造 `session` 对象时，也将 `delegate` 设为了 `self`，也就是 `AFURLSessionManager` 类。

{% codeblock AFURLSessionManager.m lang:objc %}
- (NSURLSession *)session {
    @synchronized (self) {
        if (!_session) {
            _session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
        }
    }
    return _session;
}
{% endcodeblock %}

如此三者就形成了这样循环持有关系。

![](afn-recycle.png)

要解决这个问题，有两种解决思路：

**方式一：将 `AFHTTPSessionManager` 对象设为单例**

对于客户端来说，大多数情况下都是对应同一个后台服务，所以可以将 `AFHTTPSessionManager` 对象设为单例来处理。

```objc
- (AFHTTPSessionManager *)sharedManager {
    static dispatch_once_t onceToken;
    static AFHTTPSessionManager *_manager = nil;
    dispatch_once(&onceToken, ^{
        _manager = [AFHTTPSessionManager manager];
        _manager.requestSerializer = [AFHTTPRequestSerializer serializer];
        _manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/html",@"text/json", @"text/plain", @"text/javascript",@"text/xml", nil];
        _manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    });
    return _manager;
}
```
如果要设定固定请求头， 以这种 `key-value` 形式加入到 `dispatch_once` 中。

```objc
[_manager.requestSerializer setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];
```

**缺点**：因为请求的 `header` 是由 `AFHTTPSessionManager`的 `requestSerializer.mutableHTTPRequestHeaders` 字典持有的，所以这种单例模式会导致全局共享一个 `header`，如果要处理不同自定义 `header` 的请求就会变得很麻烦。

**方式二：在请求结束时，手动销毁 `session` 对象**

由于 `session` 对象对 `delegate` 强持有，要打破循环引用，需要在请求结束后手动调用 `AFHTTPSessionManager` 对象销毁的方法。

```objc
- (AFHTTPSessionManager *)getSessionManager{
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.requestSerializer = [AFHTTPRequestSerializer serializer];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/html",@"text/json", @"text/plain", @"text/javascript",@"text/xml", nil];
    manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    return manager;
}

- (void)sendRequest{
    AFHTTPSessionManager *manager = [self getSessionManager];
    __weak typeof(manager)weakManager = manager;
    [manager GET:@"https://blog.fiteen.top" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
         __strong typeof (weakManager)strongManager = weakManager;
        NSLog(@"success 回调");
        [strongManager invalidateSessionCancelingTasks:YES];
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        __strong typeof (weakManager)strongManager = weakManager;
        NSLog(@"error 回调");
        [strongManager invalidateSessionCancelingTasks:YES];
    }];
}
```
## 非 OC 对象内存处理

虽然现在已经普及了 ARC 模式，但它仅对 OC 对象进行自动内存管理。对于非 OC 对象，比如 `CoreFoundation` 框架下的 `CI`、`CG`、`CF` 等开头的类的对象，在使用完毕后仍需我们手动释放。

比如这段获取 UUID 的代码：

```objc
CFUUIDRef puuid = CFUUIDCreate( kCFAllocatorDefault );
CFStringRef uuidString = CFUUIDCreateString( kCFAllocatorDefault, puuid );
NSString *uuid = [(NSString *)CFBridgingRelease(CFStringCreateCopy(NULL, uuidString)) uppercaseString];
// 使用完后释放 puuid 和 uuidString 对象
CFRelease(puuid);
CFRelease(uuidString);
```

还有 C 语言中，如果用 `malloc` 动态分配内存后，需要用 `free` 去释放，否则会出现内存泄漏。比如：

```c
person *p = (person *)malloc(sizeof(person));
strcpy(p->name,"fiteen");
p->age = 18;
// 使用完释放内存
free(p);
// 防止野指针
p = NULL;
```

## <span id="peak-mem-usage">循环加载引起内存峰值</span>

先看下面这段代码，看似没有内存泄漏的问题，但是在实际运行时，for 循环内部产生了大量的临时对象，会出现 CPU 暴增。

```objc
for (int i = 0; i < 1000000; i++) {
    NSString *str = @"Abc";
    str = [str lowercaseString];
    str = [str stringByAppendingString:@"xyz"];
    NSLog(@"%@", str);
}
```

这是因为循环内产生大量的临时对象，直至循环结束才释放，可能导致内存泄漏。

**解决方案**：

在循环中创建自己的 `autoreleasepool`，及时释放占用内存大的临时变量，减少内存占用峰值。

```objc
for (int i = 0; i < 100000; i++) {
    @autoreleasepool {
        NSString *str = @"Abc";
        str = [str lowercaseString];
        str = [str stringByAppendingString:@"xyz"];
        NSLog(@"%@", str);
    }
}
```

在没有手加自动释放池的情况下，`autorelease` 对象是在当前的 runloop 迭代结束时释放的，而它能够释放的原因是系统在每个 runloop 迭代中都会先销毁并重新创建自动释放池。

下面举个特殊的例子，使用**容器 block 版本的枚举器**时，内部会自动添加一个自动释放池，比如：

```objc
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    // 这里被一个局部 @autoreleasepool 包围着
}];
```


## <span id="wild-pointer-and-zoombies">野指针与僵尸对象</span>

指针指向的对象已经被释放/回收，这个指针就叫做**野指针**。这个被释放的对象就是**僵尸对象**。

如果用野指针去访问僵尸对象，或者说向野指针发送消息，会发生 `EXC_BAD_ACCESS` 崩溃，出现**内存泄漏**。

```objc
// MRC 下
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Student *stu = [[Student alloc] init];
        [stu setAge:18];
        [stu release];      // stu 在 release 之后，内存空间被释放并回收，stu 变成野指针
        // [stu setAge:20]; // set 再调用 setAge 就会崩溃
    }
    return 0;
}
```

**解决方案**：当对象释放后，应该将其置为 `nil`。


## 内存泄漏检查工具

### Instruments

Instruments 是 Xcode 自带的工具集合，为开发者提供强大的程序性能分析和测试能力。

它打开方式为：`Xcode → Open Developer Tool → Instruments`。其中的 Allocations、Leaks 和 Zombies 功能可以协助我们进行内存泄漏检查。

- Leaks：**动态**检查泄漏的内存，如果检查过程时出现了红色叉叉，就说明存在内存泄漏，可以定位到泄漏的位置，去解决问题。此外，Xcode 中还提供**静态**监测方法 Analyze，可以直接通过 `Product → Analyze` 打开，如果出现泄漏，会出现“蓝色分支图标”提示。

- Allocations：用来检查内存使用/分配情况。比如出现“[循环加载引起内存峰值](#peak-mem-usage)”的情况，就可以通过这个工具检查出来。

- Zombies：检查是否访问了[僵尸对象](#wild-pointer-and-zoombies)。

Instruments 的使用相对来说比较复杂，你也可以通过在工程中引入一些第三方框架进行检测。

### MLeaksFinder

[MLeaksFinder](https://github.com/Tencent/MLeaksFinder) 是 WeRead 团队开源的 iOS 内存泄漏检测工具。

它的使用非常简单，只要在工程引入框架，就可以在 App 运行过程中监测到内存泄漏的对象并立即提醒。MLeaksFinder 也不具备侵入性，使用时无需在 release 版本移除，因为它只会在 debug 版本生效。

不过 MLeaksFinder 的只能定位到内存泄漏的对象，如果你想要检查该对象是否存在循环引用。就结合 FBRetainCycleDetector 一起使用。

### FBRetainCycleDetector

[FBRetainCycleDetector](https://github.com/facebook/FBRetainCycleDetector) 是 Facebook 开源的一个**循环引用**检测工具。它会递归遍历传入内存的 OC 对象的所有强引用的对象，检测以该对象为根结点的强引用树有没有出现循环引用。

