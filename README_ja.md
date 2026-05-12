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

Astro の Content Collections を使って Markdown ファイルを管理し、`github-markdown-css` で GitHub README 風のスタイルで表示する静的ブログサイトです。

## 技術スタック

| カテゴリ | 技術 | バージョン |
|----------|------|-----------|
| フレームワーク | Astro (Content Layer API) | ^6.3.1 |
| 言語 | TypeScript (strict モード) | - |
| スキーマバリデーション | Zod (`astro/zod`) | 4.x |
| Markdown スタイリング | github-markdown-css (ライトテーマ) | ^5.9.0 |
| ランタイム | Node.js | >=22.12.0 |
| パッケージマネージャー | npm | - |
| CI | GitHub Actions | - |

## 特徴

- **Content Collections** — Astro Content Layer API（`glob()` ローダー + Zod スキーマバリデーション）による型安全な Markdown 管理
- **GitHub Markdown CSS** — `github-markdown-light.css` + `<article class="markdown-body">` で GitHub README ライクな表示
- **一覧・詳細ページ** — 新しい順に並べた記事一覧と動的ルーティングによる詳細ページ
- **自動公開** — `src/content/posts/` に `.md` ファイルを置くだけで自動的に記事として公開
- **日本語ロケール対応** — HTML `lang="ja"`、日付は `ja-JP` 形式で表示
- **ホットモジュールリロード** — 開発中のファイル変更を即座に反映
- **静的サイト生成** — 本番用にプリレンダリングされた HTML を出力

## ページ一覧

| ルート | ファイル | 説明 |
|-------|---------|------|
| `/` | `src/pages/index.astro` | トップページ — 記事一覧へのリンク |
| `/posts/` | `src/pages/posts/index.astro` | 記事一覧 — `pubDate` 降順（新しい順） |
| `/posts/:slug/` | `src/pages/posts/[...slug].astro` | 記事詳細 — GitHub マークダウンスタイル適用 |

## サンプル記事

| Slug | タイトル | 技術分野 |
|------|---------|---------|
| `sample` | TypeScript 5.0 の新機能を徹底解説 | TypeScript, Decorators |
| `astoro` | Astro 6.0 正式リリース | Astro, Cloudflare, CSP |
| `zed` | Zed Editor 1.0 正式リリース | Rust, GPUI, AI Agent |

## プロジェクト構成

```
astro-output/
├── .github/
│   ├── workflows/
│   │   ├── deploy.yml          # GitHub Pages デプロイ
│   │   └── test.yml            # Build テスト (push/PR時)
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── FUNDING.yml
├── public/
├── src/
│   ├── content/
│   │   └── posts/              # Markdown 記事ディレクトリ
│   │       ├── sample.md
│   │       ├── astoro.md
│   │       └── zed.md
│   ├── content.config.ts       # Content Collections 設定 (glob ローダー + Zod スキーマ)
│   ├── layouts/
│   │   └── BlogLayout.astro    # 共通ベースレイアウト (lang="ja", slot)
│   └── pages/
│       ├── index.astro         # トップページ
│       └── posts/
│           ├── index.astro     # 記事一覧 (getCollection 使用)
│           └── [...slug].astro # 動的詳細ページ (getStaticPaths 使用)
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

## インストール

```bash
# リポジトリをクローン
git clone https://github.com/watanabe3tipapa/astro-output.git

# ディレクトリに移動
cd astro-output

# 依存関係をインストール
npm install
```

## 使い方

```bash
# 開発サーバー起動（ホットリロード対応）
npm run dev
# → http://localhost:4321/

# 本番用ビルド
npm run build
# → dist/

# ビルド結果のプレビュー
npm run preview
```

`src/content/posts/` に以下の frontmatter 形式で `.md` ファイルを追加してください：

```yaml
---
title: "記事タイトル"
description: "記事の説明"
pubDate: 2026-01-01
---
```

## デプロイ

### GitHub Pages

1. `main` ブランチにプッシュ
2. GitHub Actions が自動でビルド・デプロイ（`.github/workflows/deploy.yml`）
3. `https://watanabe3tipapa.github.io/astro-output/` で公開

**前提条件:** リポジトリ Settings > Pages > Source を "GitHub Actions" に設定

## 主要な実装詳細

### Content Collections（`src/content.config.ts`）

Astro v6 の Content Layer API + `glob()` ローダーを使用。`src/content/posts/` 以下の `.md` ファイルをスキャンし、frontmatter を Zod スキーマでバリデーションします：

- `title`（文字列、必須）
- `description`（文字列、必須）
- `pubDate`（日付、任意）

### 記事詳細ページ（`src/pages/posts/[...slug].astro`）

- `getStaticPaths()` + `getCollection('posts')` で静的パスを自動生成
- `render()`（`astro:content`）で Markdown を HTML に変換
- `github-markdown-light.css` を直接インポート
- コンテンツを `<article class="markdown-body">` でラップ

### 記事一覧ページ（`src/pages/posts/index.astro`）

- `getCollection('posts')` で全記事を取得
- `pubDate` 降順（新しい順）にソート
- `pubDate` がない記事はエポックにフォールバック

## コントリビューション

コントリビューションは大歓迎です！まず[CONTRIBUTING.md](CONTRIBUTING.md)と[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)をお読みください。

1. リポジトリをフォーク
2. 機能ブランチを作成 (`git checkout -b feature/amazing-feature`)
3. 変更をコミット (`git commit -m 'Add amazing feature'`)
4. ブランチにプッシュ (`git push origin feature/amazing-feature`)
5. Pull Requestを作成

## ライセンス

MITライセンス — 詳細は[LICENSE](LICENSE)ファイルを参照してください。

## 連絡先

GitHub: [https://github.com/watanabe3tipapa/astro-output](https://github.com/watanabe3tipapa/astro-output)
