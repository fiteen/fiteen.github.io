---
title: V2Ray + CDN 中转隐藏 IP
date: 2019-12-13 01:43:01
tags:
    - V2Ray
    - Cloudflare
    - CDN
    - DNS
categories: 科学上网
thumbnail: hide-ip.png
---

> ⚠️⚠️⚠️ **声明：本文内容仅限技术交流，若有用作商业或其他违规行为，与本人无关。**

<!--more-->

IP 又双叒叕被墙了？

刚换的 IP 还没捂热又凉了，怎么办？

下面教你一招，为你的 IP 加上双重保护锁，从此躲开“中奖”，快乐省钱又省心！

## 原理

先在 VPS 服务器上用 V2Ray 伪装成一个网站，再用 CDN 中转。这时流量传递的顺序是这样的：

![CDN 中转](visitor-firewall-cdn-vps-website.png)

主要实现就是两点：
一、借助 V2Ray 代理，将我们的流量被伪装成网站流量
二、利用 CDN 中转 V2Ray 的 WebSocket 流量

这样，GFW 只知道你与 CDN 之间的联系，不知道 VPS 的实际地址，并且 CDN 会有很多 IP 地址，GFW 也不会随意封这些 IP，毕竟也有很多正规网站在使用，因此可以基本保证 IP 的安全。

## 准备工作

于是，只要有了 VPS、域名和 CDN，就能实现这套方案：

- VPS：推荐 [BandwagonHost](https://bandwagonhost.com/)、[Vultr](https://www.vultr.com)、[Hostwinds](https://www.hostwinds.com)、[HostDare](https://manage.hostdare.com/index.php)、[谷歌免费薅一年](teach-you-to-build-a-free-Shadowsocks-service)。
- 域名：通过阿里云/腾讯云/华为云等购买域名，`.xyz`、`.top` 都是性价比比较高的选择。如果不想花钱也可以在 [freenom](http://www.freenom.com/nl/index.html) 上注册一个免费域名，运气好的话域名免费有效期可以达到12个月。
- CDN：推荐使用美国的 Cloudflare，优点是免费、无需备案。

## V2Ray

### 什么是 V2Ray

V2Ray 是继 Shadowsocks 之后一款非常好用的代理软件，甚至比 Shadowsocks 更优秀，它拥有更多可选择的协议和传输载体，还有强大的路由功能。

想要知道它的工作机制、本地策略、如何配置等细节可以查看 [V2Ray 官网](https://www.v2ray.com)。

###  搭建 V2Ray 服务

V2Ray 的配置其实是比较繁琐的，可以借助这个[一键安装脚本](https://github.com/233boy/v2ray/wiki/V2Ray一键安装脚本)快速配置。

### 安装脚本

通过 SSH 连接到 VPS 主机，以 root 用户输入以下命令来安装或卸载脚本：

```
bash <(curl -s -L https://git.io/v2ray.sh)
```

### 管理 V2Ray

安装完成后，直接在终端输入 `v2ray` 就可以进行管理。面板上会出现如下选项：

```
  1. 查看 V2Ray 配置
  2. 修改 V2Ray 配置
  3. 下载 V2Ray 配置 / 生成配置信息链接 / 生成二维码链接
  4. 查看 Shadowsocks 配置 / 生成二维码链接
  5. 修改 Shadowsocks 配置
  6. 查看 MTProto 配置 / 修改 MTProto 配置
  7. 查看 Socks5 配置 / 修改 Socks5 配置
  8. 启动 / 停止 / 重启 / 查看日志
  9. 更新 V2Ray / 更新 V2Ray 管理脚本
 10. 卸载 V2Ray
 11. 其他
```

输入 `2` 进入修改 V2Ray面板，面板上会出现如下选项：

```
  1. 修改 V2Ray 端口
  2. 修改 V2Ray 传输协议
  3. 修改 V2Ray 动态端口 (如果可以)
  4. 修改 用户ID ( UUID )
  5. 修改 TLS 域名 (如果可以)
  6. 修改 分流的路径 (如果可以)
  7. 修改 伪装的网址 (如果可以)
  8. 关闭 网站伪装 和 路径分流 (如果可以)
  9. 开启 / 关闭 广告拦截
```

输入 `2` 修改 V2Ray 传输协议，终端会输出当前的传输协议，如果不是 `WebSocket + TLS`，继续在终端输入 `4` 改成这个协议。如图下所示，依次点击回车键、输入正确的域名、将域名解析到指定的 IPv4 地址。

![输入域名并解析到指定地址](domain-name-resolution.png)

关于域名解析，以阿里云为例，像这样添加一条 A 记录类型即可。

![阿里云域名解析](aliyun-resolution.png)

接下来，你还会被询问是否要**设置分流路径**和**伪装的网址**，如果没有特殊要求，回复默认项即可。

修改配置完成后，终端会输出新的配置信息，形如：

![V2Ray 配置信息](v2ray-config.png)

你也可以通过以下命令进行**快速管理**：

- `v2ray info` 查看 V2Ray 配置信息
- `v2ray config` 修改 V2Ray 配置
- `v2ray link` 生成 V2Ray 配置文件链接
- `v2ray infolink` 生成 V2Ray 配置信息链接
- `v2ray qr` 生成 V2Ray 配置二维码链接
- `v2ray ss` 修改 Shadowsocks 配置
- `v2ray ssinfo` 查看 Shadowsocks 配置信息
- `v2ray ssqr` 生成 Shadowsocks 配置二维码链接
- `v2ray status` 查看 V2Ray 运行状态
- `v2ray start` 启动 V2Ray
- `v2ray stop` 停止 V2Ray
- `v2ray restart` 重启 V2Ray
- `v2ray log` 查看 V2Ray 运行日志
- `v2ray update` 更新 V2Ray
- `v2ray update.sh` 更新 V2Ray 管理脚本
- `v2ray uninstall` 卸载 V2Ray

配置完成后，我们将信息设置到支持 V2Ray 的客户端，比如集成了 [v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin) 的 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG)、[V2rayU](https://github.com/yanue/V2rayU)、[V2RayX](https://github.com/Cenmrev/V2RayX) 等。

这时候挂上代理访问，我们流量被伪装成网站流量，当别人访问你的域名时，打开的将是你设置的伪装网址，终于你的 IP 就不会直接暴露。

不过我们 `ping` 一下域名，就会发现，显示的还是原始 IP。那么下面要做的，就是利用 CDN 中转 V2Ray 的 WebSocket 流量。

## CDN 中转

这里用到的就是 Cloudflare 的**免费**的**自带防御功能**的 CDN 服务。

### 注册 Cloudflare 账号

前往[官网](https://www.cloudflare.com/)注册一个账号，流程很简单，只需验证一下有效邮箱。

### 使用 Cloudflare 管理域名

登录后账户就会引导你添加托管域名。

![添加域名](add-site.png)

注意这里必须使用**根域名**，并确保该域名不在于 Cloudflare 官方以及百度云加速以及其他合作商的系统中。

### 选择 Free 套餐

添加好网站后，选择套餐，这里点击第一个 Free 方案即可。

![选择套餐](select-a-plan.png)

### 补全域名的解析纪录

Cloudflare 会自动搜索域名的解析记录，如果有我们需要的 DNS 记录但是没有解析出来的，可以手动添加。

找到伪装域名的解析记录，修改它 DNS 解析记录的代理状态为 Proxied，也就是橘色云朵。

![修改 DNS 记录](revise-dns-record.png)

关于 **Proxy state**：

- Proxied：解析DNS，同时该记录要经过代理
- DNS only：只解析DNS，不代理

设置完成后，然后点击 Continue。

### 替换 DNS 服务器

![替换 DNS](replace-nameservers.png)

我们看到 Cloudflare 提示我们将原来的两台 DNS 服务器换成新分配的服务器。前往自己的域名服务商修改 DNS 之后，等待生效，我10分钟左右就收到了 “Status Active” 的通知邮件，等待时间正常来说不超过 24h。

## 效果

在[IP 地址查询网](https://tool.lu/ip/) 上输入域名，看到解析出的 IP 归属地为 CloudFlare 公司CDN 节点。

![域名 IP 查询](ip-query.png)

如此，原始 IP 就被隐藏了。

同样地，如果 IP 已经被墙，也可以通过这套方案拯救。因为域名托管在 CDN 上，只要 CDN 没有被封，它就可以帮助我们代理访问到 VPS，然后借助 VPS 上的代理科学上网。