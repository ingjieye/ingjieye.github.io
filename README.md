# yyj.us

Personal blog, deployed on GitHub Pages.

Built with [Astro](https://astro.build) and the [Retypeset](https://github.com/radishzzz/astro-theme-retypeset) theme.

## Local development

```bash
pnpm install
pnpm dev      # http://localhost:4321
pnpm build    # outputs to dist/
pnpm preview
```

## Writing a post

Create a Markdown file under `src/content/posts/`:

```markdown
---
title: Title
published: 2026-09-10
tags:
  - Tag
abbrlink: url-slug   # determines the post URL: /posts/<abbrlink>/
---
```

Put images in `src/content/posts/_images/` and reference them with a relative path
like `./_images/foo.png`.

## Deployment

Pushing to `master` triggers `.github/workflows/deploy.yml`, which builds the site
and publishes it to GitHub Pages.

## History

This site ran on Hexo until 2021. The final build output of that era is preserved
in the tag `backup/hexo-master-20260910`. Redirects from the old post URLs are
configured under `redirects` in `astro.config.ts`.
