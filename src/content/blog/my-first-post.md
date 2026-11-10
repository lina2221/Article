---
title: '我的第一篇文章'
description: '这是用 Astro 写个人博客的示例模板，照着改就能变成你自己的文章'
pubDate: '2026-11-10'
updatedDate: '2026-11-10'
heroImage: '../../assets/blog-placeholder-1.jpg'
---

# 你好，这是我的博客

> 这一篇是**模板**，你直接把下面内容换成自己的就行。
> 文章是用 **Markdown** 写的，左边写、右边（网站）自动更新。

## 一、怎么写新文章

1. 打开文件夹 `my-blog/src/content/blog/`
2. **新建一个 `.md` 文件**（文件名随便起，比如 `我的生活.md`）
3. 最上面先写一段"头部信息"（下面有模板）
4. 下面空一行，开始写正文

### 头部的固定写法（必填三项）

```md
---
title: '文章标题'
description: '一句话简介，列表页和搜索引擎会用到'
pubDate: '2026-11-10'
---
```

- `title`：标题
- `description`：简介
- `pubDate`：发布日期（格式 `年-月-日`）
- `heroImage`（可选）：封面图，填 `../../assets/你的图.jpg`

## 二、Markdown 能写什么

**加粗**、*斜体*、[超链接](https://astro.build)、`行内代码` 都没问题。

- 列个点
- 再列一个点

```js
// 代码块也支持语法高亮
console.log('Hello, 我的博客!');
```

> 引用块长这样。

![示例图](../../assets/blog-placeholder-2.jpg)

## 三、写完怎么看效果

在终端里（先切到 Node 24）：

```bash
cd /c/Users/图模型250704/Desktop/my-blog
export PATH="/d/Node:$PATH"
npm run dev
```

然后浏览器打开 `http://localhost:4321/blog/`，就能看到这篇文章出现在列表里。

---

把上面这些说明删掉，换成你自己的文字，就是一篇真正的博客了。祝你写作愉快 🎉
