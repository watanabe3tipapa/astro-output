---
title: "Astro 6.0 正式リリース — 開発サーバー刷新、Cloudflare Workers 第一級対応、Live Content Collections 安定版"
description: "2026年3月に安定版がリリースされた Astro 6.0 の主要新機能を詳しく解説します。"
pubDate: 2026-05-12
---

## はじめに

2026年3月10日、JavaScript 製 Web フレームワーク **Astro 6.0** が正式リリースされました。本リリースは、同年1月の Cloudflare による Astro 開発チーム買収発表後に初のメジャーバージョンアップであり、開発サーバーの完全刷新、Cloudflare Workers の第一級サポート、Content Security Policy の安定化など、多くの重要な変更が含まれています。

本記事では Astro 6.0 の主要な新機能と、v5 からの移行時に注意すべきポイントを解説します。

## Vite Environment API による開発サーバー刷新

Astro 6 の最大の変更点は、開発サーバー `astro dev` の完全再実装です。

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare(),
});
```

従来の開発サーバーはブラウザ環境をエミュレートしていましたが、Astro 6 では Vite の **Environment API** をベースに、**本番と同じランタイム**でコードが実行されます。これにより、以下の問題が解消されました。

| 項目 | Astro 5 以前 | Astro 6 |
|------|-------------|---------|
| ランタイム | ブラウザエミュレーション | 本番環境と同一 |
| プラットフォーム固有機能 | デプロイ後まで確認不可 | 開発中に確認可能 |
| HMR | 標準 | Workerd 環境でも動作 |

## First-Class Cloudflare Workers サポート

Cloudflare による買収を経て、Astro 6 は Cloudflare Workers のランタイム（Workerd）を開発環境で直接利用できるようになりました。

```typescript
import { getRuntime } from '@astrojs/cloudflare/runtime';

export async function GET() {
  const runtime = getRuntime();
  const kv = runtime.env.MY_KV_NAMESPACE;
  const value = await kv.get('key');
  return new Response(value);
}
```

開発中に以下の Cloudflare サービスへアクセス可能です:

- **Durable Objects** — サーバーレス状態管理
- **KV Namespaces** — グローバルキーバリューストレージ
- **R2 Storage** — オブジェクトストレージ
- **Workers Analytics Engine** — リアルタイムメトリクス

## Live Content Collections 安定版

Astro 5.10 で実験的に導入された **Live Content Collections** が安定版に昇格しました。従来の Content Collections はビルド時にのみコンテンツを解決しましたが、Live Collection は**リクエスト時に**データを取得します。

```typescript
// src/content.config.ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

export const collections = {
  posts: defineCollection({
    loader: glob({ pattern: '**/*.md', base: './src/content/posts' }),
    schema: z.object({
      title: z.string(),
      description: z.string(),
      pubDate: z.date().optional(),
    }),
  }),
};
```

```typescript
// 動的 CMS データを扱う Live Collection
import { defineLiveCollection } from 'astro:content';

const updates = defineLiveCollection({
  loader: cmsLoader({ apiKey: process.env.MY_API_KEY }),
  schema: z.object({
    id: z.string(),
    title: z.string(),
    body: z.string(),
  }),
});
```

## Content Security Policy (CSP)

CSP サポートが実験的機能から安定版に昇格しました。Astro が自動的にすべてのスクリプトとスタイルのハッシュを生成し、CSP ヘッダーに注入します。

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  security: {
    csp: {
      algorithm: 'SHA-512',
      directives: ["default-src 'self'"],
    },
  },
});
```

## Fonts API

カスタムフォント管理を自動化する Fonts API が追加されました。

```typescript
// astro.config.mjs
import { defineConfig, fontProviders } from 'astro/config';

export default defineConfig({
  fonts: [{
    name: 'Noto Sans JP',
    cssVariable: '--font-noto-sans-jp',
    provider: fontProviders.fontsource(),
  }],
});
```

コンポーネント側では `<Font>` コンポーネントでフォントのプリロードを指示できます。

```astro
---
import { Font } from 'astro:assets';
---

<Font cssVariable="--font-noto-sans-jp" preload />
```

## 実験的機能

### Rust コンパイラ

従来の Go ベースの `.astro` コンパイラに代わる、Rust 製のコンパイラが実験的に利用可能です。

```typescript
import { defineConfig } from 'astro/config';

export default defineConfig({
  experimental: {
    rustCompiler: true,
  },
});
```

### Queued Rendering

キューイング方式の 2 パスレンダリングにより、従来比で最大 **2 倍高速**なレンダリングを実現します。

## 破壊的変更と移行

### Node.js 22 以上必須

Node 18 / 20 のサポートが終了しました。`^22.12.0` 以上が必要です。

### レガシー Content Collections 形式の廃止

`src/content/config.ts`（フォルダ内）は使えなくなりました。`src/content.config.ts`（プロジェクトルート）に移行し、`glob()` ローダーを使用する必要があります。

### 削除された API

| 削除された API | 代替 |
|---------------|------|
| `Astro.glob()` | `import.meta.glob()` |
| `getEntryBySlug()` | Content Layer API |
| `Astro.canonicalURL` | `Astro.url` |
| Zod `.nonempty()` | `.min(1)` |

### アップグレードされたパッケージ

- **Vite 7** — Astro 全体で採用
- **Shiki 4** — コードハイライトエンジン
- **Zod 4** — スキーマバリデーション（`astro/zod` からインポート）

## まとめ

Astro 6.0 は、開発体験と本番環境の一致を重視したメジャーアップデートです。特に Cloudflare Workers との統合強化により、エッジコンピューティングを活用したコンテンツサイト構築の可能性が大きく広がりました。

v5 から移行する際は、Content Collections の設定ファイル移行と Node.js バージョンの確認が最低限の作業となります。詳細は [Astro v6 移行ガイド](https://docs.astro.build/en/guides/upgrade-to/v6/) を参照してください。
