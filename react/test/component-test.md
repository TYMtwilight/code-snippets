# React Testing Library - コンポーネントテストテンプレート

## 概要
`@testing-library/react` と `@testing-library/user-event` を使ったコンポーネントテストのパターン。ユーザー視点（表示・操作・結果）でテストを書く。

## 使用場面
- フォームの入力・送信を検証する時
- ボタンクリック後の状態変化を確認する時
- バリデーションエラーメッセージの表示を検証する時

## コード

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import TodoForm from "../../components/TodoForm";

describe("TodoForm", () => {
  const mockOnSaveTodo = vi.fn();

  beforeEach(() => {
    mockOnSaveTodo.mockReset();
    mockOnSaveTodo.mockResolvedValue(undefined); // async 関数として扱う
  });

  // --------------------------------------------------
  // 表示確認
  // --------------------------------------------------

  it("入力欄と追加ボタンが表示される", () => {
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    expect(screen.getByPlaceholderText("新しい TODO を入力...")).toBeInTheDocument();
    expect(screen.getByRole("button", { name: "追加" })).toBeInTheDocument();
  });

  // --------------------------------------------------
  // 正常系：フォーム送信
  // --------------------------------------------------

  it("入力して submit すると onSaveTodo が呼ばれる", async () => {
    const user = userEvent.setup();
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    await user.type(screen.getByPlaceholderText("新しい TODO を入力..."), "テスト TODO");
    await user.click(screen.getByRole("button", { name: "追加" }));

    expect(mockOnSaveTodo).toHaveBeenCalledWith("テスト TODO");
  });

  it("前後のスペースはトリムされる", async () => {
    const user = userEvent.setup();
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    await user.type(screen.getByPlaceholderText("新しい TODO を入力..."), "  テスト TODO  ");
    await user.click(screen.getByRole("button", { name: "追加" }));

    expect(mockOnSaveTodo).toHaveBeenCalledWith("テスト TODO");
  });

  it("送信成功後に入力欄がクリアされる", async () => {
    const user = userEvent.setup();
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    const input = screen.getByPlaceholderText("新しい TODO を入力...");
    await user.type(input, "テスト TODO");
    await user.click(screen.getByRole("button", { name: "追加" }));

    expect(input).toHaveValue("");
  });

  // --------------------------------------------------
  // 異常系：バリデーション
  // --------------------------------------------------

  it("空のまま submit するとエラーが表示される", async () => {
    const user = userEvent.setup();
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    await user.click(screen.getByRole("button", { name: "追加" }));

    expect(screen.getByText("タイトルを入力してください")).toBeInTheDocument();
    expect(mockOnSaveTodo).not.toHaveBeenCalled();
  });

  it("スペースのみで submit するとエラーが表示される", async () => {
    const user = userEvent.setup();
    render(<TodoForm onSaveTodo={mockOnSaveTodo} />);

    await user.type(screen.getByPlaceholderText("新しい TODO を入力..."), "    ");
    await user.click(screen.getByRole("button", { name: "追加" }));

    expect(screen.getByText("タイトルを入力してください")).toBeInTheDocument();
    expect(mockOnSaveTodo).not.toHaveBeenCalled();
  });
});
```

## 説明

### ユーザー視点のテスト哲学

> The more your tests resemble the way your software is used, the more confidence they can give you.
>
> — Testing Library 公式

実装の詳細（state の中身・関数の呼び出し順）ではなく、**ユーザーが見るもの・操作するもの**を検証する。

### screen クエリ早見表

| クエリ | 取得対象 | 例 |
|---|---|---|
| `getByRole` | ARIA ロール | `getByRole("button", { name: "追加" })` |
| `getByPlaceholderText` | placeholder 属性 | `getByPlaceholderText("入力...")` |
| `getByText` | テキストコンテンツ | `getByText("エラーメッセージ")` |
| `getByLabelText` | label と紐付いた input | `getByLabelText("タイトル")` |
| `queryByText` | 存在しない要素の確認（null 許容）| `queryByText("...")` |
| `findByText` | 非同期で要素を待つ | `await findByText("読み込み完了")` |

`getBy*` は要素がなければ例外、`queryBy*` は null を返す。

### userEvent vs fireEvent

```typescript
// userEvent（推奨）: 実際のユーザー操作に近い
const user = userEvent.setup(); // テストごとに setup() を呼ぶ
await user.type(input, "テキスト");
await user.click(button);

// fireEvent（低レベル）: 単純なイベント発火
fireEvent.change(input, { target: { value: "テキスト" } });
fireEvent.click(button);
```

### モック関数のリセット

```typescript
beforeEach(() => {
  mockOnSaveTodo.mockReset(); // 呼び出し履歴と実装をリセット
  mockOnSaveTodo.mockResolvedValue(undefined); // async 関数として再設定
});
```

- `mockReset()` で前のテストの呼び出し回数が残らないようにする
- Props として渡す async 関数は `mockResolvedValue` でモック化する

### アクセシビリティ優先のクエリ選択

```
getByRole           ← 最優先（button, heading, textbox など）
getByLabelText      ← label と input が紐付いている場合
getByPlaceholderText ← placeholder がある場合
getByText           ← 表示テキストで探す
getByTestId         ← 最終手段（data-testid 属性を追加する）
```

## 参考
- [Testing Library - クエリの優先順位](https://testing-library.com/docs/queries/about#priority)
- [user-event - 公式ドキュメント](https://testing-library.com/docs/user-event/intro)

## 関連スニペット
- [MSW を使った API テスト](./msw-api-test.md)
- [Vitest 設定](../config/vite-test-config.md)

## 作成日
2026-03-21

## タグ
#react #test #testing-library #vitest #user-event #component-test #typescript
