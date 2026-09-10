# yyj.us

个人博客，部署在 GitHub Pages。

基于 [Astro](https://astro.build) 和 [Retypeset](https://github.com/radishzzz/astro-theme-retypeset) 主题。

## 本地开发

```bash
pnpm install
pnpm dev      # http://localhost:4321
pnpm build    # 输出到 dist/
pnpm preview
```

## 写文章

在 `src/content/posts/` 下新建 Markdown：

```markdown
---
title: 标题
published: 2026-09-10
tags:
  - 标签
abbrlink: url-slug   # 决定文章 URL：/posts/<abbrlink>/
---
```

图片放 `src/content/posts/_images/`，用相对路径 `./_images/xxx.png` 引用。

## 部署

push 到 `master` 由 `.github/workflows/deploy.yml` 自动构建并发布到 GitHub Pages。

## 旧站

2021 年之前是 Hexo 站点，源码在 `hexo` 分支，最后一次构建产物在 tag `backup/hexo-master-20260910`。
旧文章 URL 的重定向配置在 `astro.config.ts` 的 `redirects` 中。
