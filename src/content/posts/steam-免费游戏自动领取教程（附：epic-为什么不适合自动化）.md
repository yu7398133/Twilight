---
title: Steam 免费游戏自动领取教程（附：Epic 为什么不适合自动化）
published: 2026-10-12T21:43:00.000+08:00
updated: 2026-10-12T21:43:00.000+08:00
tags: []
draft: false
---
## 起因

Steam 每周都有免费游戏，但每次都要手动打开客户端、找页面、点领取，一周不看就错过了。Epic 那边也是一样，每周四晚上更新。

我原本想的是复刻「小黑盒」那种一键领取，最后一圈做下来，**Steam 实现了全自动，Epic 放弃了**。这篇文章把两边都讲清楚——包括 Epic 为什么不适合自动化，这部分可能比教程本身更有价值。

先说结论：

平台

结果

原因

**Steam**

✅ 全自动无人值守

有成熟的 ASF 生态，登录态自己维护

**Epic**

❌ 放弃，保持手动

结账环节有 hCaptcha，且服务端另有风控

## 一、Steam 自动领取

## 1.1 方案选型

Steam 上领取免费游戏，本质上就是调用 `AddFreeLicense` / `ajaxregisterkey` 接口把游戏塞进库里，需要的只是一个登录态的 Cookie。

自己写脚本的话，最大的痛点是 **Cookie 会频繁失效**——手动登录导出的静态 Cookie 基本撑不过一周，你必须再写一套自动续期，而续期又要过 Steam Guard。

**所以正确的做法是用 ASF（ArchiSteamFarm）。**

ASF 是 Steam 社区里最成熟的挂卡工具，它自己维护登录会话，配合 **FreePackages** 插件就能自动领取免费游戏。你只需要在首次配置时配合输入一次密码和令牌，之后它自己管。

另外提一句：我试过 `vogler/free-games-claimer` 这个项目，它的 `steam-games.js` 其实**只导出游戏库 JSON，不做领取**。Steam 侧能真正自动入库的成熟方案，基本都绑定 ASF。

## 1.2 安装 ASF

以 Debian 为例，从 [ASF Releases](https://github.com/JustArchiNET/ArchiSteamFarm/releases) 下载对应版本，解压到 `/opt/asf`：

首次运行会提示你配置。核心是 `config/bot.json`：

**关键就是** `EnableFreePackages: true`——这一行才是自动领取的开关。

如果要让它常驻，建议做成 systemd 服务：

**第一次启动需要人盯着**：ASF 会要求输入 Steam Guard 验证码（邮箱或手机令牌）。这一步过了，之后就不用管了。

> 注意：如果你的 ASF 跑在无头环境（比如 Docker），不要在 Headless 模式下用 stdin 喂验证码——实测 ASF 在 Headless 下一请求输入就退出。老老实实用 systemd 服务，或者带终端跑第一次。

## 1.3 确认它真的在领

ASF 日志里搜 `ClaimFreeApp`：

看到 `状态：OK` 就是领成功了。

有个小坑：日志里可能出现 `FetchProductInfo()` 异常，**那只是 ASF 重连瞬间的正常现象，不影响领取**，不用管。

## 1.4 加一层播报

ASF 领了游戏你不知道，所以要有个汇总播报。我用的是青龙面板（QingLong）跑一个 Python 脚本，逻辑是：

1. 查 Steam 当前 100% OFF 的游戏
2. 汇总 ASF 已自动领取的记录
3. 与上次对比，**有新增才推送**（没新增就静默，不打扰）

**这里有个部署上的坑要注意**：如果 ASF 跑在宿主机、青龙跑在容器里，青龙**读不到宿主机的 ASF 日志**。解决办法是用一个导出脚本定期把领取记录导出成 JSON，让青龙读那个文件：

配个 systemd timer 每 30 分钟跑一次即可。

播报结果大概是这个样子：

## 1.5 定时策略

**Steam 的免费游戏没有固定更新日**，这点和 Epic 不一样。所以建议：

* **每日扫描一次**（覆盖不定期出现的免费周末）
* **每周播报一次**汇总

我设的是每天 09:00 扫描、每周五 01:00 播报。

## 二、Epic 为什么放弃了

这部分是我花时间最多、也是最有意思的部分。

## 2.1 一开始方向就错了

我最早发现 Epic 的接口请求全部返回 403，页面上是 Cloudflare 的挑战页。当时我的结论是：

> **Epic 全端点被 Cloudflare 拦截，属网络层阻断，无解。**

**这个结论是错的。**

后来我用 NAS 自带的浏览器实际登录了一次，发现：**登录之后，**`store.epicgames.com` **完全正常，GraphQL 接口返回 200 和真实数据。**

那 403 只出现在**未登录的 curl / 容器请求**上——Cloudflare 拦的不是"我的网络"，而是"看起来像脚本的客户端"。

**教训：拿 curl 的 403 去判断一个网站"被墙了"，是不成立的。真实浏览器登录态的对比测试才是有效证据。**

## 2.2 真正的拦路虎：hCaptcha

登录之后，领取流程其实很清晰：

这道 hCaptcha 不只在结账时出现，**登录、结账、提交三处都有**。

我试过用 CDP 模拟人类点击（12 步渐变路径的鼠标移动 + 按下 + 抬起，`isTrusted=true`），结果是：

> **非常抱歉，验证失败。**

而且更关键的一点：**验证码没解时点「添加到库」，浏览器发出的网络请求数是 0。** 也就是说请求根本没发出去，前端直接拦住了。

## 2.3 社区证据：连付费解法都不行

我去 GitHub 上找了现成的项目，确实有看起来能用的：

* [Ronchy2000/epic-freebies-helper](https://github.com/Ronchy2000/epic-freebies-helper)（257⭐）——用智谱 `glm-4.6v` 多模态模型识别验证码
* [Magerko/Epic-Claimer](https://github.com/Magerko/Epic-Claimer)——Playwright + AI hCaptcha solver

**但是**，在 [Revadike/epicgames-freebies-claimer](https://github.com/Revadike/epicgames-freebies-claimer) 的 PR #184 里，作者给出了一个决定性的实测结果：

> 我用付费服务生成了 sitekey 和域名都正确的 hCaptcha 解法，**Epic 仍然返回 400 拒绝**。我猜测 hCaptcha 解法之外还有额外校验。

还有用户在 [vogler/free-games-claimer](https://github.com/vogler/free-games-claimer) 的 issue #483 里确认：容器环境缺少 WebGL/WebGPU 指纹时，验证码只能得到 "Incorrect response"，加上 `--enable-unsafe-webgpu` 之后才变成"可解"。

**这两条加起来说明：就算 AI 能识别图片，Epic 服务端还有一层指纹校验（社区推测涉及** `xal` **编码指纹）。**

## 2.4 最后为什么不做了

技术上：**不可靠。** 即使堆上多模态模型 + 指纹伪造，Epic 随时可以加一层校验，你的自动化就废了。

但真正让我停下的不是技术原因：

1. **违反 Epic 服务协议**——绕过人机验证属于明确的反自动化行为；
2. **这是我的主号**——为了每周省 30 秒，去赌账号安全，不值得。

**所以我把它全部删掉了**：容器、Docker 卷、目录、调试端口、脚本里的 Epic 逻辑，全部清理干净。Epic 免费游戏现在就是每周手动点一下，大概 30 秒。

## 2.5 顺带一提：那个"设备不受支持"弹窗

流程里有个 `设备不受支持 / 此产品与您当前的设备不兼容` 的弹窗，**它不代表领取失败**。

原因是那些游戏标了仅支持 Windows，而浏览器的 UA 显示 Linux。点「继续」就能正常进入结账页，**订单金额显示 ¥0.00 是正常的**。

我一开始在这个弹窗上判断错了，以为是被拦了。这个坑记一下。

## 三、最终效果

整套东西跑起来之后，每周五凌晨 1 点你会收到一条消息，告诉你这周新领了什么。没有新游戏就不推送。

## 四、一些通用教训

1. **别用 curl 的 403 判断"网站被墙了"**——先拿真实浏览器登录态对比一次。
2. **选型优先看社区成熟度**——Steam 侧我绕了一圈，最后还是回到 ASF，因为"维护登录态"这件事只有它做得好。
3. **遇到验证码，先问"值不值得绕"**——技术上能不能做，和该不该做是两回事，尤其涉及主账号。
4. **容器和宿主的路径隔离要提前想**——我在这上面 debug 了不少时间。

## 参考

* [ArchiSteamFarm](https://github.com/JustArchiNET/ArchiSteamFarm)
* [vogler/free-games-claimer](https://github.com/vogler/free-games-claimer) — issue [\#454](https://github.com/vogler/free-games-claimer/issues/454)、[\#483](https://github.com/vogler/free-games-claimer/issues/483)、[\#138](https://github.com/vogler/free-games-claimer/issues/138)
* [Revadike/epicgames-freebies-claimer](https://github.com/Revadike/epicgames-freebies-claimer) — PR #184
* [Ronchy2000/epic-freebies-helper](https://github.com/Ronchy2000/epic-freebies-helper)
* [QIN2DIM/epic-awesome-gamer](https://github.com/QIN2DIM/epic-awesome-gamer)



bashjsoninibashbashbashplaintextplaintextplaintextEND
