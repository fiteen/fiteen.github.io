---
title: 【持续更新】Hexo + inside 博客个性化定制
date: 2020-01-17 01:20:03
tags: 
	- Hexo
	- Valine
categories: 前端
thumbnail: hexo.png
---

[我的博客](https://blog.fiteen.top)采用的是 [Hexo 官方网站](https://hexo.io/themes/)上相中的 [hexo+theme+inside](https://github.com/ikeq/hexo-theme-inside) 主题。虽然开发者已经提供了主题的[使用文档](https://blog.oniuo.com/theme-inside)，但是作为一款小众的主题，一些常用功能的定制并不是那么完善，不过贴心的开发者提供了 [plugins 配置方案](https://blog.oniuo.com/theme-inside/docs/misc#plugins)。

<!--more-->

下文总结了部分功能的拓展方案，可供需要的朋友参考。

> **注意**：inside v2.6.3 之前会出现**文章中点击上/下一篇时，插件无法生效的情况**。
> 其原因是**主题插件模块存在组件缓存问题，导致切换页面时 js 执行失败**，为了保障功能正常使用，请务必将主题升级到[最新版本](https://github.com/ikeq/hexo-theme-inside/releases)。（2020.03.14 更新）

## plugin 前置准备

`themes/inside/_config.yml` 中的 plugins 支持于特定位置动态插入可执行的代码片段，或全局加载脚本/样式。

支持通过安装 html-minifier、babel 和 uglify-js 来实现代码压缩。 在项目根目录执行（Hexo 根目录，非 `themes/inside`）：

```bash
npm install babel-core babel-preset-env html-minifier uglify-js --save
```


## Font Awesome

按照 plugin 配置描述的，要支持 Font Awesome 的 CSS，只需要这样设置：

{% codeblock _config.yml lang:yaml %}
plugins:
  - //netdna.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css
{% endcodeblock %}

也就是在全局加载样式，~~不过不知道为什么没有正常生效🤔~~（**[inside-v2.6.1](https://github.com/ikeq/hexo-theme-inside/releases/tag/2.6.1) 已经修复了这个问题**，建议你升级到[最新版本](https://github.com/ikeq/hexo-theme-inside/releases)）。

> 如果你是 v2.6.0 及以下版本，可以用这个方案解决：
>
> 在 `themes/inside/layout/index.swig` 的 `<head>` 标签内加入以下代码：
>
>{% codeblock index.swig lang:html %}
<link href="//netdna.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">
{% endcodeblock %}
>
> 这时，虽然图标显示出来了，但是样式还是有点问题，可能和主题本身的 CSS 有关系，找到 `source` 目录下的 `styles.e4da61f53c7bc99becf4.css`（也可能叫别的） 里的 `.fa`，删除里面的 `margin:10rem 0 3rem;` 。
>
> ![修改 .fa 样式](alter-style-css.png)

不过个人觉得放在 CDN 上访问速度还是有点慢，所以从官网[下载](http://fontawesome.io)最新版放在主题的 `source/lib` 目录下，全局引用：

{% codeblock _config.yml lang:yaml %}
plugins:
  - lib/font-awesome/css/font-awesome.min.css
{% endcodeblock %}

或者在需要的位置引用 CSS 资源：

```html
<link href="lib/font-awesome/css/font-awesome.min.css" rel="stylesheet">
```


## 访问量统计

很多人都有站点访问量统计的需求，像这样：

![访问量统计效果](busuanzi-example.png)

我采用的是轻量的[不蒜子统计](http://ibruce.info/2015/04/04/busuanzi/)来做访问量统计。

先**安装脚本**，在使用不蒜子的页面，也就是 `sidebar` 模块插入 `busuanzi.js`。

{% codeblock _config.yml lang:yaml %}
plugins:
  - position: sidebar
    template: |
      <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
{% endcodeblock %}

再**安装标签**，官方给出了站点 PV/UV 的统计代码：

```html
<span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>
<span id="busuanzi_container_site_uv">本站访客数<span id="busuanzi_value_site_uv"></span>人次</span>
```

你也可以用这两个 id 来显示访问数：

- `busuanzi_value_site_pv`：异步回填访问数
- `busuanzi_container_site_pv`：为防止计数服务访问出错或超时（3秒）的情况下，使整个标签自动隐藏显示

在 `_config.yml` 文件里找到 `footer` 下的 `custom`，写入相关的 html 代码。比如：

{% codeblock _config.yml lang:yaml %}
# Custom text.
custom: <span id="busuanzi_container_site_uv" style='display:none'>Total <span id="busuanzi_value_site_uv"></span> visitors. </span><span id="busuanzi_container_site_pv" style='display:none'><span id="busuanzi_value_site_pv"></span> Views</span>
{% endcodeblock %}

或者使用 font-awesome 字体：

{% codeblock _config.yml lang:yaml %}
# Custom text.
custom: <span id="busuanzi_container_site_pv" style='display:none'><i class="fa fa-eye"></i> <span id="busuanzi_value_site_pv"></span></span> ｜ <span id="busuanzi_container_site_uv" style='display:none'><i class="fa fa-user"></i> <span id="busuanzi_value_site_uv"></span></span>
{% endcodeblock %}


## 代码复制

为了方便博客的读者引用代码，可以在文章中代码块的右上角加一个复制按钮，如：

![代码块复制按钮效果](clipboard-example.png)

它的实现是在页面加载完毕后，使用 js 动态为每个代码块添加一个按钮，当鼠标滑动到代码块上时显示按钮，点击按钮时复制代码块里的内容。因此需要三个文件：

- **实现复制代码块功能**的文件 `clipboard.js`，这里可以直接引用这个文件： `https://cdn.jsdelivr.net/npm/clipboard@2.0.4/dist/clipboard.js`。
- 支持**动态创建复制按钮**的文件 `clipboard-use.js`
- 复制按钮的**样式**文件 `clipboard.css`

页面载入完成后，创建一个复制按钮，上面用 font-awesome 的 clipboard 图标，实现如下：

{% codeblock clipboard-use.js lang:js %}
!function (e, t, a) {
  /* code */
  var initCopyCode = function () {
    var copyHtml = '<button class="btn-copy" data-clipboard-snippet=""><i class="fa fa-clipboard"></i></button>';
    $(".highlight .code pre").before(copyHtml);
    new ClipboardJS('.btn-copy', {
      target: function (trigger) {
        return trigger.nextElementSibling;
      }
    });
  }
  initCopyCode();
}(window, document);
{% endcodeblock %}

这里要注意的是， `clipboard-use.js` 中需要用到 `jQuery`，而 inside 里没有引入，故需要手动引入。

复制按钮的样式如下：
{% codeblock clipboard.css lang:css %}
.highlight {
    /* 方便copy代码按钮（btn-copy）的定位 */
    position: relative;
}
.btn-copy {
    border-radius: 3px;
    border-width: 0px;
    font-size: 13px;
    line-height: 20px;
    padding: 2px 6px;
    position: absolute;
    right: 5px;
    top: 5px;
    background: none;
    color: black;
    opacity: 0;
    outline: none;
    -webkit-tap-highlight-color: transparent;
    -webkit-appearance: none;
}
.btn-copy span {
    margin-left: 5px;
}
.highlight:hover .btn-copy {
    opacity: 1;
}
{% endcodeblock %}

复制按钮可以按照自己的喜好设置，如果想简单一点，直接用我的样式，可以这样配置：

{% codeblock _config.yml lang:yaml %}
plugins:
  # inside 主题没有引入 jQuery 框架，需要手动引入
  - //cdnjs.loli.net/ajax/libs/jquery/3.2.1/jquery.min.js
  # 插件生效范围：post 和 page
  - position: [post, page]
    template: |
      <script type="text/javascript" src="//cdn.jsdelivr.net/npm/clipboard@2.0.4/dist/clipboard.js"></script>
      <script type="text/javascript" src="//cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@v0.1.0/clipboard-use.js"></script>
      <link href="//cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@v0.1.1/clipboard.css" rel="stylesheet">
      <link href="lib/font-awesome/css/font-awesome.min.css" rel="stylesheet">
{% endcodeblock %}

如果已经全局引用过 font-awesome，可以把最后一条引用删除。

## 评论系统 - Valine

主题的内置评论，支持 [Disqus](https://disqus.com) 和 [LiveRe](https://livere.com)。但个人认为这两款评论系统的 UI 风格主题不是很搭配，最后还是决定采用 [Valine](https://valine.js.org)——一款基于LeanCloud的快速、简洁且高效的无后端评论系统。

虽然文档中也有提供 Valine 的配置方法，但是我实践后发现样式貌似出现了一些问题，这条 [issue](https://github.com/ikeq/hexo-theme-inside/issues/153) 也证实了这一点（**inside-2.6.1 已修复**）。所以我另找了一个 js 文件，并做了一点小改动。你可以引用我放在 CDN 上的资源 `https://cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@v0.1.0/valine.js`，或者直接把 `valine.js` 文件[下载](https://github.com/fiteen/fiteen.github.io/releases)到本地，放在 `inside/source/lib`路径下。然后写入以下代码：

{% codeblock _config.yml lang:yaml %}
plugins:
  # inside 主题没有引入 jQuery 框架，需要手动引入
  - //cdnjs.loli.net/ajax/libs/jquery/3.2.1/jquery.min.js
  - //cdn1.lncld.net/static/js/3.0.4/av-min.js
  # 引用本地 source/lib 路径下的 valine.js 文件
  # - lib/valine.js
  # 引用 CDN 上的 valine.js 文件
  - //cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@v0.1.0/valine.js
  - position: comments
    template: |
      <div id="vcomment"></div>
      <script>
        new Valine({
          el: '#vcomment',
          lang: 'en',
          admin_email: 'Your EMAIL',
          appId: 'Your APP ID',
          appKey: 'Your APP KEY',
          placeholder: 'Write a Comment',
          emoticon_url:'https://cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@0.1.0/alu',
          emoticon_list:["吐.png","喷血.png","狂汗.png","不说话.png","汗.png","坐等.png","献花.png","不高兴.png","中刀.png","害羞.png","皱眉.png","小眼睛.png","中指.png","尴尬.png","瞅你.png","想一想.png","中枪.png","得意.png","肿包.png","扇耳光.png","亲亲.png","惊喜.png","脸红.png","无所谓.png","便便.png","愤怒.png","蜡烛.png","献黄瓜.png","内伤.png","投降.png","观察.png","看不见.png","击掌.png","抠鼻.png","邪恶.png","看热闹.png","口水.png","抽烟.png","锁眉.png","装大款.png","吐舌.png","无奈.png","长草.png","赞一个.png","呲牙.png","无语.png","阴暗.png","不出所料.png","咽气.png","期待.png","高兴.png","吐血倒地.png","哭泣.png","欢呼.png","黑线.png","喜极而泣.png","喷水.png","深思.png","鼓掌.png","暗地观察.png"],
        })
      </script>
{% endcodeblock %}

关于上面的参数介绍：

- **lang**：选填，目前支持英文版 `en` 和中文版 `zh-cn` 两种，默认是 `zh-cn`。
- **admin_email**：选填，设置作者邮箱，使用该邮箱账号评论或回复，评论者名字右侧会出现一个人形小图标标识作者。
- **appId&appKey**：必填，LeanCloud 中创建应用得到的 `APP ID` 和 `APP KEY`，创建方式参照[此文](https://ioliu.cn/2017/add-valine-comments-to-your-blog/)。
- **emoticon_url**：必填，这里设置一个表情包 CDN 路径，你也可以自定义喜欢的表情包。
- **emoticon_list**：必填，`emoticon_url`里包含的表情包中需要显示在评论区的表情包名称列表。

![评论区效果](valine-comment-example.png)

这就是配置成功后的评论框效果。

目前已经有 Valine 评论系统的拓展和增强版 [Valine+Admin](https://github.com/DesertsP/Valine-Admin.git)，主要实现评论邮件通知、评论管理、垃圾评论过滤等功能，还支持自定义修改邮件通知模板、漏发邮件自动补发等。具体步骤这篇[配置手册](https://deserts.io/valine-admin-document/)已经比较清晰了，照着上面的步骤操作即可，本文就不复制粘贴了。

**注意**：想要在评论区显示自定义头像，先前往[Gravatar官网](http://cn.gravatar.com/)注册账号，注册的邮箱需要和你评论时填写的邮箱一致。如果注册成功后，头像仍没有显示，不要着急， `gravatar.cat.net` 有七天的缓存期，请耐心等待。


下面再分享几个**小功能点的配置**：


## 博客背景

修改博客背景很简单，只需修改 `themes/inside/_config.yml` 中 `appearance.background` 配置即可。

这里分享一个网站——[Subtle Patterns](https://www.toptal.com/designers/subtlepatterns/)，支持超过 500 种 PNG 高品质免费背景纹理素材，无须注册登录，可以直接下载。


## 博客字体

你可能会发现部署好的博客首次加载时的字体效果生效比较慢，这是因为主题中默认配置的字体样式用的是谷歌的服务：

{% codeblock _config.yml lang:yaml %}
appearance:
  font:
    url: //fonts.googleapis.com/css?family=Baloo+Bhaijaan|Inconsolata|Josefin+Sans|Montserrat
{% endcodeblock %}

如果被墙了就无法正常显示，因此我们可以换一个访问更快的地址，如：

{% codeblock _config.yml lang:yaml %}
appearance:
  font:
    url: //cdn.jsdelivr.net/gh/fiteen/fiteen.github.io@v0.1.0/font.css
{% endcodeblock %}

## 分享 QQ 链接

我们可以在 `sns.qq` 里配置自己想要链接的 QQ ID 信息，直接写 QQ 号当然是不可行的。需要先开通 [QQ 推广](https://shang.qq.com/v3/widget.html)。

![](qq-link.png)

`<a>` 标签里的 `href` 就是你的 **QQ 号**分享链接，形如：`https://wpa.qq.com/msgrd?v=3&uin=${YOUR-QQ-ID}&site=qq&menu=yes`。

如果你要**分享群号**，通过[加群组件](https://qun.qq.com/join.html)，拿到形如：`https://shang.qq.com/wpa/qunwpa?idkey=${YOUR-GROUP-ID-KEY}` 的链接。

## 配置 RSS

比较简单，[文档](https://blog.oniuo.com/theme-inside/docs/basic#sns)中有提到：

> 若使用 hexo-generator-feed，sns.feed 可留空，主题会尝试取 hexo.config.feed.path。可通过改变项的先后顺序来自定义排序。

因此直接在站点根目录（不是主题根目录）下执行命令即可：

```bash
npm install hexo-generator-feed --save
```

---

如对上述内容还有疑问，可以在评论区提出。如果需要我帮忙拓展某些功能，可以提 [issue](https://github.com/fiteen/fiteen.github.io/issues) 给我。