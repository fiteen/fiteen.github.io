---
title: 面试题
---

# 自我介绍

你好我是FiTeen，2016 年毕业于XX大学XXXXXXX专业，从事 iOS 开发 4 年时间，上架过近十款 App，其中涉及到 iPad 点单、电商类、母婴社交类、医疗类、金融类的 App，积累了不少实战经验。我的特长就是学习新鲜事物和写文章，离职期间对自己之前一些项目和技术点进行了总结，发表的一些文章也会一些平台收录了。

# 前端

- Koa2 是 Node 的一个框架，可以认为是后端的东西
- React/Vue：常见的前端单页框架
- JQuery 是比较早起的，方便直接操作 DOM

# Flutter

## Fish Redux

是一个基于 Redux 数据管理的组装式 Flutter 应用框架，它特别适用于构建中大型的复杂应用。

特性包括：

- 函数式编程
- 可预测的状态容器
- 可插拔组件化
- 无损性能

它的特点是配置式组装。 一方面我们将一个大的页面，对视图和数据层层拆解为互相独立的 Component|Adapter，上层负责组装，下层负责实现； 另一方面将 Component|Adapter 拆分为 View，Reducer，Effect 等相互独立的上下文无关函数。

lib//主要编写代码目录
├── app.dart//主入口
├── entity_factory.dart//模型工厂辅助类
├── home//主页面
│   ├── action.dart
│   ├── effect.dart
│   ├── home_adapter//适配器
│   │   ├── action.dart
│   │   ├── adapter.dart
│   │   └── reducer.dart
│   ├── home_component//组件
│   │   ├── action.dart
│   │   ├── component.dart
│   │   ├── effect.dart
│   │   ├── reducer.dart
│   │   ├── state.dart
│   │   └── view.dart
│   ├── page.dart
│   ├── state.dart
│   └── view.dart
├── login//login页面
│   ├── action.dart
│   ├── effect.dart
│   ├── page.dart
│   ├── state.dart
│   └── view.dart
├── main.dart
└── network//网络请求和模型实体类
    ├── handover_entity.dart
    ├── netuntil.dart
    └── user_info_entity.dart

用户进行某个操作 -> 然后调用 context.dispatch 方法发送一个由 ActionCreator 创建的 Action -> effect 接收并处理，然后 dispatch 给 reducer -> reducer 接收并产生新的 state -> state 更新导致界面显示的刷新。


- state：进行状态操作的，是数据源，你所定义的page里面的字段都需要写在这。状态类必须实现 Cloneable 接口。
- action：定义事件的，比如页面跳转，刷新数据，定义的 action。
- view：提供实现界面的方法
- effect：具体的事件实现
- adapter
- reducer：用于接收意图，该文件提供了Reducer，声明Reducer监听的action，实现监听到action的动作
- page：组装

# 计算机网络网络

## HTTP & HTTPS & SSL

HTTP：超文本转移协议。
HTTPS：安全套接字层上的超文本传输协议。

### HTTP 和 HTTPS 区别

- HTTP 是超文本传输协议，信息是明文传输，存在安全风险的问题。HTTPS 则解决 HTTP 不安全的缺陷，在 TCP 和 HTTP 网络层之间加入了 SSL/TLS 安全协议，使得报文能够加密传输。
- HTTP 连接建立相对简单，TCP 三次握手之后便可进行 HTTP 的报文传输。而 HTTPS 在 TCP 三次握手之后，还需进行 SSL/TLS 的握手过程，才可进入加密报文传输。
- HTTP 的端口号是 80，HTTPS 的端口号是 443。
- HTTPS 协议需要向 CA（证书权威机构）申请数字证书，来保证服务器的身份是可信的。

### http 请求建立的过程

1. 用户输入 URL 地址；
2. 对 URL 地址进行 DNS 域名解析；
3. 建立 TCP 连接（三次握手）；
4. 浏览器发起 HTTP 请求报文，
5. 服务器返回 HTTP 响应报文，浏览器得到 HTML 代码；
6. 关闭 TCP 连接（四次挥手）；
7. 浏览器解析 html 代码，并请求 html 代码中的资源（如 js、css、图片等）；
8. 浏览器对页面进行渲染呈现给用户。

###  HTTP 常见状态码

#### 1XX 信息

表示服务器已接收了客户端请求，客户端可继续发送请求。

- 100 continue，请求者应当继续提出请求。服务器已收到请求的一部分，正在等待其余部分。
- 101 switching protocols，指服务器将按照其上的头信息变为一个不同的协议。

#### 2XX 成功

- 200 OK，表示从客户端发来的请求在服务器端被正确处理
- 204 no content，表示请求成功，但响应报文不含实体的主体部分
- 206 partial content，进行范围请求

#### 3XX 重定向

- 301 moved permanently，永久性重定向，表示资源已被分配了新的 URL
- 302 found，临时性重定向，表示资源临时被分配了新的 URL
- 303 see other，表示资源存在着另一个 URL，应使用 GET 方法丁香获取资源
- 304 not modified，表示服务器允许访问资源，但因发生请求未满足条件的情况
- 307 temporary redirect，临时重定向，和302含义相同

#### 4XX 客户端错误

- 400 bad request，请求报文存在语法错误
- 401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
- 403 forbidden，表示对请求资源的访问被服务器拒绝
- 404 not found，表示在服务器上没有找到请求的资源

#### 5XX 服务器错误

- 500 internal sever error，表示服务器端在执行请求时发生了错误；
- 503 service unavailable，表明服务器暂时处于超负载或正在停机维护，无法处理请求。

### 一个 HTTPS 请求建立的过程

1. 客户端向服务器发起 HTTPS 请求，连接到服务器的 443 端口；
2. 服务器端有一个密钥对，即公钥和私钥，是用来进行非对称加密使用的，服务器端保存着私钥，不能将其泄露，公钥可以发送给任何人。
3. 服务器将自己的公钥发送给客户端。
4. 客户端收到服务器端的公钥之后，会对公钥进行检查，验证其合法性（服务器发送的数字证书的合法性），如果公钥合格，那么客户端会随机生成对称密钥。然后用服务器的公钥对客户端密钥进行非对称加密，这样客户端密钥就变成密文了，至此，HTTPS 中的第一次 HTTP 请求结束。
5. 客户端会发起 HTTPS 中的第二个 HTTP 请求，将加密之后的客户端密钥发送给服务器。
6. 服务器接收到客户端发来的密文之后，会用自己的私钥对其进行非对称解密，解密之后的明文就是客户端密钥，然后用客户端密钥对数据进行对称加密，这样数据就变成了密文。
7. 然后服务器将加密后的密文发送给客户端。
8. 客户端收到服务器发送来的密文，用客户端密钥对其进行对称解密，得到服务器发送的数据。这样 HTTPS 中的第二个 HTTP 请求结束，整个HTTPS传输完成。

### SSL

SSL/TLS 是 HTTPS 安全性的核心模块，TLS 的前身是 SSL。SSL/TLS是建立在 TCP 协议之上（因而也是应用层级别的协议）。其包括 TLS Record Protocol（保证数据传输过程中的完整性和私密性）和 TLS Handshaking Protocols（后者负责握手过程中的身份认证）两个模块。

SSL 旨在通过 Web 创建安全的 Internet 通信。它是一种标准协议，用于加密浏览器和服务器之间的通信。它允许通过 Internet 安全轻松地传输账号密码、银行卡、手机号等私密信息。
SSL 证书就是遵守 SSL 协议，由受信任的 CA 机构颁发的数字证书。使用 SSL 证书有许多好处：

- 保障服务器和浏览器之间的通信安全
- 验证网站的真实身份，区别于钓鱼欺诈网站
- 加密用户的敏感信息以确保安全
- 提高SEO搜索引擎排名
- 提升用户对网站的信任
- 有助于提高网站的在线销售业绩

## TCP 三次握手 & 四次挥手

**三次握手**

1. 第一次握手：起初两端都处于 CLOSED 关闭状态，Client 将标志位 SYN 置为 1，随机产生一个值 seq = x，并将该数据包发送给 Server，Client 进入 SYN-SENT 状态，等待 Server 确认。
2. 第二次握手：Server 收到数据包后由标志位 SYN = 1 得知 Client 请求建立连接，Server 将标志位 SYN 和 ACK 都置为 1，ack = x + 1，随机产生一个值 seq = y，并将该数据包发送给 Client 以确认连接请求，Server 进入 SYN-RCVD 状态，此时操作系统为该 TCP 连接分配 TCP 缓存和变量。
3. 第三次握手：Client 收到确认后，检查 seq 是否为 x + 1，ACK 是否为 1，如果正确则将标志位 ACK 置为 1，ack = y + 1，并且此时操作系统为该 TCP 连接分配 TCP 缓存和变量，并将该数据包发送给 Server，Server 检查 ack 是否为 y + 1，ACK 是否为 1，如果正确则连接建立成功，Client 和 Server 进入 established 状态，完成三次握手，随后 Client 和 Server 就可以开始传输数据。

**四次挥手**

1. 第一次挥手：Client 的应用进程先向其 TCP 发出连接释放报文段（FIN = 1，序号 seq = u），并停止再发送数据，主动关闭 TCP 连接，进入 FIN-WAIT-1（终止等待1）状态，等待 Server 的确认。
2. 第二次挥手：Server 收到连接释放报文段后即发出确认报文段，（ACK = 1，确认号 ack = u + 1，序号 seq = v），Server 进入 CLOSE-WAIT（关闭等待）状态，此时的 TCP 处于半关闭状态，Client 到 Server 的连接释放。
> 注：Client 收到 Server 的确认后，进入 FIN-WAIT-2（终止等待2）状态，等待 Server 发出的连接释放报文段。
3. 第三次挥手：Server 已经没有要向 Client 发出的数据了，Server 发出连接释放报文段（FIN = 1，ACK = 1，序号 seq = w，确认号 ack = u + 1），Server 进入 LAST-ACK（最后确认）状态，等待 Client 的确认。
4. 第四次挥手：Client 收到 Server 的连接释放报文段后，对此发出确认报文段（ACK = 1，seq = u + 1，ack = w + 1），Client 进入 TIME-WAIT（时间等待）状态。此时 TCP 未释放掉，需要经过时间等待计时器设置的时间 2MSL 后，Client 才进入 CLOSED 状态。

## IP、TCP 和 Socket

### IP 协议

TCP/IP 中的 IP 是网络协议 (Internet Protocol) 的缩写。IP 实现了分组交换网络。IP 网络负责源主机与目标主机之间的数据包传输。IP 协议在传输过程中，数据包可能会丢失，也有可能被重复传送导致目标主机收到多个同样的数据包。

### TCP 协议

TCP 是基于 IP 层的协议。TCP 是**全双工、面向连接、可靠的、有序的**、有错误检查机制的基于字节流传输的协议。这样当两个设备上的应用通过 TCP 来传递数据的时候，总能够保证目标接收方收到的数据的顺序和内容与发送方所发出的是一致的。TCP 建立的是双向连接，通信双方可以同时进行数据的传输。TCP 会确保所传输的数据的正确性，即接受方收到的数据与发出方的数据一致。

### UDP 协议

用户数据报协议，不可靠，开销小。

### Socket

Socket 翻译为套接字，是支持 TCP/IP 协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：

- 连接使用的协议
- 本地主机的IP地址
- 本地进程的协议端口
- 远地主机的 IP 地址
- 远地进程的协议端口。

socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。它不属于 OSI 七层协议，它只是对于TCP，UDP 协议的一套封装，让我们开发人员更加容易编写基于 TCP、UDP 的应用。

## GET 和 POST

GET - 从指定的资源请求数据。
  - 可被缓存
  - 保留在浏览器历史记录中
  - 不应在处理敏感数据时使用
  - 请求有长度限制
  - 请求只应当用于取回数据
POST - 向指定的资源提交要被处理的数据。
  - 不会被缓存
  - 不会保留在浏览器历史记录中
  - 对数据长度没有要求

## 长连接

### 长连接在 iOS 开发中的作用

长连接：通过三次握手建立链接之后，该条链路就一直存在，而且该链路是一种双向的通行机制，适合于频繁的网络请求，避免 Http每一次请求都会建立链接和关闭链接的操作，减少浪费，提高效率。苹果提供的 push 服务 apns 就是典型的长连接的应用，IM 应用、订单推送这些也是长连接的典型应用。

### 长连接为什么要保持心跳

国内移动无线网络运营商在链路上一段时间内没有数据通讯后, 会淘汰NAT表中的对应项, 造成链路中断。而国内的运营商一般NAT超时的时间为5分钟，所以通常我们心跳设置的时间间隔为3-5分钟。

### 长连接选择 TCP 协议还是 UDP 协议

使用 TCP 进行数据传输的话，简单、安全、可靠，但是带来的是服务端承载压力比较大。

使用 UDP 进行数据传输的话，效率比较高，带来的服务端压力较小，但是需要自己保证数据的可靠性，不作处理的话，会导致丢包、乱序等问题。

如果你的技术团队实力过硬，你可以选择 UDP 协议，否则还是使用 TCP 协议比较好。据说腾讯 IM 就是使用的 UDP 协议，然后还封装了自己的私有协议，来保证 UDP 数据包的可靠传输。

### 服务端单机最大 TCP 连接数是多少

理论上：对 IPV4，不考虑ip地址分类等因素，最大tcp连接数约为2的32次方（ip数）×2的16次方（port数），也就是server端单机最大tcp连接数约为2的48次方。

实际最大值：受到**机器资源**、**操作系统**等的限制，单机最大并发TCP连接数超过10万，甚至上百万是没问题的，国外 Urban Airship 公司在产品环境中已做到 50 万并发。

## OSI 七层网络协议

从下到上：

物理层：负责机械、电子、定时接口通信信道上的原始比特流的传输。
数据链路层：负责物理寻址，同时将原始比特流转变成逻辑传输线路。
网络层：控制子网的运行，如逻辑编址、分组传输、路由选择。
传输层：接受上一层的数据，在必要的时候把数据进行分割，并将这些数据交给网络层，且保证这些数据段有效到达对方。
会话层：不同机器上的用户之间建立以及管理回话。
表示层：信息的语法语义以及它们的关联，如加密解密、转换翻译、压缩解压缩。
应用层：各种应用程序协议，如 http、ftp、SMTP、POP3。

## iOS 的分区

C语言的内存模型分为5个区：从高到低：栈区、堆区、静态区、常量区、代码区。

- **栈区**：存放函数的**参数值**、**局部变量**等，由**编译器**自动分配和释放，通常在函数执行完后就释放了，其操作方式类似于数据结构中的**栈**。栈内存分配运算内置于 CPU 的指令集，效率很高，但是分配的内存量有限，比如 iOS 中栈区的大小是2M。
- **堆区**：就是通过 `new`、`malloc`、`realloc` 分配的内存块，编译器不会负责它们的释放工作，需要用程序员手动释放。分配方式类似于数据结构中的**链表**。在 iOS 开发中所说的“内存泄漏”说的就是堆区的内存。
- **静态区**：全局变量和静态变量（在 iOS 中就是用 `static` 修饰的局部变量或者是全局变量）的存储是放在一块的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和静态变量在相邻的另一块区域。程序结束后，由系统释放。
- **常量区**：常量存储在这里，不允许修改。
- **代码区**：存放函数体的二进制代码。

> 自由存储区：就是那些由 malloc 等分配的内存块，他和堆是十分相似，不过它是用 free 来结束自己的生命的。

### 堆和栈的区别是什么

堆空间的内存是动态分配的，一般存放对象，并且需要手动释放内存。当然，iOS 引入了 ARC（自动引用计数管理技术）之后，程序员就不需要用代码管理对象的内存了，之前 MRC（手动管理内存）的时候，程序员需要手动 release 对象。

栈空间的内存是由系统自动分配，一般存放局部变量，比如对象的地址等值，不需要程序员对这块内存进行管理，比如，函数中的局部变量的作用范围（生命周期）就是在调完这个函数之后就结束了。

从**申请的大小**方面讲：

- 栈空间比较小；
- 堆空间比较大。

从**数据存储**方面来说：

- 栈空间中一般存储基本数据类型，对象的地址；
- 堆空间一般存放对象本身，block 的 copy 等。

# 简历项目有关

## OC 和 java 区别

- OC 不支持方法重载
- OC 分类+协议实现 java 的多继承
- OC 中可以用 nil 表示 null
- OC 需要 alloc init、 java 直接 new
- OC 中有很多 C 语言函数、结构体，Java 就是纯面向对象的
- OC 的异常都继承自 NSException，也是继承自 NSObject。但是 java 有业务异常就必须捕获，不然出现编译错误
- OC 通过格式化字符串 `%@` 触发 `description` 方法，但是 java 里可以直接调 toString 方法
- OC 对象的复制需要遵循 `NSCopying` 协议，这个协议中有个复制方法 `–(id)copyWithZone:(*NSZone)zone` 需要我们去实现。就相当于java中的 `clone()`方法，也就是对象的深度复制，所谓深度复制就是重新分配一个存储空间，并将原来对象的内容都复制过来。
- 使用自动释放池，通过引用计数处理对象的内存管理
- 拥有 id 这种通用对象类型

## App 启动

### Mach-O

Mach-O 文件格式是 OS X 与 iOS 系统上的可执行文件格式，像我们编译过程产生的 `.o` 文件，以及程序的可执行文件，动态库等都是 Mach-O 文件，它的结构如下:

- Header: 保存了一些基本信息，包括了该文件运行的平台（CPU 架构）、文件类型、LoadCommands 的个数等。
- LoadCommands： 可以理解为加载命令，在加载 Mach-O 文件时会使用这里的数据来确定内存的分布以及相关的加载命令。比如我们的 **main 函数的加载地址，程序所需的 dyld 的文件路径，以及相关依赖库的文件路径**。
- Data：Data 中的每个段（segment）的数据都保存在这里，每个段都有一个或多个 Section，它存放了具体代码和数据，包含：
  - __TEXT 包含 Mach header，被执行的代码和只读常量（如C 字符串）。只读可执行（r-x）。
  - __DATA 包含全局变量，静态变量等。可读写（rw-）。
  - __LINKEDIT 包含了加载程序的元数据，比如函数的名称和地址。只读（r–-）。

### dylib

动态链接库，运行时加载的，可以被多个 App 的进程共用。

分为**系统 dylib** 和**内嵌 dylib**（embed dylib，即开发者手动引入的动态库）。系统 dylib 有：

- iOS 中用到的所有系统 framework，比如 UIKit、Foundation；
- 系统级别的 libSystem（如 libdispatch(GCD)、libsystem_c(C语言库)、libsystem_blocks(Block)、libCommonCrypto(加密库，比如常用的 md5)）；
- 加载 OC runtime 方法的 libobjc；
……

### dyld

动态链接器，其本质也是 Mach-O 文件，一个专门用来加载 dylib 文件的库。 dyld 位于 /usr/lib/dyld，可以在 mac 和越狱机中找到。dyld 会将 App 依赖的动态库和 App 文件加载到内存后执行。

### pre-main

1. 系统先**读取 App 的可执行文件**(Mach-O文件)，从里面获得 dyld 的路径，然后**加载 dyld**，dyld 去初始化运行环境。

2. 开启缓存策略，**加载程序相关依赖库**(其中也包含可执行文件)，并对这些库进行链接，最后**调用每个依赖库的初始化方法**，在这一步，runtime 被初始化。

3. 当所有依赖库的初始化后，轮到最后一位(程序可执行文件)进行初始化，在这时 runtime 会**对项目中所有类进行类机构初始化**，然后调用所有的 load 方法。最后 dyld 返回 main 函数地址，main 函数被调用，我们便来到程序入口 main 函数。

#### Load Dylibs

这一步，指的是动态库加载。在此阶段，dyld 会：

1. 分析 App 依赖的所有 dylib；
2. 找到 dylib 对应的 Mach-O 文件；
3. 打开、读取这些 Mach-O 文件，并验证其有效性；
4. 在系统内核中注册代码签名；
5. 对 dylib 的每一个 `segment` 调用 `mmap()`（把文件和对象映射到内存）。

### Rebase/Binding

这一步，做的是指针重定位。Rebase 和 Binding 属于**静态调整**（fix-up），修改的是 __DATA 段中的内容。

在 dylib 的加载过程中，系统为了安全考虑，引入了 ASLR（Address Space Layout Randomization）技术和代码签名。由于 ASLR 的存在，镜像会在新的随机地址（actual_address）上加载，和之前指针指向的地址（preferred_address）会有一个偏差（slide，slide=actual_address-preferred_address），因此 dyld 需要**修正这个偏差**，指向正确的地址。具体通过这两步实现：

第一步：Rebase，在 image 内部调整指针的指向。将 image 读入内存，并以 page 为单位进行加密验证，保证不会被篡改，性能消耗主要在 IO。

第二步：Binding，符号绑定。将指针指向 image 外部的内容。查询符号表，设置指向镜像外部的指针，性能消耗主要在 CPU 计算。

#### ObjC Setup

通知 runtime 去做一些代码运行时需要做的事情：

1. dyld 会注册所有声明过的 ObjC 类；
2. 将分类插入到类的方法列表中；
3. 检查每个 selector 的唯一性。

#### Initializers

而这里则开始动态调整，往堆和栈中写入内容。具体工作有：

1. 调用每个 Objc 类和分类中的 +load 方法；
2. 调用 C/C++ 中的构造器函数（用 attribute((constructor)) 修饰的函数）；
3. 创建非基本类型的 C++ 静态全局变量。

### main 函数阶段

#### 有 storyboard

1. main 函数
2. UIApplicationMain
  - 创建 UIApplication 对象
  - 创建 UIApplication 的 delegate
3. 根据 info.plist 获得 delegate 对象
  - 创建 UIWindow
  - 创建和设置 UIWindow 的 rootViewController
  - 显示窗口 window

#### 没有 storyboard

1. main 函数
2. UIApplicationMain
  - 创建 UIApplicationMain
  - 创建 UIApplication 的 delegate
3. delegate 对象开始处理（监听）系统事件
  - 程序启动完毕，调用 didFinishLaunching 方法
  - didFinishLaunch 方法中创建 UIWindow 设置 rootiewControoler 并显示窗口 window

### 总结优化方案

1. 进行代码瘦身，合并或删除无效的 ObjC 类、Category、方法、C++ 静态全局变量等；
2. 在闪屏页背后就进行启动时的网络数据读取，并对配置数据进行缓存，再次启动的时候直接读取本地数据；
3. 在 launch 时期去除一些不必要的初始化操作，比如友盟分享，可以等到第一次调用再出初始化；
4. 用纯代码而不是 Storyboard 的方式加载启动页；
5. 在 release 包中移除 NSLog 打印；
6. 将不必须在 +load 方法中执行的任务延迟到 +initialize 中；
7. 重新梳理架构，减少不必要的内置动态库数量；
8. webView 加载优化；
9. 在 release 包中移除 NSLog 打印；

## 项目中的性能优化

- 视图延迟/懒加载；
- 避免臃肿的 xib（xib 会被打进二进制文件中，启动时加载）；
- 尽可能将 View 设置成不透明；
- 不要阻塞主线程（执行 I/O 操作，大多数会阻塞主线程，如果需要做一些其它类型开销很大的操作，使用 GCD 或 NSOperations、NSOperationQueues）；
- 选择正确的集合，Array（有序、慢）、Dictionary（快）、Set(无序、快)
- 避免使用过大的第三方框架，用轻量级的工具类代替；
- 多使用缓存（图片、数据等）；
- 让图片大小和 UIImageView 一样大（图片的缩放非常耗费资源）；
- 优化 tableView；
  - 减少 CPU/GPU 计算量：
    - cell 复用（reuseIdentifier）
    - cell 高度的计算（对于固定高度，直接设置 `rowHeight`，不要使用 `heightForRowAtIndexPath` 方法，节省不必要的计算和开销；对于动态高度，提前计算好，存储在 model 中）
    - 减少 cell 内部控件的层级
    - 通过覆盖圆角图片来实现头像的圆角效果
    - 不要给 cell 动态添加 subView（初始化 cell 完毕后，根据需要设置 hide 属性显示和隐藏。） 
  - 按需加载 cell：
    - 在 `cellForRow` 方法中之加载可见的 cell；
    - 监听 tableView 的滚动，保存目标滚动范围的前后三行索引；
  - 异步处理 cell
    - 异步加载网络图片
    - 异步绘制本地图片
    - 异步绘制 UIView
    - 异步绘制 NSString、UILabel
- 选择正确的数据格式（JSON 快、小、便于传输；XML 处理很大数据）；
- 处理内存警告；
- 缓存（NSCache）；
- 考虑绘制
- 处理内存警告（实现在 appdelegate 中的 applicationDidReceiveMemoryWarning：方法。在UIViewController 子类中重写 didReceiveMemoryWarning 方法。）；
- 重用花销很大的对象（比如 NSDataFormatter 和 NSCalendar）；
- 设置阴影路径；
- 选择正确的数据存储方式（NSUserDefaults、XML、JSON、plist、NSCoding、SQLite、CoreData）；
- 加速启动时间；
- 使用Autorelease Pool
- 缓存图片——或者不缓存（imageNamed 可以缓存已经加载的图片，这个方法会在系统缓存中根据指定的名字寻找图片，如果找到了就返回，如果没有找到就从指定的文件中加载，并缓存起来，然后再将结果返回，适合需要重用的图片；imageWithContentsOfFile 只是简单的加载图片，并不会将图片缓存，适合加载只使用一次的图片）；
- 尽量避免 Data 格式化（日期 NSDateFormatter，改用时间戳）；

## 签名/重签名

### 简单签名

Apple 公钥内置于 iOS 系统，Apple 私钥由苹果后台保存。

我们把 App 上传到 App Store 时，苹果后台用私钥对 App 数据进行签名，iOS 系统下载 App 后，用公钥验证这个签名，如果签名正确则这个 App 肯定是由苹果后台认证的，并且没有被修改或损坏。

### 双层代码

#### 签名原理

Mac 公钥：借助钥匙串导出的 CSR 文件；
Mac 私钥：p12 文件；
Apple 公钥：内置于 iOS 系统；
Apple 私钥：.cer 证书。

**过程**：
1. 借助本地（Mac 设备）钥匙串导出 CSR 文件；
2. 在苹果后台上传 CSR 后生成 .cer 证书，并下载到本地；
3. 安装 .cer 证书，这时 .cer 会去签名 CSR 文件，可以拿到含有签名的证书，根据 CSR 可以在本地找到对应的 p12 文件；
4. 在 Apple 后台申请 App ID，配置好的 UDID（注册设备） 列表以及 App 申请的权限（Entitlements），生成的 PP 文件，下载到本地并双击执行。
5. 编译时，用 p12 签名 App，同时打包，安装中包含 PP 文件，生成的 ipa 中的描述文件就是对应 PP 文件，同时可执行文件会把签名写入描述文件中。
6. 安装前，Apple 公钥会对描述文件、证书签名，验证是否合法，CSR 也会对 App 签名验证。验证通过，就开始安装。

### 重签名原理和实现

重签名的核心是删除原来 `ipa` 包里的 `_CodeSignature` 使用 codesign 命令，将新的证书打进 ipa，生成新的 `_CodeSignature`。

步骤：

以 qq.ipa 为例。

```shell
// 1. 解压 qq.ipa 找到 Payload 文件。
unzip qq.ipa
// 2. 将 Payload 目录中的 `_CodeSignature` 文件删除。
rm -rf Payload/qq.app/_CodeSignature/
// 3. 用新证书对应的 `embedded.mobileprovision` 文件替换掉 qq.ipa 中的 `embedded.mobileprovision` 文件。
cp embedded.mobileprovision Payload/qq.app/embedded.mobileprovision
// 4. 重新签名，`iPhone Distribution: xxx Technology Co., Ltd.` 指的是自己的 `embedded.mobileprovision` 文件中用到的签名证书名称，在 Xcode 或钥匙串中可以找到。如果有 framework 或者 dylib 先分别对两者重签。
// codesign -f -s "iPhone Distribution: xxx Technology Co., Ltd. " --entitlements entitlements.plist Payload/qq.app/Frameworks/*
// 再对可执行文件重签。
codesign -f -s "iPhone Distribution: xxx Technology Co., Ltd. " --entitlements entitlements.plist Payload/qq.app
// 5. 重新打包
zip -r qq.ipa Payload
/rm -rf Payload/
```
## 防抓包/防重签/防调试

### 防止抓包

HTTP 是明文传输，首先确保链接都是 HTTPS 的。

#### HTTPS 通信过程

1. 客户端将自己支持的加密算法发送给服务器，请求服务器证书；
2. 服务器选取一组加密算法，并将公钥证书返回给客户端；
3. 客户端校验证书合法性，生成随机对称密钥，用公钥加密后发送给服务器；
4. 服务器用私钥解密出对称密钥，返回一个响应，HTTPS 连接建立完成；
5. 随后双方通过这个对称密钥进行安全的数据通信。

#### 抓包过程分析

1. 客户端将自己支持的加密算法发送给服务器 A，请求服务器证书；这个过程被中间人的窃听，中间人把消息修改后发给了真正的目的地——服务端 B。
2. 服务端 A 收到了要建立 HTTPS 连接的请求后，原本会发送当时从证书签发机构签发的公钥证书。中间人又窃听了这个过程，于是替换上自己的假公钥证书后转发给了客户端。
3. 客户端收到了中间人发过来的假公钥证书，校验证书合法性，生成随机对称密钥，用假公钥加密后发给了中间人。那么中间人用假公钥对应的私钥就可以解出真正的对称密钥，中间人再用刚才服务端返回的公钥证书加密这个对称密钥，发给服务端 A。
4. 服务端 A 收到了当时用自己下发的公钥的证书加密的对称密钥，用自己的私钥解密，也得到了对称密钥。

#### 处理方式

1. 检测是否存在代理服务器

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

2. 不给代理服务器发消息

构造 AFHTTPSessionManager 对象的时候，将它的 NSURLSessionConfiguration 的 `connectionProxyDictionary` 对象（表示网络会话连接中的代理服务器）设为空字典。

```objc
+ (instancetype)sharedManager {
    static AFHTTPSessionManager *_singleton;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _singleton = [[AFHTTPSessionManager manager] initWithSessionConfiguration:[AFHTTPSessionManager setProxyWithConfig]];
        // other code ...
    });
    return _singleton;
}

+ (NSURLSessionConfiguration *)setProxyWithConfig {
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    config.connectionProxyDictionary = @{};
    return config;
}
```

但是两种方案都有破解方案：通过 class-dump 找到对应方法，如何通过动态库注入的方式，在运行时 hook 方法修改其返回逻辑，再对 ipa 包重签名后发布。

#### AFN + SSL Pinning

客户端取得服务端提供的 `.cer` 证书，加入到项目中。

然后设置 AFSSLPinningMode（AFSSLPinningModePublicKey/AFSSLPinningModeCertificate），来验证本地证书和请求使用的证书是否一致。

但是这种方案想要破解也很简单，App 砸壳后，再显示包内容，依然可以直接把证书放在抓包工具中使用。

### 防止重签名

在程序入口处，检测 App 包内的 embedded.mobileprovision 内的**组织名称**或 **teamid** 是否和我发布时签名时的信息相同，不相同就退出。

1. 关键字符串用函数替换，达到在二进制文件中查看不到的效果；
2. 关键函数名用C语言写，这样用Class-dump等工具查不出函数名;
3. 闪退的代码用汇编写，这样在编译后的二进制文件中找不到exit的关键字
4. 关键函数名用不规则字符串替换，达到混淆效果，别人更难破解（慎用，上架可能被拒）

拓展：

如果是 **Appstore 包**：`embedded.mobileprovision` 在 Appstore 上架时，Apple 会对上架的 App 包做签名验证，然后移除 embedded.mobileprovision 这个文件，并重签名。因此如果获取不到这个文件，认为此时是安全的即可。

如果获取并重签，那么一定会有 `embedded.mobileprovision` 文件，依然执行企业包那套判断逻辑即可。

### 防调试

使用了系统函数 `ptrace` 实现的。

我们执行 `ptrace`，让 `ptrace` 发送 `PT_DENY_ATTACH`，让父进程限制对子进程的调试进程附加，一旦检测到程序进程，应用程序就会退出。

`ptrace` 是用于进程跟踪，它提供了父进程可以观察和控制其子进程执行的能力。

基本原理：当使用了 `ptrace` 跟踪后，所有发送给被跟踪的子进程的信号(除了SIGKILL)，都会被转发给父进程，而子进程则会被阻塞，这时子进程的状态就会被系统标注为 `TASK_TRACED`。而父进程收到信号后，就可以对停止下来的子进程进行检查和修改，然后让子进程继续运行。

## 内存泄漏

> 总结文：[iOS 内存泄漏场景与解决方案](https://blog.fiteen.top/2020/ios-memory-leak)

### Block

block 是 `self` 的属性、block 内部又调用了 `self`，形成循环引用。

解决：

#### Weak-Strong-Dance

先用 `__weak` 将 `self` 置为弱引用，打破“循环”关系，但是 `weakSelf` 在 block 中可能被提前释放，因此还需要在 block 内部，用 `__strong` 对 `weakSelf` 进行强引用，这样可以确保 `strongSelf` 在 block 结束后才会被释放。

#### 断开持有关系

1. 用 `__block` 修饰的局部变量代替 self，并在调用结束时置为 `nil`，打破原本的循环持有链；

2. 将 self 以传参的形式传入 block 内部。

> 为什么要用 `__block` 修饰?
> 首先，block 本身不允许修改外部变量的值。但被 `__block` 修饰的变量会被存在了一个**栈的结构体**当中，成为结构体指针。当这个对象被 block 持有，就将“外部变量”在栈中的**内存地址**放到**堆**中，进而可以在 block 内部修改外部变量的值。
> 外部变量未被 `__block` 修饰时，block 数据结构中捕获的是外部变量的值，通过 `__block` 修饰时，则捕获的是对外部变量的指针引用。

### NSTimer

只要申请了 `timer`，加入了 runloop，并且 target 是 `self`，就算不是循环引用，也会造成内存泄漏，因为 self 没有释放的时机。

#### 在合适的实际销毁 NSTimer 对象

比如：VC 中的 `didMoveToParentViewController`、`viewDidDisappear`，View 中可以选择 `removeFromSuperview` 等。

```objc
[_timer invalidate];
_timer = nil;
```

但这种方案并一定是正确可行的，可能会造成倒计时被提前销毁，

比如：在注册页面中加了一个倒计时，如果在 `viewDidDisappear` 中销毁了 `_timer`，当用户点击跳转到用户协议页面时，倒计时就已经被销毁了。

#### 使用 GCD 定时器

```c
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

#### 借助中间者销毁

用别的对象代替 target 里的 `self`，中介者绑定 selector 之后，再在 dealloc 中释放 timer。

##### 以一个 NSObject 对象作为中介者

```objc
_target = [NSObject new];
class_addMethod([_target class], @selector(timeFire), class_getMethodImplementation([self class], @selector(timeFire)), "v@:");
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:_target selector:@selector(timeFire) userInfo:nil repeats:YES];
```

##### 以 NSProxy 的子类作为中介者

创建一个继承自 NSProxy 的子类 `WeakProxy`，将 `timer` 的 target 设置为 `WeakProxy` 实例，利用完整的消息转发机制实现执行 self 中的计时方法，解决循环引用。

```objc
// WeakProxy.h
@property (nonatomic, weak, readonly) id weakTarget;

+ (instancetype)proxyWithTarget:(id)target;
- (instancetype)initWithTarget:(id)target;

// WeakProxy.m
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

// 创建 timer
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0f target:[WeakProxy proxyWithTarget:self] selector:@selector(timeFire) userInfo:nil repeats:YES];
```

#### 带 block 的 timer

iOS 10 之后，Apple 给 NStimer 新增了一种带 block 的构造方法，这种方式可以解决循环引用的问题。

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));
```

为了兼容 iOS 10 之前的方法，可以写成 NSTimer 分类的形式，将 block 作为 SEL 传入初始化方法中，统一以 block 的形式处理回调。

```objc
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
```

### 委托模式

1. delegate 对象通常用 `weak` 修饰，来解决循环引用的问题。比如 UITableViewDelegate 对象。

2. 但也有特例，比如 NSURLSession 中的 NSURLSessionDelegate 对象就是用 `retain` 修饰的，为了确保在网络请求回调之前，delegate 对象不被释放。而 AFURLSessionManager 中的 NSURLSession 对象也是被强持有的，所以 AFNetworking 中出现了循环引用。

解决方案：

1) 将 AFHTTPSessionManager 对象设为单例（AFHTTPSessionManager 继承自 AFURLSessionManager）；
2) 在每次请求结束时，用 `invalidateSessionCancelingTasks` 方法手动销毁 session 对象。

### 非 OC 对象内存处理

1. CoreFoundation 框架下的 CI、CG、CF 等开头的类的对象，在使用完毕后仍需我们手动释放。

比如这段获取 UUID 的代码：

```c
CFUUIDRef puuid = CFUUIDCreate( kCFAllocatorDefault );
CFStringRef uuidString = CFUUIDCreateString( kCFAllocatorDefault, puuid );
NSString *uuid = [(NSString *)CFBridgingRelease(CFStringCreateCopy(NULL, uuidString)) uppercaseString];
// 使用完后释放 puuid 和 uuidString 对象
CFRelease(puuid);
CFRelease(uuidString);
```

2. C 语言中，用 malloc 动态分配的内存，需要用 `free` 去释放

```c
person *p = (person *)malloc(sizeof(person));
strcpy(p->name,"fiteen");
p->age = 18;
// 使用完释放内存
free(p);
// 防止野指针
p = NULL;
```

### 循环加载引起内存峰值

用 `@autoreleasepool` 处理，及时释放占用内存大的临时变量，减少内存占用峰值。
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

### 野指针与僵尸对象

对已经释放的对象发消息，会发生崩溃。当对象释放后，应该将其置为 nil。

### 内存检查工具

#### Instruments

- Leaks（动态检查泄漏的内存）、Xcode 中的 Analyze（静态检查内存）；
- Allocations（检查内存使用/分配情况）；
- Zombies（检查僵尸对象）。

#### MLeaksFinder + FBRetainCycleDetector

MLeaksFinder 的只能定位到内存泄漏的对象，FBRetainCycleDetector 可以检查对象是否存在循环引用。

**MLeaksFinder 原理**：

打个比方：当一个 `UIViewController` 被 `pop` 或 `dismiss` 后，该 `UIViewController` 包括它的 `view`，`view` 的 `subviews` 等将很快被释放（除非你把它设计成单例，或者持有它的强引用，但一般很少这样做）。于是，我们只需在一个 `ViewController` 被 `pop` 或 `dismiss` 一小段时间后，看看该 `UIViewController`，它的 `view`，`view` 的 `subviews` 等等是否还存在。

所以具体的方法是，为基类 `NSObject` 添加一个方法 `-willDealloc` 方法，该方法的作用是，先用一个弱指针指向 `self`，并在一小段时间(2s)后，通过这个弱指针调用 `-assertNotDealloc`，而 `-assertNotDealloc` 主要作用是弹出 alert 框。

## 离屏渲染

离屏渲染（Offscreen-Renderd）：GPU（图形处理器）在当前屏幕缓冲区外新开辟一个渲染缓冲区进行工作。这会给我们带来额外的性能损耗，如果这样的操作达到一定数量，会触发缓冲区的频繁合并和上下文的的频繁切换，会出现卡顿、掉帧现象。可以通过 Core Animation 工具检测（Debug -> View Debugging -> Rendering）

造成离屏渲染的原因有很多，如：cornerRadius（圆角）、mask（遮罩层）、shadows（阴影）、shouldRasterize（光栅化）等等。

下面这些情况/操作会引发离屏渲染：

- 为图层设置遮罩（layer.mask）
- 将图层的 layer.masksToBounds / view.clipsToBounds 属性设置为true
- 将图层 layer.allowsGroupOpacity属性设置为YES和layer.opacity小于1.0
- 为图层设置阴影（layer.shadow）
- 为图层设置 layer.shouldRasterize=true
- 具有 layer.cornerRadius，layer.edgeAntialiasingMask，layer.allowsEdgeAntialiasing 的图层
- 文本（任何种类，包括UILabel，CATextLayer，Core Text 等）。
- 使用 CGContext在drawRect :方法中绘制大部分情况下会导致离屏渲染，甚至仅仅是一个空的实现。

### 圆角

**方案一：使用贝塞尔曲线 UIBezierPath 和 Core Graphics 框架画出一个圆角。**

```objc
UIImageView *imageView = [[UIImageView alloc]initWithFrame:CGRectMake(100, 100, 100, 100)]; 
imageView.image = [UIImage imageNamed:@"myImg"]; 
// 开始对imageView进行画图 
UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 1.0); 
// 使用贝塞尔曲线画出一个圆形图 
[[UIBezierPath bezierPathWithRoundedRect:imageView.bounds cornerRadius:imageView.frame.size.width] addClip];
[imageView drawRect:imageView.bounds];
imageView.image = UIGraphicsGetImageFromCurrentImageContext(); 
// 结束画图 
UIGraphicsEndImageContext();
[self.view addSubview:imageView];
```

**方案二：使用 CAShapeLayer 和 UIBezierPath 设置圆角。**（依然有离屏渲染的问题、但减少掉帧）

```objc
UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(100,100,100,100)];
imageView.image = [UIImage imageNamed:@"myImg"];
UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:imageView.bounds byRoundingCorners:UIRectCornerAllCorners cornerRadii:imageView.bounds.size];
CAShapeLayer *maskLayer = [[CAShapeLayer alloc] init];
// 设置大小
maskLayer.frame = imageView.bounds;
// 设置图形
maskLayer.path = maskPath.CGPath;
imageView.layer.mask = maskLayer;
[self.view addSubview:imageView];
```

方案二优于方案一：CAShapeLayer 动画渲染直接提交到手机的 GPU 当中，相较于 view 的 drawRect 方法使用 CPU 渲染而言，其效率极高，能大大优化内存使用情况。

**方案三：直接覆盖一张中间为圆形透明的图片。**

### 其它

- 使用 ShadowPath 指定 layer 的阴影效果路径，优化性能；
- 使用异步进行 layer 渲染（Facebook 开源的异步绘制框架 AsyncDisplayKit）；
- 尽量使用不包含透明（alpha）通道的图片资源；
- 如果 alhpa不等于1，设置 layer 的 opaque 值为 YES，减少复杂图层合成；
- 尽量设置 layer 的大小值为整数值。

## JSBridge 原理和实现

### 结构

- JSBridgeCallBackModel：存放消息通知的数据结构（callbackId、data（msg、code、args））
- JSBridgeApi：声明注册 API
- JSBridgeWebView：实现代理方法，处理注册方法等
- JSBridgeUtil：工具类方法：JSON 数据的序列化和反序列化、获取方法名、解析命名空间

### 原理

- 分别在 OC 环境和 javascript 环境都保存一个 bridge 对象，里面维持着 callbackId，以及每个 id 对应的具体实现。
- OC -> JS：调用 `stringByEvaluatingJavaScriptFromString` 方法，执行 JS 中注册的方法。
- JS -> OC：javascript 通过执行 prompt / alert 方法来触发 webview 的私有方法（重写 prompt），从而实现把 javascript 消息发送给 OC。

实现 UIWebViewDelegate/ WKUIDelegate 的三个方法：

```objc
// 解析 prompt，并实现相关的方法
- (NSString *)webView:(UIWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)text initiatedByFrame:(id)frame;
// 响应 JS 发送的 alert 消息
- (void)webView:(UIWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame;
// return YES; 即可
- (BOOL)webView:(UIWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame;
```

### WebViewJavascriptBridge

借助 bridge，完成 JS 和 OC 的交互。

#### 原理

- 分别在 OC 环境和 javascript 环境都保存一个 bridge 对象，里面维持着 responseId、callbackId，以及每个 id 对应的具体实现。
- OC 通过 javascript 环境的 window.WebViewJavascriptBridge 对象来找到具体的方法，然后执行。
- javascript 通过改变 iframe 的 src 来触发 webview 的代理方法（拦截 URL）从而实现把 javascript 消息发送给 OC。

#### 框架

- WebViewJavascriptBridge_JS：JS 文件，需要客户端在网页初始化的时候注入到网页中去；
- WebViewJavascriptBridge/WKWebViewJavascriptBridge：客户端和 H5 交互的桥梁；
- WebViewJavascriptBridgeBase：数据处理工具类
  - 客户端注册的方法以及对应实现：`@property (strong, nonatomic) NSMutableDictionary* messageHandlers;`
  - 客户端通知网页后的回调实现：`@property (strong, nonatomic) NSMutableDictionary* responseCallbacks;`
  - 网页js环境注入方法：`-(void)injectJavascriptFile;`
  - 处理从 native 到 JS 的消息注入，消息队列处理和分发
  - JSON 数据的序列化和反序列化
  - LOG 输出

### 实现

以网页通知客户端为例：客户端会将要被调用的方法存到字典中，同时拦截网页的调用，当网页调用时，从字典中取出方法并调用。调用完后，判断网页是否有调用完的回调，如果有，再将回调的 id 和参数通过客户端调用网页的方式通知过去。这就完成了网页通知客户端的总体流程。

## MVC & MVVM

### MVC

#### MVC 是什么

- View：展示用户界面；
- Model：存储数据数据；
- Controller：处理业务逻辑。

Controller 是将 Model 与 View 连接起来协同工作的部分，它能访问 Model 和 View，Model 和 View 不能互相访问。

##### Model 与 Controller 通信

1. Controller 可以直接向 Model 请求数据，一般将 Model 的类作为 Controller 的属性，直接调用相应的实例方法或者类方法；
2. Model 可以通过 Notification 或 KVO 来通知 Controller 更新 View。

##### View 与 Controller 通信

1. Controller 可以直接操作 View，比如通过 outlet 等
2. View 需要处理一些特殊 UI 逻辑或获取数据时，通过 delegate 或 data source 方式交给 Controller 来处理。

#### 缺点

随着业务增长，Controller 会越来越臃肿，很难维护。

### MVVM

#### MVVM 是什么

MVVM 就是在 MVC 的基础上分离出业务处理的逻辑 ViewModel 层。

- Model： API 请求的原始数据，数据持久化；
- View：视图展示，由 ViewController 来控制；
- Controller：不再直接和 Model 进行绑定，而是通过 ViewModel 进行持有真实的 Model。
- ViewModel：负责网络请求，业务处理和数据转化。

#### 优点

1. 低耦合；
2. 可重用；
3. 独立开发。

#### 缺点

1. 类增多。每个 VC 都附带一个 ViewModel；
2. 数据绑定使得 Bug 很难被调试；
3. 调用复杂度增加。

## RAC

ReactiveCocoa（简称 RAC），一款由 GitHub 开源的应用于 iOS 和 OS X 的框架，具有**函数式编程**和**响应式编程**的特性。

> 函数式编程：嵌套的函数，将操作尽可能写在一起。（本质：往方法里面传入 block，方法中嵌套 block 调用。）</br>
> 响应式编程：一种面向数据流和变化传播的编程范式。</br>
> 链式编程：方法的返回值必须是方法的调用者。

RAC 通过引入信号（Signal）的概念，代替传统 iOS 开发中对于控件状态变化检查的代理模式或 target-action 模式。

用 RAC 将 ViewModel 与 View 关联，View 层的变化可以直接响应 ViewModel 层的变化，这使得 Controller 变得更加简单。

RAC 有一对一的单向数据流 RACSignal 和一对一的双向数据流 RACChannel。

RACChannel 可以被理解为一个双向的连接，这个连接的两端都是 RACSignal 实例，它们可以向彼此发送消息。

### 三个步骤：

1. 创建信号：创建信号对象，然后创建一个可变数组
2. 订阅信号：创建一个订阅者，将 block 保存到订阅者中，将订阅者保存到上面的数组中
3. 发送信号：遍历信号对象中的数组，取出订阅对象，调用订阅对象中的 block 执行

### 信号依赖

```objc
  //这相当于网络请求中的依赖,必须先执行完信号A才会执行信号B
  //经常用作一个请求执行完毕后,才会执行另一个请求
  //注意信号A必须要执行发送完成信号,否则信号B无法执行
  RACSignal * concatSignal = [self.signalA concat:self.signalB];

  //这里我们是对这个拼接信号进行订阅
  [concatSignal subscribeNext:^(id x) {
      NSLog(@"%@",x);
  }];
```

### RAC 冷热信号问题

**热信号**是主动的，即使你没有订阅事件，它仍然会时刻推送。而**冷信号**是被动的，只有当你订阅的时候，它才会发送消息。

**热信号**可以有多个订阅者，是一对多，信号可以与订阅者共享信息。而冷信号只能一对一，当有不同的订阅者，消息会从新完整发送。

RacSignal 就是冷信号，RacSubject 父子是热信号。

### RAC throttle 的作用

每次发送的数据，都经过 interval 的间隔之后才发出。在 interval 时间内发送的所有信号只有最后一个数据被发送，前面的都会被抛弃。

例如实现搜索框实时搜索的功能，即用户每每输入字符，view 都要求展现搜索结果。这时如果用户搜索的字符串较长，那么由于网络请求的延时可能造成UI显示错误，并且多次不必要的请求还会加大服务器的压力，这显然是不合理的，此时我们就需要用到**节流**。

```objc
- (RACSignal*)throttle:(NSTimeInterval)interval;
```

### RAC 实战

基本控件的事件（UITextField 输入、按钮点击、计数器）、监听属性变化、遍历数组和字典、监听通知、代替代理/KVO 等。

## 直播

### 流程

> 采集（屏幕采集、摄像头采集、可拓展采集） —> 处理（美颜、水印、滤镜、声音特效、可拓展处理） —> 编码（音视频压缩）和封装 —> 推流到服务器 —> 服务器流分发 —> 播放器流播放（拉流（ijkplayer）、解码、 渲染）

### 鉴权

URL 鉴权功能是通过阿里云 CDN 加速节点与客户资源站点配合实现的一种更为安全可靠的源站资源防盗方法。由客户站点提供给用户加密 URL（包含权限验证信息），用户使用加密后的 URL 向加速节点发起请求，加速节点对加密 URL 中的权限信息进行验证以判断请求的合法性，对合法请求给予正常响应，拒绝非法请求，从而有效保护客户站点资源。

鉴权仅会在推流或者播流开始的时候进行验证，在推流或者播流过程中即不会验证，也就是说推流或者播流过程中如果超过了鉴权时间戳也可以继续播放。

# 基础

## OC 的动态特性

动态：主要是将数据类型的确定由编译时，推迟到了运行时。之所以叫做动态。

OC 的三大动态特性：

- **动态类型**：id 类型，对应静态类型（指定类型）
- **动态绑定**：消息机制
- **动态加载**：根据需求加载所需要的资源，例如不同机型适配，加载图片（1x/2x/3x）


## autoreleasepool

### autorelease 原理

autorelease 实际上只是把 release 的调用延迟了，对于每个 autorelease，系统只是把该对象放入了自动释放池中。自动释放池被销毁或者耗尽时，会向自动释放池中的所有对象发送 release 消息，释放自动释放池中的所有对象。

所有 autorelease 对象，出了作用域，就会自动添加到最近创建的自动释放池中。

### autorelease 对象的释放

- 手动释放 autoreleasepool
- RunLoop 结束后自动释放

### autoreleasepool 如何实现

autoreleasepool 以一个队列数组的形式实现，主要通过下列三个函数完成：

- `objc_autoreleasePoolPush`
- `objc_autoreleasePoolPop`
- `objc_autorelease`

### CFRunLoopObserver 与 Autorelease Pool

RunLoop 负责管理自动释放池.

- CFRunLoopObserver 监视到 kCFRunLoopEntry(将要进入 RunLoop/**启动** RunLoop)的时候，会调用 `_objc_autoreleasePoolPush()` （第一次）创建自动释放池。
- CFRunLoopObserver 监视到 kCFRunLoopBeforeWaiting(**即将休眠**) 时调用 `_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()`释放旧的池并创建新池。
- kCFRunLoopExit(即将**退出**Loop) 时会调用 `_objc_autoreleasePoolPop()` 来（最后一次）释放自动释放池。

### 手动添加 autoreleasepool 的场景

- 编写的程序不是基于 UI 框架的，比如说命令行工具；
- 编写的循环中创建了大量的临时对象；
- 创建了一个辅助线程（**自定义的 NSOperation 和 NSThread 需要手动创建自动释放池**）：比如：自定义的 NSOperation 类中的 main 方法就必须添加自动释放池，否则出了作用域，自动释放对象会因为没有自动释放池去处理它，而造成内存泄漏。但对于 NSBlockOperation 和 NSInvocationOperation 这种默认的 Operation，系统已经帮我们封装好了，不需要手动创建自动释放池。

### 重载 +load 时

重载 +load时不需要手动添加 `@autoreleasepool`吗，在 runtime 调用 `+load` 方法前后是加了 `objc_autoreleasePoolPush()` 和 `objc_autoreleasePoolPop()` 的。

## load & initialize

### load 方法调用时机

`+load` 方法会在 runtime 加载类、分类时调用，在程序运行过程中只调用一次。并且只要这个类的符号被编译到最后的可执行文件中，`+load` 方法就会被调用。

调用顺序：

1. 先调用类的 `+load`；
  - 按照编译先后顺序调用（先编译、先调用）
  - 调用子类的 `+load` 之前会先调用父类的 `+load`（但不需要手动调用 super，系统会自动调）
2. 再调用分类的 `+load`；
  - 按照编译先后顺序调用（先编译，先调用）

### initialize 方法的调用时机

initialize 在类第一次接收到消息时调用，也就是 `objc_msgSend()`。

先调用父类的 `+initialize`，再调用子类的`initialize`。

## runtime

Runtime 是指将数据类型的确定由**编译时**推迟到了**运行时**。它是一套底层的纯 C 语言 API，我们平时编写的 Objective-C 代码，最终都会转换成 runtime 的 C 语言代码。

### 消息传递

在 Objective-C 中，所有的消息传递中的“消息”都会被编译器转化为：

```
id objc_msgSend ( id self, SEL op, ... );
```

比如执行一个对象的方法：`[obj foo];`，底层运行时会被编译器转化为：`objc_msgSend(obj, @selector(foo));`。

![消息传递流程](https://blog.fiteen.top/2020/ios-runtime/message-send.png)

- 先从当前 class 的 cache 方法列表里去查找。
- 如果找到了，如果找到了就返回对应的 IMP 实现，并把当前的 class 中的 selector 缓存到 cache 里面。
- 如果类的方法列表中找不到，就到父类的方法列表中查找，一直找到 NSObject 类为止。
- 最后再找不到，就会进入动态方法解析和消息转发的机制。

> **category 是怎么将方法添加到类上去的?**
> 
> 程序运行的时候，分类中的方法列表会加入到类的方法列表当中。

### 消息转发

如果消息传递后仍无法找到 IMP，就进入了消息转发流程。

1. 通过运行期的动态方法解析功能，我们可以在需要用到某个方法时再将其加入类中。
2. 对象可以把其无法解读的某些选择子转交给备用接受者来处理。
3. 经过上述两步之后，如果还是没有办法处理选择子，那就启动完整的消息转发机制。

![消息转发流程](https://blog.fiteen.top/2020/ios-runtime/message-forwarding.png)

如果消息转发流程结束，仍无法找到 IMP，则会执行 `doesNotRecognizeSelector` 方法。

### 实践

#### 埋点

添加 UIViewController 分类，hook 每个控制器中需要埋点的方法，在 load 方法中进行方法交换(`method_exchangeImplementations`），加入埋点请求。

#### 下拉刷新

创建一个 scrollView 的分类，通过 runtime 动态（`objc_setAssociatedObject`、`objc_getAssociatedObject`）给 scrollView 的头部和尾部加入两个刷新视图。利用 KVO 监听滚动视图的偏移量（contentOffset）和区域（contentSize），也就是滚动进度，根据不同的进度来设置控件的相应文字和图片动画。

#### 自定义导航栏

给 UIViewController 添加 navigationBar 分类，使控制器直接对 bar 视图进行修改。

#### UITableView 占位符

监听 cell 的数量，如果为零，通过 tableView 的分类，给 tableView 动态添加 placeholder 自定义视图。

## RunLoop

### 定义

RunLoop 是一个事件循环**对象**。它内部是 do-while 循环，这也是 RunLoop 运行的本质。在循环内部会不断处理各种任务(比如 Source、Timer、Observer)。

RunLoop 是 iOS 事件响应与任务处理最核心的机制，它贯穿 iOS 整个系统。

CFRunLoop 的结构体：

```
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;			/* locked for accessing mode list */
    __CFPort _wakeUpPort;			// used for CFRunLoopWakeUp 
    Boolean _unused;
    volatile _per_run_data *_perRunData;              // reset for runs of the run loop
    pthread_t _pthread;
    uint32_t _winthread;
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes;
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFAbsoluteTime _runTime;
    CFAbsoluteTime _sleepTime;
    CFTypeRef _counterpart;
};
```

1. 一个 RunLoop 对应一条线程，因为对象中只有一个 `pthread_t` 对象；（但一个线程可以开启多个 RunLoop，只不过都是嵌套在最大的 RunLoop 中）
2. 同一时刻，RunLoop 只能在一个 mode 上运行，如果需要切换 mode，需要退出 currentMode，切换到指定的 mode。
3. RunLoop 有多种模式，用的比较多的是：
  - `kCFRunLoopDefaultMode` (`NSDefaultRunLoopMode`)，App 默认 mode，通常主线程在这个 mode 下
  - `UITrackingRunLoopMode`，界面跟踪 mode，用于滚动视图追踪触摸滑动，保证滑动时不受其他 mode 影响
  - `kCFRunLoopCommonModes`（`NSRunLoopCommonModes`），伪模式，kCFRunLoopDefaultMode 和 UITrackingRunLoopMode 的集合
4. 每个 mode 可以包含多个事件（source）
  - `CFMutableSetRef _sources0`，处理的是 App 内部的事件、App 自己负责管理，如按钮点击事件等。
  - `CFMutableSetRef _sources1`，由 RunLoop 和内核管理，Mach Port驱动，如 CFMachPort、CFMessagePort
  - `CFMutableArrayRef _observers`
  - `FMutableArrayRef _timers`

### 核心源码

[CFRunloop 源码](http://opensource.apple.com/source/CF/CF-855.17/CFRunLoop.c)

1. 通知观察者即将进入 runloop。
2. 通知观察者即将处理 timer 事件。
3. 通知观察者即将处理 source0（不基于 port 的输入源）事件。
4. 处理 source0（不基于 port 的输入源） 事件。
5. 如果有 source1 (基于 port 的输入源) 处于 ready 状态，直接处理这个 source1。转到步骤9。
6. 通知观察者线程即将休眠。
7. 使线程进入睡眠状态，直到发生以下事件之一：
  - source0 事件触发。
  - timer 事件触发。
  - 为 runloop 设置的超时值到期。
  - runloop 被外部手动唤醒。
8. 通知观察者线程被唤醒。
9. 处理唤醒时收到的消息。
  - 如果触发了 timer 事件，请处理 timer 事件并重启 runloop。转到步骤2。
  - 如果触发了 source0/source1，请传递事件。
  - 如果 runloop 已显式唤醒，但尚未超时，请重启 runloop。转到步骤2。
10. 通知观察者 runloop 即将退出。

底层实现函数：

```c
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled){
    // 配置 RunLoop 的 mode
    SetupCFRunLoopMode();
    // 通知 Observers 将要进入 Loop
    __CFRunLoopDoObservers(kCFRunLoopEntry);
    // 通过 GCD 设置 RunLoop 的超时时间
    SetupThisRunLoopRunTimeoutTimer();
    // RunLoop 开始处理事件  do while 循环
    do {
        // 通知 Observers 将执行 timer
        __CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
        // 通知 Observers 将执行 source0
        __CFRunLoopDoObservers(kCFRunLoopBeforeSources);
        // 执行 blocks
        __CFRunLoopDoBlocks();
        // 执行 source0
        __CFRunLoopDoSource0();
        // 问 GCD 主线程有没有需要执行的东西
        CheckIfExistMessagesInMainDispatchQueue();
        // 通知 Observers 将进入睡眠
        runtime(kCFRunLoopBeforeWaiting);
        /* 指定 唤醒端口
        监听 mach_msg 会停在这里
        进入 mach_msg_trap 状态
        */
        var wakeUpPort = SleepAndWaitForWakingUpPorts();
        // 接收到 消息  通知 Observers RunLoop被唤醒了
        __CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
        // 处理事件
        if (wakeUpPort == timerPort) {
            // 唤醒端口是 timerPort 执行 timer 回调 /* DOES CALLOUT */
            __CFRunLoopDoTimers();
        } else if (wakeUpPort == mainDispatchQueuePort) {
            // 唤醒端口 执行 mainQueue 里面的调用
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__();
        } else {
            // 唤醒端口 执行 source1回调
            __CFRunLoopDoSource1();
        }
        // 执行 blocks
        __CFRunLoopDoBlocks()；
    // 当事件处理完了、被强制停止了、超时了、mode 是空的时候就会退出循环
    } while (!stop && isStopped !timeout && !ModeIsEmpty );
    // 通知 Observers 将退出 Loop
    __CFRunLoopDoObservers(kCFRunLoopExit);
}
```

### 线程安全

Core Foundation中：`CFRunLoopRef` 对象，它提供了纯 C 函数的 API，并且这些 API 是**线程安全**的；

Foundation 中：用 `NSRunLoop` 对象来表示，它是基于 CFRunLoopRef 的封装，提供的是面向对象的 API，但这些 API **不是线程安全**的。

### 作用

1. 使程序一直运行并接收用户的输入；
2. 决定程序在何时处理哪些事件；
3. 调用解耦（主调方产生很多事件，不用等到被调方处理完事件之后，才能执行其他操作）
4. 节省 CPU 时间（当程序启动后，什么都没有执行的话，就不用让 CPU 来消耗资源来执行，直接进入睡眠状态）

### CFRunLoopTimer 的封装

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:    (NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;
+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;
```

### CFRunLoopObserver 的应用举例

CFRunLoopObserver 作用：告知外界 RunLoop 状态的更改。它的状态有：

```c
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    // 进入RunLoop开始跑了
    kCFRunLoopEntry = (1UL << 0),
    // 将要执行timer了
    kCFRunLoopBeforeTimers = (1UL << 1),
    // 将要执行Source了
    kCFRunLoopBeforeSources = (1UL << 2),
    // 将要进入睡眠
    kCFRunLoopBeforeWaiting = (1UL << 5),
    // 被唤醒
    kCFRunLoopAfterWaiting = (1UL << 6),
    // 退出
    kCFRunLoopExit = (1UL << 7),
    // 全部的状态
    kCFRunLoopAllActivities = 0x0FFFFFFFU
}
```

#### CFRunLoopObserver 与 Autorelease Pool

RunLoop 负责管理自动释放池；

CFRunLoopObserver 监视到 kCFRunLoopEntry(将要进入 RunLoop/启动 RunLoop)的时候，会调用 `_objc_autoreleasePoolPush()` （第一次）创建自动释放池。

CFRunLoopObserver 监视到 kCFRunLoopBeforeWaiting(将要进入休眠) 时调用 `_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()`释放旧的池并创建新池。

kCFRunLoopExit(即将退出Loop) 时会调用 `_objc_autoreleasePoolPop()` 来（最后一次）释放自动释放池。

#### 重绘视图

苹果为了保证界面的流畅性，（1）不会重绘属性（frame等）没有改变的视图（2）只发送一次 `drawRect:` 消息。

当相关的视图对象接收到设置属性的消息的时候，就会将自己标记为要重绘。

RunLoop 会收集所有等待重绘制的视图，苹果会注册一个 CFRunLoopObserver 来监听 kCFRunLoopBeforeWaiting 事件，当事件触发的时候，就会对所有等待重绘的视图对象发送 `drawRect:`消息。

### RunLoop 的挂起和唤醒

当 RunLoop 处于空闲状态或者点击了暂停的时候，就被挂起，具体步骤：

1. 指定用于再次唤醒的端口（mach_port）

2. 调用 `mach_msg` 监听唤醒端口。内核调用 `mach_msg_trap` 让 RunLoop 处于 `mach_msg_trap` 状态，RunLoop 就会挂起，等待激活。就像一段代码中有scanf函数，必须要接收一个输入一样，不输入就不会继续往下执行。这里要区别于sleep。或者像是 Notification,当有 post 的时候，才会被唤醒。
3. 由另一线程（或者另一个进程中的某个线程）向内核发送这个端口的 msg 后，trap 状态就会被唤醒，RunLoop 就继续工作

### GCD 和 RunLoop

在RunLoop的内部实现中，用到了很多 GCD 的东西。比如刚刚开始 run 的时候，通过 DISPATCH_SOURCE_TYPE_TIMER 该类型的 dispatch_source 设置了 RunLoop 的超时时间。还可以在上面 RunLoop 实现的伪代码中看到 `__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()`，只要是 dispatch 到 main_queue 的，Core Foundation 都会调用这个函数，之后，libdispatch.dylib 就会执行回调。

### 应用实践

#### NSTimer

NSTimer 定时器的触发正是基于 RunLoop 运行的，所以使用 NSTimer 之前必须注册到 RunLoop。

RunLoop 为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行。

但是 NSTimer 提供了一个 `tolerance` 属性用于设置宽容度，如果确实想要使用 NSTimer 并且希望尽可能的准确，则可以设置此属性。

NSTimer 的创建通常有两种方式，尽管都是类方法，一种是 timerWithXXX，另一种 scheduedTimerWithXXX。

后者除了创建一个定时器外会自动以 NSDefaultRunLoopModeMode 添加到当前线程 RunLoop 中，但前者需要手动添加。

#### 在滚动视图中延迟加载图片

当图片在 tracking mode 下加载时，会出现卡顿，所以把它指定在 defalut mode 下，等待滚动结束再加载。

```objc
[cell performSelector:@selector(setImage) withObject:nil afterDelay:0 inModes:@[NSDefaultRunLoopMode]];
```

#### 滚动视图中的倒计时控件

倒计时默认加在 default mode 下，如果想让它在滚动时继续执行 timer 事件，需要把他加在 common mode 下。

```objc
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```

#### 让 crash 的 App 回光返照

App崩溃的发生分两种情况：

(1) program received signal:SIGABRT SIGABRT 一般是过度 release 或者发送 unrecogized selector 导致。

(2) EXC_BAD_ACCESS 是访问已被释放的内存导致,野指针错误。

由 SIGABRT 引起的 Crash 是系统发这个 signal 给 App，程序收到这个 signal 后，就会把主线程的 RunLoop 杀死，程序就 Crash 了 该例只针对 SIGABRT 引起的 Crash 有效。

```c
// 创建 RunLoop
CFRunLoopRef runLoop = CFRunLoopGetCurrent();
// 获取当前所有的 modes
NSArray *allModes = CFBridgingRelease(CFRunLoopCopyAllModes(runLoop));
while (1) {
    for (NSString *mode in allModes) {
        // 快速的切换 mode 就能处理滚动、点击等事件
        CFRunLoopRunInMode((CFStringRef)mode, 0.001, false);
    }
}
```

#### AFNetworking 中 RunLoop 的创建

在 AFN 中当使用 NSURLConnection 去执行网络操作的时候，会遇到还没有收到服务器的回调，线程就已经退出了。为了解决这一问题，作者使用到了 RunLoop。下面是 AFN 中的一段代码：

```objc
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
       [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```

## crash 场景

- unrecognized selector crash
- KVO crash
- NSNotification crash
- NSTimer crash
- Container crash（数组越界，插nil等）
- NSString crash （字符串操作的crash）
- Bad Access crash （野指针）
- UI not on Main Thread Crash (非主线程刷UI(机制待改善))

## block

block 本质上也是一个 OC 对象，他内部也有一个 isa 指针。block 是封装了函数调用以及函数调用环境的 OC 对象。

```c
// block的结构体
struct __main_block_impl_0 {
  // 结构体的成员变量
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  
  // 构造函数
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

### block 变量捕获

- 全局变量   直接访问
- 局部变量
  - auto   值传递
  - static 指针传递

### block 类型

block 的类型 NSBlock 最终也是继承自 `NSObject`。

- __NSGlobalBlock__：数据区，没有访问 auto 变量
- __NSStackBlock__：栈区，访问了 auto 变量
- __NSMallocBlock__：堆区，`__NSStackBlock__` 调用了copy

1. 程序区域：代码段，用于存放代码
2. 数据区域：数据段，用于存放全局变量
3. 堆：动态分配内存，需要程序员自己申请，程序员自己管理，通常是 `alloc` 或者 `malloc` 方式申请的内存
4. 栈：用于存放局部变量，系统会自动分配内存，自动销毁内存

### `__block` 的作用

`__block` 会将变量包装成对象，然后在把值封装在结构体里面，block 内部存储的变量为结构体指针。

在 ARC 环境下，编译器会根据情况自动将栈上的 block 复制到堆上，类似以下情况：

- block 作为函数返回值时，编译器自动完成 copy 操作；
- 将 block 赋值给通过 strong 或 copy 修饰的 id 或 block 类型的成员变量时；
- block 作为 GCD 的方法参数时。

## KVC & KVO

### KVC

键值编码，可以用来访问并修改一个类的私有属性。

#### KVC 的底层实现

当一个对象调用 `setValve` 方法时，方法内部会做以下操作：

 - 检查是否存在相应 `key` 的 set 方法，存在就调用 set 方法；
 - set 方法不存在，就查找 `_key` 成员变量是否存在，存在就直接赋值
 - 如果 `_key` 没找到，就查找相同名称的属性 `key`，存在就赋值
 - 如果都没有找到，就调用 `valveForUndefineKey` 和 `setValue: forUndefinedKey`

### KVO

键值观察者模式，提供观察某一属性变化的方法。

#### 简述 KVO 的实现

当观察某对象 A 时，KVO 机制动态创建一个新的名为：`NSKVONotifying_A` 的新类，该类继承自对象 A 的本类，且 KVO 为 `NSKVONotifying_A` 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前（`-willChangeValueForKey:`）和之后（`-didChangeValueForKey:`），通知所有观察对象属性值的更改情况。
最后把这个对象的 isa 指针 (isa 指针告诉 runtime 系统这个对象的类是什么) 指向新创建的子类。

此外，Apple 还重写了 `-class` 方法，企图让我们以为我们这个类没有变，就是原本那个类。

#### 如何手动通知 KVO

重写被观察对象的 `automaticallyNotifiesObserversForKey` 方法，返回 `NO`。

要实现手动通知，需要手动在赋值前后添加 `willChangeValueForKey` 和 `didChangeValueForKey`，才可以收到观察通知。

#### 通过 KVC 修改属性会触发 KVO

会。

#### KVO 什么时候会崩溃，如何防护

1. `removeObserver` 一个未注册的 keyPath，导致错误：`Cannot remove an observer A for the key path "str"，because it is not registered as an observer.`

**解决办法**：根据实际情况，增加一个添加 keyPath 的标记，在 `dealloc` 中根据这个标记，删除观察者。

2. 添加的观察者已经销毁，但是并未移除这个观察者，当下次这个观察的 `keyPath` 发生变化时，KVO 中的观察者的引用变成了野指针，导致 crash。

**解决办法**：在观察者即将销毁的时候，先移除这个观察者。

其实还可以将观察者 observer 委托给另一个类去完成，这个类弱引用被观察者，当这个类销毁的时候，移除观察者对象，参考[KVOController](https://www.bbsmax.com/A/lk5avNZPd1/)。

#### KVO 优缺点

**优点**：

1. 能够提供一种简单的方法实现两个对象间的同步。例如：model 和 view 之间同步；
2. 能够对非我们创建的对象，即内部对象的状态改变作出响应，而且不需要改变内部对象（SKD对象）的实现；
3. 能够提供观察的属性的最新值以及先前值；
4. 用 keyPath 来观察属性，因此也可以观察嵌套对象；
5. 完成了对观察对象的抽象，因为不需要额外的代码来允许观察值能够被观察。

**缺点**：

1. 我们观察的属性必须使用 strings 来定义。因此在编译器不会出现警告以及检查；
2. 对属性重构将导致我们的观察代码不再可用；
3. 复杂的“IF”语句要求对象正在观察多个值。这是因为所有的观察代码通过一个方法来指向；
4. 当释放观察者时不需要移除观察者。
5. 只能通过重写 -observeValueForKeyPath:ofObject:change:context:方法来获得通知；
6. 不能通过指定 selector 的方式获取通知；
7. 不能通过 block 的方式获取通知。

## Notification & delegate

观察者模式，发送者和接受者的关系是**间接**、**多对多**的关系。

发送者和接受者的关系是**直接**、**一对一**的关系。

- delegate 效率比 Notification 高；
- delegate 比 Notification 更直接、需要关注返回值，常带有 should 关键词；Notification 不关心结果，常带有 did 关键词。

## 内存管理

ARC 所做的就是在代码编译器自动在合适的地方插入 release 或 autorelease，**只要没有强指针指向对象，对象就会被释放**。ARC 中不能手动使用 NSZone，也不能调用父类的 dealloc。

> 注意：release 只会让对象的引用计数 -1，当对象的引用计数器 = 0 时，会调用对象的 dealloc 方法才能释放对象的内存。

- `alloc`、`new`：类初始化方法，开辟新的内存空间，引用计数+1；
- `retain`：实例方法，不会开辟新的内存空间，引用计数+1；
- `copy`： 实例方法，把一个对象复制到新的内存空间，新的内存空间引用计数+1，旧的不会；其中分为浅拷贝和深拷贝，浅拷贝只是拷贝地址，不会开辟新的内存空间；深拷贝是拷贝内容，会开辟新的内存空间；
- `strong`：强引用； 引用计数+1；
- `release`：实例方法，释放对象；引用计数-1；
- `autorelease`： 延迟释放；autoreleasepool 自动释放池；当执行完之后引用计数-1；
- `initWithFormat` 和 `stringWithFormat` 方式创建的字符串在堆上时，引用计数+1；
- `assign` 和 `weak` 都是弱引用，不影响引用计数；

### 内存管理机制

通过**引用计数器**（retainCount）机制来决定对象是否需要释放。每次 RunLoop 完成一个循环的时候，都会检查对象的 retainCount，如果 retainCount 为 0，说明该对象没有地方需要继续使用来，就会被释放了。

只要没有强指针指向对象，对象就会被释放。

### 内存管理范围

管理所有继承自 NSObject 的对象，对基本数据类型无效，是因为对象和其它基本数据类型在系统中的存储空间不一样。其它局部变量主要存储在栈区，而对象存储于堆区，当代码块结束时，这个代码块中的所有局部变量会自动弹栈清空，指向的对象的指针也会被回收，这时对象就没有指针指向了，但依然存在于堆中，造成内存泄漏。

### 内存管理研究的对象

- 野指针：指针变量没有进行初始化或指向的空间已经被释放
  - 使用野指针调用对象方法，会报异常，程序崩溃
  - 在调用完 release 后，把保存对象指针的地址清空，赋值为 nil，找 OC 中没有空指针异常，所以 `[nil retain]` 调用方法不会有异常

- 僵尸对象：堆中已经被释放的对象（retainCount = 0）
- 空指针：指针赋值为空，nil
- 内存泄漏：如 `Person *p = [Person new];`（对象提前赋值 nil 或清空），在栈区的 p 已经释放，而堆区 new 产生的对象还没有释放，会造成内存泄漏。

## NSThread

创建方式：

```objc
- (instancetype)initWithTarget:(id)target selector:(SEL)selector object:(nullable id)argument;
+ (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(nullable id)argument;
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg;
```

## GCD

有一个底层线程池，这个池中存放的是一个个的线程。这个“池”中的线程是可以重用的，当一段时间后这个线程没有被调用胡话，这个线程就会被销毁。

### 队列

队列指执行任务的等待队列，即用来存放任务的队列。

队列是一种特殊的线性表，采用 FIFO（先进先出）的原则，即新任务总是被插入到队列的末尾，而读取任务的时候总是从队列的头部开始读取。每读取一个任务，则从队列中释放一个任务。

#### 串行队列（Serial Dispatch Queue）

每次只有一个任务被执行。让任务一个接着一个地执行。（只开启一个线程，一个任务执行完毕后，再执行下一个任务）

```objc
dispatch_queue_t serialQueue = dispatch_queue_create("com.fiteen.queue.serial", DISPATCH_QUEUE_SERIAL);
```

**主队列**也是串行队列。

```objc
dispatch_queue_t mainQueue = dispatch_get_main_queue();
```

#### 并发队列（Concurrent Dispatch Queue）

可以让多个任务并发（同时）执行。（可以开启多个线程，并且同时执行任务）

```objc
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.fiteen.queue.concurrent", DISPATCH_QUEUE_CONCURRENT);
```

> 注意：并发队列的并发功能只有在异步（dispatch_async）方法下才有效。

**全局队列**全局队列也是并发队列。

```objc
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```
### GCD 异步添加主队列

```objc
dispatch_async(dispatch_get_main_queue(), ^{
    NSLog(@"currentThread: %@", [NSThread currentThread]);
});
dispatch_async(dispatch_get_main_queue(), ^{
    NSLog(@"currentThread: %@", [NSThread currentThread]);
});
```

不会开辟主线程，所有任务都在主线程中执行。

### GCD 同步添加主队列/串行队列

```objc
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"currentThread: %@", [NSThread currentThread]);
});
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"currentThread: %@", [NSThread currentThread]);
});
```

造成死锁。线程 A 等待线程 B，线程 B 等待线程 A，互相等待，就会陷入死锁。

### NSOperation

NSOperation 是个抽象类，本身并不具备封装操作的能力，必须使用它的子类。

使用 NSOperation 子类的方式有3种：

- NSInvocationOperation
- NSBlockOperation
- 自定义子类继承 NSOperation，重写父类的- (void)main;方法，把耗时操作写到里面

### GCD 和 NSOperationQueue 和 GCD 的区别

-  GCD 是纯 C 语言的 API 。NSOperationQueue 是基于 GCD 的 OC 的封装。
-  GCD只支持 FIFO 队列，NSOperationQueue可以方便设置执行顺序，设置最大的并发数量。
-  NSOperationQueue 可是方便的设置 Operation 之间的依赖关系，GCD 则需要很多代码。
-  NSOperationQueue 支持 KVO，可以检测 Operation 是否正在执行 isExecuted ,是否结束 isFinished。是否取消 isCanceled 
-  GCD 的执行速度比 NSOperationQueue 快。

## autolayout

### 过程

updating constraints（测量阶段，更新约束）->layout（用上一步的信息设置 view 的 center 和 bounds）->display（把view渲染到屏幕上）

### 相关方法

基于约束的 AutoLayer 的方法：

1. setNeedsUpdateConstraints

当一个自定义view的某个属性发生改变，并且可能影响到constraint时，需要调用此方法去标记constraints需要在未来的某个点更新，系统然后调用updateConstraints.

2. needsUpdateConstraints

constraint-based layout system使用此返回值去决定是否需要调用updateConstraints作为正常布局过程的一部分。

3. updateConstraintsIfNeeded

立即触发约束更新，自动更新布局。

4. updateConstraints

## view layout

layoutSubviews 在以下情况下会被调用：

1. init 初始化不会触发 layoutSubviews，但是用 initWithFrame 初始化时，当 react 值不为 CGRectZero 时，也会触发；
2. addSubview 会触发；
3. 设置 view 的 frame 会触发，前提是 frame 的值设置前后发生了变化；
4. 滚动一个 UIScrollView 会触发 layoutSubviews；
5. 旋转 Screen 会触发父 UIView 上的layoutSubViews 事件；
6. 改变一个 UIView 大小的时候会触发父 UIView 上的 layoutSubviews 事件。

- setNeedsLayout 方法并不会立即刷新，立即刷新需要调用 layoutIfNeeded 方法。
## 事件传递和响应机制

### 事件处理的流程

1. 发生触摸事件，系统会将该事件加入到一个由 UIApplication 管理的事件队列中（队列特点 FIFO，先产生的事件先处理）。

2. UIApplication 从事件队列中取出最前面的事件，把事件传递给应用程序的主窗口（keyWindow）。

3. 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步。

4. 最合适的视图会调用自己的 `touches` 方法处理事件，`touches` 默认做法是把事件顺着响应者链条向上抛。

事件的传递顺序：父控件传递给子控件，产生触摸事件 -> UIApplication 事件队列 -> [UIWindow hitTest:withEvent:] -> 返回更合适的 view -> [子控件 hitTest:withEvent:] -> 返回最合适的 view。

> 如果父控件不能接受触摸事件，那么子控件也就不可能接收到触摸事件。

### 如何寻找最合适的 view

1. 首先判断主窗口（keyWindow）自己是否能接受触摸事件
2. 判断触摸点是否在自己身上
3. 子控件数组中从后往前遍历子控件，重复前面的两个步骤（所谓从后往前遍历子控件，就是首先查找子控件数组中最后一个元素，然后执行1、2步骤）
4. 如果没有符合条件的子控件，那么自己就是最合适的 view。

两个重要的方法：

```objc
// 作用：寻找并返回最合适的 view(能够响应事件的那个最合适的 view)
// 只要事件一传递给一个控件，这个控件就会调用他自己的 hitTest: withEvent: 方法
// 不管这个控件能不能处理事件，也不管触摸点在不在这个控件上，事件都会先传递给这个控件，随后再调用 hitTest: withEvent: 方法
// 如果hitTest:withEvent:方法中返回 nil，那么调用该方法的控件本身和其子控件都不是最合适的 view，也就是在自己身上没有找到更合适的 view。那么最合适的 view 就是该控件的父控件
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;
// 判断点在不在当前 view 上（方法调用者的坐标系上）如果返回 YES，代表点在方法调用者的坐标系上;返回 NO 代表点不在方法调用者的坐标系上，那么方法调用者也就不能处理事件。
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
```

### hitTest:withEvent:方法底层实现

```objc
#import "WSWindow.h"
@implementation WSWindow
// 什么时候调用:只要事件一传递给一个控件，那么这个控件就会调用自己的这个方法
// 作用:寻找并返回最合适的view
// UIApplication -> [UIWindow hitTest:withEvent:]寻找最合适的view告诉系统
// point:当前手指触摸的点
// point:是方法调用者坐标系上的点
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    // 1.判断下窗口能否接收事件
     if (self.userInteractionEnabled == NO || self.hidden == YES ||  self.alpha <= 0.01) return nil; 
    // 2.判断下点在不在窗口上 
    // 不在窗口上 
    if ([self pointInside:point withEvent:event] == NO) return nil; 
    // 3.从后往前遍历子控件数组 
    int count = (int)self.subviews.count; 
    for (int i = count - 1; i >= 0; i--) { 
        // 获取子控件
        UIView *childView = self.subviews[i]; 
        // 坐标系的转换,把窗口上的点转换为子控件上的点 
        // 把自己控件上的点转换成子控件上的点 
        CGPoint childP = [self convertPoint:point toView:childView]; 
        UIView *fitView = [childView hitTest:childP withEvent:event]; 
        if (fitView) {
        // 如果能找到最合适的view 
            return fitView; 
        }
    } 
    // 4.没有找到更合适的view，也就是没有比自己更合适的view 
    return self;
}
// 作用:判断下传入过来的点在不在方法调用者的坐标系上
// point:是方法调用者坐标系上的点
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    return NO;
}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event { 
    NSLog(@"%s",__func__);
}
@end
```

### 如何做到一个事件多个对象处理

因为系统默认做法是把事件上抛给父控件，所以可以通过重写自己的 `touches` 方法和父控件的 `touches`方法来达到一个事件多个对象处理的目的。

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{ 
    // 1. 自己先处理事件...
    NSLog(@"do somthing...");
    // 2. 再调用系统的默认做法，再把事件交给上一个响应者处理
    [super touchesBegan:touches withEvent:event]; 
}
```

### 响应者对象

只有继承了 `UIResponder` 的对象才能接受并处理事件，我们称之为“响应者对象”。如：

- UIApplication
- UIViewController
- UIView

UIResponder 中提供了以下 4 个对象方法来接收并处理事件。

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

UIView 不能接收触摸事件的三种情况：

- 不允许交互：`userInteractionEnabled = NO`；
- 隐藏：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件；
- 透明度：如果设置一个控件的透明度 `<0.01`，会直接影响子控件的透明度。`alpha：0~0.01` 为透明。

### 响应者链条

响应者链是由多个响应者对象连接起来的链条。

响应者链的事件传递过程：

1. 如果当前 view 是控制器的 view，那么控制器就是上一个响应者，事件就传递给控制器；
2. 如果当前 view 不是控制器的view，那么父视图就是当前 view 的上一个响应者，事件就传递给它的父视图；
3. 在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给 window 对象进行处理；
4. 如果 window 对象也不处理，则其将事件或消息传递给 UIApplication 对象
5. 如果 UIApplication 也不能处理该事件或消息，则将其丢弃。


## iOS 中常用的锁

### 自旋锁

自旋锁在未获得锁的情况下，一直运行（自旋），占用着 CPU。

自旋锁与互斥锁有点类似，只是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，“自旋锁”的作用是为了解决某项资源的互斥使用。因为自旋锁不会引起调用者睡眠，所以自旋锁的效率远高于互斥锁。


OSSpinLock/NSSpinLock，它现在被废弃了，它现在被废弃了，不能使用了，它是有缺陷的，会造成死锁。当低优先级线程访问了锁并执行了任务，这时恰好高的优先级线程也访问了锁，因为它的优先级较高，所以要优先执行任务，所以它会不停的访问该锁，并使得 CPU 大部分都用来访问该锁忙等了，造成低优先级的线程没有足够的 CPU 时机来执行任务，这样造成了死锁。

```objc
#import <libkern/OSAtomic.h>
// 初始化
OSSpinLock spinLock = OS_SPINLOCK_INIT;
// 加锁
OSSpinLockLock(&spinLock);
// 解锁
OSSpinLockUnlock(&spinLock);
// 尝试加锁，可以加锁则立即加锁并返回 YES,反之返回 NO
OSSpinLockTry(&spinLock)
/*
注:苹果爸爸已经在iOS10.0以后废弃了这种锁机制,使用os_unfair_lock 替换,
顾名思义能够保证不同优先级的线程申请锁的时候不会发生优先级反转问题.
*/
// os_unfair_lock(互斥锁)
#import <os/lock.h>
// 初始化
os_unfair_lock unfair_lock = OS_UNFAIR_LOCK_INIT;
// 加锁
os_unfair_lock_lock(&unfair_lock);
// 解锁
os_unfair_lock_unlock(&unfair_lock);
// 尝试加锁，可以加锁则立即加锁并返回 YES,反之返回 NO
os_unfair_lock_trylock(&unfair_lock);
/*
注:解决不同优先级的线程申请锁的时候不会发生优先级反转问题.
不过相对于 OSSpinLock , os_unfair_lock 性能方面减弱了许多.
*/
```
### 互斥锁

`p_thread_mutex`,`NSLock`,`@synthronized` 这个顺序是按照性能排序的，也是我们常用的几个互斥锁。

由于是互斥锁，当一个线程进行访问的时候，该线程获得锁，其他线程进行访问的时候，将被操作系统挂起，直到该线程释放锁，其他线程才能对其进行访问，从而却确保了线程安全。但是如果连续锁定两次，则会造成死锁问题。

```objc
- (void)mutexLock{
    // pthread_mutex
    pthread_mutex_t mutex;
    pthread_mutex_init(&mutex,NULL);
    pthread_mutex_lock(&mutex);
    pthread_mutex_unlock(&mutex);

    // NSLock
    NSLock *lock = [[NSLock alloc] init];
    lock.name = @"lock";
    [lock lock];
    [lock unlock];
    
    // synchronized
    // 底层封装的pthread_mutex的PTHREAD_MUTEX_RECURSIVE 模式，锁对象来表示是否为同一把锁。
    @synchronized (self) {
        
    }
}
```

### 递归锁

NSRecursiveLock，它是递归锁，封装的 pthread_mutex 的 PTHREAD_MUTEX_RECURSIVE 模式。它允许我们进行多次锁。不会造成死锁。

递归锁的主要意思是，同一条线程可以加多把锁。

当相同的线程访问一段代码，如果是加锁的可以继续加锁，继续往下走；

不同线程来访问这段代码时，发现有锁要等待所有锁解开之后才可以继续往下走。

```objc
- (void)RecursiveLock{
    NSRecursiveLock *lock = [NSRecursiveLock alloc];
    // 尝试加锁，可以加锁则立即加锁并返回 YES,反之返回 NO
    [lock tryLock];
    [lock lock];
    [lock lock];
    [lock unlock];
}
```

### 条件锁

NSCondition 条件锁（对象锁）。RAC 中使用了。

```objc
// 初始化
NSCondition *_condition= [[NSCondition alloc]init];
// 加锁
[_condition lock];
// 解锁
[_condition unlock];
```

其他功能接口：

- wait： 进入等待状态
- waitUntilDate： 让一个线程等待一定的时间
- signal： 唤醒一个等待的线程
- broadcast： 唤醒所有等待的线程
注: 所测时间波动太大, 有时候会快于 NSLock, 我取的中间值.

```objc
- (void)conditionLock{
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        condition = [[NSCondition alloc] init];
        [condition wait];
        NSLog(@"finish----");
    });
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [NSThread sleepForTimeInterval:5.0];
        [condition signal];
    });
}
```

NSConditionLock

```objc
// 初始化
NSConditionLock *_conditionLock = [[NSConditionLock alloc]init];
// 加锁
[_conditionLock lock];
// 解锁
[_conditionLock unlock];
// 尝试加锁，可以加锁则立即加锁并返回 YES,反之返回 NO
[_conditionLock tryLock];
/*
其他功能接口
- (instancetype)initWithCondition:(NSInteger)condition NS_DESIGNATED_INITIALIZER; //初始化传入条件
- (void)lockWhenCondition:(NSInteger)condition;//条件成立触发锁
- (BOOL)tryLockWhenCondition:(NSInteger)condition;//尝试条件成立触发锁
- (void)unlockWithCondition:(NSInteger)condition;//条件成立解锁
- (BOOL)lockBeforeDate:(NSDate *)limit;//触发锁 在等待时间之内
- (BOOL)lockWhenCondition:(NSInteger)condition beforeDate:(NSDate *)limit;//触发锁 条件成立 并且在等待时间之内
*/
```

### 信号量

semphone 在一定程度也可以当互斥锁用，它适用于编程逻辑更复杂的场景，同时它也是除了自旋锁以为性能最高的锁。

在多线程环境下用来确保代码不会被并发调用。在进入一段代码前，必须获得一个信号量，在结束代码前，必须释放该信号量，其他想要执行该代码的线程必须等待直到前者释放了该信号量。

```objc
- (void)semaphore{
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"semaphoreFinish---");
    });
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [NSThread sleepForTimeInterval:5.0];
        dispatch_semaphore_signal(semaphore);
    });
}
```

## 哈希表

### 哈希表定义/原理

哈希表（也叫散列表），是根据键（key）直接访问在内存储存位置的数据结构。

哈希表本质是一个数组，数组中的每一个元素成为一个箱子，箱子中存放的是键值对。根据下标 index 从数组中取 value。关键是如何获取 index，这就需要一个固定的函数（哈希函数），将 key 转换成 index。不论哈希函数设计的如何完美，都可能出现不同的 key 经过 hash 处理后得到相同的 hash 值，这时候就需要处理哈希冲突。

优点：哈希表可以提供快速的操作。
缺点：哈希表通常是基于数组的，数组创建后难于扩展。也没有一种简便的方法有序遍历表中的数据项。
综上，如果不需要有序遍历数据，井且可以提前预测数据量的大小。那么哈希表在速度和易用性方面是无与伦比的。

### 哈希查找步骤

1. 使用哈希函数将被查找的 key 映射（转换）为 index，理想情况下（hash 函数设计合理）不同的键映射的数组下标也不同，所有的查找时间复杂度为 O(1)。但是实际情况下不是这样的，所以哈希查找的第二步就是处理哈希冲突。

2. 处理哈希碰撞冲突。处理方法有很多，比如拉链法、线性探测法。

### 哈希函数的构造方法

1. 数字分析法：

如果事先知道关键字集合，并且每个关键字的位数比哈希表的地址码位数多时，可以从关键字中选出分布较均匀的若干位，构成哈希地址。

2. 平方取中法：

当无法确定关键字中哪几位分布较均匀时，可以先求出关键字的平方值，然后按需要取平方值的中间几位作为哈希地址。

3. 分段叠加法：

这种方法是按哈希表地址位数将关键字分成位数相等的几部分（最后一部分可以较短），然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。

4. 除留余数法：

假设哈希表长为 m(大于元素个数的最小质数)，p 为小于等于 m 的最大素数，则哈希函数为 h(k)= k % p ，其中%为模 p 取余运算。

### 哈希冲突的解决方法

1. 开放定址法：当发生冲突时，使用某种探测技术在Hash表中形成一个探测序列，然后沿着这个探测序列依次查找下去，当碰到一个空的单元时，则插入其中。基本公式为：`hash(key) = （hash(key)+di）mod TableSize`。其中di为增量序列，TableSize 为表长。根据 di 的不同我们又可以分为线性探测，平方（二次）探测、随机探测。 
2. 再哈希法：当发生冲突时，使用第二个、第三个、哈希函数计算地址，直到无冲突时。缺点：计算时间增加。
3. 链地址法/拉链法：每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向 
链表连接起来。
4. 建立公共溢出区：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表。

### Hash 在 iOS 中的应用

1. App 签名原理（MD5、MD5 可以说是目前应用最广泛的 Hash 算法）
2. weak 底层实现采用的数据存储结构（Hash 表 + 数组，采用**开放定址线性探测法**解决哈希冲突）
3. 关联对象 associatedObject 底层实现采用的数据存储结构（Hash 表 + Hash 表，采用**开放定址线性探测法**解决哈希冲突）
4. NSDictionary 的底层实现采用的数据存储结构（Hash 表，采用**开放定址线性探测法**解决哈希冲突）
5. @synchronized（Hash 表，采用**拉链法**解决哈希冲突）
6. KVO 底层实现采用的数据存储结构（Hash 表 + 数组）
7. RunLoop 保存在一个全局的可变字典中， key 是 phread_t， value 是 CFRunLoopRef
8. 不支持 Tagged Pointer 的 Swift 对象的引用计数（Hash 表）

### NSDictionary

NSDictionary（字典）是使用 Hash 表来实现 key 和 value 之间的映射和存储的， Hash 函数设计的好坏影响着数据的查找访问效率。数据在 Hash 表中分布的越均匀，其访问效率越高。

而在 Objective-C 中，通常都是利用 NSString 来作为键值，其内部使用的 Hash 函数也是通过使用 NSString 对象作为键值来保证数据的各个节点在 Hash 表中均匀分布。

#### NSDictionary 原理

```objc
- (void)setObject:(id)anObject forKey:(id <NSCopying>)aKey;
```

NSDictionary中的 key 是唯一的，key 可以是遵循 NSCopying 协议和重载方法的任何对象。也就是说在 NSDictionary 内部，会对 aKey 对象 copy 一份新的。而 anObject 对象在其内部是作为强引用（retain或strong)。

在调用 `setObject: forKey:` 后，内部会去调用 key 对象的 `- (NSUInteger)hash;` 方法确定 object 在 hash 表内的入口位置，然后会调用 `- (BOOL)isEqual:(id)object;` 来确定该值是否已经存在于 NSDictionary 中。

#### 解决哈希冲突

https://juejin.im/post/5c6abfc86fb9a049c04396a7

说法一：使用 `NSMapTable` 实现的，采用拉链法解决哈希冲突。

```objc
@interface NSMapTable : NSObject {
   NSMapTableKeyCallBacks   *keyCallBacks;
   NSMapTableValueCallBacks *valueCallBacks;
   NSUInteger             count;
   NSUInteger             nBuckets;
   struct _NSMapNode  **buckets;
}
```

从上面的结构真的能看出NSMapTable是一个 哈希表 + 链表 的数据结构吗？在NSMapTbale中插入或者删除一个对象的寻找时间 = O(1) + O(m) ，m为最坏时可能为n。
O(1) ：对key进行hash得到bucket的位置
O(m) ：不同的key得到相同的hash值的value存放到链表中，遍历链表的时间


说法二：是对 `_CFDictionary` 的封装，解决哈希冲突使用的是开放定址线性探测法。

```objc
struct __CFDictionary {
    CFRuntimeBase _base;
    CFIndex _count;
    CFIndex _capacity;
    CFIndex _bucketsNum;
    uintptr_t _marker;
    void *_context;
    CFIndex _deletes;
    CFOptionFlags _xflags;
    const void **_keys;	
    const void **_values;
};
```

从上面的数据结构可以看出 NSDictionary 内部使用了两个指针数组分别保存 keys 和 values。采用的是连续方式存储键值对。拉链法会将 key 和 value 包装成一个结果存储（链表结点），而 Dictionary 的结构拥有 keys 和 values 两个数组（开放寻址线性探测法解决哈希冲突的用的就是两个数组），说明两个数据是被分开存储的，不像是拉链法。


## 编译和启动步骤

### Clang（编译器前端）

GCC 的替代品，Clang 的编译速度比 GCC 快。

1. Lexer：读入源文件，并将其转化成字符流

2. Parser：将字符流转换成 AST（抽象语法树）

3. SemanticAnalysis：对输入的 AST 进行语法检查

4. Code Generation：代码生成，将 AST 转换成低层次的IR指令

### LLVM（编译器后端）

底层虚拟机（Low Level Virtual Machine）

1. Optimization：分析IR指令，将其中潜在会拖慢运行速度的指令干掉

2. AsmPrinter：通过IR（中间码）生成特定CPU架构的汇编代码

3. Assemble ：将汇编代码转化成机器码

4. Linker：将机器码链接成可执行程序或动态码

### BitCode

当我们提交程序到 App Store 上时，Xcode会将程序编译为一个中间表现形式( bitcode )。然后 App store 会再将这个 bitcode 编译为可执行的64位或32位程序

BitCode 类似于一个中间码，被上传到appStore之后，苹果会根据下载应用的用户的手机指令集类型生成只有该指令集的二进制进行下发，从而达到精简安装包体积的目的

bitcode 优化汇编指令

### 指令集

- 模拟器的指令集：
  - 4S：-5i386
  - 5S以上：X64_86

- 真机（iOS设备)的默认指令集：
  - armv6设备：iPhone,iPhone2,iPhone3G
  - armv7设备：iPhone3GS,iPhone4,iPhone4S
  - armv7s设备：iPhone5,iPhone5C
  - arm64设备：iPhone5S以上

### 编译时报出 i386，x86_64的错误

解决办法：

查看静态库是否支持i386/x86_64的方法：

1.如果是.a文件：

打开终端，使用lipo -info 静态包地址.a查看

2.如果是framework文件：

cd到 framework 内部，之后在使用 `lipo -info xxxFramework` 查看

## 讲一讲 NSURLProtocol

### 使用场合：

任务之间不太相互依赖：GCD
任务之间有依赖或要监听任务的执行情况：NSOperationQueue

## 本地数据持久化技术

其实就是存储到沙盒中。

- NSUserDefaults，偏好设置，存储本地信息，如用户名、手机号、登录状态等，数据自动保存在沙盒的 `Libarary/Preferences` 目录下。
- plist，属性列表，比如全国各地省市区信息
- Archieving，归档
- sqlite3（FMDB）
- Core Data（基于 sqlite3 的 OC 封装，转成 OC 对象，无需接触 sql 语句） 没用过

## FMDB 多线程的问题

文件数据库 sqlite，同一时刻允许多个进程/线程读，但同一时刻只允许一个线程写。在进行写入操作时，数据库文件被琐定，此时任何其他读/写操作都被阻塞。

以第三方框架 FMDB 为例，原先我们使用 FMDB 中的 FMDatabase 对象操作数据库，但是 FMDatabase 不能在多个线程中共用。因为这个类本身不是线程安全的，如果这样使用会造成多线程对 sqlite 的竞争，导致数据混乱、应用崩溃等问题。

如要需要多线程操作数据库，那么就需要使用 FMDatabaseQueue。

FMDatabaseQueue 底层原理：创建一个自定义串行队列（这里不能使用主队列，因为使用主队列会导致阻塞），然后将放入队列的 block 同步执行，这样避免了多线程同时访问数据库。 为了保证读写安全，我们只使用一个自定义串行队列（保证只有一个 FMDatabaseQueue 对象）

```
[dbQueue inDatabase:^(FMDatabase *db) {  
    NSString *sql = [NSString stringWithFormat:@"INSERT INTO %@(%@) VALUES (%@);", tableName, keyString, valueString];  
    res = [db executeUpdate:sql withArgumentsInArray:insertValues];  
    NSLog(res ? @"插入成功" : @"插入失败");  
}]; 
```

## WebSocket

WebSocket 是 HTML5 一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯，它建立在 TCP 之上，同 HTTP 一样通过 TCP 来传输数据，但是它和 HTTP 最大不同是：WebSocket 是一种双向通信协议.

由于项目需要创建一个聊天室，需要通过长链接，和后台保持通讯，进行聊天,并且实时进行热点消息的推送。

目前 Facebook 的 SocketRocket 应该是目前最好的 关于WebSocket使用的框架了.而且简单易用.

## iOS 内省方法

判断对象类型：

```objc
// 判断是否是这个类或者这个类的子类的实例
-(BOOL) isKindOfClass:
// 判断是否是这个类的实例
-(BOOL) isMemberOfClass:
```

判断对象 or 类是否有这个方法：
```objc
// 判断实例是否有这样方法
-(BOOL) respondsToSelector:
// 判断类是否有这个方法
+(BOOL) instancesRespondToSelector:
```
## class、objc_getClass、object_getclass

`objc_getClass`：参数是类名的字符串，返回的就是这个类的类对象；

`object_getClass`：参数是 id 类型，它返回的是这个 id 的 isa 指针所指向的 Class，如果传参是 Class，则返回该 Class 的 metaClass；

`[obj class]`：则分两种情况：

1. 当 obj 为实例对象时，[obj class] 中 class 是实例方法：`- (Class)class`，返回的 obj 对象中的 isa 指针；
2. 当 obj 为类对象（包括元类和根类以及根元类）时，调用的是类方法：`+ (Class)class`，返回的结果为其本身。

## intrinsicContentSize

intrinsicContentSize，也就是控件的内置大小。比如 UILabel、UIButton 等控件，他们都有自己的内置大小。控件的内置大小往往是由控件本身的内容所决定的，比如一个UILabel的文字很长，那么该 UILabel 的内置大小自然会很长。控件的内置大小可以通过 UIView 的 intrinsicContentSize 属性来获取，也可以通过 invalidateIntrinsicContentSize 方法来在下次 UI 规划事件中重新计算 intrinsicContentSize。如果直接创建一个原始的 UIView 对象，显然它的内置大小为0。

## assign 和 weak

### weak

相同点：都是弱引用
assign 用于修饰基本数据类型，它修饰的对象，释放后不会置为 nil，变成野指针，报 `EXC_BAD_ACCESS` 错误。
weak 修饰的对象释放后会置为 nil，通常用 weak 修饰 delegate 来防止出现循环引用。

**weak 对象如何置为 nil？**

weak 对象会放入 hash 表中，用 weak 指向的对象的**地址**作为 key，当此对象的引用计数为 0 时 dealloc，通过 key 在表中搜索出所有对应的 weak 对象，把他们置为 nil。

```c
objc_storeWeak(&a, b)
// 相当于
objc_storeWeak(value, key)
```
## layer 和 view 的区别

1. UIView 继承自 UIResponder，能接收并响应事件，负责显示内容的管理。而 CALayer 继承自 NSObject，不能响应事件，负责显示内容的绘制；
2. UIView 侧重于展示内容， CALayer 侧重于图形和界面的绘制；
3. 每个 UIView 内部都有一个 CALayer 在背后提供内容的绘制和显示，并且 View 的尺寸样式都由内部的 Layer 所提供，layer 比 view 多了一个 `anchorPoint`；
4. 当 View 展示的时候，View 是 layer 的 CALayerDelegate，View 展示的内容是由 CALayer 进行 display 的；
5. View 内容的展示依赖于 CALayer 对于内容的绘制，UIView 的 frame 也是由内部的 CALayer 进行绘制的；
6. 对于 UIView 的属性修改，不会引起动画效果，但是对于 CALayer 的属性修改，是支持默认动画效果的，在 View 执行动画的时候，View 是 layer 的代理，layer 通过 `actionForLayer:forkey` 相对应的代理 View 请求动画 action。 

## self 和 super 的区别

**self**是关键字，代表当前方法的调用者。如果是类方法，代表当前类；如果是实例方法，代表当前类的对象。

**super**是编译器指令。

其实不管是 self 还是 super 真正调用的对象都是一样的，只是查找方法的位置不一样，self 是从当前类结构中开始查找，super 是 从父类中查找，但方法真正的接受者都是当前类或者当前类的对象。

**那么初始化的时候为什么要调用 if (self = [super init])**

这里是担心父类初始化失败，如果初始化一个对象失败，就会返回 nil，当返回 nil 的时候 `self = [super init]`测试的主体就不会再继续执行。如果不这样做，你可能会操作一个不可用的对象，它的行为是不可预测的，最终可能会导致你的程序崩溃。

## Outlet 为什么要设置成 weak

vc 已经强引用了 view，当控件的父 view 销毁时，如果你还想继续拥有这个控件，就用 srtong；如果想保证控件和父 view 拥有相同的生命周期，就用 `weak`。

使用 storyboard 创建的 vc，会有一个叫 `_topLevelObjectsToKeepAliveFromStoryboard` 的私有数组强引用所有 top level 的对象，所以这时即便声明成 weak 也没关系。


## 分类与扩展的区别

1. 类扩展(extension）是 category 的一个特例，有时候也被称为匿名分类或者私有类。他的作用是为一个类添加一些私有的成员变量和方法。
2. 和分类不同，类扩展既可以声明成员变量又可以声明方法。
3. 类扩展可以定义在 .m 文件中，这种扩展方式中定义的变量都是私有的，也可以定义在 .h 文件中，这样定义的代码就是公有的，类扩展在 .m 文件中声明私有方法是非常好的方式。
4. 类扩展中添加的新方法，一定要实现。category 中没有这种限制。

## 创建/定义一个对象

### 步骤

- 开辟内存空间
- 初始化参数
- 返回内存地址值

### 对象的内存大小

首先，对象初始化需要的内存（`class_getInstanceSize`）为4/8字节（isa 指针在 32 位系统下占 4 字节，在 64 位系统下占 8 字节），实际分配的内存（`malloc_size`）为16字节，因为 `alloc` 源码实现中有：

```c
size_t instanceSize(size_t extraBytes) {
    size_t size = alignedInstanceSize() + extraBytes;
    // CF requires all objects be at least 16 bytes.
    if (size < 16) size = 16;
    return size;
}
```

如果需要的内存超过 16 字节，就要考虑**内存对齐**的问题。

1. 计算实际需要的内存空间时，内存空间大小必须是结构体中占用最大内存空间成员所占用内存大小（32 位下是 4 或 8，64 位下是 8）的倍数。

2. 计算实际分配的内存空间时，分配内存大小必须是 16 的倍数。


## 对象的内存销毁时间表

1. 调用 `-release:` 引用计数变为零
  - 对象正在被销毁，生命周期即将结束
  - 不能再有新的 `__weak` 弱引用，否则将指向 `nil`
  - 调用 [self dealloc]		
2. 子类调用 `-dealloc`
  - 继承关系中最底层的子类，调用 `-dealloc`
  - 如果是 MRC 代码，则会手动释放实例变量们（ivars）
  - 继承关系中每一层的父类，调用 `-dealloc`
3.	NSObject 调用 `-dealloc`
  - 只做一件事：调用 Objective-C runtime 中的 `object_dispose()` 方法
4.	调用 `object_dispose()`
  - 为 C++ 的实例变量们（ivars）调用 destructors		
  - 为 ARC 状态下的实例变量们（ivars）调用 `-release`	
  - 解除所有使用 runtime Associat 方法关联的对象
  - 解除所有 __weak 引用
  - 调用 `free()`

## dealloc 的调用时机

1. 当前类的 `dealloc` 的调用是在最后一次 release 执行后（引用计数为0），但此时实例变量(ivars)并未释放。
2. 父类的 `dealloc` 方法会在子类 `dealloc` 方法返回后自动执行。
3. ARC 子类的实例变量(ivars)在根类 `[NSObject dealloc]` 中释放。


## 更新 UI 的操作必须放在主线程 

因为 **UIKit 框架是线程不安全**的。

线程安全需要加锁，但加锁会消耗性能，影响处理速度，影响渲染速度，我们通常自己在写 `@property`时都会写 `nonatomic` 来追求高性能高效率。

但 UI 层面需要追求速度流畅，体验无顿挫感的，因此给 UI 加锁是不可能的。那么就强制规定所有 UI 操作必须在主线程中进行，避免多线程对 UI 进行操作，主要既高效高性能，又不会出现线程安全问题。


## 好的代码是什么样的

高内聚、低耦合、可复用、可扩展、健壮、可读性高、可维护性高

SOLID 原则：

Single responsibility principle ： 单一职责原则
Open/closed principle ：开闭原则
Liskov substitution principle ：Liskov替换原则
Interface segregation principle ：接口隔离原则
Dependency inversion principle ： 依赖反转原则

## 函数式/响应式/链式编程

**函数式编程**：嵌套的函数，将操作尽可能写在一起。（本质：往方法里面传入 block，方法中嵌套 block 调用。）
**响应式编程**：一种面向数据流和变化传播的编程范式。
**链式编程**：方法的返回值必须是方法的调用者。

### 项目中的链式调用如何实现的

**链式调用**或者称为**方法链**更合适，method chaining(named parameter idiom)，即用连续调用多个方法，每次方法调用都返回当前对象本身。

method chaining 在 Objective-C 中的应用，很有名的就有 Masonry：

```objc
make.left.equalTo(self.XXView.mas_right).mas_offset(8);
```

那它具体是如何实现的呢？

其实就是利用 **block**。`.` 其实算是语法糖。

可以将 block 作为方法的参数，同时 block 再把对象实例返回回来。

```objc
// Test.h
@interface Test : NSObject
- (Test*(^)(NSString* v1))value1;
- (Test*(^)(BOOL v2))value2;
- (Test*(^)(NSInteger v3))value3;
@end

// Test.m

@implementation Test

- (Test*(^)(NSString* v1))value1{
    @weakify(self)  
    return ^(NSString* v1){
        @strongify(self)
        mV1 = v1;
        return self;
    }
}

- (Test*(^)(BOOL v2))value2{
    @weakify(self) 
    return ^(BOOL v2){
        @strongify(self)
        mV2 = v2;
        return self;
    }
}

- (Test*(^)(NSInteger v3))value3{
    @weakify(self) 
    return ^(NSInteger v3){
        @strongify(self)
        mV3 = v3;
        return self;
    }
}
@end
```

调用

```objc
 + (Test*(^)(void))create{
    return ^(void){
        return [Test new];
    }
}

Test.create().value1(@"value").value2(true).value3(1);
```

### 函数式特点

嵌套的函数，将操作尽可能写在一起。（本质：往方法里面传入 block，方法中嵌套 block 调用。）

1. 函数是"第一等公民"。函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。
2. 只用"表达式"，不用"语句"。执行操作后需要有**返回值**。
3. 没有"副作用"，意味着函数要独立，所有功能就是返回一个新的值，没有其它行为，尤其是不得修改外部变量的值。
4. 不修改状态，使用参数保存状态，例子就是递归。
5. 引用透明，函数的运行不依赖于外部变量或状态，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同。


## 两个子视图的最近公共父视图

```objc
- (NSArray *)superClasses:(Class)class {
    if (!class) return @[];
    NSMutableArray *res = @[].mutableCopy;
    while (class) {
        [res addObject:class];
        class = [class superclass];
    }
    return [res copy];
}

- (Class)commonClass:(Class)classA andClass:(Class)classB {
    NSArray *arrA = [self superClasses:classA];
    NSSet *setB = [NSSet setWithArray:[self superClasses:classB]];
    for (NSUInteger i = 0; i < arrA.count; i++) {
        if ([setB containsObject:arrA[i]]) {
            return arrA[i];
        }
    }
    return nil;
}
```

## UIView 中 的 tintcolor 是怎么实现的

`tintColor` 是 iOS7 以后，UIView 新添加的属性，它具有**传递性**。

对一个 view 来说, 只要它的 tintColor 没有明确指定, 那么它就会采用其 superview 的 tintColor. 一旦这个 view 的 tintColor 改变了, 这种改变就会一直向其 subview 传播, 直至某个 subview 明确指定了 tintColor 为止。

此外, 在某个 view 的 tintColor 发生改变或 tintAdjustmentMode 属性改变时, 这个 view 以及它所有受影响的 subview 都会调用 `tintColorDidChange` 方法, 所以有什么自定义操作, 都可以在这个方法中进行。


UILabel 在 autolayout 的时候会自动延伸
text你觉得是怎么实现的
有三个view平行放置在superview中怎么用
optional是怎么实现的
weak 的实现方法

## UIView 的父类、UIButton 的父类，他们共同的父类

UIView 的父类 是 UIResponder；
UIButton 的父类是 UIControl，UIControl 的父类是 UIView，根类都是 NSObject。

## APNS 推送原理

1. 客户端的 App 需要把用户的 UDID 和 app 的 bundleID 发送给 APNS 服务器，进行注册， APNS 将加密后的 device Token 返回给 App；
2. App 获取 Deveice Token 后，上传到公司服务器；
3. 当需要推送通知时，公司服务器会将推送的内容和 Device Token 一起发送给 APNS 服务器；
4. APNS 再将推送内容推送到相应的客户端 App 上。

# 源码解析

## SDWebImage

1. 首先显示占位图
2. 在 webimagecache 中寻找图片对应的缓存，它是以 url 为数据索引先在内存中查找是否有缓存；
3. 如果没有缓存，就通过md5处理过的key来在磁盘中查找对应的数据，如果找到就会把磁盘中的数据加到内存中，并显示出来；
4. 如果内存和磁盘中都没有找到，就会向远程服务器发送请求，开始下载图片；
5. 下载完的图片加入缓存中，并写入到磁盘中；
5. 整个获取图片的过程是在子线程中进行，在主线程中显示。

## AFNetworking

### 内部结构

AFNetworking 是封装的 NSURLSession 的网络请求，由五个模块组成：分别由 NSURLSession, Security, Reachability,Serialization, UIKit 五部分组成。

1. NSURLSession：网络通信模块（核心模块） 对应 AFNetworking中的 AFURLSessionManager和对HTTP协议进行特化处理的AFHTTPSessionManager，AFHTTPSessionManager 是继承于 AFURLSessionmanager 的。
2. Security：网络通讯安全策略模块，对应 AFSecurityPolicy
3. Reachability：网络状态监听模块，对应 AFNetworkReachabilityManager
4. Serialization：网络通信信息序列化、反序列化模块对应 AFURLResponseSerialization
5. UIKit：对于 iOS UIKit的扩展库

### AFN2.0 为什么添加一条常驻线程

AFN2.0里面把每一个网络请求的发起和解析都放在了一个线程里执行。正常来说，一个线程执行完任务后就退出了。开启 runloop 是为了防止线程退出。一方面避免每次请求都要创建新的线程；另一方面，因为 connection 的请求是异步的，如果不开启 runloop，线程执行完代码后不会等待网络请求完的回调就退出了，这会导致网络回调的代理方法不执行。

这是一个单例，用NSThread创建了一个线程，并且为这个线程添加了一个 runloop，并且加了一个 NSMachPort，来防止 runloop 直接退出。

### AFN3.0 为什么不再需要常驻线程

NSURLConnection 的一大痛点就是：发起请求后，这条线程并不能随风而去，而需要一直处于等待回调的状态。

苹果也是明白了这一痛点，从iOS9.0开始 deprecated 了 NSURLConnection。替代方案就是NSURLSession。

```objc
self.operationQueue = [[NSOperationQueue alloc] init];
self.operationQueue.maxConcurrentOperationCount = 1;
self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
```

从上面的代码可以看出，NSURLSession 发起的请求，不再需要在当前线程进行代理方法的回调，而是可以指定回调的 delegateQueue，这样我们就不用为了等待代理回调方法而苦苦保活线程了。

同时还要注意一下，指定的用于接收回调的 Queue 的 `maxConcurrentOperationCount` 设为了1，这里目的是想要让并发的请求串行地进行回调。
 
### 为什么 AFN3.0 中需要设置 self.operationQueue.maxConcurrentOperationCount = 1; 而 AFN2.0 却不需要

功能不一样：AFN 3.0的 operationQueue 是用来接收 NSURLSessionDelegate 回调的，鉴于一些多线程数据访问的安全性考虑，设置了 `maxConcurrentOperationCount = 1`来达到串行回调的效果。

而 AFN 2.0 的 operationQueue 是用来添加 operation 并进行并发请求的，所以不要设置为1。

# 算法

## 翻转字符串

```c
void Flip(char*array,int len) {
  int left = 0;
  int right = len;
  while (left <= right) {
    char tmp = array[left];//数组内容进行翻转
    array[left] = array[right];
    array[right] = tmp;
    left++;//对循环条件进行调整
    right--;//对循环条件进行调整
  }
}
int main() {
  char array[] = "tneduts a ma i";
  int len = strlen(array)-1;//取长度注意是取下标最后一位
  Flip(array, len);
  for (int i = 0; i <= len; ++i) {
    printf("%c", array[i]);
  }
  printf("\n");
  system("pause");
  return 0;
}
```
## 判断一颗二叉树是不是对称的

注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的.

解析：思路，递归，从根节点开始，判断左右子节点是否对称，若对称，递归，若不对称，则返回 NO。

```c
bool isMirror(struct TreeNode* p, struct TreeNode* q) {
    if(!p && !q)
        return 1;
    if(!p || !q)
        return 0;
    if(p->val == q->val)
        return isMirror(p->left, q->right) && isMirror(p->right, q->left);
    else
        return 0;
}

bool isSymmetric(struct TreeNode* root) {
    return isMirror(root, root);
}
```

## 如何删除单链表中的第 i 个元素

因为数据结构是单链表，要想删除第 i 个结点，就要先找到这个结点；

1. 找到第 i-1 个结点；
2. 让第 i-1 个结点的后继指向第 i 个结点的后继，达到删除的目的。

## RLE 算法

编写一个函数，实现统计字符次数的功能：例如输入为 `aaabbccc`，输出为 `a3b2c3`。不限语言。

```c
#include <stdio.h>
#define N 100

void compress(char str[]);

void main(void){
	char str[N];

	printf("Enter a string:\n");
	scanf("%Ns", str);
	compress(str);
}

void compress(char str[]){
	int i = 0, cnt = 1;
	
	while(str[i] != '\0'){
		if(str[i+1] == str[i])
			cnt++;
		else{
			printf("%c%d", str[i], cnt);
			cnt = 1;
		}
		i++;
	}
	printf("\n");
}
```

## 各种排序算法的原理实现

> [总结](https://blog.fiteen.top/2019/sorting-algorithm)

**比较类排序**：通过比较来决定元素间的相对次序，时间复杂度为 O(nlogn)～O(n²)。属于比较类的有：

|         排序算法          | 时间复杂度 | 最差情况 | 最好情况 | 空间复杂度 | 排序方式 | 稳定性 |
| :-----------------------: | :--------: | :------: | :------: | :--------: | :----: | :-------: |
|  冒泡排序  |   O(n²)    |  O(n²)   |   O(n)   |    O(1)​    |  In-place  | ✔ |
|   快速排序   |  O(nlogn)​  |  O(n²)   | O(nlogn)​ |  O(logn)​   | In-place | ✘ |
| 插入排序 |   O(n²)    |  O(n²)   |   O(n)​   |    O(1)​    |  In-place  | ✔ |
|   希尔排序   |  O(nlog²n)​  |  O(n²)   |   O(n)​   |    O(1)​    | In-place | ✘ |
| 选择排序 |   O(n²)    |  O(n²)   |  O(n²)   |    O(1)​    | In-place | ✘ |
|    堆排序     |  O(nlogn)​  | O(nlogn) | O(nlogn)​ |    O(1)​    | In-place | ✘ |
|   归并排序   |  O(nlogn)​  | O(nlogn) | O(nlogn)​ |    O(n)​    |  Out-place  | ✔ |

**非比较类排序**：不通过比较来决定元素间的相对次序，其时间复杂度可以突破 O(nlogn)，以线性时间运行。属于非比较类的有：

|         排序算法         | 时间复杂度 | 最差情况  | 最好情况 | 空间复杂度 | 排序方式 | 稳定性 |
| :----------------------: | :--------: | :-------: | :------: | :--------: | :----: | :-------: |
|   桶排序   |   O(n+nlog(n/r))​   |   O(n²)   |   O(n)​   |   O(n+r)​   |  Out-place  | ✔ |
| 计数排序 |   O(n+r)​   |  O(n+r)​   |  O(n+r)​  |   O(n+r)​   |  Out-place  | ✔ |
|  基数排序   |  O(d(n+r))​  | O(d(n+r)) |  O(d(n+r))  |   O(n+r)​   |  Out-place  | ✔ |

### 冒泡排序

冒泡排序指**越小的元素会经由交换慢慢“浮”到数列的顶端**。

原理：

1. 从左到右，依次比较相邻的元素大小，更大的元素交换到右边；
2. 从第一组相邻元素比较到最后一组相邻元素，这一步结束最后一个元素必然是参与比较的元素中最大的元素；
3. 按照大的居右原则，重新从左到后比较，前一轮中得到的最后一个元素不参与比较，得出新一轮的最大元素；
4. 按照上述规则，每一轮结束会减少一个元素参与比较，直到没有任何一组元素需要比较。

![](https://blog.fiteen.top/2019/sorting-algorithm/bubble-sort.gif)

```c
void bubble_sort_quicker(int arr[], int n) {
    int i, j, flag;
    for (i = 0; i < n - 1; i++) {
        flag = 0;
        for (j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j+1);
                flag = 1;
            }
        }
        if (!flag) return;
    }
}
```

冒泡排序属于交换排序，是稳定排序，平均时间复杂度为 O(n²)（用上面的实现，可以达到 O(n)），空间复杂度为 O(1)。

### 快速排序

快速排序（Quick Sort），是冒泡排序的改进版，之所以“快速”，是因为使用了**分治法**。它也属于交换排序，通过元素之间的位置交换来达到排序的目的。

> **基本思想**：在序列中随机挑选一个元素作基准，将小于基准的元素放在基准之前，大于基准的元素放在基准之后，再分别对小数区与大数区进行排序。

一趟快速排序的具体做法是：

1. 设两个指针 i 和 j，分别指向序列的头部和尾部；
2. 先从 j 所指的位置向前搜索，找到第一个比基准小的数值后停下来，再从 i 所指的位置向后搜索，找到第一个比基准大的数值后停下来，把 i 和 j 指向的两个值交换位置；
3. 重复步骤2，直到 i = j，最后将相遇点指向的值与基准交换位置。

![](https://blog.fiteen.top/2019/sorting-algorithm/selection-sort.gif)

```c
/* 选取序列的第一个元素作为基准 */
int select_pivot(int arr[], int low) {
    return arr[low];
}

/* 随机选择基准的位置，区间在 low 和 high 之间 */
int select_pivot_random(int arr[], int low, int high) {
    srand((unsigned)time(NULL));
    int pivot = rand()%(high - low) + low;
    swap(arr, pivot, low);
    
    return arr[low];
}

/* 取起始位置、中间位置、末尾位置指向的元素三者的中间值作为基准，此方案最佳 */
int select_pivot_median_of_three(int arr[], int low, int high) {
    // 计算数组中间的元素的下标
    int mid = low + ((high - low) >> 1);
    // 排序，使 arr[mid] <= arr[low] <= arr[high]
    if (arr[mid] > arr[high]) swap(arr, mid, high);
    if (arr[low] > arr[high]) swap(arr, low, high);
    if (arr[mid] > arr[low]) swap(arr, low, mid);
    // 使用 low 位置的元素作为基准
    return arr[low];
}

void quick_sort(int arr[], int low, int high) {
    int i, j, pivot;
    if (low >= high) return;
    pivot = select_pivot_median_of_three(arr, low);
    i = low;
    j = high;
    while (i != j) {
        while (arr[j] >= pivot && i < j) j--;
        while (arr[i] <= pivot && i < j) i++;
        if (i < j) swap(arr, i, j);
    }
    arr[low] = arr[i];
    arr[i] = pivot;
    quick_sort(arr, low, i - 1);
    quick_sort(arr, i + 1, high);
}
```

快速排序是不稳定排序，它的平均时间复杂度为 O(nlogn)，平均空间复杂度为 O(logn)。

### 插入排序

插入排序就是不断地将尚未排好序的数插入到已经排好序的部分，好比打扑克牌时一张张抓牌的动作。

**基本思想**：先将第一个元素视为一个有序子序列，然后从第二个元素起逐个进行插入，直至整个序列变成元素非递减有序序列为止。如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入大相等元素的后面。整个排序过程进行 n-1 趟插入。

![](https://blog.fiteen.top/2019/sorting-algorithm/insertion-sort.gif)

```c
void insertion_sort(int arr[], int n) {
    int i, j, temp;
    for (i = 1; i < n; i++) {
        temp = arr[i];
        for (j = i; j > 0 && arr[j - 1] > temp; j--)
            arr[j] = arr[j - 1];
        arr[j] = temp;
    }
}
```

插入排序是稳定排序，平均时间复杂度为 O(n²)，空间复杂度为 O(1)。


### 希尔排序

是直接插入排序的改进版，又称“缩小增量排序”。它与直接插入排序不同之处在于，它会优先比较距离较远的元素。

**基本思想**：先将整个待排序列分割成若干个字序列分别进行直接插入排序，待整个序列中的记录“基本有序”时，再对全体记录进行一次直接插入排序。子序列的构成不是简单地“逐段分割”，将相隔某个增量的记录组成一个子序列，让增量逐趟缩短，直到增量为 1 为止。

![](https://blog.fiteen.top/2019/sorting-algorithm/shell-sort.gif)

如果增量序列满足 [n / 2, n / 2 / 2, …, 1]，可以这样实现：

```c
void shell_sort_split_half(int arr[], int n) {
    int i, j, dk, temp;
    for (dk = n >> 1; dk > 0; dk = dk >> 1) {
        for (i = dk; i < n; i++) {
            temp = arr[i];
            for (j = i - dk; j >= 0 && arr[j] > temp; j -= dk)
                arr[j + dk] = arr[j];
            arr[j + dk] = temp;
        }
    }
}
```

如果增量满足自定义序列，这样实现：

```c
void shell_insert(int arr[], int n, int dk) {
    int i, j, temp;
    for (i = dk; i < n; i += dk) {
        temp = arr[i];
        j = i - dk;
        while (j >= 0 && temp < arr[j]) {
            arr[j + dk] = arr[j];
            j -= dk;
        }
        arr[j + dk] = temp;
    }
}

void shell_sort(int arr[], int n, int dlta[], int t) {
    int k;
    for (k = 0; k < t; ++k) {
        // 一趟增量为 dlta[k] 的插入排序
        shell_insert(arr, n, dlta[k]);
    }
}
```

希尔排序是不稳定排序，它的分析是一个复杂的问题，因为它的运行时间依赖于增量序列的选择，它的平均时间复杂度为O(n^1.3)，最好情况是 O(n)，最差情况是 O(n²)。空间复杂度为 O(1)。

### 选择排序

选择排序的基本思想就是：每一趟 n-i+1(i=1,2,…,n-1) 个记录中选取关键字最小的记录作为有序序列的第 i 个记录。

![](https://blog.fiteen.top/2019/sorting-algorithm/selection-sort.gif)

**步骤**：

1. 在未排序序列中找到最小（大）元素，存放到排序序列的起始位置;
2. 在剩余未排序元素中继续寻找最小（大）元素，放到已排序序列的末尾;
3. 重复步骤2，直到所有元素排序完毕。

```c
void selection_sort(int arr[], int n) {
    int i, j;
    for (i = 0; i < n - 1; i++) {
        int min = i;
        for (j = i + 1; j < n; j++) {
            if (arr[j] < arr[min])
                min = j;
        }
        swap(arr, min, i);
    }
}
```

选择排序是不稳定排序，时间复杂度固定为 O(n²)，因此它不适用于数据规模较大的序列。空间复杂度为 O(1)。

### 堆排序

**基本思想**：

实现堆排序需要解决两个问题：

- 如何将一个无序序列构建成堆？
- 如何在输出堆顶元素后，调整剩余元素成为一个新的堆？

以升序为例，算法实现的思路为：

1. 建立一个 `build_heap` 函数，将数组 tree[0,...n-1] 建立成堆，n 表示数组长度。函数里需要维护的是所有节点的父节点，最后一个子节点下标为 n-1，那么它对应的父节点下标就是 (n-1-1)/2。
2. 构建完一次堆后，最大元素就会被存放在根节点 tree[0]。将 tree[0] 与最后一个元素交换，每一轮通过这种不断将最大元素后移的方式，来实现排序。
3. 而交换后新的根节点可能不满足堆的特点了，因此需要一个调整函数 heapify 来对剩余的数组元素进行最大堆性质的维护。如果 tree[i] 表示其中的某个节点，那么 tree[2*i+1] 是左孩子，tree[2*i+2] 是右孩子，选出三者中的最大元素的下标，存放于 max 值中，若 max 不等于 i，则将最大元素交换到 i 下标的位置。但是，此时以 tree[max] 为根节点的子树可能不满足堆的性质，需要递归调用自身。

![](https://blog.fiteen.top/2019/sorting-algorithm/heap-sort.gif)

```c
void heapify(int tree[], int n, int i) {
    // n 表示序列长度，i 表示父节点下标
    if (i >= n) return;
    // 左侧子节点下标
    int left = 2 * i + 1;
    // 右侧子节点下标
    int right = 2 * i + 2;
    int max = i;
    if (left < n && tree[left] > tree[max]) max = left;
    if (right < n && tree[right] > tree[max]) max = right;
    if (max != i) {
        swap(tree, max, i);
        heapify(tree, n, max);
    }
}

void build_heap(int tree[], int n) {
    // 树最后一个节点的下标
    int last_node = n - 1;
    // 最后一个节点对应的父节点下标
    int parent = (last_node - 1) / 2;
    int i;
    for (i = parent; i >= 0; i--) {
        heapify(tree, n, i);
    }
}

void heap_sort(int tree[], int n) {
    build_heap(tree, n);
    int i;
    for (i = n - 1; i >= 0; i--) {
        // 将堆顶元素与最后一个元素交换
        swap(tree, i, 0);
        // 调整成大顶堆
        heapify(tree, i, 0);
    }
}
```

堆排序是不稳定排序，适合数据量较大的序列，它的平均时间复杂度为 Ο(nlogn)，空间复杂度为 O(1)。堆排序仅需一个记录大小供交换用的辅助存储空间。

### 归并排序

归并排序是建立在**归并**操作上的一种排序算法。它和快速排序一样，采用了**分治法**。

**基本思想**：

假如初始序列含有 n 个记录，则可以看成是 n 个有序的子序列，每个子序列的长度为 1。然后两两归并，得到 n/2 个长度为2或1的有序子序列；再两两归并，……，如此重复，直到得到一个长度为 n 的有序序列为止，这种排序方法称为 **二路归并排序**。

![](https://blog.fiteen.top/2019/sorting-algorithm/merge-sort.gif)

```c
/* 将 arr[L..M] 和 arr[M+1..R] 归并 */
void merge(int arr[], int L, int M, int R) {
    int LEFT_SIZE = M - L + 1;
    int RIGHT_SIZE = R - M;
    int left[LEFT_SIZE];
    int right[RIGHT_SIZE];
    int i, j, k;
    // 以 M 为分割线，把原数组分成左右子数组
    for (i = L; i <= M; i++) left[i - L] = arr[i];
    for (i = M + 1; i <= R; i++) right[i - M - 1] = arr[i];
    // 再合并成一个有序数组（从两个序列中选出最小值依次插入）
    i = 0; j = 0; k = L;
    while (i < LEFT_SIZE && j < RIGHT_SIZE) arr[k++] = left[i] < right[j] ? left[i++] : right[j++];
    while (i < LEFT_SIZE) arr[k++] = left[i++];
    while (j < RIGHT_SIZE) arr[k++] = right[j++];
}

void merge_sort(int arr[], int L, int R) {
    if (L == R) return;
    // 将 arr[L..R] 平分为 arr[L..M] 和 arr[M+1..R]
    int M = (L + R) / 2;
    // 分别递归地将子序列排序为有序数列
    merge_sort(arr, L, M);
    merge_sort(arr, M + 1, R);
    // 将两个排序后的子序列再归并到 arr
    merge(arr, L, M, R);
}
```

归并排序是**稳定排序**，它和选择排序一样，性能不受输入数据的影响，但表现比选择排序更好，它的时间复杂度始终为 O(nlogn)，但它需要额外的内存空间，空间复杂度为 O(n)。


### 基数排序

基数排序（Radix Sort）是**非比较型**排序算法，利用了“**桶**”的概念。基数排序不需要进行记录关键字间的比较，是一种**借助多关键字排序的思想对单逻辑关键字进行排序**的方法。比如数字100，它的个位、十位、百位就是不同的关键字。

那么，对于一组乱序的数字，基数排序的实现原理就是将整数按位数（关键字）切割成不同的数字，然后按每个位数分别比较。对于关键字的选择，有最高位优先法（MSD法）和最低位优先法（LSD法）两种方式。MSD 必须将序列先逐层分割成若干子序列，然后再对各子序列进行排序；而 LSD 进行排序时，不必分成子序列，对每个关键字都是整个序列参加排序。

**算法步骤**：

以 **LSD 法**为例：
1. 将所有待比较数值（非负整数）统一为同样的数位长度，数位不足的数值前面补零
2. 从最低位（个位）开始，依次进行一次排序
3. 从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列

如果要支持负数参加排序，可以将序列中所有的值加上一个常数，使这些值都成为非负数，排好序后，所有的值再减去这个常数。

![](https://blog.fiteen.top/2019/sorting-algorithm/radix-sort.gif)

```c
// 基数，范围0~9
#define RADIX 10

void radix_sort(int arr[], int n) {
    // 获取最大值和最小值
    int max = arr[0], min = arr[0];
    int i, j, l;
    for (i = 1; i < n; i++) {
        if (max < arr[i]) max = arr[i];
        if (min > arr[i]) min = arr[i];
    }
    // 假如序列中有负数，所有数加上一个常数，使序列中所有值变成正数
    if (min < 0) {
        for (i = 0; i < n; i++) arr[i] -= min;
        max -= min;
    }
    // 获取最大值位数
    int d = 0;
    while (max > 0) {
        max /= RADIX;
        d ++;
    }
    int queue[RADIX][n];
    memset(queue, 0, sizeof(queue));
    int count[RADIX] = {0};
    for (i = 0; i < d; i++) {
        // 分配数据
        for (j = 0; j < n; j++) {
            int key = arr[j] % (int)pow(RADIX, i + 1) / (int)pow(RADIX, i);
            queue[key][count[key]++] = arr[j];
        }
        // 收集数据
        int c = 0;
        for (j = 0; j < RADIX; j++) {
            for (l = 0; l < count[j]; l++) {
                arr[c++] = queue[j][l];
                queue[j][l] = 0;
            }
            count[j] = 0;
        }
    }
    // 假如序列中有负数，收集排序结果时再减去前面加上的常数
    if (min < 0) {
        for (i = 0; i < n; i++) arr[i] += min;
    }
}
```

基数排序是**稳定排序**，适用于关键字取值范围固定的排序。

基数排序可以看作是若干次“分配”和“收集”的过程。假设给定 n 个数，它的最高位数是 d，基数（也就是桶的个数）为 r，那么可以这样理解：共进行 d 趟排序，每趟排序都要对 n 个数据进行分配，再从 r 个桶中收集回来。所以算法的时间复杂度为 O(d(n+r))，在整数的排序中，r = 10，因此可以简化成 O(dn)，是**线性阶**的排序。

它占用额外内存，需要创建 r 个桶的额外空间，以及 n 个元素的额外空间，所以空间复杂度为 O(n+r)。
