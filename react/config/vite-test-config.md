# Vitest 設定テンプレート（React + TypeScript）

## 概要
Vite プロジェクトで Vitest を使ったテスト環境を構築する設定。`jsdom` 環境で React コンポーネントをテストするための最小設定。

## 使用場面
- Vite + React + TypeScript プロジェクトにテストを追加する時
- Jest 互換の API（`describe`, `it`, `expect`, `vi`）を使いたい時
- `@testing-library/jest-dom` のカスタムマッチャー（`toBeInTheDocument` など）を使いたい時

## コード

### vite.config.ts

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,           // describe / it / expect / vi をグローバルに使用可能
    environment: "jsdom",    // ブラウザ互換の DOM 環境（React コンポーネントのテストに必要）
    setupFiles: "./src/test-setup.ts", // テスト前に実行するセットアップファイル
  },
});
```

### src/test-setup.ts

```typescript
import "@testing-library/jest-dom/vitest";
// toBeInTheDocument, toHaveValue, toHaveTextContent などのカスタムマッチャーを追加
```

### package.json（テスト関連の依存関係）

```json
{
  "devDependencies": {
    "vitest": "^3.x.x",
    "@vitejs/plugin-react": "^4.x.x",
    "jsdom": "^26.x.x",
    "@testing-library/react": "^16.x.x",
    "@testing-library/user-event": "^14.x.x",
    "@testing-library/jest-dom": "^6.x.x",
    "msw": "^2.x.x"
  }
}
```

### テスト実行コマンド

```bash
# 一度だけ実行（CI 向け）
npx vitest run

# ウォッチモード（開発中）
npx vitest

# UI モード（ブラウザで結果確認）
npx vitest --ui
```

## 説明

### globals: true の効果

```typescript
// globals: true の場合（import 不要）
describe("...", () => {
  it("...", () => {
    expect(value).toBe(expected);
    vi.fn(); // Vitest のモック関数
  });
});

// globals: false の場合（import が必要）
import { describe, it, expect, vi } from "vitest";
```

### environment の選択

| 値 | 説明 | 用途 |
|---|---|---|
| `"jsdom"` | ブラウザ互換の DOM 環境 | React コンポーネント・DOM 操作のテスト |
| `"node"` | Node.js 環境（デフォルト）| API クライアント・ユーティリティ関数のテスト |
| `"happy-dom"` | jsdom より高速な軽量実装 | jsdom の代替 |

React コンポーネントのテストには `"jsdom"` が必要（`document`、`window` などが使える）。

### setupFiles の役割

テスト実行前に一度だけ実行されるファイル。以下のような共通設定に使う。

```typescript
// src/test-setup.ts

// @testing-library/jest-dom のカスタムマッチャーを登録
import "@testing-library/jest-dom/vitest";

// 全テストで使うグローバルモックを設定
// global.fetch = vi.fn();
```

### @testing-library/jest-dom が提供するマッチャー

```typescript
expect(element).toBeInTheDocument()  // DOM に存在する
expect(element).toHaveValue("text")  // input の値
expect(element).toHaveTextContent("text") // テキスト内容
expect(element).toBeDisabled()       // disabled 属性
expect(element).toBeVisible()        // 表示されている
expect(element).toHaveClass("cls")   // CSS クラスを持つ
```

## 参考
- [Vitest - 公式ドキュメント](https://vitest.dev/)
- [Vitest - React Testing Guide](https://vitest.dev/guide/#trying-vitest-online)
- [@testing-library/jest-dom](https://github.com/testing-library/jest-dom)

## 関連スニペット
- [React Testing Library - コンポーネントテスト](../test/component-test.md)
- [MSW を使った API テスト](../test/msw-api-test.md)

## 作成日
2026-03-21

## タグ
#react #vitest #vite #testing #jsdom #configuration #typescript
