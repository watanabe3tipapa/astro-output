[![Astro Output](https://via.placeholder.com/150x50?text=Astro+Output)](https://github.com/watanabe3tipapa/astro-output)

<!-- badges -->
[![License](https://img.shields.io/github/license/watanabe3tipapa/astro-output.svg)](LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/watanabe3tipapa/astro-output/main.svg)](https://github.com/watanabe3tipapa/astro-output/commits/main)
[![Tests](https://github.com/watanabe3tipapa/astro-output/actions/workflows/test.yml/badge.svg)](https://github.com/watanabe3tipapa/astro-output/actions)
[![Node](https://img.shields.io/badge/Node-22.12%2B-339933)](https://nodejs.org)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Maintenance](https://img.shields.io/badge/Maintenance-Active-brightgreen.svg)](https://github.com/watanabe3tipapa/astro-output)

[English](README.md) | [日本語](README_ja.md)

# Astro Output

A static blog site built with Astro that renders Markdown files into GitHub README-style HTML pages using `github-markdown-css`.

## Tech Stack

| Category | Technology | Version |
|----------|-----------|---------|
| Framework | Astro (Content Layer API) | ^6.3.1 |
| Language | TypeScript (strict mode) | - |
| Schema Validation | Zod (via `astro/zod`) | 4.x |
| Markdown Styling | github-markdown-css (light theme) | ^5.9.0 |
| Runtime | Node.js | >=22.12.0 |
| Package Manager | npm | - |
| CI | GitHub Actions | - |

## Features

- **Content Collections** — Type-safe Markdown management using Astro's Content Layer API with `glob()` loader and Zod schema validation
- **GitHub Markdown CSS** — Renders articles in GitHub README-like style with `github-markdown-light.css` and `<article class="markdown-body">`
- **List & Detail Pages** — Index page listing all posts (newest first) and dynamic route for individual articles
- **Auto Discovery** — Place `.md` files in `src/content/posts/` and they automatically become published pages
- **Japanese Locale** — HTML `lang="ja"`, dates formatted in `ja-JP` locale
- **Hot Module Replacement** — Instant feedback during development
- **Static Site Generation** — Pre-rendered HTML output for production

## Pages

| Route | File | Description |
|-------|------|-------------|
| `/` | `src/pages/index.astro` | Top page — links to article list |
| `/posts/` | `src/pages/posts/index.astro` | Article index — sorted by `pubDate` (newest first) |
| `/posts/:slug/` | `src/pages/posts/[...slug].astro` | Article detail — GitHub markdown style applied |

## Articles

| Slug | Title | Tech |
|------|-------|------|
| `sample` | TypeScript 5.0 の新機能を徹底解説 | TypeScript, Decorators |
| `astoro` | Astro 6.0 正式リリース | Astro, Cloudflare, CSP |
| `zed` | Zed Editor 1.0 正式リリース | Rust, GPUI, AI Agent |

## Project Structure

```
astro-output/
├── .github/
│   ├── workflows/
│   │   ├── deploy.yml          # GitHub Pages deploy
│   │   └── test.yml            # Build test on push/PR
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── FUNDING.yml
├── public/
├── src/
│   ├── content/
│   │   └── posts/              # Markdown articles directory
│   │       ├── sample.md
│   │       ├── astoro.md
│   │       └── zed.md
│   ├── content.config.ts       # Content Collections config (glob loader + Zod schema)
│   ├── layouts/
│   │   └── BlogLayout.astro    # Shared base layout (lang="ja", slot)
│   └── pages/
│       ├── index.astro         # Top page
│       └── posts/
│           ├── index.astro     # Article listing via getCollection()
│           └── [...slug].astro # Dynamic detail page via getStaticPaths()
├── astro.config.mjs
├── package.json
├── tsconfig.json
├── LICENSE                     # MIT
├── CHANGELOG.md
├── CONTRIBUTING.md
├── CONTRIBUTING_ja.md
├── CODE_OF_CONDUCT.md
├── CODE_OF_CONDUCT_ja.md
├── SECURITY.md
├── SECURITY_ja.md
├── README.md
├── README_ja.md
└── DEV-MEMO.md
```

## Installation

```bash
# Clone the repository
git clone https://github.com/watanabe3tipapa/astro-output.git

# Navigate to the directory
cd astro-output

# Install dependencies
npm install
```

## Usage

```bash
# Start dev server with HMR
npm run dev
# → http://localhost:4321/

# Build static site for production
npm run build
# → dist/

# Preview production build
npm run preview
```

Add Markdown files to `src/content/posts/` with frontmatter:

```yaml
---
title: "Article Title"
description: "Article description"
pubDate: 2026-01-01
---
```

## Deployment

### GitHub Pages

1. Push changes to the `main` branch
2. GitHub Actions automatically builds and deploys via `.github/workflows/deploy.yml`
3. Published at `https://watanabe3tipapa.github.io/astro-output/`

**Prerequisites:** Go to repository Settings > Pages > Source > "GitHub Actions"

## Key Implementation Details

### Content Collections (`src/content.config.ts`)

Uses Astro v6's Content Layer API with `glob()` loader. The `posts` collection scans `src/content/posts/` for `.md` files and validates frontmatter against a Zod schema:

- `title` (string, required)
- `description` (string, required)
- `pubDate` (date, optional)

### Article Detail Page (`src/pages/posts/[...slug].astro`)

- Generates static paths via `getStaticPaths()` with `getCollection('posts')`
- Renders markdown content using `render()` from `astro:content`
- Applies `github-markdown-light.css` via direct import
- Wraps content in `<article class="markdown-body">`

### Article Listing (`src/pages/posts/index.astro`)

- Fetches all posts with `getCollection('posts')`
- Sorts by `pubDate` descending (newest first)
- Handles missing `pubDate` gracefully (falls back to epoch)

## Contributing

Contributions are welcome! Please read the [CONTRIBUTING.md](CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) guidelines first.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT License — see the [LICENSE](LICENSE) file for details.

## Contact

GitHub: [https://github.com/watanabe3tipapa/astro-output](https://github.com/watanabe3tipapa/astro-output)
