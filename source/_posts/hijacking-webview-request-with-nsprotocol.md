---
title: 深度理解 NSURLProtocol
date: 2020-02-19 10:30:49
tags: NSURLProtocol
categories: iOS
thumbnail: nsurlprotocol.png
---

> NSURLProtocol is both the most obscure and the most powerful part of the URL Loading System. 
> <p style="text-align:right">——Mattt Thompson</p>

<!--more-->

## NSURLProtocol 是什么

NSURLProtocol 是 Foundation 框架中 [URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system?language=objc) 的一部分。它可以让开发者可以在不修改应用内原始请求代码的情况下，去改变 URL 加载的全部细节。换句话说，NSURLProtocol 是一个被 Apple 默许的中间人攻击。

虽然 NSURLProtocol 叫“Protocol”，却不是协议，而是一个抽象类。

既然 NSURLProtocol 是一个抽象类，说明它无法被实例化，那么它又是如何实现网络请求拦截的？

答案就是通过**子类化**来定义新的或是已经存在的 URL 加载行为。如果当前的网络请求是可以被拦截的，那么开发者只需要将一个自定义的 NSURLProtocol 子类注册到 App 中，在这个子类中就可以拦截到所有请求并进行修改。

那么到底哪些网络请求可以被拦截？

## NSURLProtocol 使用场景

前面已经说了，NSURLProtocol 是 URL Loading System 的一部分，所以它可以拦截所有基于 URL Loading System 的网络请求：

- NSURLSession
- NSURLConnection
- NSURLDownload
- NSURLResponse
 - NSHTTPURLResponse
- NSURLRequest
 - NSMutableURLRequest

相应的，基于它们实现的第三方网络框架 [AFNetworking](https://github.com/AFNetworking/AFNetworking) 和 [Alamofire](https://github.com/Alamofire/Alamofire) 的网络请求，也可以被 NSURLProtocol 拦截到。

但早些年基于 CFNetwork 实现的，比如 [ASIHTTPRequest](https://github.com/pokeb/asi-http-request)，其网络请求就无法被拦截。

另外，**UIWebView 也是可以被 NSURLProtocol 拦截的，但 WKWebView 不可以。**（因为 WKWebView 是基于 WebKit，并不走 C socket。）

因此，在实际应用中，它的功能十分强大，比如：

- 重定向网络请求，解决 DNS 域名劫持的问题
- 进行全局或局部的网络请求设置，比如修改请求地址、header 等
- 忽略网络请求，使用 H5 离线包或是缓存数据等
- 自定义网络请求的返回结果，比如过滤敏感信息

下面来看一下 NSURLProtocol 的相关方法。

## NSURLProtocol 的相关方法

### 创建协议对象

```objc
// 创建一个 URL 协议实例来处理 request 请求
- (instancetype)initWithRequest:(NSURLRequest *)request cachedResponse:(NSCachedURLResponse *)cachedResponse client:(id<NSURLProtocolClient>)client;
// 创建一个 URL 协议实例来处理 session task 请求
- (instancetype)initWithTask:(NSURLSessionTask *)task cachedResponse:(NSCachedURLResponse *)cachedResponse client:(id<NSURLProtocolClient>)client;
```

### 注册和注销协议类

```objc
// 尝试注册 NSURLProtocol 的子类，使之在 URL 加载系统中可见
+ (BOOL)registerClass:(Class)protocolClass;
// 注销 NSURLProtocol 的指定子类
+ (void)unregisterClass:(Class)protocolClass;
```

### 确定子类是否可以处理请求

子类化 NSProtocol 的首要任务就是告知它，需要控制什么类型的网络请求。

```objc
// 确定协议子类是否可以处理指定的 request 请求，如果返回 YES，请求会被其控制，返回 NO 则直接跳入下一个 protocol
+ (BOOL)canInitWithRequest:(NSURLRequest *)request;
// 确定协议子类是否可以处理指定的 task 请求
+ (BOOL)canInitWithTask:(NSURLSessionTask *)task;
```

### 获取和设置请求属性

NSURLProtocol 允许开发者去获取、添加、删除 request 对象的任意元数据。这几个方法常用来处理请求无限循环的问题。

```objc
// 在指定的请求中获取与指定键关联的属性
+ (id)propertyForKey:(NSString *)key inRequest:(NSURLRequest *)request;
// 设置与指定请求中的指定键关联的属性
+ (void)setProperty:(id)value forKey:(NSString *)key inRequest:(NSMutableURLRequest *)request;
// 删除与指定请求中的指定键关联的属性
+ (void)removePropertyForKey:(NSString *)key inRequest:(NSMutableURLRequest *)request;
```

### 提供请求的规范版本

如果你想要用特定的某个方式来修改请求，可以用下面这个方法。

```objc
// 返回指定请求的规范版本
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request;
```

### 确定请求是否相同

```objc
// 判断两个请求是否相同，如果相同可以使用缓存数据，通常只需要调用父类的实现
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b;
```

### 启动和停止加载

这是子类中最重要的两个方法，不同的自定义子类在调用这两个方法时会传入不同的内容，但共同点都是围绕 protocol 客户端进行操作。

```objc
// 开始加载
- (void)startLoading;
// 停止加载
- (void)stopLoading;
```

### 获取协议属性

```objc
// 获取协议接收者的缓存
- (NSCachedURLResponse *)cachedResponse;
// 接受者用来与 URL 加载系统通信的对象，每个 NSProtocol 的子类实例都拥有它
- (id<NSURLProtocolClient>)client;
// 接收方的请求
- (NSURLRequest *)request;
// 接收方的任务
- (NSURLSessionTask *)task;
```

NSURLProtocol 在实际应用中，主要是完成两步：拦截 URL 和 URL 转发。先来看如何拦截网络请求。

## 如何利用 NSProtocol 拦截网络请求

### 创建 NSURLProtocol 子类

这里创建一个名为 `HTCustomURLProtocol` 的子类。

```objc
@interface HTCustomURLProtocol : NSURLProtocol
@end
```
### 注册 NSURLProtocol 的子类

在合适的位置注册这个子类。对基于 NSURLConnection 或者使用 `[NSURLSession sharedSession]` 初始化对象创建的网络请求，调用 `registerClass` 方法即可。

```objc
[NSURLProtocol registerClass:[NSClassFromString(@"HTCustomURLProtocol") class]];
// 或者
// [NSURLProtocol registerClass:[HTCustomURLProtocol class]]; 
```

如果需要全局监听，可以设置在 `AppDelegate.m` 的 `didFinishLaunchingWithOptions` 方法中。如果只需要在单个 UIViewController 中使用，记得在合适的时机注销监听：

```objc
[NSURLProtocol unregisterClass:[NSClassFromString(@"HTCustomURLProtocol") class]];
```

如果是基于 NSURLSession 的网络请求，且不是通过 `[NSURLSession sharedSession]` 方式创建的，就得配置 NSURLSessionConfiguration 对象的 `protocolClasses` 属性。

```objc
NSURLSessionConfiguration *sessionConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
sessionConfiguration.protocolClasses = @[[NSClassFromString(@"HTCustomURLProtocol") class]];
```

### 实现 NSURLProtocol 子类

实现子类分为五个步骤：

> 注册 → 拦截 → 转发 → 回调 → 结束

以拦截 UIWebView 为例，这里需要重写父类的这五个核心方法。

```objc
// 定义一个协议 key
static NSString * const HTCustomURLProtocolHandledKey = @"HTCustomURLProtocolHandledKey";

// 在拓展中定义一个 NSURLConnection 属性。通过 NSURLSession 也可以拦截，这里只是以 NSURLConnection 为例。
@property (nonatomic, strong) NSURLConnection *connection;
// 定义一个可变的请求返回值，
@property (nonatomic, strong) NSMutableData *responseData;

// 方法 1：在拦截到网络请求后会调用这一方法，可以再次处理拦截的逻辑，比如设置只针对 http 和 https 的请求进行处理。
+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
    // 只处理 http 和 https 请求
    NSString *scheme = [[request URL] scheme];
    if ( ([scheme caseInsensitiveCompare:@"http"] == NSOrderedSame ||
          [scheme caseInsensitiveCompare:@"https"] == NSOrderedSame)) {
        // 看看是否已经处理过了，防止无限循环
        if ([NSURLProtocol propertyForKey:HTCustomURLProtocolHandledKey inRequest:request]) {
            return NO;
        }
        // 如果还需要截取 DNS 解析请求中的链接，可以继续加判断，是否为拦截域名请求的链接，如果是返回 NO
        return YES;
    }
    return NO;
}

// 方法 2：【关键方法】可以在此对 request 进行处理，比如修改地址、提取请求信息、设置请求头等。
+ (NSURLRequest *) canonicalRequestForRequest:(NSURLRequest *)request {
    // 可以打印出所有的请求链接包括 CSS 和 Ajax 请求等
    NSLog(@"request.URL.absoluteString = %@",request.URL.absoluteString);
    NSMutableURLRequest *mutableRequest = [request mutableCopy];
    return mutableRequest;
}

// 方法 3：【关键方法】在这里设置网络代理，重新创建一个对象将处理过的 request 转发出去。这里对应的回调方法对应 <NSURLProtocolClient> 协议方法
- (void)startLoading {
    // 可以修改 request 请求
    NSMutableURLRequest *mutableRequest = [[self request] mutableCopy];
    // 打 tag，防止递归调用
    [NSURLProtocol setProperty:@YES forKey:HTCustomURLProtocolHandledKey inRequest:mutableRequest];
    // 也可以在这里检查缓存
    // 将 request 转发，对于 NSURLConnection 来说，就是创建一个 NSURLConnection 对象；对于 NSURLSession 来说，就是发起一个 NSURLSessionTask。
    self.connection = [NSURLConnection connectionWithRequest:mutableRequest delegate:self];
}

// 方法 4：主要判断两个 request 是否相同，如果相同的话可以使用缓存数据，通常只需要调用父类的实现。
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b {
    return [super requestIsCacheEquivalent:a toRequest:b];
}

// 方法 5：处理结束后停止相应请求，清空 connection 或 session
- (void)stopLoading {
    if (self.connection != nil) {
        [self.connection cancel];
        self.connection = nil;
    }
}

// 按照在上面的方法中做的自定义需求，看情况对转发出来的请求在恰当的时机进行回调处理。
#pragma mark- NSURLConnectionDelegate

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}

#pragma mark - NSURLConnectionDataDelegate

// 当接收到服务器的响应（连通了服务器）时会调用
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    self.responseData = [[NSMutableData alloc] init];
    // 可以处理不同的 statusCode 场景
    // NSInteger statusCode = [(NSHTTPURLResponse *)response statusCode];
    // 可以设置 Cookie
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
}

// 接收到服务器的数据时会调用，可能会被调用多次，每次只传递部分数据
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.responseData appendData:data];
    [self.client URLProtocol:self didLoadData:data];
}

// 服务器的数据加载完毕后调用
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
}

// 请求错误（失败）的时候调用，比如出现请求超时、断网，一般指客户端错误
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}

```
上面用到的一些 NSURLProtocolClient 方法：

```objc
@protocol NSURLProtocolClient <NSObject>
// 请求重定向
- (void)URLProtocol:(NSURLProtocol *)protocol wasRedirectedToRequest:(NSURLRequest *)request redirectResponse:(NSURLResponse *)redirectResponse;
// 响应缓存是否合法
- (void)URLProtocol:(NSURLProtocol *)protocol cachedResponseIsValid:(NSCachedURLResponse *)cachedResponse;
// 刚接收到 response 信息
- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveResponse:(NSURLResponse *)response cacheStoragePolicy:(NSURLCacheStoragePolicy)policy;
// 数据加载成功
- (void)URLProtocol:(NSURLProtocol *)protocol didLoadData:(NSData *)data;
// 数据完成加载
- (void)URLProtocolDidFinishLoading:(NSURLProtocol *)protocol;
// 数据加载失败
- (void)URLProtocol:(NSURLProtocol *)protocol didFailWithError:(NSError *)error;
// 为指定的请求启动验证
- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
// 为指定的请求取消验证
- (void)URLProtocol:(NSURLProtocol *)protocol didCancelAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge;
@end
```

## 补充内容

### 使用 NSURLSession 时的注意事项

如果在 NSURLProtocol 中使用 NSURLSession，需要注意：

- 拦截到的 request 请求的 HTTPBody 为 nil，但可以借助 HTTPBodyStream 来获取 body；
- 如果要用 `registerClass` 注册，只能通过 `[NSURLSession sharedSession]` 的方式创建网络请求。

### 注册多个 NSURLProtocol 子类

当有多个自定义 NSURLProtocol 子类注册到系统中的话，会按照他们注册的反向顺序依次调用 URL 加载流程，也就是最后注册的 NSURLProtocol 会被优先判断。

对于通过配置 NSURLSessionConfiguration 对象的 `protocolClasses` 属性来注册的情况，`protocolClasses` 数组中只有第一个 NSURLProtocol 会起作用，后续的 NSURLProtocol 就无法拦截到了。

所以 [OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs) 在注册 NSURLProtocol 子类的时候是这样处理的：

```objc
+ (void)setEnabled:(BOOL)enable forSessionConfiguration:(NSURLSessionConfiguration*)sessionConfig
{
    // Runtime check to make sure the API is available on this version
    if ([sessionConfig respondsToSelector:@selector(protocolClasses)]
        && [sessionConfig respondsToSelector:@selector(setProtocolClasses:)])
    {
        NSMutableArray * urlProtocolClasses = [NSMutableArray arrayWithArray:sessionConfig.protocolClasses];
        Class protoCls = HTTPStubsProtocol.class;
        if (enable && ![urlProtocolClasses containsObject:protoCls])
        {
            // 将自己的 NSURLProtocol 插入到 protocolClasses 的第一个，进行拦截
            [urlProtocolClasses insertObject:protoCls atIndex:0];
        }
        else if (!enable && [urlProtocolClasses containsObject:protoCls])
        {
            // 拦截完成后移除
            [urlProtocolClasses removeObject:protoCls];
        }
        sessionConfig.protocolClasses = urlProtocolClasses;
    }
    else
    {
        NSLog(@"[OHHTTPStubs] %@ is only available when running on iOS7+/OSX9+. "
              @"Use conditions like 'if ([NSURLSessionConfiguration class])' to only call "
              @"this method if the user is running iOS7+/OSX9+.", NSStringFromSelector(_cmd));
    }
}
```

### 如何拦截 WKWebView

虽然 NSURLProtocol 无法直接拦截 WKWebView，但其实还是有解决方案的。就是使用 `WKBrowsingContextController` 和 `registerSchemeForCustomProtocol`。

```objc
// 注册 scheme
Class cls = NSClassFromString(@"WKBrowsingContextController");
SEL sel = NSSelectorFromString(@"registerSchemeForCustomProtocol:");
if ([cls respondsToSelector:sel]) {
    // 通过 http 和 https 的请求，同理可通过其他的 Scheme 但是要满足 URL Loading System
    [cls performSelector:sel withObject:@"http"];
    [cls performSelector:sel withObject:@"https"];
}
```

但由于这涉及到了私有方法，直接引用无法过苹果的机审，所以使用的时候需要对字符串做下处理，比如对方法名进行算法加密处理等，实测也是可以通过审核的。

总之，NSURLProtocol 非常强大，无论是优化 App 的性能，还是拓展功能，都具有很强的可塑空间，但在使用的同时，又要多关注它带来的问题。尽管它在很多框架或者知名项目中都已经得以应用，其奥义依然值得开发者们去深入研究。