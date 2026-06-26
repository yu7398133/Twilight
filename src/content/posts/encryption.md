---
title: 加密示例
published: 2020-02-02
description: '密码：123456'
encrypted: true
pinned: false
password: "123456"
tags: [Encryption]
category: 示例
---


# 密码保护文章

这是 Twilight 主题中密码保护文章的示例。以下内容使用 AES 加密，只有输入正确密码才能查看。


## Frontmatter 示例

```yaml
---
title: 加密示例
published: 2020-02-02
encrypted: true
password: "your-password"
...
---
```

- `encrypted` - 是否启用文章加密。
- `password` - 解锁内容所需的密码。


## 注意事项

:::warning
请勿将此功能用于银行密码或私钥等极其敏感的信息。加密在客户端进行，密码本身存储在文章的元数据中（虽然通常不会直接显示）。
:::
