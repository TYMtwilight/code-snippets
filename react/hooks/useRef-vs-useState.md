# useRef vs useState の使い分け

## 概要
useRefとuseStateの違いを理解し、適切に使い分ける方法

## 使用場面
- 再レンダリングを引き起こしたくない値を保持する時
- setIntervalのIDやDOM参照など、画面に表示しない値を管理する時
- useEffectのクリーンアップ関数で最新の値を参照したい時

## コード

### useRefの使用例

```typescript
// ❌ useStateで実装（悪い例）
const [intervalId, setIntervalId] = useState<NodeJS.Timeout | null>(null);

const startTimer = () => {
  const id = setInterval(() => { ... }, 1000);
  setIntervalId(id); // ← 再レンダリングが発生！
  // → 無限ループの危険性
};

// ✅ useRefで実装（良い例）
const intervalRef = useRef<NodeJS.Timeout | null>(null);

const startTimer = () => {
  intervalRef.current = setInterval(() => { ... }, 1000);
  // ← 再レンダリングなし！
};

const stopTimer = () => {
  if (intervalRef.current) {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  }
};
```

### アンマウント時に最新の値を参照する

```typescript
// 問題: useEffectのクリーンアップ関数は古い値をキャプチャする
const [timeLeft, setTimeLeft] = useState(1500);

useEffect(() => {
  return () => {
    console.log(timeLeft); // ← マウント時の値（1500）のまま
  };
}, []); // 依存配列が空

// 解決策: useRefで常に最新の値を保存
const stateRef = useRef({ timeLeft, isRunning, hasStarted });

// 最新の状態を常に保存
useEffect(() => {
  stateRef.current = { timeLeft, isRunning, hasStarted };
}, [timeLeft, isRunning, hasStarted]);

// アンマウント時に最新の値を参照
useEffect(() => {
  return () => {
    console.log(stateRef.current.timeLeft); // ← 最新の値！
    saveTimerState(stateRef.current); // 最新の状態を保存
  };
}, []);
```

## 説明

### useRefの特徴

1. **再レンダリングを引き起こさない**
   ```typescript
   const countRef = useRef(0);
   countRef.current = 1; // ← 再レンダリングなし
   ```

2. **コンポーネントのライフサイクル全体で値を保持**
   ```typescript
   // レンダリング1回目
   myRef.current = 100;

   // レンダリング2回目
   console.log(myRef.current); // 100 ← 保持されている
   ```

3. **`.current` プロパティで値にアクセス**
   ```typescript
   const myRef = useRef<number>(0);
   myRef.current // ← ここに実際の値
   ```

### 使い分けの基準

| 特徴 | useState | useRef |
|------|----------|--------|
| 値の更新 | `setState(新しい値)` | `ref.current = 新しい値` |
| 再レンダリング | 発生する | 発生しない |
| 画面表示 | 適している | 適していない |
| 用途 | UIの状態管理 | 裏で使う値の保存 |
| 例 | `timeLeft`, `isRunning` | `intervalRef`, `startTimeRef` |

### 覚え方

- **「見せる値」 → useState**
- **「隠す値」 → useRef**

## 実用例

```typescript
// タイマー実装での使用例
export const useTimer = () => {
  // ✅ 画面に表示する値 → useState
  const [timeLeft, setTimeLeft] = useState(1500);
  const [isRunning, setIsRunning] = useState(false);

  // ✅ 画面に表示しない値 → useRef
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  const startTimeRef = useRef<Date | null>(null);
  const endTimeRef = useRef<number | null>(null);
  const stateRef = useRef({ timeLeft, isRunning });

  // 最新の状態を保存
  useEffect(() => {
    stateRef.current = { timeLeft, isRunning };
  }, [timeLeft, isRunning]);

  // アンマウント時に最新の状態を参照
  useEffect(() => {
    return () => {
      if (stateRef.current.timeLeft > 0) {
        saveTimerState(stateRef.current);
      }
    };
  }, []);
};
```

## 参考
- [React公式: useRef](https://react.dev/reference/react/useRef)
- [A Complete Guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/)

## 関連スニペット
- [useEffect のクリーンアップ](./useEffect-cleanup.md)
- [useMemo で参照の安定性を保つ](./useMemo-reference-stability.md)

## 作成日
2026-01-31

## タグ
#react #hooks #useRef #useState #performance #lifecycle
