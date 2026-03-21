# MSW (Mock Service Worker) - API テストテンプレート

## 概要
`msw` の `setupServer` を使い、ネットワークレベルで API をモック化して API クライアント関数をテストするパターン。fetch / axios を差し替えずに実際の HTTP リクエストをインターセプトできる。

## 使用場面
- API クライアント関数（`fetchTodos`, `createTodo` など）を単体テストする時
- エラーレスポンス（400 / 404 / 500）時の例外処理を検証する時
- テストごとにモックの振る舞いを上書きしたい時（`server.use()`）

## コード

```typescript
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";
import { fetchTodos, createTodo, updateTodo, toggleTodo, deleteTodo } from "../../lib/api";

// --- モックサーバーの定義 ---

const mockTodo = {
  id: 1,
  title: "テスト TODO",
  completed: false,
  createdAt: "2026-03-01T10:00:00",
  updatedAt: "2026-03-01T10:00:00",
};

const server = setupServer(
  // GET /api/todos
  http.get("http://localhost:8080/api/todos", () => {
    return HttpResponse.json([mockTodo]);
  }),

  // POST /api/todos
  http.post("http://localhost:8080/api/todos", async ({ request }) => {
    const body = (await request.json()) as { title: string };
    return HttpResponse.json(
      { ...mockTodo, id: 2, title: body.title },
      { status: 201 }
    );
  }),

  // PUT /api/todos/:id
  http.put("http://localhost:8080/api/todos/:id", async ({ request }) => {
    const body = (await request.json()) as { title: string };
    return HttpResponse.json({ ...mockTodo, ...body });
  }),

  // PATCH /api/todos/:id/toggle
  http.patch("http://localhost:8080/api/todos/:id/toggle", () => {
    return HttpResponse.json({ ...mockTodo, completed: true });
  }),

  // DELETE /api/todos/:id
  http.delete("http://localhost:8080/api/todos/:id", () => {
    return new HttpResponse(null, { status: 204 });
  })
);

// --- テストのライフサイクル ---
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers()); // テストごとに上書きをリセット
afterAll(() => server.close());

// --- テスト ---

describe("API クライアント", () => {

  // --------------------------------------------------
  // 正常系
  // --------------------------------------------------

  it("fetchTodos で TODO 一覧を取得できる", async () => {
    const todos = await fetchTodos();

    expect(todos).toHaveLength(1);
    expect(todos[0].title).toBe("テスト TODO");
    expect(todos[0].completed).toBe(false);
  });

  it("createTodo で新しい TODO を作成できる", async () => {
    const todo = await createTodo("新規 TODO");

    expect(todo.title).toBe("新規 TODO");
    expect(todo.id).toBe(2);
  });

  it("updateTodo で TODO を更新できる", async () => {
    const todo = await updateTodo(1, { title: "更新された TODO" });

    expect(todo.title).toBe("更新された TODO");
  });

  it("toggleTodo で完了状態を切り替えられる", async () => {
    const todo = await toggleTodo(1);

    expect(todo.completed).toBe(true);
  });

  it("deleteTodo で TODO を削除できる", async () => {
    await expect(deleteTodo(1)).resolves.toBeUndefined();
  });

  // --------------------------------------------------
  // 異常系：server.use() でテストごとにモックを上書き
  // --------------------------------------------------

  it("fetchTodos でサーバーエラー時に例外を投げる", async () => {
    server.use(
      http.get("http://localhost:8080/api/todos", () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    await expect(fetchTodos()).rejects.toThrow("TODO の取得に失敗しました");
  });

  it("createTodo でバリデーションエラー時に例外を投げる", async () => {
    server.use(
      http.post("http://localhost:8080/api/todos", () => {
        return HttpResponse.json(
          { status: 400, error: "Bad Request", message: "タイトルは必須です" },
          { status: 400 }
        );
      })
    );

    await expect(createTodo("")).rejects.toThrow("TODO の作成に失敗しました");
  });

  it("updateTodo で存在しない ID はエラーになる", async () => {
    server.use(
      http.put("http://localhost:8080/api/todos/:id", () => {
        return HttpResponse.json(
          { status: 404, message: "TODO が見つかりません" },
          { status: 404 }
        );
      })
    );

    await expect(updateTodo(9999, { title: "更新" })).rejects.toThrow("TODO の更新に失敗しました");
  });
});
```

## 説明

### MSW の仕組み

```
テストコード
  └─ api.fetchTodos() を呼ぶ
       └─ fetch("http://localhost:8080/api/todos")
            ↓ MSW がインターセプト
       ← HttpResponse.json([mockTodo]) を返す
  └─ 実際のサーバーには届かない
```

### setupServer のライフサイクル

```typescript
beforeAll(() => server.listen());       // テストスイート開始前にサーバー起動
afterEach(() => server.resetHandlers()); // テストごとに server.use() の上書きをリセット
afterAll(() => server.close());          // テストスイート終了後にサーバー停止
```

`afterEach` で `resetHandlers()` を呼ばないと、あるテストで上書きしたハンドラが次のテストでも残ってしまう。

### テストごとにレスポンスを上書き

```typescript
// デフォルト: 200 OK を返す
const server = setupServer(
  http.get(".../api/todos", () => HttpResponse.json([mockTodo]))
);

// 特定のテストだけ 500 を返す
it("エラー時のテスト", async () => {
  server.use(
    http.get(".../api/todos", () => new HttpResponse(null, { status: 500 }))
  );
  // このテストだけ 500 が返る
});
// afterEach の resetHandlers() でデフォルトに戻る
```

### HttpResponse の主要パターン

```typescript
// JSON レスポンス
HttpResponse.json(data)
HttpResponse.json(data, { status: 201 })
HttpResponse.json(errorData, { status: 400 })

// ボディなしレスポンス
new HttpResponse(null, { status: 204 })
new HttpResponse(null, { status: 500 })
```

### リクエストボディの読み取り

```typescript
http.post("/api/todos", async ({ request }) => {
  const body = (await request.json()) as { title: string };
  // body.title でリクエストデータを使えるため、動的なレスポンスが作れる
  return HttpResponse.json({ id: 1, title: body.title, completed: false });
})
```

## 参考
- [MSW - 公式ドキュメント](https://mswjs.io/docs/)
- [MSW - Node.js 統合（setupServer）](https://mswjs.io/docs/integrations/node)

## 関連スニペット
- [React Testing Library - コンポーネントテスト](./component-test.md)
- [カスタム Hook - API 通信パターン](../hooks/use-api-hook.md)
- [Vitest 設定](../config/vite-test-config.md)

## 作成日
2026-03-21

## タグ
#react #test #msw #mock-service-worker #api-test #vitest #typescript
