---
title: App Store 审核经验
date: 2018-06-14 15:56:22
tags: 上架审核
categories: iOS
---
## 相关资料

### 审核指南

- [《App Store 审核指南》](https://developer.apple.com/cn/app-store/review/guidelines/) 
- [《苹果开发者计划许可协议》](https://developer.apple.com/terms/)

<!--more-->

苹果官方会不定期更新 Guidelines 和 PLA，请及时关注。

### 关键概念

- [iTunes Connect](https://developer.apple.com/support/itunes-connect/cn/)

  iTunes Connect 是一套以网页为基础的工具，用于管理在 App Store 上销售的面向 iPhone、iPad、Mac、Apple Watch、Apple TV 和 iMessage 的 app；同时也用于管理 iTunes Store 和 iBooks Store 上的内容。开发者通过 iTunes Connect 提交和管理 app，邀请用户使用 TestFlight 进行测试，添加税务和银行信息，以及访问销售报告等。

- 元数据

  元数据指的是 iTunes Connect 中输入的 App 信息和平台版本信息——例如，App 名称、描述、关键词和屏幕快照。此信息的部分显示在 App Store 产品页面，并且可以被本地化。

- 二进制文件

  包含在 ipa 包中的一个可执行文件，提审时需重点检查包括但不限于 info.plist、包／文件大小、icon 规格、私有 API、第三方 SDK、64 位等内容。

### 审核状态

开发者在审核过程中需要特别关注的两个 [App 状态](https://help.apple.com/itunes-connect/developer/?lang=zh-cn#/dev18557d60e)为：

- 正在等待审核（Waiting For Review）

  您已经提交了一个新的 App 或者更新了一个版本。Apple 已经收到了您的 App 但还没有开始审核。在该状态下可以： 

  - [将构建版本从审核中移除](https://help.apple.com/itunes-connect/developer/?lang=zh-cn#/dev04f55d711)
  - 编辑某些 [App 信息](https://help.apple.com/itunes-connect/developer/?lang=zh-cn#/dev219b53a88)

- 正在审核（In Review）

  Apple 正在审核您的 App。您可以[将构建版本从审核中移除](https://help.apple.com/itunes-connect/developer/?lang=zh-cn#/dev04f55d711)。

一般这两个过程都会在 24-48 小时内完成，即从你提交到审核完成正常应在 2 天内结束，App 首次提交除外。

当 App 被拒超过三次，“正在等待审核”过程会延长，极有可能持续一周；若未按照苹果的要求操作，可能被拉入黑名单，“正在审核”过程无限延长。

## 近期被拒案例及应对措施

### 账号资质问题

对于监管敏感的行业和应用，App Store 的审核会更为苛刻。这类案例主要体现在理财、借贷、医疗类的 App，相关的应对方法有：

1. 证明你的公司，有提供相关资质。

   如果 App 的公司主体具备资质，直接讲资质证明（如营业执照、政府背书）发给苹果审核团队；若不具备，需要将 App 放在有资质的公司主体的账号下提交。

   如果苹果审核团队方面对 App 的性质存在误解，提供相关证明并及时沟通。

2. 如果是个人开发者账号提交的应用，须升级为企业开发者账号后再提交。

3. 如果是其他开发者账号（比如外包）替你开发，须将其他开发账号添加到你的苹果开发者账号下（在“用户和职能-添加 iTunes Connect 用户”操作）。

4. 尽可能体现 App 产品与公司品牌的关联性，包括但不限于以下几点：

   -  App 名称的择定
   - 在 App 的“关于我们”中，中英文介绍公司
   - 提交“软件著作权登记证书”，或者“商标证书”
   - 向苹果审核团队阐述 App 功能的运营主体、技术支持网站等

5. 设置开关，将敏感内容在审核期间隐藏，审核过后再显示。但近期苹果已经发现这一现象，会不定期抽查过审应用。这种做法也有被竞争对手举报的可能，一旦被查到可能面临被直接下架的风险。

### 元数据不规范

2018 年伊始，苹果爸爸就抛出了重磅炸弹——苹果 2.1 狗年大礼包。我们需要对照[元数据规范](https://help.apple.com/itc/appsspec/#/)对本地信息进行修改和调整。

如果不需要更新 ipa 包，可以直接在被拒信息下面回复，明确告知对方：App 不存在这些问题或者我们已经对相关资源和功能作出了调整，请重新审核。切勿不沟通，直接重新提包，会被苹果认定默认存在大礼包中提及的问题。

iTC 中在上传屏幕快照时，以 5.5 寸为基准，条件允许时，为不同机型定制不同的屏幕快照更佳。

### 内购

订阅、游戏内货币、游戏关卡、课程、会员等非实物交换类的虚拟物品，必须且只允许走内购渠道。此外，需要注意以下几点：

- 支付页面不能使用网页作为载体，苹果会认为存在变更支付方式的可能
- 类别（如消耗型/非消耗性、自动续订/非续订）需要选择正确
- 提高产品审核通过率，iTC 中信息尽可能补充完整

### 隐私

在 Apple 生态体系中，保护用户隐私总是第一要务。当需要访问用户的相册、相机、通讯录、位置、日历等，App 描述中应当注明 app 会要求访问哪些内容类型 (例如，位置、通讯录和日历等)，并说明当用户不授予许可时，app 的哪些功能会无法正常工作。

## 如何提高过审速度

### 沟通原则

- 尊重

  回复时称对方为审核员，沟通过程中保持严肃、友好、认同的态度，对给出任何审核结果表示感谢。

- 积极

  及时主动告知审核员我方的处理进度。中英文表达皆可，面对积极回复且礼貌的开发者，审核人员更愿意给出直接的意见。

### 以往经验

- In Review 状态不要手动撤回，可能会导致后续审核速度变慢。
- 对被拒原因不认可，可以直接在被拒消息后申诉。即使回复后直接重新提交新版本，审核员也会看到消息，可以在消息中告知，已经按照要求进行修改，这种情况下苹果的处理效率会高一些。
- 处理当前的审核结果一般是同一个审核员，提审后及时查看审核状态，一旦被拒及时回复，可以得到尽可能快对回应和处理。
- 提审之前，请先反复检查，避免低级问题或常见问题遗漏到苹果审核人员手中。若连续审核通过，后续审核速度会越来越快，反之，若连续被拒极有可能进入黑名单，审核速度越来越慢。
- 遇到竞争对手侵权可以向苹果投诉，但不要多人重复投诉，不然可能会拉长处理时间。

