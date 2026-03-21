# カスタム Hook - API 通信パターン（useXxx）

## 概要
API 通信ロジックを `useState` / `useEffect` / `useCallback` を使ってカスタム Hook に切り出すパターン。コンポーネントをシンプルに保ちながら、loading / error 状態・CRUD 操作を一元管理する。

## 使用場面
- 複数のコンポーネントから同じ API を呼び出す時
- loading / error 状態をコンポーネントから分離したい時
- CRUD 操作（取得・作成・更新・削除）をまとめて管理したい時

## コード

```typescript
import { useState, useEffect, useCallback } from "react";
import type { Todo, FilterStatus } from "../types/todo";
import * as api from "../lib/api";

export function useTodos() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<FilterStatus>("ALL");
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  // 一覧取得（filter が変わるたびに再取得）
  const loadTodos = useCallback(async () => {
    setLoading(true);
    setError("");
    try {
      const data = await api.fetchTodos(filter);
      setTodos(data);
    } catch {
      setError("TODO の読み込みに失敗しました");
    } finally {
      setLoading(false);
    }
  }, [filter]); // filter が依存配列に入る

  useEffect(() => {
    loadTodos();
  }, [loadTodos]);

  // 作成：楽観的更新（先頭に追加）
  const addTodo = async (title: string) => {
    const created = await api.createTodo(title);
    setTodos((prev) => [created, ...prev]);
  };

  // 更新：ID で対象を置換
  const editTodo = async (id: number, title: string) => {
    const updated = await api.updateTodo(id, { title });
    setTodos((prev) =>
      prev.map((todo) => (todo.id === id ? updated : todo))
    );
  };

  // 完了トグル
  const toggleTodo = async (id: number) => {
    const updated = await api.toggleTodo(id);
    setTodos((prev) =>
      prev.map((todo) => (todo.id === id ? updated : todo))
    );
  };

  // 削除：ID でフィルタリング
  const removeTodo = async (id: number) => {
    await api.deleteTodo(id);
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  };

  return {
    todos,
    filter,
    loading,
    error,
    setFilter,
    addTodo,
    editTodo,
    toggleTodo,
    removeTodo,
    reload: loadTodos,
  };
}
```

## 説明

### 責務の分離

```
App コンポーネント
  └─ useTodos()          ← API 通信・状態管理（このファイル）
       └─ api.fetchTodos() ← HTTP リクエスト（lib/api.ts）

App コンポーネントは useTodos() が返す値・関数を使うだけ
```

### useCallback で関数を安定化

```typescript
const loadTodos = useCallback(async () => {
  // ...
}, [filter]); // filter が変わったら関数を再作成
```

- `loadTodos` を `useEffect` の依存配列に入れるため、`useCallback` でメモ化する
- `filter` が依存配列に含まれているため、filter 変更 → `loadTodos` 再作成 → `useEffect` 発火 → API 再取得

### 楽観的更新 vs 再取得

| 方法 | メリット | デメリット |
|---|---|---|
| 楽観的更新（`setTodos(prev => ...)`) | 即座に UI に反映 | API エラー時に状態がずれる |
| 再取得（`loadTodos()`） | DB の最新状態に同期 | API 呼び出しが増える |

このパターンでは作成・更新・削除後に**楽観的更新**を使用。エラー処理が必要な場合は `try/catch` で `reload()` を呼ぶ。

### 戻り値の設計

```typescript
return {
  // 状態（読み取り専用として使う）
  todos,
  filter,
  loading,
  error,

  // アクション（コンポーネントから呼び出す）
  setFilter,
  addTodo,
  editTodo,
  toggleTodo,
  removeTodo,
  reload: loadTodos, // エイリアスで意図を明確に
};
```

### コンポーネント側での使い方

```tsx
function App() {
  const { todos, filter, loading, error, setFilter, addTodo, removeTodo } = useTodos();

  if (loading) return <p>読み込み中...</p>;
  if (error) return <p>{error}</p>;

  return (
    <>
      <TodoForm onSaveTodo={addTodo} />
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} onDelete={removeTodo} />
      ))}
    </>
  );
}
```

## 参考
- [React - カスタム Hook を使ったロジックの再利用](https://ja.react.dev/learn/reusing-logic-with-custom-hooks)
- [React - useCallback](https://ja.react.dev/reference/react/useCallback)

## 関連スニペット
- [MSW を使った API テスト](../test/msw-api-test.md)
- [useCallback の依存配列ルール](./useCallback-dependencies.md)

## 作成日
2026-03-21

## タグ
#react #hooks #custom-hook #api #usestate #useeffect #usecallback #typescript
