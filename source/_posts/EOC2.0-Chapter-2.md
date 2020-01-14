---
title: 《Effective Objective-C 2.0》整理（二）：对象、消息、运行时
date: 2016-09-19 13:20:00
tags: 《Effective Objective-C 2.0》
categories: iOS
---

### 第6条：理解“属性”这一概念
实例变量一般通过“存取方法”来访问。

* 获取方法（getter）：读取变量值
* 设置方法（setter）：写入变量值

<!--more-->

属性能够访问封装在对象里的数据，意味着编译器会自动写出一套存取方法。

```objective-c
@property NSString *firstName; // Same as:
- (NSString *)firstName;
- (void)setFirstName:(NSString *)firstName;
```

也可以用“点语法”访问属性。
```objective-c
aPerson.firstName = @"Bob"; // Same as:
[aPerson setFirstName:@"Bob"];

NSString *lastName = aPerson.lastName; // Same as:
NSString *lastName = [aPerson lastName];
```

下面区分一下3种声明类型：

声明类型|描述
-|-
@property | 在**头文件**中声明 getter 和 setter 方法
@synthesize | 在**实现文件**中生成相应的 getter 和 setter 方法
@dynamic |  告诉编译器开发者会自己实现 getter 和 setter 方法。若未实现，编译通过但程序运行时会崩溃

属性各种特质设定会影响编译器所生成的存取方法，介绍以下特质：

**原子性**
* atomic：原子性，不声明即默认。存取过程中线程安全，系统会自动的创建 lock 锁，锁定变量。
* nonatomic：非原子性的。线程不安全，性能更好。开发时应使用 nonatomic。

**读/写权限**
* readwrite： 属性拥有 getter 和 setter，若该属性由 @synthesize 实现，则编译器会自动生成这两个方法。
* readonly：一种“拥有关系”，设置新值时，设置方法会保留新值，并释放旧值，再将新值设置上去。

**内存管理语义**
* assign： “设置方法”只针对“纯量类型”（CGFloat、NSInteger等）的简单赋值操作。不进行任何 retain 操作。
* strong：一种“拥有关系”，设置新值时，设置方法会保留新值，并释放旧值，再将新值设置上去。
* weak： 一种“非拥有关系”，设置新值时，既不保留新值，也不释放旧值。在属性所指的对象遭到摧毁时，属性值也会清空。
* unsafe_unretained：语义和 assign 相同，适用于“对象类型”。非拥有（“不保留”，unretained），当属性所指的对象遭到摧毁时，属性值不会自动清空（“不安全”，unsafe）。
* copy：所属关系与 strong 类似，但设置方法并不保留新值，而是将其 copy。

问题：为什么NSString *要用copy 修饰？
答案：因为传递给setter的新值有可能指向一个 NSMutableString 类的实例，它是 NSString 的子类，表示一种可以修改其值的字符串，此时若是不拷贝字符串，那么设置完属性后，字符串的值就可能会在对象不知情的情况下遭人更改。

**方法名**
* getter=<name>指定“获取方法”的方法名。
```objective-c
// UISwitch类中表示“开关”是否打开的属性如下定义：
@property (nonatomic, getter=isOn) BOOL on;
```
* setter=<name>指定“设置方法”的方法名，用法不常见。

通过上述特质，可以微调由编译器所合成的存取方法。但若是自己实现存取方法，应该保证其具备相关属性所声明的特质。

### 第7条：在对象内部尽量直接访问实例变量

在对象之外访问实例变量时，总是通过属性来做，但在对象内部访问实例变量一直存在争议。

笔者建议在读取实例变量时采用直接访问的形式，设置实例变量的时候通过属性来做。

举例：
```objective-c
@interface EOCPerson : NSObject
@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
// Convenience for firstName + " " + lastName;
- (NSString *)fullName;
- (void)setFullName:(NSString *)fullName;
@end
```

`fullName` 和 `setFullName` 可以这样实现：
```objective-c
// 使用点语法
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@",self.firstName,self.lastName];
}
- (void)setFullName:(NSString *)fullName {
    NSArray *components = [fullName componentsSeparatedByString:@" "];
    self.firstName = [components objectAtIndex:0];
    self.lastName = [components objectAtIndex:1];
}
// 直接访问实例变量
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@",_firstName,_lastName];
}
- (void)setFullName:(NSString *)fullName {
    NSArray *components = [fullName componentsSeparatedByString:@" "];
    _firstName = [components objectAtIndex:0];
    _lastName = [components objectAtIndex:1];
}
```
这两种写法有以下区别：
1. 由于不经过“方法派发”（详见第11条），直接访问实力变量的速度比较快。在这种情况下，编译器所生成的代码会直接访问保存对象实例变量的那块内存。
2. 直接访问实例变量时，不会调用 setter 方法，那就绕过了第6条所提及的“内存管理语义”，比如：在 ARC 下直接访问一个声明为 copy 的属性，不会拷贝属性，只会保留新值并释放旧值。
3. 直接访问实例变量，不会触发 KVO 通知。
4. 通过属性来访问有助于排查与之相关的错误，因为可以给 getter/setter 方法新增断点，监控该属性的调用者及其访问时机。

由此衍生一种折中方案：**写入实例变量时，通过其“设置方法”来做，读取时直接访问之**。此方法既能提高读取操作的速度，又能控制对属性的写入操作。

注意：如果使用懒加载，必须通过存取方法来访问属性，否则实例变量永远不会初始化。

### 第8条：理解“对象等同性”这一概念

NSObject 协议中有两个用于判断等同性的关键方法：

```objective-c
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```
NSObject 类对这两个方法的默认实现是：当且仅当其“指针值”完全相等时，这两个对象才相等。若想在自定义的对象中正确覆写这些方法，就必须先理解其约定。

如果 “isEqual:” 方法判定两个对象相等，那么其 hash 方法也必须返回同一个值。但是，如果两个对象的 hash 方法返回同一个值，那么 “isEqual:” 方法未必会认为两者相等。

比如下面这个类：
```objective-c
@interface EOCPerson : NSObject
@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, assign) NSUInteger age;
@end
 
// 我们认为，如果两个 EOCPerson 的所有字段都相等，那么两个对象就相等。

- (BOOL)isEqual:(id)object {
    if (self == object) return YES;
    if ([self class] != [object class]) return NO;
    
    EOCPerson *otherPerson = (EOCPerson *)object;
    if (![_firstName isEqualToString:otherPerson.firstName])
        return NO;
    if (![_lastName isEqualToString:otherPerson.lastName])
        return NO;
    if (_age != otherPerson.age)
        return NO;
    return YES;
}

- (NSUInteger)hash {
    NSUInteger firstNameHash = [_firstName hash];
    NSUInteger lastNameHash = [_lastName hash];
    NSUInteger ageHash = _age;
    return firstNameHash ^ lastNameHash ^ ageHash;
}
```
isEqual 检测规则：只要其中有不相等的属性，就判定两对象不等，否则两对象相等。

collection 在检索哈希表时，会把对象的哈希码做索引。在写 hash 方法时，需要考虑性能以及减小创建字符串的开销，在减少碰撞频度与降低运算复杂程度之间做出取舍。

**特定类所具有的等同性判定方法**

由于 Objective-C 在编译器不做**强类型**检查，这样容易不小心传入类型错误的对象，因此做判定时应确保所传对象的类型正确性。

以 EOCPerson 类为例：

```objective-c
- (BOOL)isEqualToPerson:(EOCPerson *)otherPerson {
    if (self == otherPerson) return YES;
    
    if (![_firstName isEqualToString:otherPerson.firstName])
        return NO;
    if (![_lastName isEqualToString:otherPerson.lastName])
        return NO;
    if (_age != otherPerson.age)
        return NO;
    return YES;
}

- (BOOL)isEqual:(id)object {
    // 如果受测参数与接受该消息的对象都属于同一个类，那么调用自己编写的判定方法，否则交由超类来判断。
    if ([self class] != [object class]) {
        return [self isEqualToPerson:(EOCPerson *)object];
    } else {
        return [super isEqual:object];
    }
}
```
**等同性判定的执行深度**

不要盲目地逐个检测每条属性，而是应该依照具体需求来制定检测方案。

**容器中可变类的等同性**

如果把某对象放入set之后又修改其内容，可能会出现容器中有相同对象的情况，要注意其隐患的发生。

### 第9条：以“类族模式”隐藏实现细节

“类族”是一种可以隐藏“抽象基类”背后实现细节的模式，在 Objective-C 系统框架中普遍使用。

**创建类族**

举例创建一个处理雇员的类族：

```objective-c
typedef NS_ENUM (NSUInteger, EOCEmployeeType) {
    EOCEmployeeTypeDeveloper,
    EOCEmployeeTypeDesigner,
    EOCEmployeeTypeFinance,
};

@interface EOCEmployee : NSObject

@property (copy) NSString *name;
@property NSInteger salary;

// 创建雇员对象
+ (EOCEmployee*)employeeWithType:(EOCEmployeeType)type;

// 雇员的日常工作
- (void)doADaysWork;

@end

@implementation EOCEmployee

+ (EOCEmployee*)employeeWithType:(EOCEmployeeType)type {
    switch (type) {
        case EOCEmployeeTypeDeveloper:
            return [EOCEmployeeDeveloper new];
            break;
        case EOCEmployeeTypeDesigner:
            return [EOCEmployeeDesigner new];
            break;
        case EOCEmployeeTypeFinance:
            return [EOCEmployeeFinance new];
            break;
    }
}

- (void)doADaysWork {
    // 供子类实现
}

@end

// 每个“实体子类”都从基类继承而来
@interface EOCEmployeeDeveloper:EOCEmployee
@end

@implementation EOCEmployeeDeveloper

- (void)doADaysWork {
    [self writeCode];
}

@end
```

本例中，基类实现了一个“类方法”，该方法根据待创建的雇员类别分配好对应的雇员类实例。这种“工厂模式”是创建类族方法之一。

**Cocoa 里的类族**

系统框架中有许多类族，大部分 collection（集合）类都是类族。例如 NSArray 与其可变版本 NSMutableArray，由此可见实际上有两个抽象基类，一个用于不可变数组，另一个用于可变数组。

抽象基类：为了给子类继承实现具体的功能，它是”残缺的类“，里面没有抽象方法的具体代码，里面的抽象方法是被子类重写的。

在 Employee 这个例子中，若是没有“工厂方法”的源代码，就无法向其中新增雇员类别。然而对于 Cocoa 中 NSArray 这样的类族来说，还是有办法新增子类的， 但需要遵守几条规则：

- 子类应该继承自类族的抽象基类。
若要编写 NSArray 类族的子类，则需令其继承自不可变的数组和基类或可变数组的基类。

- 子类应该定义自己的数据存储方式。
NSArray 本身只不过是包在其他隐藏对象外卖的壳，它仅仅定义了所有数组都需具备的一些接口。对于这个自定义的数组子类来说，可以用 NSArray 来保存其实例。

- 子类应当覆写超类文档中指明需要覆写的方法。
在每个抽象基类中，都有一些子类必须覆写的方法。比如说，想要编写 NSArray 的子类，就需要实现 count 及 “objectAtIndex:” 方法。像 lastObject 这种方法则无须实现，因为基类可以根据前两个方法推演它。

### 第10条：在既有类中使用关联对象存放自定义数据

要在对象中存放相关信息，我们通常会从对象所属的类中继承一个子类，再改写子类对象。有时候类的实例可能是由某种机制所创建的，这就引入了一个强大的特性——“关联对象”。

可以给某对象关联许多其他对象，这些对象通过“键”来区分。存储对象值的时候，可以指明“存储策略”，用以维护相应的“内存管理语义”。存储策略由名为 objc_AssociationPolicy 的枚举所定义，下表列出该枚举的取值和与之等效的 @property 属性。

关联类型|等效的 @property 属性
:-|:-
OBJC_ASSOCIATION_ASSIGN|assign
OBJC_ASSOCIATION_RETAIN_NONATOMIC|nonatomic, retain
OBJC_ASSOCIATION_COPY_NONATOMIC|nonatomic, copy
OBJC_ASSOCIATION_RETAIN|retain
OBJC_ASSOCIATION_COPY|copy

下面的方法可以管理关联对象：

- void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
此方法以给定的键和策略为某对象设置关联对象值。

- id objc_getAssociatedObject(id object, const void *key)
此方法根据给定的键从某对象中获取相应的关联对象值。

- void objc_removeAssociatedObjects(id object)
此方法移除指定对象的全部关联对象。

在设置关联对象值时，通常使用静态全局变量做键。

“关联对象”缺点：常会引入难以查找的bug。


### 第11条：理解 objc_msgSend 的作用

在对象上调用方法又叫“传递消息”，消息有“名称”（name）或“选择子”（selector），可以接受参数，而且可能还有返回值。传递消息会使用**动态绑定**机制来决定需要调用的方法。

给对象发送消息可以这样来写：
```objective-c
id returnValue = [someObject messageName:parameter];
```

在本例中，someObject叫做“接受者”（receiver），messageName 叫做“选择子”（selector）。选择子和参数合起来称为“消息”（message）。编译器看到消息后，将其转换为一条标准的C语言函数调用，也是消息传递机制中的核心函数，叫做 objc_msgSend，其“原型”如下：
```objective-c
void objc_msgSend(id self, SEL cmd, ...)
```

这是个“参数个数可变的函数”，能接受两个及以上的参数。第一个参数代表接受者，第二个参数代表选择子（SEL 是选择子的类型），后续参数就是消息中的参数，其顺序不变。编译器会把刚才那个例子中的消息转换为如下函数：
```objective-c
id returnValue = objc_msgSend(someObject,
                              @selector(messageName:),
                              parameter);
```

objc_msgSend 函数会依据接受者与选择子的类型来调用适当的方法。为了完成此操作，该方法需要在接受者所属的类中搜寻其“方法列表”，如果能找到与选择子名称相符的方法，就跳至其实现代码。若是找不到，就沿着继承体系继续向上查找，找到合适的方法再跳转。如果最终还是找不到相符的方法，就执行“消息转发”操作。

还有一些特殊情况的函数：

- objc_msgSend_stret：待发送的消息要返回结构体

- objc_msgSend_fpret：消息返回的是浮点数
- objc_msgSendSuper：要给超类发信息，例如 `[super message:parameter];`

### 第12条：理解消息转发机制

编译器无法确定某类型对象到底能解读多少种选择子，因为运行期还可向其中动态新增。

当对象收到无法解读的消息后，就会启动“消息转发”机制。消息转发流程：

1. 通过运行期的动态方法解析功能，我们可以在需要用到某个方法时再将其加入类中。
2. 对象可以把其无法解读的某些选择子转交给其他对象来处理。
3. 经过上述两步之后，如果还是没有办法处理选择子，那就启动完整的消息转发机制。

**动态方法解析**

对象在收到无法解读的消息后，首先将调用其所属类的这个类方法：

```objective-c
// 表示这个类是否能新增一个实例方法用以处理选择子
+ (BOOL)resolveInstanceMethod:(SEL)selector
// 表示这个类是否能新增一个类方法用以处理选择子
+ (BOOL)resolveClassMethod:(SEL)selector
```

使用此方法的前提：相关方法的实现代码已经写好，只等着运行的时候动态插在类里面。此方案常用来实现@dynamic属性。

**备援接受者**

当前接受者还有第二次处理未知选择子的机会。这一步中，运行期系统会问：能否将这条消息转给其他接受者处理，对应方法：

```objective-c
- (id)forwardingTargetForSelector:(SEL)selector
```

若当前接受者能找到备援对象，则将其返回，若找不到，就返回nil。

注意：我们无法操作经由这一步所转发的消息，若想在发送给备援接受者之前先修改消息内容，得通过完整的消息转发机制。

**完整的消息转发**

首先创建 NSInvocation 对象，把与尚未处理的那条消息有关的全部细节（包括选择子、目标及参数）都封于其中。在触发 NSInvocation 对象时，“消息派发系统”将亲自出马，把消息指派给目标对象。此步骤会调用：

```objective-c
- (void)forwardInvocation:(NSInvocation)
```

实现此方法时，若发现某调用操作不应由本类处理，则需调用超类的同名方法，直至 NSObject。如果调用了 NSObject 类，那么该方法还会继而调用“doesNotRecognizeSelector:”以抛出异常，此异常表明选择子最终未能得到处理。

**消息转发全流程**

![消息转发](message-forwarding.png)

### 第13条：用“方法调配技术”调试“黑盒方法”

在运行期，可以向类中新增或替换选择子所对应的方法实现。

使用另一份实现来替换原有的方法实现，这道工序叫“方法调配”，开发者常用此技术向原有实现中添加新功能。

类的方法列表会把选择子的名称映射到相关的方法实现之上，使得“动态消息派发系统”能够根据此找到应该调用的方法。这些方法均以函数指针的形式来表示，这种指针叫做IMP，其原型如下：

```objective-c
id (*IMP)(id, SEL,...)
```

以互换NSString大小写的两个方法为例：

```objective-c
Method originalMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
Method swappedMethod = class_getInstanceMethod([NSString class], @selector(uppercaseString));
method_exchangeImplementations(originalMethod, swappedMethod)
```

一般来说，只有调试程序的时候才需要在运行期修改方法实现，这种做法不宜滥用，否则会令代码变得不易读懂且难以维护。

### 第14条：理解"类对象"的用意

对象类型并非在编译器就绑定好了，而是在运行期查找。有个特殊的类型叫做 id，它能指代任意的 Objective-C 对象类型。

“在运行期检视对象类型”这一操作也叫做“类型信息查询”（“内省”），这个强大而有用的特性内置于 Foundation 框架的 NSObject 协议里，凡事由公共根类集成而来的对象都要遵从此协议。

Objective-C 对象的本质是什么？

每个 Objective-C 对象实例都是指向某块内存数据的指针。

```objective-c
NSString *pointerVariable = @"Some string";
```

对于通用的对象类型 id，由于其本身已经是指针了，所以可以这样写：

```objective-c
id genericTypeString = @"Some string";
```

假设有个名为 SomeClass 的子类从 NSObject 中继承而来，则其继承体系如下图所示：

![SomeClass 继承体系](class-hierarchy-for-instances-of-someclass.png)

super_class 指针确立了继承关系，而 isa 指针描述实例所属的类。

如果对象类型无法在编译器确定，那么就应该使用类型信息查询方法来探知。“isMemberOfClass:”能够判断出对象是否为某个特定类的实例，而“isKindOfClass:” 则能够判断出对象是否为某类或其派生类的实例，例如：

```objective-c
NSMutableDictionary *dict = [NSMutableDictionary new];
[dict isMemberOfClass:[NSDictionary class]]; /// <NO
[dict isMemberOfClass:[NSMutableDictionary class]]; /// <YES
[dict isKindOfClass:[NSDictionary class]]; /// <YES
[dict isKindOfClass:[NSArray class]]; /// <NO
```

尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能。

---

参考资料：[《Effective Objective-C 2.0》编写高质量iOS与OS X代码的52个有效方法](https://book.douban.com/subject/25829244/)