---
title: "Zed Editor 1.0 正式リリース — Rust 製 GPU アクセラレーションエディタの現在地"
description: "元 Atom 開発チームが手がける Rust 製エディタ Zed が 1.0 に到達。GPU 駆動の圧倒的な速度と AI エージェント機能の全容を解説します。"
pubDate: 2026-05-12
---

## はじめに

2026年4月29日、Rust 製コードエディタ **Zed** が **バージョン 1.0** を迎えました。Atom エディタの創設者 Nathan Sobo を中心とする Zed Industries が開発する本エディタは、Electron に依存せず GPU を直接活用する独自アーキテクチャにより、従来のエディタを凌駕するパフォーマンスを実現しています。

本記事では Zed 1.0 のコア技術と主要機能を詳しく紹介します。

## GPUI アーキテクチャ — Electron からの決別

Zed の最大の特徴は、**GPUI** と呼ばれる独自の GPU アクセラレーション UI フレームワークです。

```rust
// GPUI のコンポーネント例（概念）
struct Editor {
    buffer: Buffer,
    selections: Vec<Selection>,
    scroll_position: Point,
}

impl Render for Editor {
    fn render(&mut self, _: &mut WindowContext) -> impl IntoElement {
        div()
            .size_full()
            .child(self.render_gutter())
            .child(self.render_lines())
    }
}
```

VS Code（Electron ベース）との比較:

| 項目 | VS Code | Zed |
|------|---------|-----|
| ベース技術 | Electron (Chromium) | GPUI (Rust + GPU シェーダー) |
| 起動時間 | 数秒 | 瞬時 |
| メモリ使用量 | 300MB〜1GB+ | 〜100MB |
| ファイル切り替え | ミリ秒単位 | マイクロ秒単位 |
| 100万行ファイル | 動作が重い | 問題なく動作 |

## AI 機能の全体像

Zed は「AI ネイティブエディタ」として設計されており、AI 機能が後付けではなく基盤レベルで統合されています。

### エージェント編集

スレッドサイドバーから AI エージェントに指示を送り、コードの読み取り・編集・実行を任せられます。

```
スレッド例:
「このプロジェクトに Zod を使ったバリデーションスキーマを追加して」

→ エージェントがコードベースを検索 → 関連ファイルを特定
  → スキーマファイルを作成 → 既存コードに統合 → 結果を diff 表示
```

### 並列エージェント

複数のエージェントを同時に実行できます。それぞれ異なるモデルを使い、異なるタスクを処理させることが可能です。

### Edit Prediction（編集予測）

キー入力ごとに次の編集を予測します。デフォルトのプロバイダは Zed 独自のオープンソースモデル **Zeta** ですが、GitHub Copilot や Codestral に切り替えることもできます。

### マルチモデル対応

```json
{
  "features": {
    "edit_prediction_provider": "zeta",
    "agent": {
      "default_model": {
        "provider": "anthropic",
        "model": "claude-sonnet-4-20260506"
      }
    },
    "inline_assistant": {
      "provider": "google",
      "model": "gemini-2.5-pro"
    }
  }
}
```

以下のプロバイダに対応しています:

- Anthropic (Claude Opus / Sonnet)
- OpenAI (GPT-4.1, o3)
- Google (Gemini 2.5 Pro)
- Ollama (ローカルモデル)
- GitHub Copilot
- その他 8 以上のプロバイダ

### Agent Client Protocol (ACP)

外部の CLI エージェント（Claude Agent, Codex, Cursor, OpenCode など）を Zed 内で直接実行できるプロトコルを備えています。

## リアルタイムコラボレーション

Zed のコラボレーション機能は、エディタの中核プロトコルに組み込まれています。

- **チャンネル機能** — チームメイトをプロジェクトに招待
- **マルチカーソル編集** — それぞれ独立したカーソルと Undo 履歴
- **共有ターミナル** — 同じシェルセッションを共有
- **画面共有** — VS Code の拡張機能に依存しないネイティブ実装

## Git 統合

Zed 1.0 では Git 機能が大幅に強化されました。

- ステージング/コミット/プッシュ/プル
- diff ビューの自動スプリット→統一表示切り替え
- 未コミット変更数のバッジ表示
- ファイルタイプアイコン対応
- 「コミットを表示」コマンドパレットアクション

## 拡張機能 — WebAssembly サンドボックス

Zed の拡張機能は WebAssembly 上で動作するため、エディタ本体をクラッシュさせることなく安全に実行できます。現時点で約 1,000 の拡張機能が利用可能です（VS Code の 100,000+ には及びませんが、主要言語の LSP サポートは標準搭載されています）。

## プライバシーとセキュリティ

Zed は**プライバシーバイデフォルト**を掲げています。

- AI 機能のデータ収集は **オプトイン**
- 自分の API キーを使う場合、Zed とプロバイダ間でゼロデータ保持契約
- プライバシーモードがデフォルト設定
- エディタ全体が GPL、サーバーコンポーネントが AGPL、GPUI フレームワークが Apache 2.0 でオープンソース
- **AI 機能を完全に無効化する設定**も UI からワンクリック

## 2026年現在の課題

- **拡張機能エコシステム** — VS Code の 100,000+ に対して約 1,000 と差が大きい
- **日本語入力** — macOS 以外での IME 対応は改善中
- **学習コスト** — VS Code からの移行にはキーバインドの違いに慣れる必要がある

## まとめ

Zed 1.0 は、Electron 依存からの脱却と GPU 駆動によるパフォーマンス重視の設計が際立つエディタです。特に AI エージェント機能は「機能の後付け」ではなく、エディタの基盤レベルで統合されている点が他エディタとの差別化ポイントです。

VS Code の重さに不満を感じている開発者や、AI アシスタントを日常的に活用する開発者にとって、一度試す価値のあるエディタです。

公式サイト: [https://zed.dev](https://zed.dev)
