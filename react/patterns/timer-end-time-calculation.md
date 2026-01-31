# 終了時刻逆算方式のタイマー実装

## 概要
バックグラウンドタブでも正確に動作するタイマーを、終了時刻を基準に逆算する方式で実装する方法

## 使用場面
- ブラウザのタブがバックグラウンドになっても正確に動作するタイマーが必要な時
- ポモドーロタイマー、カウントダウンタイマーなど
- setIntervalの遅延の影響を受けたくない時

## コード

### デクリメント方式（従来・問題あり）

```typescript
// ❌ 従来のデクリメント方式：バックグラウンドでズレる
useEffect(() => {
  if (isRunning && timeLeft > 0) {
    intervalRef.current = setInterval(() => {
      setTimeLeft((prev) => prev - 1); // 1秒ごとに1減らす
    }, 1000);
  }

  return () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };
}, [isRunning, timeLeft]);

// 問題点：
// - バックグラウンドタブではsetIntervalが遅延する
// - 1秒のはずが5秒、10秒と遅れることがある
// - 結果として計測時間がずれる
```

### 終了時刻逆算方式（推奨）

```typescript
// ✅ 終了時刻逆算方式：バックグラウンドでも正確
const [timeLeft, setTimeLeft] = useState(1500);
const [isRunning, setIsRunning] = useState(false);
const endTimeRef = useRef<number | null>(null);
const intervalRef = useRef<NodeJS.Timeout | null>(null);

// タイマー開始時：終了時刻を計算
const startTimer = () => {
  endTimeRef.current = Date.now() + timeLeft * 1000;
  setIsRunning(true);
};

// カウントダウン処理
useEffect(() => {
  if (isRunning && endTimeRef.current) {
    intervalRef.current = setInterval(() => {
      const now = Date.now();
      const remaining = Math.max(
        0,
        Math.ceil((endTimeRef.current! - now) / 1000)
      );

      setTimeLeft(remaining);

      if (remaining === 0) {
        setIsRunning(false);
        endTimeRef.current = null;
        if (intervalRef.current) {
          clearInterval(intervalRef.current);
          intervalRef.current = null;
        }
        onComplete?.();
      }
    }, 250); // 250msごとに更新（より正確な計測）
  }

  return () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };
}, [isRunning, onComplete]);
```

## 説明

### 従来のデクリメント方式の問題点

```
【バックグラウンドタブでの動作】

期待：1秒ごとに1減る
1500 → 1499 → 1498 → 1497 ...

実際：setIntervalが遅延
1500 → (5秒後) → 1499 → (10秒後) → 1498 ...
     ↑ ブラウザがバックグラウンドタブの処理を抑制

結果：25分のタイマーが30分、40分とズレる
```

### 終了時刻逆算方式の仕組み

```
【開始時】
現在時刻: 10:00:00
残り時間: 1500秒（25分）
→ 終了時刻を計算: 10:25:00
→ endTimeRef.current = Date.now() + 1500 * 1000

【250msごとの計算】
現在時刻: 10:10:30
→ 残り時間 = (10:25:00 - 10:10:30) / 1000 = 870秒
→ 画面に表示: 14分30秒

現在時刻: 10:10:30.25（0.25秒後）
→ 残り時間 = (10:25:00 - 10:10:30.25) / 1000 = 869秒
→ 画面に表示: 14分29秒

【バックグラウンドから復帰】
（5分間バックグラウンドだったが、復帰した）
現在時刻: 10:15:30
→ 残り時間 = (10:25:00 - 10:15:30) / 1000 = 570秒 ← 正確！
→ 画面に表示: 9分30秒
```

### 重要なポイント

**1. 終了時刻を保存（useRef）**
```typescript
const endTimeRef = useRef<number | null>(null);

// タイマー開始時
endTimeRef.current = Date.now() + timeLeft * 1000;

// 一時停止時
endTimeRef.current = null;
```

**2. 毎回終了時刻から逆算**
```typescript
const now = Date.now();
const remaining = Math.ceil((endTimeRef.current! - now) / 1000);
```

**3. 更新間隔を短く（250ms）**
```typescript
setInterval(() => {
  // 残り時間を計算
}, 250); // 1000msより短くして滑らかに
```

**4. Math.ceil で切り上げ**
```typescript
// Math.floor（切り捨て）だと不自然
// 残り0.9秒 → 0秒と表示される

// Math.ceil（切り上げ）で自然な表示
// 残り0.1秒 → 1秒と表示される
Math.ceil((endTimeRef.current! - now) / 1000);
```

## 実用例：完全なタイマーフック

```typescript
export const useTimer = (
  initialSeconds: number,
  onComplete?: () => void
) => {
  const [timeLeft, setTimeLeft] = useState(initialSeconds);
  const [isRunning, setIsRunning] = useState(false);
  const [hasStarted, setHasStarted] = useState(false);

  const endTimeRef = useRef<number | null>(null);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  // タイマーをスタート/停止
  const toggleTimer = useCallback(() => {
    setIsRunning((prev) => {
      const newState = !prev;

      if (newState) {
        // 開始時：終了時刻を計算
        setTimeLeft((currentTimeLeft) => {
          endTimeRef.current = Date.now() + currentTimeLeft * 1000;
          return currentTimeLeft;
        });
        setHasStarted(true);
      } else {
        // 一時停止時：終了時刻をクリア
        endTimeRef.current = null;
      }

      return newState;
    });
  }, []);

  // リセット
  const resetTimer = useCallback(() => {
    setIsRunning(false);
    setTimeLeft(initialSeconds);
    setHasStarted(false);
    endTimeRef.current = null;
  }, [initialSeconds]);

  // 終了時刻逆算方式によるカウントダウン
  useEffect(() => {
    if (isRunning && endTimeRef.current) {
      intervalRef.current = setInterval(() => {
        const now = Date.now();
        const remaining = Math.max(
          0,
          Math.ceil((endTimeRef.current! - now) / 1000)
        );

        setTimeLeft(remaining);

        if (remaining === 0) {
          setIsRunning(false);
          endTimeRef.current = null;

          if (intervalRef.current) {
            clearInterval(intervalRef.current);
            intervalRef.current = null;
          }

          onComplete?.();
        }
      }, 250);
    }

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [isRunning, onComplete]);

  return {
    timeLeft,
    isRunning,
    hasStarted,
    toggleTimer,
    resetTimer,
  };
};

// 使用例
function TimerComponent() {
  const { timeLeft, isRunning, toggleTimer, resetTimer } = useTimer(
    1500, // 25分
    () => console.log('タイマー完了！')
  );

  const minutes = Math.floor(timeLeft / 60);
  const seconds = timeLeft % 60;

  return (
    <div>
      <div>{minutes}:{seconds.toString().padStart(2, '0')}</div>
      <button onClick={toggleTimer}>
        {isRunning ? 'PAUSE' : 'START'}
      </button>
      <button onClick={resetTimer}>RESET</button>
    </div>
  );
}
```

## 比較：デクリメント vs 終了時刻逆算

| 項目 | デクリメント方式 | 終了時刻逆算方式 |
|------|----------------|----------------|
| 実装 | `prev - 1` | `(endTime - now) / 1000` |
| 更新間隔 | 1000ms | 250ms（推奨） |
| バックグラウンド | ズレる ❌ | 正確 ✅ |
| 精度 | 低い | 高い |
| 複雑さ | シンプル | やや複雑 |
| 推奨 | ❌ | ✅ |

## 参考
- [MDN: Date.now()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date/now)
- [Background Tabs and Performance](https://developer.chrome.com/blog/timer-throttling-in-chrome-88/)

## 関連スニペット
- [useRef vs useState](../hooks/useRef-vs-useState.md)
- [useEffect のクリーンアップ](../hooks/useEffect-cleanup-unmount.md)

## 作成日
2026-01-31

## タグ
#react #patterns #timer #setInterval #background-tab #accuracy
