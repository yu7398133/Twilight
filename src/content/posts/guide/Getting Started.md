---
title: 模板指南 - 快速开始
published: 2001-10-02
description: "如何使用此博客模板。"
cover: "./Cover - Getting Started.jpg"
coverInContent: false
pinned: true
tags: []
category:
    - 指南:
        - 快速开始
draft: false
---


提示：对于本指南中未涉及的内容，你可以在 [Astro 文档](https://docs.astro.build/) 中找到答案。


## 文章的 Front-matter

```yaml
---
title: 我的第一篇博客文章
published: 2020-02-02
description: 这是我的 Astro 博客的第一篇文章。
cover: ./cover.jpg
coverInContent: false
tags: []
category: 日常
comment: true
draft: false
---
```


| 属性             | 说明             |
|------------------|------------------|
| `title`          | 文章的标题。 |
| `published`      | 文章的发布日期。 |
| `pinned`         | 是否将此文章置顶在文章列表顶部。 |
| `description`    | 文章的简短描述，显示在首页。 |
| `cover`          | 文章的封面图片路径。<br/>1. 以 `http://` 或 `https://` 开头：使用网络图片 <br/>2. 以 `/` 开头：使用 `public` 目录中的图片 <br/>3. 无前缀：相对于 Markdown 文件的路径 |
| `coverInContent` | 是否在文章内容中显示封面图片。 |
| `tags`           | 文章的标签。 |
| `category`       | 文章的分类。<br/>1. 单个分类：`category: 指南` <br/>2. 多个分类：`category: [指南, 快速开始]` |
| `licenseName`    | 文章内容的许可证名称。 |
| `author`         | 文章的作者。 |
| `sourceLink`     | 文章内容的来源链接或参考。 |
| `comment`        | 是否启用评论功能，默认为 `true`。 |
| `draft`          | 如果文章仍为草稿状态，则不会被显示。 |


## 文章文件的存放位置

文章文件应放置在 `src/content/posts/` 目录下。你也可以创建子目录来更好地组织文章和资源文件。

```
src/content/posts/
├── post-1.md
└── post-2/
    ├── cover.jpg
    └── index.md
```
