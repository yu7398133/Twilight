---
title: 模板指南 - 高级自定义
published: 2024-02-10
description: "掌握 Twilight 模板的高级功能和自定义选项。"
cover: "./Cover - Advanced Customization.jpg"
coverInContent: false
pinned: false
tags: []
category:
    - 指南:
        - 高级自定义
draft: false
---


本指南涵盖了 Twilight 模板中可用的高级自定义选项和功能，包括全局配置和 Markdown 扩展。


## 全局配置

`twilight.config.yaml` 文件是博客配置的核心。以下是一些你可以调整的高级设置：

### 站点与本地化

- **语言与翻译**：通过 `site.translate.enable` 启用客户端翻译。你可以选择不同的翻译服务并配置自动检测。

- **自定义字体**：在 `site.font` 下提供 CSS 链接或文件路径来添加自定义字体。

### 视觉效果

- **主题色**：通过调整 `site.themeColor.hue`（0-360）来更改博客的主色调。

- **壁纸模式**：可选择 `banner`、`fullscreen` 或 `none`。你还可以通过 `carousel` 启用多壁纸轮播，并搭配 `kenBurns` 效果。

- **波浪效果**：通过 `site.wallpaper.banner.waves.enable` 切换横幅上的动态水波纹效果。

- **粒子效果**：通过 `particle.enable` 启用背景中的浮动粒子。

### 用户界面

- **导航栏透明度**：通过 `site.wallpaper.banner.navbar.transparentMode` 调整，可选 `semi`、`full` 或 `semifull`。

- **侧边栏组件**：在 `sidebar.components` 中重新排列或切换侧边栏组件，如 `profile`（个人简介）、`announcement`（公告）、`categories`（分类）、`tags`（标签）、`toc`（目录）和 `statistics`（统计）。


## Markdown 扩展

### GitHub 仓库卡片

你可以添加链接到 GitHub 仓库的动态卡片，页面加载时会从 GitHub API 拉取仓库信息。

::github{repo="Spr-Aachen/Twilight"}

使用代码 `::github{repo="Spr-Aachen/Twilight"}` 创建 GitHub 仓库卡片。

```markdown
::github{repo="Spr-Aachen/Twilight"}
```

### 音乐卡片

- 在线音乐
::music{meting="https://meting.spr-aachen.com/api?server=netease&type=song&id=1390882521"}

```markdown
::music{meting="https://meting.spr-aachen.com/api?server=netease&type=song&id=1390882521"}
```

- 本地音乐
::music{title="深海之息" artist="Youzee Music" cover="https://p1.music.126.net/PhKOqFtljgHDDpKYM2ADUA==/109951169858309716.jpg" audio="assets/music/深海之息.m4a" lrc="assets/music/深海之息.lrc"}

```markdown
::music{title="深海之息" artist="Youzee Music" cover="https://p1.music.126.net/PhKOqFtljgHDDpKYM2ADUA==/109951169858309716.jpg" audio="assets/music/深海之息.m4a" lrc="assets/music/深海之息.lrc"}
```

### 提示框

支持以下类型的提示框：`note` `tip` `important` `warning` `caution`

:::note
即使在快速浏览时，也值得关注的信息。
:::

:::tip
帮助用户更好完成操作的可选信息。
:::

:::important
用户成功完成操作所必需的关键信息。
:::

:::warning
由于潜在风险，需要用户立即关注的关键内容。
:::

:::caution
某个操作可能带来的负面后果。
:::

- **基本语法**

    ```markdown
    :::note
    即使在快速浏览时，也值得关注的信息。
    :::

    :::tip
    帮助用户更好完成操作的可选信息。
    :::
    ```

- **自定义标题**

    提示框的标题可以自定义。
    :::note[自定义标题]
    这是一个带有自定义标题的提示框。
    :::
    ```markdown
    :::note[自定义标题]
    这是一个带有自定义标题的提示框。
    :::
    ```

- **GitHub 语法**

    > [!TIP]
    > 也支持 [GitHub 语法](https://github.com/orgs/community/discussions/16925)。
    ```markdown
    > [!TIP]
    > 也支持 GitHub 语法。
    ```

- **剧透文本**

    你可以在文本中添加剧透内容。剧透文本也支持 **Markdown** 语法。

    内容 :spoiler[被隐藏了 **嘿嘿**]！
    ```markdown
    内容 :spoiler[被隐藏了 **嘿嘿**]！
    ```

---

更多详情请查阅[文档](https://docs.twilight.spr-aachen.com)。
