---
title: 《Effective Objective-C 2.0》整理（三）：接口与 API 设计
date: 2016-09-22 17:55:21
tags: 《Effective Objective-C 2.0》
categories: iOS
---

### 第15条：用前缀避免命名空间冲突

Objective-C 没有其他语言那种内置的命名空间（namespace）机制，我们需要变相实现命名空间。

而 Apple 宣称其保留使用所有“两字母前缀”的权利，因此我们选用的前缀应该是三个字母的，一般选用与公司、应用程序或与二者有关联之名称作为类名的前缀，并在所有代码中均只用这一前缀。

<!--more-->

### 第16条：提供“全能初始化方法”

所有对象均要初始化。我们将可为对象提供必要信息以便其能完成工作的初始化方法叫做“全能初始化方法”。

以某个矩形类为例，它的全能初始化方法为：

```
- (id)initWithWidth:(float)width andHeight:(float)height{
    if ((self = [super init])) {
        _width = width;
        _height = height;
    }
    return self; 
}
```

若全能初始化方法与超类不同，则需覆写超类中的对应方法。如果超类的初始化方法不适用于子类，那么应该覆写这个超类方法，并在其中抛出异常。

举例，某个正方形类作为矩形类的子类，它需要满足 width 和 height 一致的条件，

那么它的初始化方法为：

```
- (id)initWithDimension:(float)dimension{
    return [super initWithWidth:dimension andHeight:dimension];
}
```

然后覆写矩形类的全能初始化：

```
- (id)initWithWidth:(float)width andHeight:(float)height{
    float dimension = MAX(width, height);
    return [self initWithDimension:dimension];
}
```

并抛出异常：

```
- (id)initWithWidth:(float)width andHeight:(float)height{
    @throw [NSException exceptionWithName:NSInternalInconsistencyException reason:@"Must use initWithDimension: instead." userInfo:nil];
}
```

### 第17条：实现 description 方法

调试程序时，经常需要打印并查看对象信息。我们通常会使用下面的方法打印：

```
NSLog(@"object = %@", object);
```

假如 object 是个自定义类，输出的信息形如：

```
object = <CustomClass: 0x7fd9a1600600>
```

想要看到类中完整的信息，需要在类中覆写 description 方法。建议在该方法中打印出类的名字和指针地址。

```
- (NSString*)description{
    return [NSString stringWithFormat:@"<%@:%p,\"%@ %@\">",[self class],self,_parm1,_parm2];
}
```

NSObject 协议中有一个 debugDescription 方法，它是开发者在调试器中以控制台命令打印对象时才调用的。Foundation 框架的 NSArray 类就是实现了 debugDescription。

### 第18条：尽量使用不可变对象

设计类时，应充分运用属性来封装数据。默认情况下，属性是 read-write，这样设计出来的类都是“可变的”。

如果把可变对象放入 collection 之后又修改其内容，那么很容易破坏 set 的内部数据结构，使其失去固有的语义，因此应该尽量减少对象中的可变内容，即定义为 readonly 属性。

不要把可变的 collection 作为属性公开，而应提供相关方法，一次修改对象中的可变collection。例如，某个 EOCPerson 类，假如要改变 friends 数据集，可通过 addFriend 和 removeFriend 实现：

```
#import <Foundation/Foundation.h>

@interface EOCPerson : NSObject

@property(nonatomic,copy,readonly)NSString *firstName;
@property(nonatomic,copy,readonly)NSString *lastName;
@property(nonatomic,strong,readonly)NSSet *friends;

- (id)initWithFirstNmae:(NSString *)firstName
               lastName:(NSString *)lastName;
- (void)addFriend:(EOCPerson*)person;
- (void)removeFriend:(EOCPerson*)person;

@end
```

### 第19条：使用清晰而协调的命名方式

**方法命名规则：**

- 如果方法的返回值是新创建的，那么方法名的首个词应是返回值的类型，除非前面还有修饰语，例如localizedString。属性的存取方法不遵循这种命名方式，因为一般认为这些方法不会创建新对象。
- 应该把表示参数类型的名词放在次参数前面。
- 如果方法要在当前对象上执行操作，那么就应该包含动词；若执行操作时还需要参数，则应该在动词后面加上一个或多个名词。
- 不要使用 str 这种简称，应该用 string 这样的全称。
- 如果某方法返回非属性的Boolean值，那么应该根据其功能，选用 has 或 is 当前缀。
- 将 get 整个前缀留给那些借由“输出参数”来保存返回值的方法，比如说，把返回值填充到“C语言数组”里的那种方法就可以使用这个词做前缀。

**类与协议的命名**

- 给方法起名时的第一要务就是确保其风格与你自己的代码所要继承的框架相符。


- 最重要的一点就是，命名方式要协调一致。如果要从其他框架中集成子类，那么务必遵循其命名惯例。

### 第20条：为私有方法名加前缀

编写类的实现代码时，经常要写一些只在内部使用的方法。为这种方法的名称加上某些前缀，这就可以轻易将公共方法和私有方法区分开，有助于调试。

例如，使用 _p 作为前缀，p 表示 “private”，而下划线可以把这个字母和真正的方法名区隔开：

```
@interface EOCObject : NSObject
- (void)publicMethod;
@end

@implementation EOCObject
- (void)publicMethod {}
- (void)p_privateMethod {}
@end
```

但是，需要注意的是，**不要单用一个下划线做私有方法的前缀**，因为这种做法是预留给苹果公司的。

### 第21条：理解 Objective-C 错误模型

不同于 Java 等编程语言，面对异常处理，Objective-C 现在采用的方法是：只有在极其罕见的情况下抛出异常，异常抛出之后，无须考虑恢复问题，而且应用程序此时也应该退出。

而出现“不那么严重的错误”时，Objective-C 语言所使用的编程范式为：令方法返回 nil/0，或是使用 NSError，以表明其中有错误发生。

NSError 的用法更加灵活，因为经由此对象，我们可以把导致错误的原因反馈给调用者。NSError 对象里封装了三条信息：

- Error domain（错误范围，类型为字符串）
- Error code（错误码，类型为整数）
- User info（用户信息/有关此错误的额外信息，类型为字典）

NSError 经常由“输出参数”返回给调用者，例如：

```
- (BOOL)doSomething:(NSError**)error {
    // Do something that may cause an error
    
    if ( /* there was an error*/ ) {
        if (error) {
            // Pass the 'error' through the out-parameter
            *error = [NSError errorWithDomain:domain code:code userInfo:userInfo];
        }
        return NO; ///< Indicate failure
    } else {
        return YES; ///< Indicate success
    }
}
```

调用者可以根据错误类型分别处理各种错误，错误范围应该定义成 NSString 型的全局常量，而错误码则定义成枚举类型为佳，如：

 ```
// EOCErrors.h
extern NSString *const EocErrorDomain;

typedef NS_ENUM(NSUInteger,EOCError) {
    EOCErrorUnknown              = -1;
    EOCErrorInternalInconsistency= 100;
    EOCErrorGeneralFault         = 105;
    EOCErrorBadInput             = 500;
};
// EOCErrors.m
NSString *const EOCErrorDomain = @"EOCErrorDomain";
 ```

### 第22条：理解 NSCopying 协议

在 Objective-C 中，对象的拷贝通过 copy 完成。如果想要自定义的类支持拷贝，那就要实现 NSCopying 协议，该协议只有一个方法：

```
- (id)copyWithZone:(NSZone *)zone;
```

如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。

```
- (id)mutableCopyWithZone:(NSZone *)zone;
```

复制对象时需决定采用浅拷贝和深拷贝。

**深拷贝：在拷贝对象自身时，将其底层数据也一并复制过去；**

**浅拷贝：只拷贝容器对象本身，而不复制其中数据。**

一般情况下应该尽量执行浅拷贝，如：

```
- (void)copyWithZone:(NSZone *)zone {
    EOCPerson *copy = [[[self class] allocWithZone:zone] initWithFirstNmae:_firstName lastName:_lastName];
    copy->_friends = [_friends mutableCopy];
    return copy;
}
```

如果你写的对象需要深拷贝，那么可以考虑新增一个专门执行深拷贝的方法，如下：。

```
- (void)deepCopy {
    EOCPerson *copy = [[[self class] alloc]initWithFirstNmae:_firstName lastName:_lastName];
    copy->_friends = [[NSMutableSet alloc]initWithSet:_friends copyItems:YES];
    return copy;
}
```


---

参考资料：[《Effective Objective-C 2.0》编写高质量iOS与OS X代码的52个有效方法](https://book.douban.com/subject/25829244/)