---
title: "Astro 7.2 の増分静的ビルドを解説 — Content Collections を使うブログを速く育てる"
description: "Astro 7.0〜7.2の主要アップデートを整理し、Astro 7.2で登場した実験的な増分静的ビルドをContent CollectionsとGitHub Actionsで活用する方法を解説します。"
pubDate: 2026-08-22
---

## はじめに

Astro 7 系では、静的サイトのビルド速度、Content Collections の運用性、そして開発ワークフローに関わる機能が連続して強化されました。特に **Astro 7.2** で導入された実験的な増分静的ビルドは、更新されていないページの再レンダリングを避けられるため、記事数が増え続けるブログやドキュメントサイトで注目したい機能です。[1] [2]

本記事では、Astro 7.0 から 7.2 までの流れを整理したうえで、Content Collections を使う静的サイトへ増分静的ビルドを導入するための実装と CI 設定を解説します。なお、増分静的ビルドは **experimental（実験的機能）** です。導入前には、対象プロジェクトのビルド時間、キャッシュの復元可否、互換性要件を確認してください。[2]

## Astro 7 系の重要なアップデート

Astro 7.0 では、`.astro` コンパイラと既定の Markdown/MDX 処理が Rust ベースへ移行し、Vite 8 も採用されました。Astro公式のベンチマークでは、対象サイトによりビルド時間が **15〜61%** 短縮されています。[3] また、Astro 7.1 では、大規模な Content Collections におけるメモリ使用量を調整する `deferRender` と、コレクションストレージを分割する実験的な `collectionStorage` が追加されました。[4]

| バージョン | 主なテーマ | Content Collections を使う静的サイトへの意味 |
| --- | --- | --- |
| Astro 7.0 | Rust コンパイラ、Sätteri、Vite 8 | Markdown中心のサイトで、ビルド全体の高速化が期待できる。 |
| Astro 7.1 | `deferRender`、分割可能なコレクションストレージ | 記事数やMarkdownの出力量が大きい場合に、同期時のメモリ消費を調整できる。 |
| Astro 7.2 | 実験的な増分静的ビルド | 変更のない詳細ページを再利用し、継続的なビルドのレンダリング負荷を抑えられる。 |

Astro 7.2 の増分静的ビルドは、前回のビルド結果から **コードとデータの双方が同じ** ページを再利用します。Astro は、レイアウト、コンポーネント、読み込むアセット、依存パッケージを含むモジュール依存グラフをハッシュし、ページ側では `cacheKey` によってデータの変化を通知します。[1] [2]

> **ポイント:** `cacheKey` を返さないパスは従来どおり毎回レンダリングされます。つまり、既存プロジェクトの挙動を変えず、適用対象の動的ルートを明示的に選べます。[2]

## まずは Astro 7 への移行を確認する

Astro 7.2 を使うには、まず Astro 7 系へ更新します。公式には `@astrojs/upgrade` が推奨されています。[1] ただしメジャーアップデートのため、更新前にブランチを切り、ローカルとCIでビルドを実行してください。

```bash
npx @astrojs/upgrade
npm run build
```

Astro 7 では Vite 8 が使われるため、Vite 固有のプラグインや設定を利用しているプロジェクトは Vite 8 の移行情報も確認します。また、Rust コンパイラでは閉じ忘れたタグがエラーになり、従来のコンパイラが補正していた不正なHTMLのネストは補正されません。[5] Content Collections を中心とするシンプルな静的ブログでも、アップグレード後には実際の画面を確認することが重要です。

## Content Collections の詳細ページを増分ビルド対応にする

`getStaticPaths()` で記事詳細ページを生成している場合、各エントリーの `digest` を `cacheKey` として返せます。`digest` はローダーがデータ変更に応じて更新する値であり、Content Collections での利用が公式に案内されています。[1] [2]

以下は、`src/pages/posts/[...slug].astro` の最小例です。

```astro
---
import { getCollection, render } from 'astro:content';
import BlogLayout from '../../layouts/BlogLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('posts');

  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
    cacheKey: String(post.digest),
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<BlogLayout title={post.data.title} description={post.data.description}>
  <article class="markdown-body">
    <Content />
  </article>
</BlogLayout>
```

続いて、`astro.config.mjs` で実験的フラグを有効にします。

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  experimental: {
    incrementalBuild: true,
  },
});
```

この構成では、ある記事のMarkdownまたはフロントマターを変更すると、その記事の `digest` が変わり、当該ページは再生成されます。一方で変更のない記事は、前回の出力を再利用できます。レイアウト、コンポーネント、Astro設定、依存関係に変更があった場合は、Astro が依存グラフや設定の変化を検知し、関連ページまたは全キャッシュを無効化します。[2]

## GitHub Actions で `.astro` キャッシュを復元する

ローカルでは、既定の `cacheDir` である `node_modules/.astro/` に増分ビルドのキャッシュが保存されます。CIでは実行環境が毎回新しくなるため、このディレクトリをビルド前に復元しなければ、再利用対象のページも通常どおりレンダリングされます。[2]

GitHub Actionsでは、次のように `actions/cache` を追加できます。キーにはOS、Node.jsバージョン、依存関係のハッシュを含め、依存関係の変更時に古いキャッシュを安全に使い回さないようにします。

```yaml
- name: Restore Astro incremental build cache
  uses: actions/cache@v4
  with:
    path: node_modules/.astro
    key: ${{ runner.os }}-astro-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-astro-

- name: Build
  run: npm run build
```

`npm ci` の後、`npm run build` の前に置くことが重要です。なお、既存の `actions/setup-node` による npm キャッシュはパッケージ取得を高速化するものであり、増分静的ビルドが再利用するページ出力キャッシュとは別物です。両方を併用することで、依存関係の導入と静的ページ生成の両面を効率化できます。

## 導入時に確認すべき制約

増分静的ビルドは、すべての構成で無条件に効果を発揮するわけではありません。特に `build.concurrency` が 1 より大きいときは増分キャッシュが無効になり、通常のレンダリングに戻ります。また、サーバーアイランドを使うページでは、安定した `ASTRO_KEY` を設定しない限りページが毎回再レンダリングされます。プリレンダリング結果を変えるミドルウェアを変更したときは、`astro build --force` でキャッシュを無視して再ビルドする必要があります。[2]

| 確認事項 | 推奨する対応 |
| --- | --- |
| Astroのバージョン | 増分ビルドと最適化画像に関する修正を含む Astro 7.2.2 以降を使う。[6] |
| キャッシュの保存 | CIで `node_modules/.astro/` をビルド前に復元する。[2] |
| 動的詳細ページ | `getStaticPaths()` の各パスに、コンテンツ更新とともに変わる `cacheKey` を付ける。[2] |
| 並列ビルド | `build.concurrency` を 1 より大きくしない。[2] |
| 検証方法 | 初回は通常ビルド、以降は記事を1件だけ変更したビルドを実行し、生成結果とログを確認する。 |

## まとめ

Astro 7 系は、Rustベースのコンパイラ・Markdown処理・Vite 8による基礎的な高速化に加え、大規模なコンテンツサイトの実運用を意識した機能を段階的に追加しています。特に Astro 7.2 の増分静的ビルドは、記事の大半が変わらないブログやドキュメントサイトで、継続的なデプロイの待ち時間を減らせる可能性があります。[1] [2] [3]

まずは Astro 7 への移行をテストし、Content Collections の詳細ページに `cacheKey: String(entry.digest)` を追加してください。そのうえでCIに `node_modules/.astro/` のキャッシュ復元を組み込み、実測値と生成結果を確認しながら段階的に有効化するのが安全です。実験的機能である点を踏まえ、更新時にはリリースノートも継続して確認しましょう。

## 参考資料

[1]: https://astro.build/blog/astro-720/ "Astro 7.2"
[2]: https://docs.astro.build/en/reference/experimental-flags/incremental-build/ "Experimental incremental static builds"
[3]: https://astro.build/blog/astro-7/ "Astro 7"
[4]: https://astro.build/blog/astro-710/ "Astro 7.1"
[5]: https://docs.astro.build/en/guides/upgrade-to/v7/ "Upgrade to Astro v7"
[6]: https://github.com/withastro/astro/releases/tag/astro%407.2.2 "astro@7.2.2"
