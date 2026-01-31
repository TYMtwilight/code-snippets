# useCallback の依存配列ルール

## 概要
useCallbackで関数をメモ化する際の依存配列の正しい設定方法と、ESLintの警告を適切に対処する方法

## 使用場面
- 子コンポーネントに渡す関数をメモ化する時
- useEffectの依存配列に関数を含める時
- 不要な再レンダリングを防ぎたい時

## コード

### 基本的な依存配列のルール

```typescript
const resetTimer = useCallback(() => {
  setIsRunning(false);
  setTimeLeft(initialTime);      // ← useMemoの値を使用
  setHasStarted(false);
  clearTimerState();             // ← Contextから来た関数を使用

  startTimeRef.current = null;
  endTimeRef.current = null;
}, [initialTime, clearTimerState]); // ← 使用している外部の値・関数を含める
```

### ❌ 悪い例：依存配列が不足

```typescript
const resetTimer = useCallback(() => {
  setTimeLeft(initialTime);      // ← initialTimeを使っている
  clearTimerState();             // ← clearTimerStateを使っている
}, []); // ← 依存配列が空！

// 問題点：
// 1. ESLintが警告を出す
// 2. 古いinitialTimeの値を使い続ける可能性
// 3. 古いclearTimerState関数を使い続ける可能性
```

### ✅ 良い例：適切な依存配列

```typescript
const resetTimer = useCallback(() => {
  setTimeLeft(initialTime);
  clearTimerState();
}, [initialTime, clearTimerState]); // ✅ 使用している値を全て含める
```

## 説明

### 依存配列に含めるべきもの

```
useCallback内で使っている外部の値や関数
    ↓
   YES
    ↓
それは関数内で定義した変数？（例：const x = 10）
    ↓
   NO（外部から来ている）
    ↓
それは以下のどれか？
- Props から来た値や関数
- Context から来た値や関数
- useMemo や useState の値
- useCallback で作った関数
    ↓
   YES
    ↓
依存配列に含める必要がある ✅
```

### 含めるべきもの vs 含めなくて良いもの

**✅ 依存配列に含めるべき**
```typescript
const resetTimer = useCallback(() => {
  setTimeLeft(initialTime);      // ← useMemoの値
  clearTimerState();             // ← Contextから来た関数
  props.onReset?.();             // ← Propsから来た関数
}, [initialTime, clearTimerState, props.onReset]);
```

**❌ 依存配列に含めなくて良い**
```typescript
const resetTimer = useCallback(() => {
  const localVar = 100;          // ← 関数内で定義
  setIsRunning(false);           // ← setStateは安定している
  startTimeRef.current = null;   // ← useRefは常に同じ
}, []); // これらは依存配列に不要
```

### 実例：toggleTimer関数

```typescript
// ❌ 悪い例：依存配列にtimeLeftを含めると毎秒再作成される
const toggleTimer = useCallback(() => {
  setIsRunning((prev) => {
    const newState = !prev;
    if (newState) {
      endTimeRef.current = Date.now() + timeLeft * 1000; // timeLeftを使用
    }
    return newState;
  });
}, [timeLeft]); // ← timeLeftが変わるたびに関数が再作成される

// ✅ 良い例：setStateのコールバック形式で最新の値を取得
const toggleTimer = useCallback(() => {
  setIsRunning((prev) => {
    const newState = !prev;
    if (newState) {
      // setTimeLeftのコールバック形式で最新のtimeLeftを取得
      setTimeLeft((currentTimeLeft) => {
        endTimeRef.current = Date.now() + currentTimeLeft * 1000;
        return currentTimeLeft;
      });
    }
    return newState;
  });
}, []); // ← 依存配列を空にできる
```

### eslint-disable-next-line の使い方

```typescript
// 状況1: 意図的に古い値を使いたい場合（稀）
const handleClick = useCallback(() => {
  console.log(count); // 常にマウント時のcountを参照したい
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);

// 状況2: 依存配列を修正する方が良い場合（推奨）
const handleClick = useCallback(() => {
  console.log(count); // 最新のcountを参照
}, [count]); // ✅ eslint-disableせず、正しく依存配列に含める
```

## 実用例

```typescript
export const useTimer = (timerType: TimerType, onComplete?: () => void) => {
  const { settings } = useSettings();
  const { clearTimerState } = useTimerState();

  const initialTime = useMemo(() => {
    return SECONDS * settings.focusTime;
  }, [settings.focusTime]);

  // ✅ 外部から来た値・関数を全て依存配列に含める
  const resetTimer = useCallback(() => {
    setIsRunning(false);
    setTimeLeft(initialTime);    // useMemoの値
    clearTimerState();           // Contextの関数
    startTimeRef.current = null; // useRef（不要）
  }, [initialTime, clearTimerState]); // ✅

  // ✅ setStateのコールバック形式を使って依存配列を減らす
  const toggleTimer = useCallback(() => {
    setIsRunning((prev) => !prev);
    setHasStarted(true);
  }, []); // ← 空でOK

  return { resetTimer, toggleTimer };
};
```

## 参考
- [React公式: useCallback](https://react.dev/reference/react/useCallback)
- [ESLint Plugin: react-hooks/exhaustive-deps](https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks)

## 関連スニペット
- [useMemo で参照の安定性を保つ](./useMemo-reference-stability.md)
- [useRef vs useState](./useRef-vs-useState.md)

## 作成日
2026-01-31

## タグ
#react #hooks #useCallback #dependencies #eslint #performance
