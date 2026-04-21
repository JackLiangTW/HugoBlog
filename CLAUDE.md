# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Jack Liang's personal portfolio and blog, built with [Hugo](https://gohugo.io/) using the `hugo-creative-portfolio-theme`. Deployed to GitHub Pages at https://jackliangtw.github.io/. Most authored content is in Traditional Chinese; `config.toml` still declares `languageCode = 'en-us'`.

## Repository layout

This repo is a wrapper. All Hugo source lives one level down in `hugoblog/`:

- `hugoblog/config.toml` — site config, theme selection, sidebar/nav links, social links.
- `hugoblog/content/blog/` — blog posts (flat directory of `.md` files).
- `hugoblog/content/portfolio/` — portfolio entries shown on the homepage grid.
- `hugoblog/content/about/_index.md` — about page.
- `hugoblog/content/portfolioBackup/` — old portfolio drafts, not rendered by active layouts.
- `hugoblog/themes/hugo-creative-portfolio-theme/` — vendored theme (not a submodule); layouts, partials, theme archetypes live here.
- `hugoblog/static/img/` — user-added images referenced from content (`img/portfolio/*`, `img/blog/*`).
- `hugoblog/archetypes/default.md` — site-level archetype (YAML `---` front matter, `draft: true`). The theme also ships `archetypes/portfolio.md` used when you `hugo new portfolio/...`.

### Submodules (important)

`.gitmodules` declares **two submodules pointing at the same repo** (`jackliangtw.github.io`):

- `hugoblog/public/` — Hugo's build output directory. `hugo` writes generated HTML here; committing and pushing this submodule is how the site deploys.
- `hugoblog/jackliangtw.github.io/` — a second checkout of the same production repo at a sibling path.

When editing, assume `hugoblog/public/` is generated output — do not hand-edit files there. Regenerate via `hugo` and commit inside the submodule to deploy.

## Commands

All Hugo commands must be run from `hugoblog/` (the directory containing `config.toml`).

```bash
cd hugoblog

# Build the site into ./public (the GitHub Pages submodule)
hugo -t hugo-creative-portfolio-theme

# Local dev server with live reload, including drafts
hugo server -D

# Scaffold a new blog post / portfolio entry from archetypes
hugo new blog/My_New_Post.md
hugo new portfolio/MyProject.md
```

There are no tests, linters, or package manager — this is a pure Hugo site.

## Content conventions

Existing content files use **TOML front matter** (`+++ ... +++`), not the YAML `---` form the site archetype emits. When adding new posts, match the surrounding style in the same folder.

Portfolio entries expect these fields (see `content/portfolio/Ezcon.md` for a reference):

```toml
+++
date = "..."
title = "..."
draft = false
image = "img/portfolio/<file>.jpg"   # relative to static/
showonlyimage = false
weight = 1                            # controls grid ordering
+++
```

Blog posts use `tags = "Hugo,Web"` (comma-separated string, not a list) — the theme's blog layout reads it in that form.

Filenames use PascalCase / snake_case mix (e.g. `AWS_Boto3_Lambda.md`, `TypeScript_extends_vs_implement.md`) — follow neighbors when adding files so URLs stay consistent with existing linked posts.

## Theme notes

The theme is vendored (not a git submodule), so it's safe to edit files under `hugoblog/themes/hugo-creative-portfolio-theme/layouts/` when a layout change is needed. Prefer overriding via `hugoblog/layouts/` if the override is site-specific — Hugo resolves site layouts before theme layouts.

Theme params available in `config.toml` include `style` (color scheme), `sidebarAbout`, `navlinks`, and `social` — do not invent new param keys without checking the partials in `themes/.../layouts/partials/` first.

## Deployment flow

1. Edit content under `hugoblog/content/`.
2. Run `hugo -t hugo-creative-portfolio-theme` from `hugoblog/` to regenerate `hugoblog/public/`.
3. Commit & push inside the `hugoblog/public/` submodule (this is `jackliangtw.github.io`, the GitHub Pages repo).
4. Commit the submodule pointer update in the outer repo.
