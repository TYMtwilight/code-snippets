# AtomicLong

## 概要
スレッドセーフな`long`型カウンター。複数のリクエストが同時に来ても安全にカウントアップできる。

## 使用場面
- REST APIでユニークなIDを発行する
- アクセスカウンター
- 統計情報の集計
- マルチスレッド環境でのカウンター

## コード
```java
import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {

    // カウンターを初期値0で作成
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam String name) {
        // incrementAndGet(): 1増やして、増やした後の値を返す
        long id = counter.incrementAndGet();  // 1, 2, 3, 4...
        return new Greeting(id, "Hello, " + name);
    }
}
```

## 説明

### なぜ普通の`long`ではダメなのか

普通の`long`の場合、`counter++`は内部で3つの操作に分かれる：
1. 値を読む
2. 1を足す
3. 値を書き戻す

この間に他のスレッドが割り込むと、IDが重複する可能性がある。

```
// 2つのリクエストが同時に来た場合
スレッド1: counter読む → 0
スレッド2: counter読む → 0  ← まだ0のまま
スレッド1: 0+1=1 → counterに書く
スレッド2: 0+1=1 → counterに書く  ← 両方とも1！
```

### AtomicLongなら安全

`incrementAndGet()`は「不可分操作（atomic operation）」として一気に実行される。

```
スレッド1: incrementAndGet() → 1 （他のスレッドは待機）
スレッド2: incrementAndGet() → 2 （順番に実行）
```

### 主なメソッド

```java
AtomicLong counter = new AtomicLong();     // 初期値0
AtomicLong counter = new AtomicLong(100);  // 初期値100

// 取得
long value = counter.get();

// 設定
counter.set(50);

// インクリメント（よく使う）
long result = counter.incrementAndGet();  // 先に増やす → 増やした値を返す
long result = counter.getAndIncrement();  // 先に取得 → 後で増やす

// デクリメント
long result = counter.decrementAndGet();

// 加算
counter.addAndGet(5);  // 5を足して結果を返す
```

## 参考
- [AtomicLong (Java SE 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/atomic/AtomicLong.html)

## 関連スニペット
- [REST Controller](../spring/rest-controller.md)

## 作成日
2026-01-30

## タグ
#java #concurrent #thread-safe #counter
