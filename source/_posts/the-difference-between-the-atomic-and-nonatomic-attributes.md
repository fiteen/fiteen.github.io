---
title: iOS 中 atomic 和 nonatomic 的区别
date: 2017-04-08 22:51:03
tags: 原子性
categories: iOS
---

`nonatomic`（非原子性） 和 `atomic`（原子性） 是 iOS 开发中用 @property 声明属性时，常用的两个关键字。

<!--more-->

看下面三种属性的声明方式：

```objective-c
@property(nonatomic, retain) UITextField *name;
@property(atomic, retain) UITextField *name;
@property(retain) UITextField *name;
```

2、3 的意思是一样的，不写的时候默认声明成 `atomic`。

## 内部实现

如[苹果官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocProperties.html)中描述的那样，它们系统生成的存取方法是不一样的：

`nonatomic` 对象的存取方法实现如下：

```objective-c
- (UITextField *) name {
    return _name;
}

- (void) setName:(UITextField *)name {
    if (_name != name) {
    	[_name release];
    	_name = [name retain];
    }
}
```

而系统为 `atomic` 对象生成的存取方法会进行**加锁**：

```objective-c
- (UITextField *) name {
    UITextField *res = nil;
    @synchronized(self) {
        res = [[_name retain] autorelease];
    }
    return res;
}

- (void) setName:(UITextField *)name {
    @synchronized(self) {
    	if (_name != name) {
      	    [_name release];
      	    _name = [name retain];
    	}
    }
}
```

## 线程安全

`atomic` 可以保证 setter 和 getter 操作不受其它线程影响，因为锁的缘故，能够优先执行完当前操作：

> 线程 A 的 setter **进行到一半**，线程 B 调用了 getter，那么会执行完 setter 再执行 getter，线程 B 还是能得到线程 A setter 后**完好无损**的对象。

那么它能保证整个对象就是线程安全的吗？

答案是并不能，当几个线程同时调用 setter/getter 时，能得到一个完整的值，但这个值无法确定，举个例子：

> 线程 A 调了 getter，**与此同时**线程 B、C 调了 setter，那么 A 最后 getter到的值，可能是
>
> 1. B、C 未 setter 之前的原始值
> 2. B setter 后的值
> 3. C setter 后的值

除了存取之外，线程安全还有其它的操作，比如：

> 线程 A 正在 getter/setter 时，线程 B 同时进行 release，可能会直接 crash。

因此，我们只能认定 `atomic` 是**存取过程**中的线程安全，并不是完全线程安全，别的线程也可以进行存取之外的操作，真正的线程安全需要开发者自己来保证。

而 `nonatomic` 明显就是线程不安全的，如果有两个线程访问同一个属性，会出现无法预料的结果。因此 `nonatomic` 耗费的资源少，速度要比 `atomic` 快，性能也更好。

## 使用

在 iOS 应用中，大多数情况都是用在主线程上，不存在并发的问题，出于性能考虑，更倾向于用 `nonatomic`。

而在 OSX 中，需要考虑多线程通讯，更适合用相对安全的 `atomic` 处理。

## 总结

综上，两者的区别可以总结如下：

|              |       atomic       |    nonatomic     |
| :----------- | :----------------: | :--------------: |
| 是否默认     |         √          |        ×         |
| 内部实现     |   存取过程中加锁   | 存取过程中不加锁 |
| 是否线程安全 | 存取过程中线程安全 |    线程不安全    |
| 性能         |        一般        |        好        |
| 适用于       |      OSX 系统      |     iOS 系统     |

