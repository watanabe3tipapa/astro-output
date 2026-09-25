---
title: "GitHub Actions で Astro サイトを GitHub Pages に自動デプロイする——本サイトの実装を解説"
description: "本ブログ（Astro 静的サイト）を GitHub Actions で GitHub Pages へ自動デプロイするまでの workflow を、実ファイルを交えて詳しく紹介します。"
pubDate: 2026-09-25
---

## はじめに

静的サイトをホストする最も手軽な方法のひとつが **GitHub Pages** です。このブログも Astro でビルドした成果物を GitHub Pages に公開していますが、そのデプロイはすべて **GitHub Actions** に任せています。

本記事では、このサイトで実際に動いている workflow（`.github/workflows/` 配下のファイル）を紹介し、GitHub Pages への自動デプロイの仕組みを解説します。

## 全体像

`.github/workflows/` には次の 2 つの workflow を配置しています。

| ファイル名 | 役割 | トリガー |
|-----------|------|---------|
| `test.yml` | ビルドの正常性確認 | `push` / `pull_request`（main 限定） |
| `deploy.yml` | ビルド + GitHub Pages へのデプロイ | `push`（main 限定） / `workflow_dispatch` |

`deploy.yml` にビルドも含むため一見 `test.yml` は冗長に見えますが、**PR 段階でビルド可否を素早く検知**するしくみとして独立させています。

## test.yml — ビルドテスト

```yaml
name: test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
```

ポイントは `npm ci` を使っている点です。`npm install` と違い `package-lock.json` を厳密に尊重するため、ローカルと CI とで依存関係が一致し、再現性のあるビルドが行えます。

## deploy.yml — デプロイ

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - name: Delete old artifacts
        run: |
          gh api /repos/${{ github.repository }}/actions/artifacts \
            --paginate --jq '.artifacts[] | select(.name == "github-pages") | .id' \
            | xargs -I {} gh api -X DELETE /repos/${{ github.repository }}/actions/artifacts/{}
        continue-on-error: true
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### 各セクションの役割

#### permissions

GitHub Pages へのデプロイに必要な権限を最小構成で宣言しています。

| 権限 | 用途 |
|------|------|
| `contents: read` | リポジトリの読み取り |
| `pages: write` | Pages への書き込み |
| `id-token: write` | OIDC トークン発行（デプロイの認証に使用） |

#### concurrency

連続した push が起きたときにデプロイが競合しないよう、`group: pages` で直列化しています。`cancel-in-progress: false` なので、進行中のデプロイを無理に中断せず、終了後に次のデプロイが実行されます。

#### ビルド工程

```yaml
- run: npm ci
- run: npm run build
```

`astro build` の出力先はデフォルトで `dist/` です。これを `actions/upload-pages-artifact@v3` でアップロードします。

```yaml
- uses: actions/upload-pages-artifact@v3
  with:
    path: dist
```

#### デプロイ工程

`deploy` ジョブは `actions/deploy-pages@v4` を実行するだけのシンプルな構成です。`build` ジョブがアップロードした artifact を GitHub 側が展開して公開します。

```yaml
environment:
  name: github-pages
  url: ${{ steps.deployment.outputs.page_url }}
```

`environment` を指定することで、Actions 画面上で「デプロイ済み環境」として管理でき、公開 URL も確認できます。

## 旧方式から変更した点

当初は `actions/configure-pages` を挟む構成にしていましたが、`astro build` の結果（フル静的サイト）をそのまま公開できる本構成では **設定ファイル（`_config.yml`）や `configure-pages` が不要** なため、削除しています。構成がシンプルになり、ワークフローの失敗箇所が減りました。

あわせて、artifact 名の重複で旧バージョンが残り続ける問題（"duplicate artifact" エラー）にも遭遇しました。対応として、アップロード前に既存の `github-pages` artifact を `gh api` で削除するステップを追加しています。

```yaml
- name: Delete old artifacts
  run: |
    gh api /repos/${{ github.repository }}/actions/artifacts \
      --paginate --jq '.artifacts[] | select(.name == "github-pages") | .id' \
      | xargs -I {} gh api -X DELETE /repos/${{ github.repository }}/actions/artifacts/{}
  continue-on-error: true
```

## GitHub Pages 側の設定

workflow だけでなく、リポジトリ側にも次の設定が必要です。

1. リポジトリの **Settings → Pages** を開く
2. **Build and deployment / Source** を **GitHub Actions** に変更
3. 保存後、`main` へ push すれば workflow が動きデプロイされる

source が「Deploy from a branch」のままだと、Actions からのデプロイが反映されないので注意してください。

## 公開 URL の差異

`astro.config.mjs` には `site` と `base` を設定しています。

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://watanabe3tipapa.github.io',
  base: '/astro-output/',
});
```

`base` はリポジトリ名に揃えます。ユーザーページ（`<user>.github.io`）の場合は `/` でよいですが、プロジェクトページ（`<user>.github.io/<repo>`）ではリポジトリ名を指定しないとアセットのパスが崩れます。

## まとめ

GitHub Pages への公開はわずか 2 つの workflow ファイルで完結します。

- `test.yml` でビルドの可否を PR 段階でチェック
- `deploy.yml` で `build` → artifact アップロード → `deploy` の 2 段階ジョブで公開
- リポジトリ設定の **Source を GitHub Actions** に変更するのを忘れない

ワークフロー全体の動きは [こちら](https://github.com/watanabe3tipapa/astro-output/actions) でも確認できます。静的サイトのホスティング先に GitHub Pages を検討している方の参考になれば幸いです。