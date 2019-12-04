---
title: 手把手教你免费搭建 Shadowsocks 服务
date: 2018-12-27 21:43:02
tags: 代理
categories: 技巧
---

### 一、申请免费试用GCP

每位新注册的用户可以在谷歌云平台 [GCP](https://cloud.google.com/free/) (Google Cloud Platform)获得第一年$300 的免费赠送额度。一年后若不主动选择继续使用不会扣费的。

注册账户的准备工作：

1、可用的 VPN，用于正常访问 GCP；

2、具有 VISA、MasterCard 等海外支付功能的信用卡一张；

<!--more-->

有账户的可以直接登录，没有的就创建一个。

 {% asset_img create-account.jpg 创建账号 %}

如果阅读英文不习惯，可以将左下方的语言改成简体中文。登录成功后进入 [GCP 试用申请](https://console.cloud.google.com/freetrial)：

**第1步 - 同意条款**：注意选择国家/地区时避免选择“中国”，因为根据 Google Cloud 的政策，不支持中国使用，直接使用默认的“美国”即可。

 {% asset_img apply-for-a-free-trial.jpg 申请免费试用 %}

**第2步 - 填写客户信息和付款方式**

客户信息的账户类型选择“个人”，通过[虚拟美国人信息生成工具](http://www.haoweichi.com/Index/random)，补充完成“姓名和地址”信息。

 {% asset_img account-information.jpg 客户信息 %}

填写付款方式时，务必填入**正确真实**的信用卡信息，不能再使用生成工具里的虚拟信息。可以取消“信用卡或借记卡账单邮寄地址与上述地址相同”的勾选，输入真实的地址。

 {% asset_img payment-method.jpg 付款方式 %}

申请成功会扣除$1，验证后将返回。至此，试用 GCP 免费申请完成。



### 二、部署虚拟机

#### 1、修改防火墙

在菜单中依次点击 【网络】 –>【VPC 网络】 –>[【防火墙规则】](https://console.cloud.google.com/networking/firewalls/list)–>【创建防火墙规则】，如下图创建一条入站规则：

 {% asset_img firewall-rules.jpg 防火墙规则 %}

注意点：

- 目标：网络中的所有实例；如果选择指定标签，需要在后续的配置中输入标签

- IP地址范围： 0.0.0.0/0

- 协议和端口：全部允许

#### 2、保留静态地址

在菜单中依次点击 【网络】 –>【VPC 网络】 –>[【外部 IP 地址】](https://console.cloud.google.com/networking/addresses/list)–>【保留静态 IP】

 {% asset_img static-ip.jpg 保留静态地址 %}

静态 IP 只能申请一个。区域可以选择亚洲东部、欧洲、美国等地，推荐使用 asia-east1，对应台湾地区，访问速度较快。

#### 3、创建计算引擎

在菜单中依次点击 【计算】 –>【Compute Engine】 –>[【VM 实例】](https://console.cloud.google.com/networking/addresses/list)–>【创建实例】

 {% asset_img create-instance.jpg 创建实例 %}

注意点：

- 区域：与创建静态地址时一致

- 机器类型：最便宜的“微型”即可

- 启动磁盘：Ubuntu 16.04 LTS Minimal

展开“管理、安全、磁盘、网络、单独租用”，外部 IP 选择第2步的静态 IP。到这里，虚拟机部署完成。

 {% asset_img VM-instance.jpg VM实例 %}



### 三、搭建 SSR + BBR

在 VM 实例列表中找到刚才创建好的实例，点击上图红框内的 SSH，会弹出终端，如下图所示。如果用的是谷歌浏览器可以使用 [SSH 插件](https://chrome.google.com/webstore/detail/ssh-for-google-cloud-plat/ojilllmhjhibplnppnamldakhpmdnibd)

 {% asset_img SSH.jpg SSH %}

- 获得 root 权限

```
sudo -i
```

- 检查内核版本

```
uname –sr
```

正常情况下，当前的内核版本都是超过 4.9，无需升级，可以直接进入下一步；如果需要升级，按照以下步骤进行

```
// 更新系统
apt update
apt upgrade
// 安装指定的新内核
apt install linux-image-4.10.0-20
// 卸载旧内核
apt autoremove
// 启用新内核
update-grub
// 重启
reboot
// 获得 root 权限
sudo -i
// 验证内核版本
uname –r
```

- 写入配置

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

- 配置生效

```
sysctl -p
```

- 检验是否开启成功

```
lsmod | grep bbr
```

如果看到回显`tcp_bbr 20480 `说明已经成功开启 BBR。

### 四、搭建 Shadowsocks Server

- 更新 apt-get 软件包

```
sudo apt-get update
```

- 通过 apt-get 安装 python-pip

```
sudo apt-get install python-pip
```

- 使用 pip 安装 shadowsocks 服务

```
sudo pip install shadowsocks
```

如果出现类似 `Successfullying installed shadowsocks - x.x.x`的提示说明安装成功。

- 创建  Shadowsocks Server 配置文件

```
sudo vim /etc/ss-conf.json
```

回车之后会进入这个创建的文件，windows 下点击键盘上的 insert 键进入编辑，mac 系统则随便输入一个字母可以进入编辑。输入以下内容：

```
{
"server":"0.0.0.0",
"server_port":8838,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"fiteen",
"timeout":600,
"method":"aes-256-cfb"
}

// server_port 与 password 分别对应 Shadowsocks 客户端上配置使用的端口和密码，内容请自定义
```

点击 ESC 键，左下角的 insert 标志消失，同时按下"shift" 和":"键，左下角出现":" 标志，输入"wq"，接着回车即保存退出文件。

- 用配置文件启动 Shadowsocks Server

```
sudo ssserver -c /etc/ss-conf.json -d start
```

如果要设置开机启动，可以参考这篇[文章](https://my.oschina.net/oncereply/blog/467349)。

服务搭建已经完成了，以 [Mac 客户端](<https://github.com/shadowsocks/shadowsocks-iOS/releases>)为例，输入上面配置的内容，确定后开启服务便可以科学上网了。

 {% asset_img server-settings.png 服务器设定 %}

