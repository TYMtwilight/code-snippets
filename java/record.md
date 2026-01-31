# Java Record（レコード）

## 概要
データを保持するためのシンプルなクラスを簡潔に定義できる構文（Java 14以降）。REST APIのレスポンス用DTOとしてよく使われる。

## 使用場面
- REST APIのレスポンスデータ（Resource Representation Class）
- DTOクラスの定義
- イミュータブルなデータオブジェクト
- 設定値の保持

## コード
```java
// record版（簡潔）
public record Greeting(long id, String content) { }

// これだけでOK！以下が自動生成される：
// - コンストラクタ
// - getter（id(), content()）
// - equals(), hashCode(), toString()
```

## 説明

### recordが自動生成するもの

```java
public record Greeting(long id, String content) { }
```

↓ 上記は以下と同等 ↓

```java
public class Greeting {
    private final long id;
    private final String content;

    // コンストラクタ
    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    // getter（※getterではなくid()形式）
    public long id() { return id; }
    public String content() { return content; }

    // equals, hashCode, toString
    @Override
    public boolean equals(Object o) { /* 自動実装 */ }
    @Override
    public int hashCode() { /* 自動実装 */ }
    @Override
    public String toString() { /* 自動実装 */ }
}
```

### REST APIでの使い方

```java
// レスポンス用のrecord
public record Greeting(long id, String content) { }

@RestController
public class GreetingController {

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam String name) {
        return new Greeting(1, "Hello, " + name);
    }
}
```

**レスポンス（自動でJSONに変換）**:
```json
{
    "id": 1,
    "content": "Hello, Hiroki"
}
```

### Entityとの違い

| 項目 | Record（DTO） | Entity |
|------|---------------|--------|
| 目的 | APIレスポンスの形を定義 | DBテーブルと対応 |
| アノテーション | なし | @Entity |
| 可変性 | イミュータブル（不変） | ミュータブル（可変） |
| 用途 | コントローラー ↔ クライアント | リポジトリ ↔ DB |

### recordの特徴

```java
public record Greeting(long id, String content) { }

// ✅ イミュータブル（変更不可）
Greeting g = new Greeting(1, "Hello");
// g.id = 2;  // コンパイルエラー！setterがない

// ✅ 値の取得
long id = g.id();         // 1
String content = g.content();  // "Hello"

// ✅ 便利なtoString
System.out.println(g);  // Greeting[id=1, content=Hello]
```

### メソッドの追加も可能

```java
public record Greeting(long id, String content) {

    // カスタムメソッドを追加できる
    public String formattedContent() {
        return "[" + id + "] " + content;
    }
}
```

### バリデーション付きコンストラクタ

```java
public record Greeting(long id, String content) {

    // コンパクトコンストラクタ
    public Greeting {
        if (id < 0) {
            throw new IllegalArgumentException("id must be positive");
        }
        if (content == null || content.isBlank()) {
            throw new IllegalArgumentException("content is required");
        }
    }
}
```

## 参考
- [Record Classes (Java SE 17)](https://docs.oracle.com/en/java/javase/17/language/records.html)
- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service)

## 関連スニペット
- [REST Controller](../spring/rest-controller.md)

## 作成日
2026-01-30

## タグ
#java #record #dto #immutable #rest-api
