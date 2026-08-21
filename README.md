# Astro Output

[![Astro](https://img.shields.io/badge/Astro-6.3-BC52EE?logo=astro)](https://astro.build)
[![Node.js](https://img.shields.io/badge/Node.js-22.12%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Build](https://github.com/watanabe3tipapa/astro-output/actions/workflows/test.yml/badge.svg)](https://github.com/watanabe3tipapa/astro-output/actions/workflows/test.yml)
[![License](https://img.shields.io/github/license/watanabe3tipapa/astro-output)](LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/watanabe3tipapa/astro-output/main)](https://github.com/watanabe3tipapa/astro-output/commits/main)

**Publish Markdown as clearly as a GitHub README.**

Astro Output is a small technical blog that turns Markdown articles managed with Astro Content Collections into a static site styled like GitHub. Adding an article to `src/content/posts/` provides one coherent workflow: type-safe metadata validation, inclusion in the article index, static generation of the detail page, and publication on GitHub Pages.

[![Open the Live Site](https://img.shields.io/badge/Live%20Site-Open-0969DA?style=for-the-badge)](https://watanabe3tipapa.github.io/astro-output/)

[English](README.md) | [日本語](README_ja.md)

## Concept

A CMS or complex backend is not always required to publish technical writing. Astro Output uses Markdown as the single source of content, keeps its history in Git, renders it as readable HTML, and publishes the resulting static site.

Content Collections and a Zod schema validate the title and description required for each article. The `[...slug].astro` route generates each static page, while `github-markdown-css` renders headings, code blocks, tables, and quotations in a GitHub README-like appearance.

| Need | How Astro Output addresses it |
| --- | --- |
| Add articles quickly | Add a Markdown file under `src/content/posts/`; the post is discovered automatically. |
| Avoid incomplete metadata | Validate frontmatter through Content Collections and a Zod schema. |
| Present technical content clearly | Apply `github-markdown-light.css` for GitHub-like Markdown rendering. |
| Keep operations simple | Generate static HTML and deploy it to GitHub Pages through GitHub Actions. |

## Key Features

| Feature | Description |
| --- | --- |
| **Content Collections** | A `glob()` loader discovers Markdown posts and a Zod schema validates `title`, `description`, and `pubDate`. |
| **Article index and detail pages** | Generates an index sorted by publication date and individual article pages with `getStaticPaths()`. |
| **GitHub-like presentation** | Uses `<article class="markdown-body">` and `github-markdown-light.css` for readable tables, code, and quotations. |
| **Japanese-friendly defaults** | Sets the document language to `ja` and formats publication dates with the `ja-JP` locale. |
| **Continuous quality checks** | GitHub Actions runs a production build on pushes to `main` and on pull requests. |
| **Automatic publication** | A push to `main` builds and deploys the static site to GitHub Pages. |

## Quick Start

### Prerequisites

| Tool | Required version | Check command |
| --- | --- | --- |
| Node.js | `>=22.12.0` | `node --version` |
| npm | Bundled with Node.js | `npm --version` |

### 1. Clone the repository and install dependencies

```bash
git clone https://github.com/watanabe3tipapa/astro-output.git
cd astro-output
npm ci
```

### 2. Start the development server

```bash
npm run dev
```

Open `http://localhost:4321/` in your browser. The development server reflects changes when you save an article Markdown file.

### 3. Verify a production build

```bash
npm run build
npm run preview
```

`npm run build` writes static files to `dist/`. Use `npm run preview` to inspect the production build locally.

## Add and Publish an Article

Create a `.md` file with any name under `src/content/posts/`. The file name becomes the slug in the article URL.

```md
---
title: "Article title"
description: "A concise summary of the article."
pubDate: 2026-08-22
---

## Introduction

Write the article body in Markdown here.
```

After writing the post, verify the local build and then update the `main` branch.

```bash
npm run build
git add src/content/posts/your-article.md
git commit -m "docs: add article"
git push origin main
```

When GitHub Actions finishes building and deploying, the article appears in the [live site](https://watanabe3tipapa.github.io/astro-output/) index and at `/posts/your-article/`.

## Technology Stack

| Category | Technology | Role in this repository |
| --- | --- | --- |
| Framework | [Astro](https://astro.build/) `^6.3.1` | Static-site generation, routing, and Content Collections. |
| Language | TypeScript | Type safety for content configuration and pages. |
| Content validation | Zod | Schema validation for article frontmatter. |
| Markdown presentation | [github-markdown-css](https://github.com/sindresorhus/github-markdown-css) | GitHub-like Markdown styling. |
| Runtime | Node.js `>=22.12.0` | Local development, builds, and CI. |
| CI/CD | GitHub Actions + GitHub Pages | Build verification and automated static-site deployment. |

## Project Structure

```text
astro-output/
├── .github/
│   └── workflows/
│       ├── test.yml                    # Build verification on push / PR
│       └── deploy.yml                  # GitHub Pages deployment
├── public/                             # Static assets such as favicons
├── src/
│   ├── content/
│   │   └── posts/                      # Published Markdown articles
│   │       ├── astro-7-2-incremental-builds.md
│   │       ├── astro.md
│   │       ├── sample.md
│   │       └── zed.md
│   ├── content.config.ts               # Content Collections and Zod schema
│   ├── layouts/
│   │   └── BlogLayout.astro            # Shared HTML layout
│   └── pages/
│       ├── index.astro                 # Home page
│       └── posts/
│           ├── index.astro             # Article index
│           └── [...slug].astro         # Dynamic article-detail route
├── astro.config.mjs                    # GitHub Pages site/base configuration
├── package.json
├── README.md
└── README_ja.md
```

## Published Articles

| Slug | Article | Topics |
| --- | --- | --- |
| [`astro-7-2-incremental-builds`](https://watanabe3tipapa.github.io/astro-output/posts/astro-7-2-incremental-builds/) | Astro 7.2 incremental static builds | Astro, Content Collections, GitHub Actions |
| [`astro`](https://watanabe3tipapa.github.io/astro-output/posts/astro/) | Astro 6.0 release | Astro, Cloudflare, CSP |
| [`zed`](https://watanabe3tipapa.github.io/astro-output/posts/zed/) | Zed Editor 1.0 release | Rust, GPUI, AI agents |
| [`sample`](https://watanabe3tipapa.github.io/astro-output/posts/sample/) | A practical guide to TypeScript 5.0 features | TypeScript, Decorators |

## Quality Checks and Deployment

Run the following command locally to verify the production build, including Content Collections synchronization.

```bash
npm run build
```

| Event | Executed work | Workflow |
| --- | --- | --- |
| Push to `main` | Install dependencies and run a production build | [`test.yml`](.github/workflows/test.yml) |
| Pull request | Install dependencies and run a production build | [`test.yml`](.github/workflows/test.yml) |
| Push to `main` | Build the static site and deploy to GitHub Pages | [`deploy.yml`](.github/workflows/deploy.yml) |

When using GitHub Pages for the first time, select **GitHub Actions** under **Settings → Pages → Source**. The site is published at [`https://watanabe3tipapa.github.io/astro-output/`](https://watanabe3tipapa.github.io/astro-output/).

## Documentation

| Document | Description |
| --- | --- |
| [CHANGELOG.md](CHANGELOG.md) | Change history. |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution workflow. |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | Code of conduct. |
| [SECURITY.md](SECURITY.md) | Security-reporting policy. |
| [DEV-MEMO.md](DEV-MEMO.md) | Development notes. |

## Contributing

Suggestions and pull requests are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) first. After making a change, run `npm run build` and confirm that the build passes before opening a pull request.

## License

This repository is available under the [MIT License](LICENSE).
