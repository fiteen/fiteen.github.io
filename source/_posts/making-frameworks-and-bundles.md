---
title: iOS 中 framework 和 bundle 的制作
date: 2018-01-05 13:57:10
tags:
    - SDK
    - framework
    - bundle
categories: iOS
---

## Framework

**Framework** 是**资源的集合**，将静态库和其头文件包含到一个结构中，让 Xcode 可以方便地把它纳入到你的项目中。

<!--more-->

在运行时，库中按你的想法暴露需要的头文件，整个工程都可以调用暴露出来的接口和参数，这样减少了内存消耗，提高了系统的性能。

### 为什么使用 framework

与别人分享自己开发的组件，有两种方式。
- 直接提供源代码。
- 将组件代码编译成静态库，供他人调用。

第一种方式容易被人看到具体实现的细节，这些可能是你不想暴露出来的。此外，开发者也可能并不想看到你的所有代码，而仅仅是希望将功能的一部分植入到自己的应用中。

因此很多组件的封装采用第二种方式，这也是下文介绍的主要内容。

### 配置静态库工程

**步骤 1**：打开 Xcode ，依次点击 `Create a new Xcode project → iOS → Cocoa Touch Framework`，在 `Product Name` 中填写名称。（注：这就是最后 framework 的名称。）

**步骤 2**：假如你的项目依赖某些系统库，那么需要通过点击 `Targets → Build Phases → Link Binary with Libraries`，点击 `+` 符号将它们添加到工程中。

**步骤 3**：修改 Project 中的 iOS Deployment Target 版本号，选择你的框架最低支持的 iOS 版本。

**步骤 4**：如果组件中存在 `.xib` 文件，请确保 `TARGETS → Build Phases → Copy Bundle Resources` 下存在该 xib 文件。

**步骤 5**：将封装好的组件文件夹拖入到项目目录下，选择你要公开的头文件。

**步骤 6**：依次点击 `TARGETS → Build Phases → Headers`，目录下有：

- Public：存放公开的头文件，给外部调用。
- Private：存放私有的 Header，但头文件在编译之后还会存在。一般用来存放项目中需要调用但又不想给别人看到其内部实现的文件。
- Project：隐藏的文件。

### 导出 framework

选中 Scheme 选择当前项目，然后右边设备依次选中 Generic iOS Device（通用真机版本）和任一模拟器，分别编译（command + B），成功将会自动跳转到打出的 `.framework` 文件相应的目录下。

### 合成 framework

为了让用户能统一调用一个 framework ，还需要将二者合成为一个 framework 。这里介绍一种简单的方法：

1、新建一个 target，依次点击 `TARGETS 左下角的加号按钮 → Cross-platform → Other 下的 Aggregate`。

2、点击工程文件，选 `TARGETS → 刚才创建的 Aggregate → Build Phases → + → New Run Script Phases`。在当前栏目里会多出一个 `Run Script` ，在里面输入以下脚本：

```shell
#!/bin/sh
#要build的target名
TARGET_NAME=${PROJECT_NAME}
if [[ $1 ]]
then
TARGET_NAME=$1
fi
UNIVERSAL_OUTPUT_FOLDER="${SRCROOT}/${PROJECT_NAME}/"

#创建输出目录，并删除之前的 framework 文件
mkdir -p "${UNIVERSAL_OUTPUT_FOLDER}"
rm -rf "${UNIVERSAL_OUTPUT_FOLDER}/${TARGET_NAME}.framework"

#分别编译模拟器和真机的 framework
xcodebuild -target "${TARGET_NAME}" ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphoneos BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" clean build
xcodebuild -target "${TARGET_NAME}" ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphonesimulator BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" clean build

#拷贝 framework 到 univer 目录
cp -R "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${TARGET_NAME}.framework" "${UNIVERSAL_OUTPUT_FOLDER}"

#合并 framework，输出最终的 framework 到 build 目录
lipo -create -output "${UNIVERSAL_OUTPUT_FOLDER}/${TARGET_NAME}.framework/${TARGET_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${TARGET_NAME}.framework/${TARGET_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${TARGET_NAME}.framework/${TARGET_NAME}"

#删除编译之后生成的无关的配置文件
dir_path="${UNIVERSAL_OUTPUT_FOLDER}/${TARGET_NAME}.framework/"
for file in ls $dir_path
do
if [[ ${file} =~ ".xcconfig" ]]
then
rm -f "${dir_path}/${file}"
fi
done
#判断 build 文件夹是否存在，存在则删除
if [ -d "${SRCROOT}/build" ]
then
rm -rf "${SRCROOT}/build"
fi
rm -rf "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator" "${BUILD_DIR}/${CONFIGURATION}-iphoneos"
#打开合并后的文件夹
open "${UNIVERSAL_OUTPUT_FOLDER}"
```

3、使用脚本进行编译 (command + B)，成功后将会自动跳转到打出的 `.framework` 文件相应的目录下。

### 查看是否成功

**步骤 1**：打开终端，进入到你的 framework 文件所在的目录

```bash
cd ${yourFrameworkName}.framework
```

**步骤 2**：查看架构支持

```bash
lipo -info ${yourFrameworkName}.framework/${yourFrameworkName}
```

可以看到输出：

```bash
Architectures in the fat file: ${yourFrameworkName} are: i386 x86_64 armv7 arm64 (支持的架构显示在这)
```

**设备的 CPU 架构(指令集)**

- 模拟器
  - **i386**: 针对 intel 通用微处理器 32 位架构，如 iPhone 4s-5:
  - **x86_64**: 针对 x86 架构的 64 位处理器
- 真机
  - **armv6**: iPhone、iPhone 2、iPhone 3G、iPod 1G/2G（Xcode4.5 起已不再支持 armv6）
  - **armv7**: iPhone 3Gs、iPhone 4、iPhone 4s、iPod 3G/4G/5G、iPad、iPad 2、iPad 3、iPad Mini
  - **armv7s**: iPhone 5、iPhone 5c、iPad 4
  - **arm64**: iPhone 5s、iPhone 6(Plus)、iPhone 6s(Plus)、iPad Air(2)、Retina iPad Mini(2,3)
  - **arm64e**:  iPhone XS\XR\XS Max

### 引入 framework 的注意事项

引入的 framework 里存在分类的话，编译运行项目会报形如 `xxx unrecognized selector sent to class xxx` 的错误。

**解决办法：**

选中左边栏的项目文件，然后依次点击 `Targets → Build Settings → Linking → Other Linker Flags`，在里面添加 `-ObjC` 再次编译就能正常运行。

**补充知识：**

从 C 代码到可执行文件经历编译步骤是：

> 源代码 > 预处理器 > 编译器 > 汇编器 > 机器码 > 链接器 > 可执行文件。

在最后一步需要把 `.o` 文件和 C 语言运行库链接起来，这时需要用到 `ld` 命令。源文件经过一系列处理后，会生成对应的 `.obj` 文件，一个项目必然会有多个 `.obj` 文件，并且这些文件之间存在各种联系，如函数调用等。链接器做的事就是把目标文件和所用的一些库链接在一起形成一个完整的可执行文件。`Other Linker Flags` 设置的值实际上就是 `ld` 命令执行时后面所加的参数。下面介绍 3 个常用参数：

|     参数      | 描述                                                         |
| :-----------: | ------------------------------------------------------------ |
|    `-ObjC`    | 链接器会把静态库中所有的 Objective-C 类和分类都加载到最后的可执行文件中 |
|  `-all_load`  | 链接器会让所有找到的目标文件都加载到可执行文件中             |
| `-force_load` | 需要指定要进行全部加载的库文件的路径                         |

**注意：千万不要随便使用 `-all_load ` 这个参数！假如你使用了不止一个静态库，然后又使用了这个参数，那么很有可能会遇到 `ld: duplicate symbol` 错误，因为不同的库文件里面可能会有相同的目标文件，所以建议在遇到 `-ObjC` 失效的情况下使用 `-force_load` 参数。**

## Bundle

### 什么是 bundle

**Bundle** 可以理解为一个**资源目录**，并包含了程序中会用到的资源，如图像、声音、编译好的代码或 `nib` 文件等。

### 创建 bundle

Bundle 创建有两种方式：

#### 简单的创建 bundle

创建一个文件夹，强制重命名该文件夹为 `${yourBundleName}.bundle`。

#### 通过 Xcode 创建 bundle

**步骤 1**：新建一个项目，依次点击 `TARGETS → + → mac OS → Framework & Library → Bundle`，输入 `Product Name`即建立出 bundle 工程；

**步骤 2**：在 bundle 目录下添加需要的资源文件，编译之后在整个项目工程的 Products 文件夹下得到资源文件 bundle 。

这样做默认情况下 bundle 里面的 `png` 图片会被转为 `tiff` 的格式。因此在编译前需要做一步设置：找到 bundle 的工程，把 `Build Settings` 里的 `COMBINE_HIDPI_IMAGES` 设置为 `NO` 之后再编译运行。
