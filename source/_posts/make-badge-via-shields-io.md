---
title: 为你的 GitHub 开源项目制作高大上的徽标
date: 2019-10-01 22:17:23
tags: 
    - GitHub
    - shield.io
categories: 技巧
---

经常逛 GitHub 的同学会发现，很多优秀的开源框架里都会出现这样的小徽标。

<!--more-->

![](badge-sample.png)

它的实现其实非常简单，借助一些小工具即可，比如：[shield.io](https://shields.io)、[Badgen](https://badgen.net)、[Open Source Badges](https://ellerbrock.github.io/open-source-badges/)、[Version Badge](https://badge.fury.io)、[FOR THE BADGE](https://forthebadge.com) 等。这里推荐最经典全面的 shield.io。

## 静态徽标

一个简单的静态徽标链接的标准格式为：

```
https://img.shields.io/badge/${label}-${message}-${color}.svg
```

**label** 表示徽标左半部分信息，可选填，**message** 表示徽标右半部分信息，**color** 表示徽标右半部分的背景颜色。`.svg` 可以省略。

如果徽标里的文字包含 `-`，需要写成 `--`，比如：

那么 ![](https://img.shields.io/badge/language-Objective--C-green) 徽标，就要这样写：

```
// 加上 .svg
![](https://img.shields.io/badge/language-Objective--C-green.svg)
// 省略 .svg
![](https://img.shields.io/badge/language-Objective--C-green)
```

如果你不需要两部分信息，比如我的带链接的博客徽标 [![](https://img.shields.io/badge/@FiTeen-grey)](https://blog.fiteen.top) ，就可以这样写：

```
[![](https://img.shields.io/badge/@FiTeen-grey)](https://blog.fiteen.top)
```

### color

关于 **color**，你可以直接填入**颜色英文**，比如：

<span>![](https://img.shields.io/badge/-red-red) ![](https://img.shields.io/badge/pink-pink) ![](https://img.shields.io/badge/-orange-orange.svg) ![](https://img.shields.io/badge/yellow-yellow) ![](https://img.shields.io/badge/brightgreen-brightgreen) ![](https://img.shields.io/badge/green-green) ![](https://img.shields.io/badge/yellowgreen-yellowgreen) ![](https://img.shields.io/badge/blue-blue) ![](https://img.shields.io/badge/royalblue-royalblue) ![](https://img.shields.io/badge/cyan-cyan.svg) ![](https://img.shields.io/badge/blueviolet-blueviolet) ![](https://img.shields.io/badge/purple-purple) ![](https://img.shields.io/badge/grey-grey) ![](https://img.shields.io/badge/lightgrey-lightgrey)</span>

也可以用这些**特殊词汇**来代替颜色：

<span>![](https://img.shields.io/badge/success-success) ![](https://img.shields.io/badge/important-important) ![](https://img.shields.io/badge/critical-critical) ![](https://img.shields.io/badge/informational-informational) ![](https://img.shields.io/badge/inactive-inactive)</span>

或者直接通过**十六进制颜色码**，比如：

<span>![](https://img.shields.io/badge/ffb6c1-ffb6c1) ![](https://img.shields.io/badge/7b68ee-7b68ee) ![](https://img.shields.io/badge/5f9ea0-5f9ea0)</span>

### 样式

目前支持五种徽标样式，具体实现就是在 svg 路径后面拼接参数。`flat` 是默认样式。

- `?style=plastic` ![](https://img.shields.io/badge/style-plastic-green?style=plastic.svg)
- `?style=flat` ![](https://img.shields.io/badge/style-flat.svg-green?style=flat)
- `?style=flat-square` ![](https://img.shields.io/badge/style-flat--square-green.svg?style=flat-square)
- `?style=for-the-badge` ![](https://img.shields.io/badge/style-for--the--badge-green.svg?style=for-the-badge)
- `?style=social` ![](https://img.shields.io/badge/style-social-green?style=social)

除此之外，还有一些 query string 参数：

- `label` - 覆盖原有的 label 文本内容。
- `labelColor` 或 `labelA` - 覆盖原有的 label 背景颜色，默认颜色是 `grey`。注意这里不能用特殊词汇表示颜色。
- `logo` - 可以插入以下名称之一的徽标（bitcoin、dependabot、discord、gitlab、npm、paypal、serverfault、stackexchange、superuser、telegram、travis）或简单图标。使用简单图标站点上显示的名称来引用简单图标。如果名称中包含空格，用短划线 `-` 代替(例如: `?logo=visual-studio-code`) ![](https://img.shields.io/badge/IDE-VSCode-green?logo=visual-studio-code)。或者插入自定义徽标 logo 图像（高度≥14px）。
- `logoColor` - 设置徽标 logo 的颜色。
- `logoWidth` - 设置徽标 logo 的水平宽度。
- `link` - 指定徽标左/右侧部分的点击操作，格式为：`?link=${label-url}&link=${message-url}`。
- `color` 或 `colorB` - 覆盖原有的 message 背景颜色。
- `cacheSeconds` - 设置 HTTP 缓存生存期（规则适用于根据每个徽章推断默认值，低于默认值的任何指定值都将被忽略）。还支持传统名称“ maxAge”。

## 动态徽标

动态徽标是指会随着项目状态变化，自动更新状态的徽标。GitHub 项目中常用的动态徽标有：

### build 状态

- **Travis（.org）：**`https://travis-ci.org/:user/:repo`

- **Travis（.org）branch：**`https://travis-ci.org/:user/:repo/:branch`

- **GitHub Workflow Status：**`/github/workflow/build/:user/:repo/:workflow`

- **GitHub Workflow Status (branch)：**`https://github.com/:user/:repo/workflows/build/badge.svg?branch=${branch}`

例如 [AFNetworking](https://github.com/AFNetworking/AFNetworking) 的 build 状态为：[![Build Status](https://travis-ci.org/AFNetworking/AFNetworking.svg)](https://travis-ci.org/AFNetworking/AFNetworking)

```
[![Build Status](https://travis-ci.org/AFNetworking/AFNetworking.svg)](https://travis-ci.org/AFNetworking/AFNetworking)
```

而 [Kingfisher](https://github.com/onevcat/Kingfisher)-master 分支的 build 状态为：[![Build Status](https://github.com/onevcat/kingfisher/workflows/build/badge.svg?branch=master)](https://github.com/onevcat/Kingfisher/actions?query=workflow%3Abuild)

```
[![Build Status](https://github.com/onevcat/kingfisher/workflows/build/badge.svg?branch=master)](https://github.com/onevcat/Kingfisher/actions?query=workflow%3Abuild)
```


要知道项目在其它平台的持续集成状态，具体参照 [shields.io - build](https://shields.io/category/build)

### 许可协议

- **Cocoapods：**`/cocoapods/l/:spec`

- **GitHub：**`/github/license/:user/:repo`

- **NPM：**`/npm/l/:packageName`

比如 [Kingfisher](https://github.com/onevcat/Kingfisher) 许可协议支持 Cocoapods 和 GitHub 两种写法： [![license](https://img.shields.io/cocoapods/l/Kingfisher?style=flat)](https://raw.githubusercontent.com/onevcat/Kingfisher/master/LICENSE) 和 [![license](https://img.shields.io/github/license/onevcat/Kingfisher)](https://raw.githubusercontent.com/onevcat/Kingfisher/master/LICENSE)

[debug](https://github.com/visionmedia/debug )的许可协议为 [![license](https://img.shields.io/npm/l/debug)](https://github.com/visionmedia/debug/blob/master/LICENSE)

```
// Kingfisher Cocoapods License：
[![license](https://img.shields.io/cocoapods/l/Kingfisher?style=flat)](https://raw.githubusercontent.com/onevcat/Kingfisher/master/LICENSE)
// Kingfisher GitHub License
[![license](https://img.shields.io/github/license/onevcat/Kingfisher)](https://raw.githubusercontent.com/onevcat/Kingfisher/master/LICENSE)
// debug NPM Licese
[![license](https://img.shields.io/npm/l/debug)](https://github.com/visionmedia/debug/blob/master/LICENSE)
```

要知道项目在其他平台的许可协议，具体参照 [shields-license](https://shields.io/category/license)。

### 平台&版本支持

- **Cocoapods Platform**：`/cocoapods/p/:repo`

- **Cocoapods Compatible**：`/cocoapods/v/:repo`

比如 [Kingfisher](https://github.com/onevcat/Kingfisher) 当前支持的平台有 ![platform](https://img.shields.io/cocoapods/p/Kingfisher)，pod 版本号为 ![version](https://img.shields.io/cocoapods/v/Kingfisher)

```
// Platform Support
![platform](https://img.shields.io/cocoapods/p/Kingfisher)
// Version Support
![version](https://img.shields.io/cocoapods/v/Kingfisher)
```

要知道相关的其它信息，具体参照 [shields.io - platform & version support](https://shields.io/category/platform-support)

### 代码测试覆盖率

针对不同的代码测试平台，有不同的获取方法，例如：

- **Codecov：**`https://codecov.io/github/:user/:repo/coverage.svg?token=${token}`

- **Codecov Branch：** `https://codecov.io/github/:user/:repo/coverage.svg?branch=${branch}&token=${token}`

以 [AFNetworking](https://github.com/AFNetworking/AFNetworking/master)-master 分支为例：[![codecov.io](https://codecov.io/github/AFNetworking/AFNetworking/coverage.svg?branch=master)](https://codecov.io/github/AFNetworking/AFNetworking?branch=master)

```
[![codecov.io](https://codecov.io/github/AFNetworking/AFNetworking/coverage.svg?branch=master)](https://codecov.io/github/AFNetworking/AFNetworking?branch=master)
```

要知道项目在其它平台的测试覆盖率，具体参照 [shields.io - coverage](https://shields.io/category/coverage)。

### 项目信息

- **GitHub Followers：**`/github/followers/:user?label=Follow`

- **GitHub Forks：**`/github/forks/:user/:repo?label=Fork`

- **GitHub Stars：**`/github/stars/:user/:repo?style=social`

- **GitHub Watchers：**`/github/watchers/:user/:repo?label=Watch`

以我本人的 [GitHub](https://github.com/fiteen) 和项目 [HTCart](https://github.com/fiteen/HTCart) 为例：

<span>![followers](https://img.shields.io/github/followers/fiteen?label=Follow) ![forks](https://img.shields.io/github/forks/fiteen/HTCart?label=Fork) ![stars](https://img.shields.io/github/stars/fiteen/HTCart?style=social) ![watchers](https://img.shields.io/github/watchers/fiteen/HTCart.svg?label=Watchers)</span>

```
// Followers
![followers](https://img.shields.io/github/followers/fiteen?label=Follow)
// Forks
![forks](https://img.shields.io/github/forks/fiteen/HTCart?label=Fork)
// Stars
![stars](https://img.shields.io/github/stars/fiteen/HTCart?style=social)
// Watchers
![watchers](https://img.shields.io/github/watchers/fiteen/HTCart.svg?label=Watchers)
```

### 下载量

- **GitHub All Releases：**`/github/downloads/:user/:repo/total`

- **GitHub Releases：**`/github/downloads/:user/:repo/:tag/total`

以 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG) 为例：

<span>![GitHub All Releases](https://img.shields.io/github/downloads/shadowsocks/ShadowsocksX-NG/total) ![GitHub Releases](https://img.shields.io/github/downloads/shadowsocks/ShadowsocksX-NG/v1.7.1/total)</span>

```
// 总下载量
![GitHub All Releases](https://img.shields.io/github/downloads/shadowsocks/ShadowsocksX-NG/total)
// v1.7.1 的下载量
![GitHub Releases](https://img.shields.io/github/downloads/shadowsocks/ShadowsocksX-NG/v1.7.1/total)
```

### 其他

当然，可支持动态的徽标还有很多，本文就不一一列举，有兴趣的可以直接在[官网](https://shields.io)查询。