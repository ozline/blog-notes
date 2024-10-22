+++
title = '我有独特的 macOS 工作流'
date = 2024-10-22T22:00:00+08:00
tags = ["pipeline"]
+++

> 这篇文章将会以我几年来对 macOS 的使用为背景，总结我自身认为最合适的 macOS 工作流，里面也许有一些软件需要付费，但是我们不仅可以使用慈禧付费版，也可以低价(有时候是 1 折)在淘宝上购买 Key 或者使用家庭车。

> 需要注意的是，我没有列举出来的软件可能是我没有用过，但更大可能是我用了然后卸载了，我使用 macOS 已经近十年了

## 软件选取

### 系统优化

原生的 macOS 系统还是有蛮多局限性的，但是软件生态弥补了这一点

#### AltTab - 窗口切换

软件地址：https://alt-tab-macos.netlify.app/

相较于原生枯燥无味的切屏，AltTab 增强了其实现，并进行了类 Windows 的切窗方式，在这里我提供我自己的设置，我印象里我的设置比较接近默认设置（也许现在的默认设置已经变成了这个）

![控制](https://img.w2fzu.com/etc/202410222205994.png)

![外观](https://img.w2fzu.com/etc/202410222206914.png)

#### RayCast - Spotlight 高级优化

地址：https://www.raycast.com/

默认的 Spotlight 在诞生之初的确技惊四座，比当时的 Windows 高到不知道哪里去了，但是现在的 Spotlight 只能说是平庸

这个 RayCast 是非常好的上位替代，提供了大量增强特性，支持安装插件、AI特性（付费，所以我没用），同时它的动画非常流畅，简洁且好用

比较好用的就是剪切板历史、高级计算器、文件查找、翻译、简单功能实现（如图片压缩等），除此之外它支持 1Password 插件（自动生成高级密码、查找密码）、在线 ChatGPT、VSCode、翻译、JSON 格式化等等等等，相当于提供了一个基础入口，我们可以在它的基础上实现各个应用之间的连接，而只需要一个`Command + Space`呼出

> Windows 上有类似实现，但是我的体感体验明显不如 RayCast，然而 RayCast 目前没有 Windows 版本

#### Bartender5 - 菜单管理

地址：https://www.macbartender.com/

macOS 一个最大的好用特性就是它的Menu Bar，但菜单栏似乎没有考虑过大量的 icon，Bartender 的作用就是负责精简它，它的作用只有一个——在屏幕不够使的时候将不需要的 Item 移到它自己的小窗上。

不要看这个功能似乎可有可无，它大大增强了我的系统体验，但是它的价格不太美丽，这里如果不想支持的话可以用慈禧付费版，没有区别

#### 微信输入法 - 丢掉卡顿的原生输入法

地址：https://z.weixin.qq.com/

这个在 2024年的当下是最优解，没有什么好说的，也许在未来也是最优解，完美适配国内，无广告，已经把搜狗输入法吊着打了，不要说原生输入法，原生输入法中文语境下会卡顿，而 Apple 似乎并没有计划去优化它

关于输入法卡顿，这里提供v2ex来源：

2022年 - https://www.v2ex.com/t/898932
2023年 - https://www.v2ex.com/t/952579
2024年 - https://www.v2ex.com/t/1053751

#### Magnet - 窗口尺寸调节

地址：https://magnet.crowdcafe.com/

嗯，比 macOS15上原生的好用，原生的总是这样，提供最基础的支持，但稍微优化一点都不行，这个毛子的应用很好用，除了在软件清理时被CleanMyMac 标记为可疑项（仅仅因为它是毛子的）

#### iShot Pro - 截图

地址：https://www.better365.cn/ishot.html

国人写的，它的 web 页面一股山寨味儿，但它非常的好用，平庸的外表难掩绝佳的体验，功能超多，占用不高，唯一的缺点也许是付费


#### Mos - 鼠标滚动优化

地址：https://github.com/Caldis/Mos

我的 MX Master 搭配这个直接无敌，好用无需多言

#### Karabiner-Elements - 键盘强化

地址：https://karabiner-elements.pqrs.org/

喜欢搞学术、喜欢霓虹的有福了，可以直接看他们的 Sponsors。这个软件解决了 macOS 下键盘定制强化羸弱的功能。它的功能十分强大，有需要的可以自己去了解，我使用的最大且最好用的一个功能是它可以在我键盘切换的时候自动屏蔽内置键盘。


#### 1Password - 多端密码管理

地址：https://1password.com/zh-cn

在使用 1Password 之前，我尝试了大量的密码管理工具，都不尽如人意，这也是开源免费的弊端，1Password 恰恰相反，它支持多端，且 UI 动效清晰优美，同步很顺利，而且解决了目前的 Passkey 的一些弊端，也可以储存 SSH 密钥。

唯一的弊端是它的价格实在是太贵了，国区也得 36 美元，淘宝基本都是家庭车，但我觉得这种密码管理工具不能拼家庭，找了半天找到了个支持个人版续费的，也要 100多。

> 但它的体验优化值得 100+/year

#### Keka - 比 Bandizip 好用

地址：https://www.keka.io/en/

真正实现了压缩和解压的优雅，我在压缩软件上踩了很多坑，这个是我唯一的最优解

### 写作

#### PicGo - 图床

地址：https://github.com/Molunerfinn/PicGo

无需多言，支持大部分云服务商，甚至包括又拍云

#### Typora - 写作场景？

带一个问号是因为我现在已经不使用 typora 了，但是还有一些痛点没有解决，所以还没有删除它。我现在写作场景区分为以下部分：

| 场景                                   | 解决方案                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 手写笔记（含学习笔记、带学生的笔记等） | iPad 的 MarginNote4 吊着打了 Documents、Prodrafts 和 Goodnotes，这个软件好用到我开了 328 的 Max 版 |
| 简易文档                               | VScode 的 markdown 可以覆盖全场景了，如果需要适配富文本场景和便携性，飞书文档（Dark Docs）是目前最优解，我 Notion 使用的并不习惯，它太割裂了，拆成在线文档和手写笔记会更好一些 |
| 标准文章                               | 我使用的是 Texifier，在比赛和一些需要使用 Latex 的场景下非常好用，只不过它只支持 macOS，而且需要一些配置。在不考虑协作的情况下，它比 OverLeaf 来的好用，现在我在考虑使用 Typst 了 |

而我现在可以说 Typora 它解决了什么痛点——我可以粘贴其他格式的表格，它会自动转换成 markdown 形式的，然后我就可以粘贴在其他地方，比如上面。的确有其他软件可以支持同样功能，但它们都没有 Typora 轻量

### 网络

这篇是重中之中，可以搭建一个非常好用的网络过滤处理体系，我参考了大多数人的解法，但最后还是 fallback 到 Sukka 的解决方案上，然后我在它的方案基础上做了我的定制化处理

#### 我的网络过滤体系

在这里不赘述在路由器上的优化，因为我们侧重点在 macOS 上。我目前使用的是 Surge + AdGuard 的体系，在这里的 Surge 改成 Clash X Pro 也是相同的效果（规则通用）。不要尝试使用 Clash Verge，不要尝试使用 Clash Verge，不要尝试使用 Clash Verge

> 幽默的吐槽一下，Surge 一直标榜自己为一个网络调试工具，但谁都知道它安的什么心，抓包不如 Proxyman，流量分析不如 Gost，Debugging Proxy 不如 Whistle。

它很中庸，但因为它是个付费软件，所以它有原生好看的界面，以及一丢丢的性能优化（在我大量的规则下我完全没看出来它和 Clash 有什么区别）

#### 上下游配置

虽然大家都知道 DOMAIN-SET 的出现就是为了做广告过滤，但很明显，怎么做也没有付费的、专门的广告过滤软件好用。我们无法替代 AdGuard（除了 AdGuard Home）

但是 AdGuard 也内置了 DNS Server，也有 HTTP 代理，怎么取舍呢？

我的配置目前如下

- 由 Surge 提供而关闭的：DNS 保护、HTTP 代理
- 在默认配置基础上开启的：自动激活过滤器、高级跟踪保护（拦截 webRTC、拦截跟踪器、移除跟踪参数）、拓展（弹窗拦截，AdGuard Extra、Web of Trust）

之后我们需要衔接 Surge，我们可以在**出站代理**上做手脚，众所周知 surge 的监听地址一般都是`0.0.0.0:6153`，那么我们的出站代理就设置为它（方式为 SOCKS5）

![image-20241023001908134](https://img.w2fzu.com/etc/202410230019172.png)

#### 过滤规则

> 过滤规则同样适用于 Clash X Pro，本质上都是同一套解析流程

我参考了 Sukka 今年上半年的文章（[我有特别的 Surge 配置和使用技巧](https://blog.skk.moe/post/i-have-my-unique-surge-setup/)）和 [217heitai/adblockfilters](https://github.com/217heidai/adblockfilters)，结合我自己的使用场景定制了这些策略，24 小时更新一次（上游则是 8 小时更新一次）

在 AdGuard 中，我导入了adblockfilters 中的插件拦截规则（[加速链接](https://mirror.ghproxy.com/https://raw.githubusercontent.com/217heidai/adblockfilters/main/rules/adblockfilters.txt)）

在 Surge 中，我编辑了这一套配置（不需要做额外修改的配置我沿用了机场配置）

```text
[General]
#!include Cylink_241018.conf

[Proxy]
#!include Cylink_241018.conf

[Proxy Group]
#!include Cylink_241018.conf

[Rule]
# Non IP
# 基础的 12 万拦截域名
DOMAIN-SET,https://ruleset.skk.moe/List/domainset/reject.conf,REJECT
# 额外 20 万拦截域名，作为基础的补充，启用时需要搭配基础一起使用
# 在 Surge 5 for Mac（或更新版本），即使同时启用基础和额外的拦截域名也不会导致匹配性能下降或内存占用过高
DOMAIN-SET,https://ruleset.skk.moe/List/domainset/reject_extra.conf,REJECT
RULE-SET,https://ruleset.skk.moe/List/non_ip/reject.conf,REJECT,extended-matching
RULE-SET,https://ruleset.skk.moe/List/non_ip/reject-no-drop.conf,REJECT-NO-DROP,extended-matching
RULE-SET,https://ruleset.skk.moe/List/non_ip/reject-drop.conf,REJECT-DROP,extended-matching
# URL-REGEX
# 需搭配 Surge 模块 https://ruleset.skk.moe/Modules/sukka_mitm_hostnames.sgmodule 使用
# MITM 和 URL-REGEX 性能开销极大，不推荐使用
# RULE-SET,https://ruleset.skk.moe/List/non_ip/reject-url-regex.conf,REJECT
# IP
# RULE-SET,https://ruleset.skk.moe/List/ip/reject.conf,REJECT-DROP
# 屏蔽特定的网络连接以改善日常上网体验，第一个会阻止视频平台 CDN 的 QUIC，第二个是丢弃Adobe软件内部的跟踪打点
RULE-SET,https://ruleset.skk.moe/List/non_ip/reject-no-drop.conf,REJECT-NO-DROP,extended-matching
RULE-SET,https://ruleset.skk.moe/List/non_ip/reject-drop.conf,REJECT-DROP,extended-matching
# iCloud Privacy Relay
DOMAIN-SET,https://ruleset.skk.moe/List/domainset/icloud_private_relay.conf,🍃 Proxies
# 国内常见域名
RULE-SET,https://ruleset.skk.moe/List/non_ip/domestic.conf,DIRECT,extended-matching
# 内网域名、局域网IP、和2个国内IP段
RULE-SET,https://ruleset.skk.moe/List/non_ip/lan.conf,DIRECT
RULE-SET,https://ruleset.skk.moe/List/ip/lan.conf,DIRECT
RULE-SET,https://ruleset.skk.moe/List/ip/domestic.conf,DIRECT
RULE-SET,https://ruleset.skk.moe/List/ip/china_ip.conf,DIRECT
# 广告拦截IP（运营商）
RULE-SET,https://ruleset.skk.moe/List/ip/reject.conf,REJECT-DROP
# Telegram IP规则，只放行telegramIP，对它其余的网络连接全部丢弃
RULE-SET,https://ruleset.skk.moe/List/ip/telegram.conf,🍃 Proxies
PROCESS-NAME,Telegram,REJECT-DROP
# Apple & Microsoft 国内 CDN 域名，以及 Apple CN 域名（云上贵州需要国内访问）
RULE-SET,https://ruleset.skk.moe/List/non_ip/apple_cdn.conf,DIRECT
RULE-SET,https://ruleset.skk.moe/List/non_ip/microsoft_cdn.conf,DIRECT
RULE-SET,https://ruleset.skk.moe/List/non_ip/apple_cn.conf,DIRECT
# 排除了拥有国内CDN后剩余的苹果和微软的域名
RULE-SET,https://ruleset.skk.moe/List/non_ip/apple_services.conf,🍃 Proxies,extended-matching
RULE-SET,https://ruleset.skk.moe/List/non_ip/microsoft.conf,🍃 Proxies,extended-matching
# 常见海外域名
RULE-SET,https://ruleset.skk.moe/List/non_ip/global.conf,🍃 Proxies,extended-matching
# 软件、游戏和驱动的下载和更新域名
DOMAIN-SET,https://ruleset.skk.moe/List/domainset/download.conf,DIRECT,extended-matching
RULE-SET,https://ruleset.skk.moe/List/non_ip/download.conf,DIRECT,extended-matching
# 没有匹配到的情况
FINAL,DIRECT,dns-failed
```

经过这一套配置后，目前电脑的访问速度有最基础的保障（M1 Pro 性能还是不强的），在访问网站时经过 AdGuard 和 Surge 的过滤后基本上没有特别明显的广告（除站内广告，站内广告大多数是拦不掉的）

#### 其他

实际上我 HTTP 代理默认是留空的，因为有其他软件会使用到代理

这一套体系在深信服的 EasyConnect 和 aTrust 软件场景中同样适用兼容(2024.01.05在其他学校经过验证)，但我还是更建议使用第三方容器版的，深信服的软件我不信任。

在这里不涉及开发，只使用最基础的系统优化，开发的话相信看到这篇文章的基本也是同好了，各自有各自的用法，不是这里的重点