---
title: iOS runtime 机制解读（结合 objc4 源码）
date: 2020-02-10 15:11:08
tags: runtime
categories: iOS
thumbnail: runtime.png
---

Runtime 是指将数据类型的确定由**编译时**推迟到了**运行时**。它是一套底层的纯 C 语言 API，我们平时编写的 Objective-C 代码，最终都会转换成 runtime 的 C 语言代码。

<!--more-->

不过，runtime API 的实现是用 C++ 开发的（源码中的实现文件都是 `.mm` 文件）。

为了更全面地理解 runtime 机制，我们结合最新的[objc4 源码](https://opensource.apple.com/source/objc4/)来进行解读。

## 消息传递

我们知道 Objective-C 是面向对象开发的，而 C 语言则是面向过程开发，这就需要**将面向对象的类转变成面向过程的结构体**。

在 Objective-C 中，所有的消息传递中的“消息”都会被编译器转化为：

```
id objc_msgSend ( id self, SEL op, ... );
```

比如执行一个对象的方法：`[obj foo];`，底层运行时会被编译器转化为：`objc_msgSend(obj, @selector(foo));`。

那么方法内部的执行流程究竟是怎么样的呢？我先来了解一些概念。

### 概念

#### objc_object

Objective-C 对象是由 `id` 类型表示的，它本质上是一个指向 `objc_object` 结构体的指针。

{% codeblock objc-private.h lang:c %}
typedef struct objc_object *id;

union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        ISA_BITFIELD;  // defined in isa.h
    };
#endif
};

struct objc_object {
private:
    isa_t isa;
// public & private method...
}
{% endcodeblock %}

我们看到 `objc_object` 的结构体中只有一个对象，就是指向其类的 `isa` 指针。

当向一个对象发送消息时，runtime 会根据实例对象的 `isa` 指针找到其所属的类。

#### objc_class

Objective-C 的类是由 `Class` 类型来表示的，它实际上是一个指向 `objc_class` 结构体的指针。

{% codeblock objc.h lang:c %}
typedef struct objc_class *Class;
{% endcodeblock %}

`objc_class` 结构体中定义了很多变量：

{% codeblock objc-runtime-new.h lang:c %}
struct objc_class : objc_object {
    // 指向类的指针(位于 objc_object)
    // Class ISA;
    // 指向父类的指针
    Class superclass;
    // 用于缓存指针和 vtable，加速方法的调用
    cache_t cache;             // formerly cache pointer and vtable
    // 存储类的方法、属性、遵循的协议等信息的地方
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
    // class_data_bits_t 结构体的方法，用于返回class_rw_t 指针（）
    class_rw_t *data() { 
        return bits.data();
    }
    // other methods...
}

struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;
    
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
    
    Class firstSubclass;
    Class nextSiblingClass;
    
    char *demangledName;

#if SUPPORT_INDEXED_ISA
    uint32_t index;
#endif
    // other methods
}
{% endcodeblock %}

`objc_class` 继承自 `objc_object`，因此它也拥有了 `isa` 指针。除此之外，它的结构体中还保存了指向父类的指针、缓存、实例变量列表、方法列表、遵守的协议等。

#### 元类

元类（metaclass）是类对象的类，它的结构体和 `objc_class` 是一样的。

由于所有的类自身也是一个对象，我们可以向这个对象发送消息，比如调用类方法。那么为了调用类方法，这个类的 `isa` 指针必须指向一个包含类方法的一个 `objc_class` 结构体。而类对象中只存储了实例方法，却没有类方法，这就引出了元类的概念，元类中保存了创建类对象以及类方法所需的所有信息。

![](instance-class-meta-isa-chain.png)

为了更方便理解，举个例子：

```
- (void)eat;    // 一个实例方法
+ (void)sleep;  // 一个类方法

// 那么实例方法需要由类对象来调用：
[person eat];
// 而类方法需要由元类来调用：
[Person sleep];
```

假如 `person` 对象也能调用 `sleep` 方法，那我们就无法区分它调用的就究竟是 `+ (void)sleep;` 还是 `- (void)sleep;`。

**类对象是元类的实例，类对象的 `isa` 指针指向了元类。**

这个说法可能有点绕，借助这张经典的图来理解：

![](instance-class-meta-chain.png)

当向对象发消息，runtime 会在这个对象所属类方法列表中查找发送消息对应的方法，但当向类发送消息时，runtime 就会在这个类的 meta class 方法列表里查找。所有的 meta class，包括 Root class，Superclass，Subclass 的 isa 都指向 Root class 的 meta class，这样能够形成一个闭环。

#### Method(method_t)

Method 是一个指向 `method_t` 结构体的指针，我们找到关于它的定义：

{% codeblock objc-private.h lang:c %}
typedef struct method_t *Method;
{% endcodeblock %}

{% codeblock objc-runtime-new.h lang:c %}
struct method_t {
    // 方法选择器
    SEL name;
    // 类型编码
    const char *types;
    // 方法实现的指针
    MethodListIMP imp;
}
{% endcodeblock %}

所以 Method 和 SEL、IMP 的关系就是 Method = SEL + IMP + types。

关于 types 的写法，参考 [Type Encodings](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)。

#### SEL(objc_selector)

SEL 又称**方法选择器**，是一个指向 `objc_selector` 结构体的指针，也是 `objc_msgSend` 函数的第二个参数类型。

{% codeblock objc.h lang:c %}
typedef struct objc_selector *SEL;
{% endcodeblock %}

方法的 `selector` 用于表示运行时方法的名称。代码编译时，会根据方法的名字（不包括参数）生成一个唯一的整型标识（ Int 类型的地址），即 SEL。

**一个类的方法列表中不能存在两个相同的 SEL**，这也是 **Objective-C 不支持重载**的原因。

**不同类之间可以存在相同的 SEL**，因为不同类的实例对象执行相同的 `selector` 时，会在各自的方法列表中去寻找自己对应的 IMP。

**获取 SEL** 的方式有三种：

- `sel_registerName` 函数
- Objective-C 编译器提供的 `@selector()` 方法
- `NSSeletorFromString()` 方法

#### IMP

IMP 本质上就是一个函数指针，**指向方法实现的地址**。

{% codeblock objc.h lang:c %}
typedef void (*IMP)(void /* id, SEL, ... */ ); 
{% endcodeblock %}

参数说明：

- id：指向 self 的指针（如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针）
- SEL：方法选择器
- ...：方法的参数列表

SEL 与 IMP 的关系类似于哈希表中 key 与 value 的关系。采用这种哈希映射的方式可以加快方法的查找速度。

#### cache_t

`cache_t` 表示类缓存，是 object_class 的结构体变量之一。

{% codeblock objc-runtime-new.h lang:c %}
struct cache_t {
    // 存放方法的数组
    struct bucket_t *_buckets;
    // 能存储的最多数量
    mask_t _mask;
    // 当前已存储的方法数量
    mask_t _occupied;
    // ...
}
{% endcodeblock %}

为了加速消息分发，系统会对方法和对应的地址进行缓存，就放在 `cache_t` 中。

实际运行中，大部分常用的方法都是会被缓存起来的，runtime 系统实际上非常快，接近直接执行内存地址的程序速度。

#### category_t

`category_t` 表示一个指向分类的结构体的指针。

{% codeblock objc-runtime-new.h lang:c %}
struct category_t {
    // 是指类名，而不是分类名
    const char *name;
    // 要扩展的类对象，编译期间是不会定义的，而是在运行时阶段通过name对应到相应的类对象
    classref_t cls;
    // 实例方法列表
    struct method_list_t *instanceMethods;
    // 类方法列表
    struct method_list_t *classMethods;
    // 协议列表
    struct protocol_list_t *protocols;
    // 实例属性
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    // 类（元类）属性列表
    struct property_list_t *_classProperties;
    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
{% endcodeblock %}

这里涉及到一个经典问题：

**分类中可以添加实例变量/成员变量/属性吗？**

首先，**分类中无法直接添加实例变量和成员变量**。

实践一下，我们就会发现，在分类中添加实例变量/成员变量，在编译阶段，就会报错，但添加属性是允许的。

![](category-add-instance-variable-error.png)

这是因为**在分类的结构体当中，没有“实例变量/成员变量”的结构，但是有“属性”的结构**。

那么分类中就可以直接添加属性吗？

其实也不然，虽然分类的 `.h` 中没有报错信息，`.m` 中却报出了如下的警告，且运行时会报错。

![](category-add-property-warn.png)

警告提示上表明有两种解决方法：

第一种：用 `@dynamic修饰`。但实际上，`@dynamic` 修饰只是告诉编译器，属性的 setter 和 getter 方法会由用户自行实现。但这样做只能消除警告，无法解决问题，运行时依然会崩溃。

第二种：给分类手动添加 setter 和 getter 方法，这是一种有效的方案。

我们知道 `@property = ivar + setter + getter`。

可以通过 `objc_setAssociatedObject` 和 `objc_getAssociatedObject` **向分类中动态添加属性**，具体实现见下文中的[“关联对象给分类增加属性”](#add-prop-to-category-with-associated-objects)。

### 流程

![消息传递流程](message-send.png)

也就是查找 IMP 的过程：

- 先从当前 class 的 cache 方法列表里去查找。
- 如果找到了，如果找到了就返回对应的 IMP 实现，并把当前的 class 中的 selector 缓存到 cache 里面。
- 如果类的方法列表中找不到，就到父类的方法列表中查找，一直找到 NSObject 类为止。
- 最后再找不到，就会进入动态方法解析和消息转发的机制。

## 消息转发

如果消息传递后仍无法找到 IMP，就进入了**消息转发**流程。

1. 通过运行期的**动态方法解析**功能，我们可以在需要用到某个方法时再将其加入类中。
2. 对象可以把其无法解读的某些选择子转交给**备用接受者**来处理。
3. 经过上述两步之后，如果还是没有办法处理选择子，那就启动**完整的消息转发**机制。

### 动态方法解析

动态方法解析的两个方法：

{% codeblock NSObject.h lang:objc %}
// 添加类方法
+ (BOOL)resolveClassMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);
// 添加实例方法
+ (BOOL)resolveInstanceMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);
{% endcodeblock %}

我们再看看这两个方法在源码中的调用：

{% codeblock objc-class.mm lang:c %}
void _class_resolveMethod(Class cls, SEL sel, id inst)
{
	// 判断是不是元类
    if (! cls->isMetaClass()) {
        // try [cls resolveInstanceMethod:sel]
		// 调用类的 resolveInstanceMethod 方法，动态添加实例方法
        _class_resolveInstanceMethod(cls, sel, inst);
    } 
    else {
        // try [nonMetaClass resolveClassMethod:sel]
        // and [cls resolveInstanceMethod:sel]
		// 调用元类的 resolveClassMethod 方法，动态添加类方法
        _class_resolveClassMethod(cls, sel, inst);
        if (!lookUpImpOrNil(cls, sel, inst, 
                            NO/*initialize*/, YES/*cache*/, NO/*resolver*/)) 
        {
            _class_resolveInstanceMethod(cls, sel, inst);
        }
    }
}
{% endcodeblock %}

下面看一个动态方法解析的例子。

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [self performSelector:@selector(foo)];
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(foo)) {
        class_addMethod([self class], sel, (IMP)fooMethod, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

void fooMethod(id obj, SEL _cmd) {
    NSLog(@"Doing foo");
}
```

可以看到虽然没有实现 `foo` 这个函数，但是我们通过 `class_addMethod` 动态添加 `fooMethod` 函数，并执行 `fooMethod` 这个函数的IMP。

如果 `resolveInstanceMethod:` 方法返回 NO ，运行时就会移到下一步：`forwardingTargetForSelector:`。

### 备用接收者

如果目标对象实现了 `forwardingTargetForSelector:` 方法，runtime 就会调用这个方法，给你把这个消息转发给其他接受者的机会。

实现一个备用接收者的例子如下：

```objc
#import "ViewController.h"
#import <objc/runtime.h>

@interface Person: NSObject

@end

@implementation Person

- (void)foo {
    NSLog(@"Doing foo");//Person的foo函数
}

@end

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self performSelector:@selector(foo)];
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    // 返回 NO，进入下一步转发。
    return NO;
}

- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(foo)) {
        //返回 Person对象，让 Person 对象接收这个消息
        return [Person new];
    }
    return [super forwardingTargetForSelector:aSelector];
}

@end
```

上面的实现就是利用 `forwardingTargetForSelector` 把当前 `ViewController` 类的方法 `foo` 转发给了备用接受者 `Person` 类去执行了。

### 完整的消息转发

如果在上一步还无法处理未知消息，唯一能做的就是启用**完整的消息转发**机制。

主要涉及到两个方法：

- 发送 `methodSignatureForSelector`进行方法签名，这可以将函数的参数类型和返回值封装。如果返回 nil，runtime 会发出 `doesNotRecognizeSelector` 消息，程序同时崩溃。
- 如果返回了一个函数签名，runtime 就会创建一个 `NSInvocation` 对象并发送 `forwardInvocation` 消息给目标对象。

实现一个完整转发的例子如下：

```objc
#import "ViewController.h"
#import <objc/runtime.h>

@interface Person: NSObject

@end

@implementation Person

- (void)foo {
    NSLog(@"Doing foo");
}

@end


@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self performSelector:@selector(foo)];
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    // 返回 NO，进入下一步转发。
    return NO;
}

- (id)forwardingTargetForSelector:(SEL)aSelector {
    // 返回 nil，进入下一步转发。
    return nil;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    if ([NSStringFromSelector(aSelector) isEqualToString:@"foo"]) {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];// 签名，进入 forwardInvocation
    }
    return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    SEL sel = anInvocation.selector;
    Person *p = [Person new];
    if([p respondsToSelector:sel]) {
        [anInvocation invokeWithTarget:p];
    } else {
        [self doesNotRecognizeSelector:sel];
    }
}

@end
```

通过签名，runtime 生成了一个对象 `anInvocation`，发送给方法 `forwardInvocation`，我们在方法中让 `Person` 对象执行 `foo` 函数。

![消息转发流程](message-forwarding.png)

以上就是 runtime 的三次转发流程，下面列举一下 runtime 的实际应用。

## 应用

### <span id="add-prop-to-category-with-associated-objects">关联对象给分类增加属性</span>

关联对象(Associated Objects) 是 Objective-C 运行时的特性，允许开发者向已经存在的类在扩展中添加自定义属性。

关联对象 runtime 提供了3个 API 接口：

```
// 获取关联的对象
id objc_getAssociatedObject(id object, const void *key);
// 设置关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
// 移除关联的对象
void objc_removeAssociatedObjects(id object);
```

参数说明：

- `object`：被关联的对象
- `key`：关联对象的唯一标识
- `value`： 关联的对象
- `policy`：内存管理的策略

关于**内存管理的策略**，源码中这样描述：

{% codeblock runtime.h lang:c %}
/* Associative References */

/**
 * Policies related to associative references.
 * These are options to objc_setAssociatedObject()
 */
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
{% endcodeblock %}

我们看看内存策略对应的属性修饰。

| 内存策略                          | 属性修饰                                            | 描述                                           |
| --------------------------------- | --------------------------------------------------- | ---------------------------------------------- |
| OBJC_ASSOCIATION_ASSIGN           | @property (assign) 或 @property (unsafe_unretained) | 指定一个关联对象的弱引用。                     |
| OBJC_ASSOCIATION_RETAIN_NONATOMIC | @property (nonatomic, strong)                       | 指定一个关联对象的强引用，不能被原子化使用。     |
| OBJC_ASSOCIATION_COPY_NONATOMIC   | @property (nonatomic, copy)                         | 指定一个关联对象的 copy 引用，不能被原子化使用。 |
| OBJC_ASSOCIATION_RETAIN           | @property (atomic, strong)                          | 指定一个关联对象的强引用，能被原子化使用。       |
| OBJC_ASSOCIATION_COPY             | @property (atomic, copy)                            | 指定一个关联对象的 copy 引用，能被原子化使用。   |

下面利用关联对象实现一个“在分类中增加一个用 `copy` 修饰的非原子性属性 `prop`的功能。

上文中，我们已经知道分类中不能直接添加属性，需要手动添加存取方法：

{% codeblock NSObject+AssociatedObject.h lang:objc %}
#import <Foundation/Foundation.h>

@interface NSObject (AssociatedObject)

@property (nonatomic, copy) NSString *prop;

@end
{% endcodeblock %}

{% codeblock NSObject+AssociatedObject.m lang:objc %}
#import "NSObject+AssociatedObject.h"
#import <objc/runtime.h>

// key 有三种常见写法：
//
// 1. static void *propKey = &propKey;
// 2. static NSString *propKey = @"propKey";
// 3. static char propKey;

static NSString *propKey = @"propKey";

@implementation NSObject (AssociatedObject)

- (void)setProp:(NSString *)prop {
    objc_setAssociatedObject(self, &propKey, prop, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)prop {
    return objc_getAssociatedObject(self, &propKey);
}

@end
{% endcodeblock %}

### 黑魔法添加和替换方法

黑魔法是方法交换（method swizzling），也就是交换方法的 IMP 实现。

一般是在 `+ (void)load;` 中执行方法交换。因为它的加载时机较早，基本能确保方法已交换。

#### 方法添加

在动态方法解析中已经提到了“方法添加”。

```
//class_addMethod(Class  _Nullable __unsafe_unretained cls, SEL  _Nonnull name, IMP  _Nonnull imp, const char * _Nullable types)
class_addMethod([self class], sel, (IMP)fooMethod, "v@:");
```

参数说明：

- `cls`：被添加方法的类
- `name`：添加的方法的名称的 SEL
- `imp`：方法的实现。该函数必须至少要有两个参数，self,_cmd
- `types`：类型编码

#### 方法替换

方法替换就是改变类的选择子映射表。

![](method-swizzling.png)

如果要互换两个已经写好的方法实现，可以用下面的函数

```objc
void method_exchangeImplementations(Method m1, Method m2);
```

方法实现可以通过下面的函数获得：

```objc
void class_getInstanceMethod(Class aClass, SEL aSelector);
```

下面实现一个替换 `ViewController` 中 `viewDidLoad` 方法的例子。

```objc
@implementation ViewController
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        SEL originalSelector = @selector(viewDidLoad);
        SEL swizzledSelector = @selector(msviewDidLoad);
        
        Method originalMethod = class_getInstanceMethod(class,originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class,swizzledSelector);
        
        // 判断 original 的方法是否已经实现，如果未实现，将 swizzledMethod 的实现和类型添加进 originalSelector 中
        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
        if (didAddMethod) {
            // 将 originalMethod 的实现和类型替换到 swizzledSelector 中
            class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        }
        else {
            // 交换 originalMethod 和 swizzledMethod
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (void)msviewDidLoad {
    NSLog(@"msviewDidLoad");
    [self msviewDidLoad];
}

- (void)viewDidLoad {
    NSLog(@"viewDidLoad");
    [super viewDidLoad];
}
@end
```

### KVO 实现

KVO 全称是 Key-value observing，也就是键值观察者模式，它提供了一种当其它对象属性被修改的时候能通知到当前对象的机制。

KVO 的实现也是依赖于 runtime 中的 `isa-swizzling`。

当观察某对象 A 时，KVO 机制动态创建一个新的名为：`NSKVONotifying_A` 的新类，该类继承自对象 A 的本类，且 KVO 为 `NSKVONotifying_A` 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。

举个例子：

```objc
#import "ViewController.h"
#import <objc/runtime.h>
#import "A.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    A *a = [A new];
    NSLog(@"Before KVO: [a class] = %@, a -> isa = %@", [a class], object_getClass(a));
    [a addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    NSLog(@"After KVO: [a class] = %@, a -> isa = %@", [a class], object_getClass(a));
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
}

@end
```

程序运行的结果为：

```
Before KVO: [a class] = A, a -> isa = A
After KVO: [a class] = A, a -> isa = NSKVONotifying_A
```

可以看到当对 a 进行观察后，虽然对象 `a` 的 `class` 还是 `A`，isa 实际指向了它的子类 `NSKVONotifying_A`，来实现当前类属性值改变的监听；

所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 KVO 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为 `NSKVONotifying_A` 的类，就会发现系统运行到注册 KVO 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 `NSKVONotifying_A` 的中间类，并指向这个中间类了。

那么子类 `NSKVONotifying_A` 的 setter 方法里具体实现了什么？

KVO 的键值观察通知依赖于 NSObject 的两个方法：

- `-willChangeValueForKey:`：被观察属性发生改变之**前**，改方法被调用，通知系统该 keyPath 的属性值**即将变更**；

- `-didChangeValueForKey:`：被观察属性发生改变之**后**，改方法被调用，通知系统该 keyPath 的属性值**已经变更**。方法 `observeValueForKey:ofObject:change:context:`也会被调用。且重写观察属性的 setter 方法这种继承方式的注入是在运行时而不是编译时实现的。

因此，KVO 为子类的观察者属性重写调用存取方法的工作原理在代码中相当于：

```objc
- (void)setName:(NSString *)name {
	// KVO 在调用存取方法之前总调用 
    [self willChangeValueForKey:@"name"];
	// 调用父类的存取方法 
    [super setValue:newName forKey:@"name"];
	// KVO 在调用存取方法之后总调用
    [self didChangeValueForKey:@"name"];
}
```

### 实现字典和模型之间的转换（MJExtension）

**原理**：

通过在 `NSObject` 的分类中添加方法 `-initWithDict:`。

具体实现为：用 runtime 提供的函数 `class_copyPropertyList` 获取属性列表，再遍历 `Model` 自身所有属性（通过 `property_getName` 函数获得属性的名字，通过 `property_getAttributes` 函数获得属性的类型）。如果属性在 `json` 中有对应的值，则将其赋值。

**源码**：

```objc
- (instancetype)initWithDict:(NSDictionary *)dict {
    if (self = [self init]) {
        // 1、获取类的属性及属性对应的类型
        NSMutableArray * keys = [NSMutableArray array];
        NSMutableArray * attributes = [NSMutableArray array];
        /*
         * 例子
         * name = value3 attribute = T@"NSString",C,N,V_value3
         * name = value4 attribute = T^i,N,V_value4
         */
        unsigned int outCount;
        objc_property_t * properties = class_copyPropertyList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            objc_property_t property = properties[i];
            // 通过 property_getName 函数获得属性的名字
            NSString * propertyName = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            [keys addObject:propertyName];
            //通过 property_getAttributes 函数获得属性类型
            NSString * propertyAttribute = [NSString stringWithCString:property_getAttributes(property) encoding:NSUTF8StringEncoding];
            [attributes addObject:propertyAttribute];
        }
        // 立即释放properties指向的内存
        free(properties);

        // 2、根据类型给属性赋值
        for (NSString * key in keys) {
            if ([dict valueForKey:key] == nil) continue;
            [self setValue:[dict valueForKey:key] forKey:key];
        }
    }
    return self;
}
```
### 实现 NSCoding 的自动归档和解档

**原理**：

在 `Model` 的基类中重写方法：`-initWithCoder:` 和 `-encodeWithCoder:`。

具体实现为：用 runtime 提供的函数 `class_copyIvarList` 获取实例变量列表，再遍历 `Model` 自身所有属性，并对属性进行 `encode` 和 `decode` 操作。

**源码**：

```objc
- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int outCount;
        Ivar * ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar ivar = ivars[i];
            NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            [self setValue:[aDecoder decodeObjectForKey:key] forKey:key];
        }
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
        [aCoder encodeObject:[self valueForKey:key] forKey:key];
    }
}
```

### JSPatch

JSPatch 是一款 iOS 动态更新框架，只需要在项目中引入引擎，就可以使用 JavaScript 调用所有 Objective-C 原生接口，从而实现热更新。

它通过**完整的消息转发**实现了获取参数的问题。

原理：当调用一个 NSObject 对象不存在的方法时，并不会马上抛出异常，而是会经过多层转发，层层调用对象的 `-resolveInstanceMethod:`、`-forwardingTargetForSelector:`、`-methodSignatureForSelector:`、`-forwardInvocation:` 等方法，其中 `-forwardInvocation:` 里的 `NSInvocation` 对象会保存了这个方法调用的所有信息，包括方法名、参数和返回值类型等。所以只需要让被 JS 替换的方法最后都调用到 `-forwardInvocation:`，就可以解决无法拿到参数值的问题了。