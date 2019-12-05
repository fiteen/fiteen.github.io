---
title: iOS 逆向分析之 class-dump
date: 2017-10-05 00:13:05
tags: 逆向
categories: iOS
---

## class-dump

class-dump 是一个命令行工具，通过利用 Objective-C 语言的 runtime 特性，提取存储在 Mach-O 文件中的类文件、协议、分类等信息，并统一表现在 .h 头文件中。

<!--more-->

### 安装

1. 下载 [class-dump-3.5.dmg](http://stevenygard.com/download/class-dump-3.5.dmg)（若链接无效，请戳[官方网址](!http://stevenygard.com/projects/class-dump/ )）

2. 打开终端，输入

   ```bash
   open /usr/local/bin
   ```

3. 将下载拿到的 class-dump 拷贝到 /usr/local/bin 目录下

4. 赋予其可执行权限，终端输入:

   ````bash
   sudo chmod 777 /usr/local/bin/class-dump
   ````

5. 至此安装成功，并可以通过 `class-dump --help` 查看用法和版本

### 使用

1. 下载一个 ipa 文件，先将文件改为 zip 格式，解压后得到 .app 的目标文件
2. 终端输入命令，格式为 `class-dump -H [.app文件路径] -o [输出文件夹路径]`
3. 假如此时输出的文件中未得到目标的 .h，结果中什么都没有或者只有一个 CDStructures.h，说明需要砸壳

## dumpdecrypted

从 AppStore 下载安装的 App 被苹果默认加了一层壳，需要通过砸壳进行逆向分析。

### 工具

- [dumpdecrypted.dylib](https://github.com/stefanesser/dumpdecrypted)

  - [下载](https://github.com/stefanesser/dumpdecrypted/archive/master.zip)

  - 编译安装

    1、在终端进入下载的解压文件的目录：

    ````bash
    cd [dumpdecrypted-master's filePath]
    ````

    2、执行 `ls` 里面存在三个文件：Makefile、README、dumpdecrypted.c

    3、执行 `make` ，在当前目录下会多出 dumpdecrypted.dylib 和 dumpdecrypted.o，前者就是我们需要的工具

- 一台越狱手机

### 操作步骤

使用越狱手机前往 AppStore 下载目标 App 并打开。

#### 1. 使用 ssh 连接手机

1.1 越狱手机和电脑连同一个 wifi，查看手机所处当前网络的 IP 地址，打开终端 A，输入指令：

````bash
ssh root@[手机当前网络的 IP 地址]
````

1.2 通过命令`ps -e`找到目标 App 对应的进程，如果该 App 为当前打开的应用，可以关注最下面的几条进程，形如：

```bash
[进程号] ??         [时间] [目标 App 在手机中路径]
```

路径形如 

`/var/mobile/Containers/Bundle/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xx.app/xx
`

将其记录下来备用。


1.3 附加进程指令：`cycript -p [进程号]`

获取 App 在沙盒 Documents 的路径：

```
[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask]
```

路径形如：`/var/mobile/Containers/Data/Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents`

将其记录下来备用。

#### 2. 注入 dumpdecrypted.dylib

2.1 新开终端 B（可使用快捷键 command + T）

2.2 使用 scp 指令将 dumpdecrypted.dylib 拷贝到目标 App 的 Documents 目录下。

指令为：

```bash
scp [dumpdecrypted.dylib 所在的完整路径] root@[手机当前网络的 IP 地址]:[目标 App 在手机中路径]
```

终端会提示输入密码，默认为 ` alpine`。

#### 3. 砸壳

3.1 回到终端 A，`cd`  进入步骤 1.3 中 App 在沙盒 Documents 的路径

3.2 执行如下指令：

```bash
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib [步骤 1.2 中目标 App 在手机中路径]
```

3.3 执行 `ls` 指令查看当前目录下是否有 `.decrypted` 的文件来确定砸壳是否成功

#### 4. class-dump 导出 App 头文件

4.1 回到终端 B，将.decrypted 文件拷贝到电脑目录下，指令为：

```bash
scp root@[手机当前网络的 IP 地址]:[步骤 1.3 中App 在沙盒 Documents 的路径]/WeChat.decrypted [自定义的电脑目录]
```

终端会提示输入密码，默认为 ` alpine`。

4.2 通过如下指令获取目标 App 的所有头文件

```bash
class-dump -s -S -H --arch [指令集] [步骤 4.1 中的.decrypted 文件路径] -o [自定义的输出目录]
```

指令集需对应当前越狱手机的型号，参考下表：

```
armv6：iPhone | iPhone2 | iPhone3G
armv7：iPhone3GS | iPhone4 | iPhone4S
armv7s：iPhone5 | iPhone5C
arm64：iPhone5S | iPhone6 | iPhone6Plus | iPhone6S  | iPhone6SPlus | iPhone7 | iPhone7Plus | iPhone8 | iPhone8Plus | iPhoneX
```
