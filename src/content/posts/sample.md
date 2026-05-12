---
title: "TypeScript 5.0 の新機能を徹底解説"
description: "TypeScript 5.0 で導入された主要な機能について、実践的なコード例を交えて解説します。"
pubDate: 2026-03-15
---

## はじめに

2026年3月、Microsoft は TypeScript 5.0 の正式リリースを発表しました。本記事では、このバージョンで追加された主要な機能を詳しく見ていきます。

## Decorators（デコレータ）

TypeScript 5.0 では、ECMAScript の Stage 3 に準拠した **Decorators** が正式にサポートされました。

```typescript
function log(target: any, context: ClassMethodDecoratorContext) {
  const methodName = String(context.name);

  function replacementMethod(this: any, ...args: any[]) {
    console.log(`Calling ${methodName} with`, args);
    const result = target.call(this, ...args);
    console.log(`Result:`, result);
    return result;
  }

  return replacementMethod;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(1, 2);
// Calling add with [1, 2]
// Result: 3
```

### 従来のデコレータとの違い

TypeScript 5.0 のデコレータは、従来の実験的デコレータと以下の点で異なります。

| 項目 | 従来（experimental） | 標準デコレータ |
|------|---------------------|---------------|
| コンパイラオプション | `experimentalDecorators: true` | 不要 |
| コンテキストオブジェクト | なし | `ClassMethodDecoratorContext` |
| 戻り値の型 | `any` | 型安全 |

## const 型パラメータ

ジェネリクスの型推論がより正確になりました。

```typescript
function getNames<const T extends readonly string[]>(names: T): T {
  return names;
}

// 推論結果: readonly ["foo", "bar"]
const names = getNames(["foo", "bar"]);
```

## 複数の設定ファイルの継承

`extends` 配列で複数の設定ファイルを継承できるようになりました。

```json
{
  "extends": ["./tsconfig.base.json", "./tsconfig.paths.json"],
  "compilerOptions": {
    "strict": true
  }
}
```

## パフォーマンス改善

TypeScript 5.0 では、ビルドパフォーマンスが大幅に改善されました。

- パッケージサイズが従来比で **約40%削減**
- ビルド時間が **平均で10〜25%高速化**
- Node.js 14 / 16 のサポート終了

## まとめ

TypeScript 5.0 は、言語機能とパフォーマンスの両面で大きな進化を遂げたバージョンです。特に Decorators の標準化は、フレームワーク開発者にとって重要なマイルストーンとなります。

詳しくは [TypeScript 5.0 リリースノート](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/) をご覧ください。
