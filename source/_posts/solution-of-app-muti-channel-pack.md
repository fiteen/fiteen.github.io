---
title: App 多渠道打包方案
date: 2019-03-03 02:41:39
tags: App 打包
categories: 
   - [iOS]
   - [Android]
---

众所周知，渠道包是国内 Android 应用市场中常用的分发方式。渠道包中会包含不同的渠道信息，方便我们后续统计 App 在各分发渠道的下载量、用户量、留存率等，有针对地调整应用内容或是推广方案等。随着国内 iOS 应用上架越来越难，衍生出了很多企业包，为了方便采集数据，也会用多渠道的方案。

<!--more-->

另外，项目进展过程中，可能会出现一些临时新增渠道的需求，这时回到工程中重新打包是比较费时的，有没有办法加快打包速度呢？下文中分享了一些方案。

## iOS 多渠道打包方案

iOS 打渠道包目前想到的就只有两种方式，一种是通过[多 target 方式](#muti-target-way)，另一种是[修改 plist 文件方式](#revise-plist-way)。

### <span id="muti-target-way">多 target 方式</span>

点击项目中的 target，右键选择 `Duplicate`。可以修改下图标红框的三处：target 名称、plist 名称和 scheme 名称。

{% asset_img target-copy.png %}

判断当前是哪个 target，可以通过添加宏定义实现，方式就是在 `Build Settings` 找到 `Preprocessor Macros`，填入宏定义名。

代码中这样判断：
```objective-c
#ifdef  TARGET1MACROS
    // target1
#elif defined TARGET2MACROS
    // target2
#endif
```

具体打包脚本就不介绍了，读者可以自行网上搜索，这种方式的缺点是一个渠道打一次，效率较低。下面着重分享修改 plist 的批量打包方式。

### <span id="revise-plist-way">修改 plist 方式</span>

下面用一个简单的 Demo 演示一下：

**第一步：**创建工程名为 MultiChannelDemo 的项目，并在项目中新建一个 `Channel.plist` 文件，plist 中设置 Channel 字段，值为 channel01。然后在页面上设置一个 label 标签用于显示当前的渠道名称，渠道名可以通过下面的代码获取到：

```objective-c
NSDictionary *channelDic = [NSDictionary dictionaryWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"Channel" ofType:@"plist"]];
NSString *channel = channelDic[@"Channel"];
```
**第二步：**把这个项目用可用的证书正常打一个母包，解压这个 ipa 包可以获得一个名为 `Payload` 的文件夹，里面是一个 .app 文件，右键显示其包内容，内容如下：。

{% asset_img package-content.png %}

可以看到，里面的 `Channel.plist` 也就是在前面工程中新建的存储渠道信息的 plist，我们会修改里面的 Channel 再生成新的渠道包。

**第三步：**提取描述文件用于重签名，上一步中 Payload 的文件夹里有一个 `embedded.mobileprovision` 文件，这就是我们需要的文件。

**第四步：**新建一个纯文本，里面输入你要新增的渠道号，如：

{% asset_img channel-list-txt.png %}

**第五步：**写一个脚本文件，内容如下：

```shell
#!/bin/bash

# 输入的包名

name="MultiChannelDemo"

echo "------SDK渠道包----------"

appName="${name}.app"

plistBuddy="/usr/libexec/PlistBuddy"

configName="Payload/${appName}/Channel.plist"

ipa="${name}.ipa"

# 输出的新包所在的文件夹名

outUpdateAppDir="ChannelPackages"

# entitlements.plist路径

entitlementsDir="entitlements.plist"

# 切换到当前目录

currDir=${PWD}

cd ${currDir}

echo "-----${currDir}"

rm -rf Payload

# 解压缩-o：覆盖文件 -q：不显示解压过程

unzip -o -q ${ipa}

# 删除旧的文件夹，重新生成

rm -rf ${outUpdateAppDir}

mkdir ${outUpdateAppDir}

# 删除旧的 entitlements.plist，重新生成

rm -rf ${entitlementsDir}

/usr/libexec/PlistBuddy -x -c "print :Entitlements " /dev/stdin <<< $(security cms -D -i Payload/${appName}/embedded.mobileprovision) > entitlements.plist

echo "------------------------开始打包程序------------------------"

# 渠道列表文件开始打包

for line in $(cat ChannelList.txt)

# 循环数组，修改渠道信息

do

# 修改 plist 中的 Channel 值

$plistBuddy -c "Set :Channel $line" ${configName}

# app 重签名

rm -rf Payload/${appName}/_CodeSignature

cp embedded.mobileprovision "Payload/${appName}/embedded.mobileprovision"

# 填入可用的证书 ID

codesign -f -s "iPhone Distribution: XXXXXX." Payload/${appName}  --entitlements ${entitlementsDir}

# 若输出 Payload/MultiChannelDemo.app: replacing existing signature 说明重签名完成

# 压缩 -r:递归处理，将指定目录下的所有文件和子目录一并处理 -q:不显示处理过程

zip -rq "${outUpdateAppDir}/$line.ipa" Payload

echo "........渠道${line}打包已完成"

done
```
脚本里的信息请根据你实际情况修改，。到这里准备工作都完成了，需要的文件如下图所示：

{% asset_img prepare-file.png %}

**第六步：**在当前目录下执行脚本文件：

```bash
sh ChannelPackage.sh
```

打包完成后生成的 `ChannelPackages` 文件夹下，就是我们需要的渠道包：

{% asset_img package-result.png %}

这种自动化打包的方式，可以规避掉 Xcode 本身打包编译的部分时间，快速出包。

## Android 多渠道打包方案

下文介绍的是美团技术团队开源的 [Walle](https://github.com/Meituan-Dianping/walle)，它有 [Gradle 插件](#gradle-way)和[命令行](#command-way)两种使用方式，前者快速集成，后者满足自定义需求。

### <span id="gradle-way">Gladle 插件方式</span>

#### 配置 build.gradle

在项目根目录下的 `build.gradle` 文件中添加 Walle 插件依赖：

```java
buildscript {
    dependencies {
        classpath 'com.meituan.android.walle:plugin:1.1.6'
    }
}
```

在 app 目录下的 `build.gradle` 文件中 apply 插件：

```java
apply plugin: 'walle'

dependencies {
    compile 'com.meituan.android.walle:library:1.1.6'
}
```

#### 配置插件

在 app 目录下的 `build.gradle` 文件中进行渠道配置：

```java
walle {
    // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels");
    // 定制渠道包的APK的文件名称
    apkFileNameFormat = '${appName}_v${versionName}_${channel}.apk';
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```
渠道配置文件里的内容格式详见：[渠道配置文件示例](https://github.com/Meituan-Dianping/walle/blob/master/app/channel)。

#### 如何获取渠道信息

在需要填写渠道信息的地方引用这段代码：

```java
String channel = WalleChannelReader.getChannel(this.getApplicationContext());
```

#### 如何生成渠道包

用 `assemble${variantName}Channels` 指令，导出 apk 包。

### <span id="command-way">命令行方式</span>

通过命令行方式，可以不打开 IDE，直接导出新渠道的 apk。步骤如下：

首先，新建一个文件夹，取用一个上面步骤导出的 apk 包，再下载 [walle-cli-all.jar](https://github.com/Meituan-Dianping/walle/releases)，两者都放置在这个文件夹目录下。

然后，在文件夹目录下执行命令：

```bash
java -jar walle-cli-all.jar put -c ${channelName} ${apkName}.apk
```

若上面的命令执行成功，会在当前目录下生成新的渠道包，名称为 `${apkName}_${channelName}.apk`

如果要批量写入渠道，可以这样，渠道之间用逗号隔开：

```bash
java -jar walle-cli-all.jar batch -c ${channelName0},${channelName1},${channelName2} ${apkName}.apk
```

或者指定渠道配置文件：

```bash
java -jar walle-cli-all.jar batch -c ${channelFile} ${apkName}.apk
```

> 如果要写入额外信息，参考[官方文档](https://github.com/Meituan-Dianping/walle/blob/master/walle-cli/README.md)。

如果要检查/显示渠道，命令为：

```bash
java -jar walle-cli-all.jar show ${apkName}.apk
```

Walle 现在既能满足新应用签名方案对安全性的要求，也能满足对渠道包打包时间的要求，有需要的可以尝试。
