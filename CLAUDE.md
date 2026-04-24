# lxun.org — Astro Blog

## 工作流程

用户将文章内容放到 `drafts/` 文件夹，我需要：

1. 读取 `drafts/` 中的内容，整理成正式的 Markdown（包含 frontmatter）
2. 写入 `src/content/posts/` 下
3. 运行 `npm run build` 确保构建通过
4. 清理 `drafts/` 中的草稿
5. 执行 `git add/commit/push` 推送到 GitHub
6. 通知用户去 Cloudflare Pages 手动部署或确认自动部署完成

## 草稿目录

- `drafts/` — 用户放入原始内容的地方，我来处理和清理

## 项目结构

```
src/
  content/posts/     ← 博客文章（Markdown）
  layouts/           ← 页面布局
  pages/             ← 页面组件
    index.astro          ← 首页（自动列出文章）
    posts/[...slug].astro  ← 文章详情页
astro.config.mjs     ← Astro 配置
package.json         ← 依赖与脚本
```

## 构建命令

- `npm run dev` — 本地开发预览
- `npm run build` — 构建到 `dist/`
- `npm run preview` — 预览构建结果

## Frontmatter 格式

```yaml
---
title: "文章标题"
description: "文章简介（首页卡片展示）"
date: 2025-03-20
tags:
  - Tag1
  - Tag2
---
```
