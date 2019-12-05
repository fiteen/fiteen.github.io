---
title: 阿里云直播鉴权算法
date: 2017-06-29 10:30:49
tags: 直播
categories: 算法
---

> 阿里云官方给出的文档：[用户指南-直播鉴权](https://help.aliyun.com/document_detail/45210.html?spm=5176.2020520107.108.2.kYdTTA)

<!--more-->

## 参数描述
要配置出正确的鉴权，需要明确以下几个参数：

**推流地址**

完整的推流地址，形如：

`rtmp://video-center.alivecdn.com/{AppName}/{StreamName}?vhost={yourdomain}`

**鉴权类型**

阿里云CDN 兼容并支持 A、B、C 三种鉴权方式，具体见[ URL 鉴权方式](https://intl.aliyun.com/help/zh/doc-detail/27135.htm)。这里选择的是 A 类型

**鉴权KEY**

`privatekey` 字段用户可以自行设置

**时间戳**

时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。

**有效时间**

以秒为单位的整数时间，用来控制直播推流时效


## 鉴权方法

 用户访问加密 URL ：

> rtmp://video-center.alivecdn.com/{AppName}/{StreamName}?vhost={yourdomain}&auth_key={timestamp}-{rand}-{uid}-{hashvalue}

auth_key字段     | 描述
-------- | ---
timestamp |  失效时间=时间戳+有效时间，CDN 服务器拿到请求后，首先会判断请求中的失效时间是否小于当前时间，如果小于，则认为过期失效并返回 HTTP 403 错误。
rand | 随机数，一般设成0
uid    | 暂未使用（设置成0)
hashvalue     | 通过 md5 加密算法计算出的32位验证串

`hashvalue` 计算方式如下：

> sstring = /{AppName}/{StreamName}-{timestamp}-{rand}-{uid}-{privatekey}
> hashvalue = md5(sstring)

输入OBS中的鉴权内容如下：

> rtmpURL：rtmp://video-center.alivecdn.com/{AppName}
> 流密钥：{StreamName}?vhost={yourdomain}&auth_key={timestamp}-{rand}-{uid}-{hashvalue}

