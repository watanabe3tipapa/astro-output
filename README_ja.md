# Astro Output

[![Astro](https://img.shields.io/badge/Astro-6.3-BC52EE?logo=astro)](https://astro.build)
[![Node.js](https://img.shields.io/badge/Node.js-22.12%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Build](https://github.com/watanabe3tipapa/astro-output/actions/workflows/test.yml/badge.svg)](https://github.com/watanabe3tipapa/astro-output/actions/workflows/test.yml)
[![License](https://img.shields.io/github/license/watanabe3tipapa/astro-output)](LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/watanabe3tipapa/astro-output/main)](https://github.com/watanabe3tipapa/astro-output/commits/main)

**Markdown を、GitHub の README のように読みやすく公開する。**

Astro Output は、Astro の Content Collections で管理する Markdown 記事を、GitHub 風のスタイルで静的サイトとして公開する小さな技術ブログです。記事ファイルを `src/content/posts/` に追加するだけで、型安全なメタデータ検証、記事一覧への掲載、個別ページの静的生成、GitHub Pages への公開までを一貫して行えます。

[![公開サイトを開く](https://img.shields.io/badge/Live%20Site-Open-0969DA?style=for-the-badge)](https://watanabe3tipapa.github.io/astro-output/)

[English](README.md) | [日本語](README_ja.md)

## コンセプト

技術記事を公開するために、CMSや複雑なバックエンドが常に必要とは限りません。Astro Output は、Git で変更履歴を管理できる Markdown をコンテンツの唯一の情報源とし、読みやすいHTMLへ変換して公開する構成を示します。

Content Collections と Zod スキーマにより、公開時に必要なタイトルと説明を検証します。`[...slug].astro` が各記事の静的ページを生成し、`github-markdown-css` がコードブロック、表、見出しを含む Markdown を GitHub README に近い見た目で表示します。

| 課題 | Astro Output の対応 |
| --- | --- |
| 記事を素早く追加したい | `src/content/posts/` に Markdown ファイルを追加するだけで記事を検出する。 |
| メタデータの不備を避けたい | Content Collections と Zod スキーマでフロントマターを検証する。 |
| 読みやすい技術記事を表示したい | `github-markdown-light.css` を使い、GitHub 風の Markdown 表示を適用する。 |
| 運用をシンプルに保ちたい | 静的HTMLを生成し、GitHub Actions から GitHub Pages へ自動デプロイする。 |

## 主な機能

| 機能 | 内容 |
| --- | --- |
| **Content Collections** | `glob()` ローダーが記事Markdownを収集し、Zod スキーマで `title`、`description`、`pubDate` を検証する。 |
| **記事一覧と詳細ページ** | 公開日が新しい順の記事一覧と、`getStaticPaths()` による個別記事ページを生成する。 |
| **GitHub 風の表示** | `<article class="markdown-body">` と `github-markdown-light.css` により、表・コード・引用を読みやすく表示する。 |
| **日本語向けの既定値** | HTML の言語を `ja` に設定し、公開日は `ja-JP` 形式で表示する。 |
| **継続的な品質確認** | `main` へのプッシュとプルリクエストで、GitHub Actions が本番ビルドを実行する。 |
| **自動公開** | `main` へのプッシュを契機に、生成した静的サイトを GitHub Pages へデプロイする。 |

## クイックスタート

### 前提条件

| ツール | 必要バージョン | 確認コマンド |
| --- | --- | --- |
| Node.js | `>=22.12.0` | `node --version` |
| npm | Node.js に同梱 | `npm --version` |

### 1. リポジトリを取得して依存関係を導入する

```bash
git clone https://github.com/watanabe3tipapa/astro-output.git
cd astro-output
npm ci
```

### 2. 開発サーバーを起動する

```bash
npm run dev
```

ブラウザで `http://localhost:4321/` を開きます。記事Markdownを保存すると、開発サーバーが変更を反映します。

### 3. 本番ビルドを確認する

```bash
npm run build
npm run preview
```

`npm run build` は静的ファイルを `dist/` に生成します。`npm run preview` では、本番ビルドの結果をローカルで確認できます。

## 記事を追加して公開する

`src/content/posts/` に任意の名前で `.md` ファイルを作成します。ファイル名は記事URLのslugになります。

```md
---
title: "記事タイトル"
description: "記事の概要を簡潔に記述します。"
pubDate: 2026-08-22
---

## はじめに

ここから Markdown で本文を書きます。
```

記事を追加したら、ローカルビルドで検証してから `main` ブランチへ反映します。

```bash
npm run build
git add src/content/posts/your-article.md
git commit -m "docs: add article"
git push origin main
```

GitHub Actions がビルドとデプロイを完了すると、記事は [公開サイト](https://watanabe3tipapa.github.io/astro-output/) の記事一覧と `/posts/your-article/` に表示されます。

## 技術構成

| 分類 | 技術 | このリポジトリでの役割 |
| --- | --- | --- |
| フレームワーク | [Astro](https://astro.build/) `^6.3.1` | 静的サイト生成、ルーティング、Content Collections。 |
| 言語 | TypeScript | コンテンツ設定とページ実装の型安全性。 |
| コンテンツ検証 | Zod | 記事フロントマターのスキーマ検証。 |
| Markdown表示 | [github-markdown-css](https://github.com/sindresorhus/github-markdown-css) | GitHub 風の Markdown スタイル。 |
| 実行環境 | Node.js `>=22.12.0` | ローカル開発、ビルド、CI。 |
| CI/CD | GitHub Actions + GitHub Pages | ビルド検証と静的サイトの自動公開。 |

## プロジェクト構成

```text
astro-output/
├── .github/
│   └── workflows/
│       ├── test.yml                    # push / PR 時のビルド検証
│       └── deploy.yml                  # GitHub Pages へのデプロイ
├── public/                             # favicon などの静的アセット
├── src/
│   ├── content/
│   │   └── posts/                      # 公開するMarkdown記事
│   │       ├── astro-7-2-incremental-builds.md
│   │       ├── astro.md
│   │       ├── sample.md
│   │       └── zed.md
│   ├── content.config.ts               # Content Collections とZodスキーマ
│   ├── layouts/
│   │   └── BlogLayout.astro            # HTML共通レイアウト
│   └── pages/
│       ├── index.astro                 # トップページ
│       └── posts/
│           ├── index.astro             # 記事一覧
│           └── [...slug].astro         # 記事詳細の動的ルート
├── astro.config.mjs                    # GitHub Pagesのsite/base設定
├── package.json
├── README.md
└── README_ja.md
```

## 掲載記事

| Slug | 記事 | 分野 |
| --- | --- | --- |
| [`astro-7-2-incremental-builds`](https://watanabe3tipapa.github.io/astro-output/posts/astro-7-2-incremental-builds/) | Astro 7.2 の増分静的ビルドを解説 | Astro、Content Collections、GitHub Actions |
| [`astro`](https://watanabe3tipapa.github.io/astro-output/posts/astro/) | Astro 6.0 正式リリース | Astro、Cloudflare、CSP |
| [`zed`](https://watanabe3tipapa.github.io/astro-output/posts/zed/) | Zed Editor 1.0 正式リリース | Rust、GPUI、AIエージェント |
| [`sample`](https://watanabe3tipapa.github.io/astro-output/posts/sample/) | TypeScript 5.0 の新機能を徹底解説 | TypeScript、Decorators |

## 品質確認とデプロイ

ローカルでは、次のコマンドでContent Collectionsの同期を含む本番ビルドを確認できます。

```bash
npm run build
```

| イベント | 実行される処理 | ワークフロー |
| --- | --- | --- |
| `main` へのプッシュ | 依存関係の導入と本番ビルド | [`test.yml`](.github/workflows/test.yml) |
| プルリクエスト | 依存関係の導入と本番ビルド | [`test.yml`](.github/workflows/test.yml) |
| `main` へのプッシュ | 静的サイトのビルドとGitHub Pagesへのデプロイ | [`deploy.yml`](.github/workflows/deploy.yml) |

GitHub Pages を初めて利用する場合は、リポジトリの **Settings → Pages → Source** で **GitHub Actions** を選択してください。公開先は [`https://watanabe3tipapa.github.io/astro-output/`](https://watanabe3tipapa.github.io/astro-output/) です。

## 関連文書

| 文書 | 内容 |
| --- | --- |
| [CHANGELOG.md](CHANGELOG.md) | 変更履歴。 |
| [CONTRIBUTING_ja.md](CONTRIBUTING_ja.md) | コントリビューションの手順。 |
| [CODE_OF_CONDUCT_ja.md](CODE_OF_CONDUCT_ja.md) | 行動規範。 |
| [SECURITY_ja.md](SECURITY_ja.md) | セキュリティ報告方針。 |
| [DEV-MEMO.md](DEV-MEMO.md) | 開発メモ。 |

## コントリビューション

改善提案とプルリクエストを歓迎します。まず [CONTRIBUTING_ja.md](CONTRIBUTING_ja.md) と [CODE_OF_CONDUCT_ja.md](CODE_OF_CONDUCT_ja.md) を確認してください。変更後は `npm run build` を実行し、ビルドが成功することを確認したうえでプルリクエストを作成してください。

## ライセンス

このリポジトリは [MIT License](LICENSE) の下で公開されています。
