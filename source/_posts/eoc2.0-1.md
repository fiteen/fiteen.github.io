---
title: 《Effective Objective-C 2.0》整理（一）：熟悉Objective-C
date: 2016-08-25 22:55:33
tags: 《Effective Objective-C 2.0》
categories: iOS
---

## 第1条：了解 Objective-C 语言的起源

Objective-C 由消息型语言的鼻祖 [Smalltalk](https://en.wikipedia.org/wiki/Smalltalk) 演化而来，是一门面向对象的语言，具有**封装**、**继承**、**多态**的特性。它还包括三大动态特性：

<!--more-->

* 动态类型：id 类型，静态类型是弱类型，动态类型是强类型
* 动态绑定：基于动态类型，一旦类型被确定，对象对应的属性和相应消息都被确定
* 动态加载：按需加载，例如不同机型适配，加载图片（1x/2x/3x），按需加载可执行代码，而非所有组件


|语言类型| 运行时执行的代码由谁决定 |举例|
|:-------- |:--------:| -----:|
|消息结构|运行环境|Objective-C|
|函数调用|编译器|C++|

因此，Objective-C 使用动态绑定的消息结构，在运行时才会检查对象类型。接受一条消息后，究竟执行何种代码，由运行期环境而非编译器决定。

Objective-C 的重要工作都由“运行期组件”（ runtime component ）完成，它面向对象所需的全部数据结构及函数特性都在运行期组件里。运行器组件本质上就是一种与开发者所编代码相链接的“动态库”，其代码能把开发者编写的所有程序粘连起来。

Objective-C 是 C 的超集，因此 C 的所有功能在Objective-C代码中依然适用。理解 C 中的内存模型（ memory model ）有助于理解 Objective-C 的内存模型和“引用计数”（ reference counting ）机制的工作原理。

Objective-C 语言中的指针是用来指示对象的，对象所占内存总是分配在“堆空间”中，而不会分配在栈上。分配在堆上的内存必须直接管理，而分配在栈上用于保存变量的内存则会在其栈帧弹出时清理。

## 第2条：在类的头文件中尽量少引入其他头文件

在类的头文件中声明其他类有以下两种选择：
``` 
1 - #import "类名.h" // 需要知道该类的全部细节
2 - @class 类名; // 向前声明，不需要知道该类全部细节，能解决了两个类循环引用的问题
```
除非却有必要，否则不要引入头文件。一般，在头文件中向前声明某类，并在实现文件引入某类头文件，这样做可以降低类之间的耦合，以减少编译时间。

以下情况必须在头文件中引入其他头文件：
* 如果类继承自某个父类，必须引入定义那个父类的头文件；
* 声明的类遵循某个协议，该协议必须有完整定义，且不能使用向前声明。因此最好把协议单独放在一个头文件中。

针对一些委托协议，建议在实现文件中引入头文件，在“ class-continue 分类”中遵循协议。

## 第3条：多用字面语法，少用与之等价的方法

字面量语法采用类C的定义方式，可以缩减源代码长度，易读性强。

常规做法：`Number *someNumber = [NSNumber numberWithInt:1];` => 字面量语法：`Number * someNumber = @1;`

** 字面数值 **
```objc
Number *intNumber = @1;
Number *floatNumber = @2.5f;
Number *doubleNumber = @3.14159;
Number *charNumber = @'a';
```
** 字面量数组 **

 ```objc
// 常规做法
NSArray *animals = [NSArray arrayWithObjects:@"cat",@"dog",@"mouse",nil]; // 发现空值nil创建结束
NSString *dog = [animals objectAtIndex:1];

// 字面量语法（更安全，出现nil对象，编译器时就会发现异常）
NSArray *animals = @[@"cat",@"dog",@"mouse"]; // 发现空值nil会抛出异常
NSString *dog = animals[1];
 ```
** 字面量字典 **

 ```objc
// 常规做法
NSDictionary *personData = [NSDictionary dictionaryWithObjectsAndKeys:@"Matt",@"firstName",[NSNumber numberWithInt:28],@"age",nil];
NSString *firstName = [personData objectForKey:@"firstName"];

// 字面量语法
NSDictionary *personData = @{@"firstName" : @"Matt", @"age" : @28};
NSString *firstName = personData[@"firstName"];
 ```
** 可变数组与字典 **

```objc
// 常规做法
[mutableArray replaceObjectAtIndex:1 withObject:@"dog"];
[mutableDictionary setObject:@"Matt" forKey:@"firstName"];

// 字面量语法
mutableArray[1] = @"dog";
mutableDictionary[@"firstName"] = @"Matt";
```

注意：用字面量语法创建数组或字典时，务必确保值中不含nil。

## 第4条：多用类型常量，少用 #define 预处理指令

定义常量时，尽量不要使用 #define 预处理指令，由于没有声明明确的类型信息，会将相同名字的常量值批量替换。取而代之的，采用`static const 类型 常量名 = 常量值`的形式。

派发通知时，需要使用字符串来表示此项通知的名称，而这个名字就可以声明为一个外界可见的常值变量。此类常值变量需放在“全局符号表”中，以便在定义的编译单元之外使用，定义方式如下：
```objc
// In the header file
extern NSString *const EOCStringConstant; // 注意const修饰符在常量类型中的位置

// In the implementation file
NSString *const EOCStringConstant = @"VALUE"; // 解读：一个常量，而这个常量是指针，指向NSString对象
```
extern这个关键字就是告诉编译器无须查看常量的定义，直接允许使用。其他类型的常量也是如此：
```objc
// 在头文件 EOCAnimatedView.h 中使用extern来声明全局常量
// 这种常量要出现在全局符号表中，所以其名称通常以与之相关的类名做前缀
extern const NSTimeInterval EOCAnimationDuration;

// 在实现文件 EOCAnimatedView.m 中定义其值
const NSTimeInterval EOCAnimationDuration = 0.3;
```

## 第5条：用枚举表示状态、选项、状态码

枚举是一种常量命名方式，某个对象所经历的各种状态可定义为一个简单的枚举集。定义方式如下：
```objc
// 方式一：
enum EOCConnectionState {
    // 编译器会为枚举分配一个独有的编号，从0开始，每个枚举递增1
    EOCConnectionStateDisconnected, // 0
    EOCConnectionStateConnecting, // 1
    EOCConnectionStateConnected, // 2
};

enum EOCConnectionState state = EOCConnectionStateConnected;

// 方式二：
enum EOCConnectionState {
    EOCConnectionStateDisconnected = 1, // 1
    EOCConnectionStateConnecting, // 2
    EOCConnectionStateConnected, // 3
};
typedef enum EOCConnectionState EOCConnectionState;

EOCConnectionState state = EOCConnectionStateConnected;
```

如果把传递给某个方法的选项表示为枚举类型，而多个选项又可同时使用，就将各选项值定义为2的幂，以便通过安位或操作将其组合。
```objc
typedef enum EOCPermittedDirection : int EOCPermittedDirection;
enum EOCPermittedDirection : int {
    EOCPermittedDirectionUp    = 1 << 0,
    EOCPermittedDirectionDown  = 1 << 1,
    EOCPermittedDirectionLeft  = 1 << 2,
    EOCPermittedDirectionRight = 1 << 3,
};

EOCPermittedDirection *permittedDirections = EOCPermittedDirectionLeft | EOCPermittedDirectionUp;
```

用 `NS_ENUM`（不需要互相组合） 与 `NS_OPTIONS`（需要安位或组合） 宏来定义枚举类型，并指明其底层数据类型，这样做可以确保枚举是开发者所选的底层数据类型实现出来的。

在处理枚举类型的switch语句中不要实现default分支。因为这样相当于加入了一种新的枚举类型。

---

参考资料：[《Effective Objective-C 2.0》编写高质量iOS与OS X代码的52个有效方法](https://book.douban.com/subject/25829244/)