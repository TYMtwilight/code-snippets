# useEffect のクリーンアップとアンマウント

## 概要
useEffectのクリーンアップ関数を使って、アンマウント時やエフェクト再実行前に適切な後処理を行う方法

## 使用場面
- setIntervalやsetTimeoutをクリアする時
- イベントリスナーを削除する時
- コンポーネントのアンマウント時に状態を保存する時
- WebSocketやAPI接続を切断する時

## コード

### 基本的なクリーンアップパターン

```typescript
useEffect(() => {
  // ↓ マウント時、または依存配列が変わった時に実行
  console.log('コンポーネントがマウントされました');
  const subscription = api.subscribe();

  // ↓ アンマウント時、または次の実行前に実行
  return () => {
    console.log('クリーンアップします');
    subscription.unsubscribe();
  };
}, [依存配列]);
```

### setIntervalのクリーンアップ

```typescript
// ❌ 悪い例：クリーンアップなし
useEffect(() => {
  const id = setInterval(() => {
    console.log('実行中');
  }, 1000);
  // クリーンアップしないと...
  // → コンポーネントが消えてもsetIntervalが動き続ける
  // → メモリリーク発生
}, []);

// ✅ 良い例：クリーンアップあり
useEffect(() => {
  const id = setInterval(() => {
    console.log('実行中');
  }, 1000);

  return () => {
    clearInterval(id); // アンマウント時に必ずクリア
  };
}, []);
```

### アンマウント時に状態を保存

```typescript
useEffect(() => {
  // マウント時：保存された状態から復元
  if (autoStart && !startTimeRef.current) {
    startTimeRef.current = new Date();
    endTimeRef.current = Date.now() + timeLeft * 1000;
  }

  if (savedState) {
    clearTimerState();
  }

  // アンマウント時：現在の状態を保存
  return () => {
    const currentState = stateRef.current;

    if (currentState.timeLeft > 0) {
      // タイマーが残っている場合は保存
      saveTimerState({
        timerType,
        timeLeft: currentState.timeLeft,
        isRunning: currentState.isRunning,
        hasStarted: currentState.hasStarted,
        startTime: startTimeRef.current,
      });
    } else {
      // タイマーが完了している場合はクリア
      clearTimerState();
    }
  };
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []); // 空の依存配列 = マウント時に1回だけ実行
```

## 説明

### マウント・アンマウントとは

**マウント（Mount）**
- コンポーネントが画面に表示される瞬間
- 例：ユーザーが `/timer/work` にアクセス

**アンマウント（Unmount）**
- コンポーネントが画面から消える瞬間
- 例：ユーザーが別ページに移動

### クリーンアップ関数が実行されるタイミング

```typescript
useEffect(() => {
  console.log('A: エフェクト実行');

  return () => {
    console.log('B: クリーンアップ');
  };
}, [count]);

// 動作の流れ：
// 1. マウント時
//    → A: エフェクト実行

// 2. countが変わった時
//    → B: クリーンアップ（前回のエフェクトを片付ける）
//    → A: エフェクト実行（新しいエフェクトを実行）

// 3. アンマウント時
//    → B: クリーンアップ（最後の片付け）
```

### 依存配列が空の場合

```typescript
useEffect(() => {
  console.log('マウント時のみ実行');

  return () => {
    console.log('アンマウント時のみ実行');
  };
}, []); // 空の依存配列

// 実行タイミング：
// - マウント時: エフェクト実行
// - アンマウント時: クリーンアップ実行
// - 途中の再レンダリング: 何もしない
```

※依存配列が空の場合、クリーンアップ関数はマウント時の変数の値をキャプチャしたままとなる。
　つまり、マウント時の状態しか見ないので、もしその変数の値が更新されたとしても、クリーンアップ関数では
　古い状態の値が参照されてしまう。

### よくあるクリーンアップパターン

**1. タイマーのクリア**
```typescript
useEffect(() => {
  const intervalId = setInterval(() => { ... }, 1000);
  return () => clearInterval(intervalId);
}, []);

useEffect(() => {
  const timeoutId = setTimeout(() => { ... }, 5000);
  return () => clearTimeout(timeoutId);
}, []);
```

**2. イベントリスナーの削除**
```typescript
useEffect(() => {
  const handleResize = () => { ... };
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

**3. API購読の解除**
```typescript
useEffect(() => {
  const subscription = api.subscribe(data => { ... });
  return () => subscription.unsubscribe();
}, []);
```

**4. 状態の保存**
```typescript
useEffect(() => {
  return () => {
    localStorage.setItem('state', JSON.stringify(state));
  };
}, []);
```

## 実用例：タイマーの完全な実装

```typescript
export const useTimer = (timerType: TimerType) => {
  const [timeLeft, setTimeLeft] = useState(1500);
  const [isRunning, setIsRunning] = useState(false);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  const stateRef = useRef({ timeLeft, isRunning });

  // 最新の状態を保存（アンマウント時に参照するため）
  useEffect(() => {
    stateRef.current = { timeLeft, isRunning };
  }, [timeLeft, isRunning]);

  // マウント・アンマウント処理
  useEffect(() => {
    console.log('🟢 タイマーがマウントされました');

    return () => {
      console.log('🔴 タイマーがアンマウントされます');
      // 最新の状態を保存
      if (stateRef.current.timeLeft > 0) {
        saveTimerState(stateRef.current);
      }
    };
  }, []);

  // タイマー処理
  useEffect(() => {
    if (isRunning) {
      intervalRef.current = setInterval(() => {
        setTimeLeft(prev => prev - 1);
      }, 1000);
    }

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [isRunning]);

  return { timeLeft, isRunning };
};
```

### ユーザー行動との対応

```
1. ユーザーが /timer/work にアクセス
   → マウント
   → console.log('🟢 タイマーがマウントされました')

2. ユーザーがタイマーを開始
   → isRunning = true
   → setInterval開始

3. ユーザーが設定ページに移動
   → アンマウント
   → clearInterval実行
   → console.log('🔴 タイマーがアンマウントされます')
   → 現在の状態をローカルストレージに保存

4. ユーザーが /timer/work に戻る
   → 再マウント
   → 保存された状態を復元
```

## 参考
- [React公式: useEffect](https://react.dev/reference/react/useEffect)
- [A Complete Guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/)

## 関連スニペット
- [useRef vs useState](./useRef-vs-useState.md)
- [useCallback の依存配列](./useCallback-dependencies.md)

## 作成日
2026-01-31

## タグ
#react #hooks #useEffect #cleanup #unmount #lifecycle #memory-leak
