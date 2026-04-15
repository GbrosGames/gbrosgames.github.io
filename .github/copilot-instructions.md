# Gbros Games Blog — Project Guidelines

Jekyll blog for **Gbros Games**, a Unity-focused indie game studio. Built on the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy) (~5.0), hosted on GitHub Pages.

## Build & Dev

```bash
bundle install
bundle exec jekyll serve   # local preview at http://127.0.0.1:4000
```

Deploy is automated via GitHub Actions (`push` to `main`). The `tools/deploy.sh` script only runs inside the Actions environment — do **not** invoke it locally.

> GitHub Actions uses `fetch-depth: 0` (full history). This is required by `_plugins/posts-lastmod-hook.rb`, which reads git commit dates to populate `last_modified_at` on posts.

## Writing Posts

**File naming**: `_posts/YYYY-MM-DD-Title-With-Hyphens.md`

**Required front matter**:
```yaml
---
title: "Post Title"
date: YYYY-MM-DD
categories: [Unity]
tags: [Unity, Intermediate]
---
```

- `categories` — single-element array matching existing category (e.g. `[Unity]`)
- `tags` — array of relevant keywords
- No `image:` field is used; featured images are embedded inline in the post body

## Images & Assets

Store post images under `assets/img/posts/<topic>/` — create a new topic subfolder per post series.

- Preferred format: `.webp` (used in newer posts); `.jpg` acceptable
- Reference images with absolute paths: `/assets/img/posts/<topic>/filename.webp`
- **Avoid spaces in filenames** — existing posts use URL-encoded paths (`%20`) which is error-prone

## Conventions

- All posts have **TOC enabled** globally (via `_config.yml` defaults)
- Comments are **disabled globally**; enable per-post with `comments: true`
- Kramdown is the Markdown renderer; Rouge is the syntax highlighter
- Localization strings live in `_data/locales/` — `en.yml` is the source of truth
- Social links configured in `_data/contact.yml`; sharing targets in `_data/share.yml`
- Theme-level layouts/includes/sass are provided by the gem — do not recreate them locally unless overriding

## Key Files

| File | Purpose |
|------|---------|
| `_config.yml` | Site-wide config (author, URL, analytics, defaults) |
| `_plugins/posts-lastmod-hook.rb` | Auto-sets `last_modified_at` from git history |
| `_tabs/about.md` | About page content |
| `tools/deploy.sh` | CI build + html-proofer test + deploy (CI only) |
