---
title: 《Effective Objective-C 2.0》整理（四）：协议与分类
date: 2016-09-28 20:21:06
tags: 《Effective Objective-C 2.0》
categories: iOS
---

## 第23条：通过委托与数据源协议进行对象间通信
Objective-C 开发中广泛使用“委托模式”来实现对象间的通信，该模式的主旨是：定义一套接口，某对象若想接受另一个对象的委托，则需遵从此接口，以便成为其“委托对象”（delegate）。而“另一个对象”则可以给其委托对象回传一些信息，也可以在发生相关事件时通知委托对象。

<!--more-->

此模式可将数据和业务逻辑解耦。例如，用户界面有个显示一系列数据所用的视图，视图对象的属性中，可以包含负责数据和事件处理的对象。这两种对象分别称为“数据源”（data source）与“委托”（delegate）。

委托（代理模式）（Delegate）：委托别人办事，自己不处理，交给别人处理；

协议（Protocol）：使用了这个协议就要按照协议办事

下面总结一下委托模式的实现，委托方：

```objc
// .h 文件中
// 定义协议
@protocol ClassADelegate
// 协议中不标注，默认为 @required 类型，必须实现
- (void)methodA;
@optional
// 特别标注了 @optional 类型，表示可以不实现
- (void)methodB;
@end

@interface ClassA : NSObject
// 引用，存代理方
@property (nonatomic, weak) id <ClassADelegate> delegate;
@end

// .m 中在合适的时机给代理方发消息
@implementation ClassB
- (void)rightTimeMethod {
    // 实现 @requiered 方法 methodA
    [self.delegate methodA];
    // 实现 @optional 方法 methodB
    if ([self.delegate respondsToSelector:@selector(methodB)]) {
        [self.delegate methodB];
    }
}
@end
```

在这里需要注意的是：**delegate 属性需要定义成 weak，而非 strong。因为两者之间必须为“非拥有关系”，否则会造成循环引用，从而导致内存泄漏**。

而代理方则需要：

```objc
// 遵守协议
@interface ClassB() <ClassADelegate>
@end

@implementation ClassB
// 实现方法
#pragma mark - ClassADelegate
- (void)methodA {
    // how to implementation methodA
}
// 将自己设置为代理方
- (void)rightTimeMethod {
	ClassA classA = [ClassA new];
	classA.delegate = self;
}
@end
```



## 第24条：将类的实现代码分散到便于管理的数个分类之中

当一个类中充斥了大量的方法实现时，可以通过分类这种模式将这个庞大的类打散，例如：

- EOCPerson+Friendship(.h/.m)
- EOCPerson+Work(.h/.m)
- EOCPerson+Play(.h/.m)

通过分类机制，可以把类的代码分成多个易于管理的小块，归入不同的“功能区”，以便单独检视，也便于调试。

在编写准备分享给其他开发者使用的程序库时，可以考虑创建 Private 分类，如果程序库中的某个地方要用到这些方法，那就引入此分类的头文件。而分类的头文件并不随程序库一并公开，于是该库的使用者也就不知道库里还有这些私有方法了。



## 第25条：总是为第三方类的分类名称加前缀

如果分类中有何原有类同名的方法，会优先调用分类中的方法，同名方法调用的优先级为**分类 > 本类 > 父类**。如果多个分类中都有和原有类中同名的方法, 那么调用该方法的时候执行谁由编译器决定，编译器会执行最后一个参与编译的分类中的方法。

为了避免分类覆盖，可以通过给类名和方法名都加专属前缀的方式解决。

例如：

```objc
@interface NSString (ABC_HTTP)
// Encode a string with URL encoding
- (NSString *)abc_urlEncodedString;
// Decode a URL encode string
- (NSString *)abc_urlDecodedString;
@end
```



## 第26条：勿在分类中声明属性

除了“class-continuation 分类”之外，其他分类都无法向类中新增实例变量。原因是分类无法合成与属性相关的实例变量。分类中可以写 @property，但不会生成 setter/getter 方法，也不会生成实现以及私有的成员变量，会编译通过，但是引用变量会报错。

简单地说，分类是运行期决议的，在运行期，对象的内存布局已经确定了，如果此时添加实例变量会破坏类的内部结构。

但是如果一定要添加，也是可以通过分类中为该属性实现存取方法来实现。如下：

```objc
#import <objc/runtime.h>

static const char *kFriendsPropertyKey = "kFriendsPropertyKey";

@implementation EOCPerson (Friends)
- (NSArray *)friends {
    return objc_getAssociatedObject(self, kFriendsPropertyKey);
}
- (void)setFriends:(NSArray *)friends {
    objc_setAssociatedObject(self,
                             kFriendsPropertyKey,
                             friends,
                             OBJC_ASSCIATIOM_RETAIN_NONATOMIC);
}
@end
```

但是这种做法并不推荐。分类机制，应该理解为一种手段，目标在于拓展类的功能，而非封装数据。最好的做法，就是将封装数据所用的全部属性都定义在主接口里。

## 第27条：使用“class-continuation 分类”隐藏实现细节

分类的主要作用是为已经存在的类添加方法，因为分类的结构体指针中，没有属性列表，只有方法列表。本章介绍的是一种特殊的分类“class-continuation”，用于定义一些无需对外公布的方法及实例变量。形如：

```objc
#import "ClassA.h"
@interface ClassA ()
// 定义你所需要的私有变量或方法
@end

@implementation ClassA
// 实现
@end
```

若想使类所遵循的协议不为人所知，也可以在“class-continuation 分类”中声明。



## 第28条：通过协议提供匿名对象

协议定义了一系列方法，遵从此协议的对象应该实现它们。于是，我们可以用协议把自己所写的 API 之中的实现细节隐藏起来，将返回的对象设计为遵从此协议的纯 id 类型，这样，想隐藏的类型就不会出现在 API 之中了。例如 ClassA、ClassB 都会遵循某个协议 EOCDelegate，假如指定类型，就会这样约定：

```objc
@property (nonatomic ,weak) ClassA <EOCDelegate> delegate;
@property (nonatomic ,weak) ClassB <EOCDelegate> delegate;
```

如果不想指明具体使用哪个类，就可以将 delegate 对象约定成 纯 id 类型，这个对象也可以称之为“匿名对象”。

```objc
@property (nonatomic ,weak) id <EOCDelegate> delegate;
```

总结一下：

- 协议可以在某种程度上提供匿名对象，具体的对象类型可以淡化成遵从某协议的 id 类型，协议里规定了对象所应实现的方法。
- 使用匿名对象来隐藏类型名称（或类名）。
- 如果具体类型不重要，重要的是对象能否处理好一些特定的方法，那么就可以使用这种协议匿名对象来完成。



---

参考资料：[《Effective Objective-C 2.0》编写高质量iOS与OS X代码的52个有效方法](https://book.douban.com/subject/25829244/)