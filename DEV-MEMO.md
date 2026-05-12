# DEV-MEMO

開発者向けメモ。プロジェクト構築の記録と実装のポイント。

## プロジェクト概要

Astro の Content Collections 機能を使って Markdown ファイルを管理し、`github-markdown-css` で GitHub README 風のスタイルで HTML 表示する静的ブログ。

## セットアップ手順（実績）

```bash
# 1. Astro プロジェクト初期化（v6.3.1）
npm create astro@latest . -- --template basics --typescript strict --no-install

# 2. 依存関係インストール
npm install
npm install github-markdown-css

# 3. src/content/posts/ に Markdown ファイルを配置
# 4. src/content.config.ts に Content Collections 設定
# 5. 各種ページ・レイアウトを作成
```

## 技術スタック（確定）

| カテゴリ | 技術 | バージョン | 備考 |
|---------|------|-----------|------|
| Framework | Astro | ^6.3.1 | Content Layer API 使用 |
| Language | TypeScript | - | strict モード |
| Schema | Zod | 4.x | `astro/zod` から import |
| CSS | github-markdown-css | ^5.9.0 | ライトテーマのみ |
| Node.js | Node | >=22.12.0 | Astro v6 要件 |
| CI | GitHub Actions | - | test.yml (build) |

## ディレクトリ構成（実態）

```
astro-output/
├── src/
│   ├── content.config.ts          # Content Collections 設定（glob loader + Zod schema）
│   ├── content/posts/             # Markdown 記事置き場
│   │   ├── sample.md              # TypeScript 5.0 解説
│   │   ├── astoro.md              # Astro 6.0 新機能
│   │   └── zed.md                 # Zed Editor 1.0
│   ├── layouts/
│   │   └── BlogLayout.astro       # 共通レイアウト（lang="ja", slot）
│   └── pages/
│       ├── index.astro            # トップページ
│       └── posts/
│           ├── index.astro        # 記事一覧（getCollection, 日付降順）
│           └── [...slug].astro    # 詳細ページ（getStaticPaths + render）
├── .github/
│   ├── workflows/
│   │   └── test.yml               # build テスト
│   ├── ISSUE_TEMPLATE/
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── FUNDING.yml
├── astro.config.mjs
├── package.json
├── tsconfig.json
├── LICENSE (MIT)
└── README.md / README_ja.md など
```

## 実装のポイント・注意点

### 1. Astro v6 Content Collections（v5 からの変更）

- `src/content/config.ts` → `src/content/config.ts` から `src/content.config.ts`（プロジェクトルート直下）に変更
- `type: 'content'` → `loader: glob({...})` に変更
- `post.render()` → `render(post)`（`astro:content` から import）に変更
- `getEntryBySlug()` は削除済み

### 2. Zod インポート元

Astro v6 では `z` を `astro:content` ではなく `astro/zod` から import する（Zod 4 対応）。ただし現状の `astro/zod` 経由でも動作確認済み。

### 3. github-markdown-css の適用

```astro
---
import 'github-markdown-css/github-markdown-light.css';
---
<article class="markdown-body">
  <Content />
</article>
```

### 4. getStaticPaths の型

`getStaticPaths()` で生成した props は `Astro.props` で受け取る。Content Collection entry はそのまま渡す。

### 5. 日付処理

`pubDate` は `z.date().optional()` で任意。一覧ページでは `pubDate` がない場合エポックにフォールバックし、降順ソートに組み込む。

## 今後の TODO

- [ ] `astro.config.mjs` に `site` / `base` を設定（GitHub Pages 対応）
- [ ] `.github/workflows/deploy.yml` を作成
- [ ] GitHub Pages 公開テスト
- [ ] OGP 画像の設定
- [ ] 記事の追加・充実
