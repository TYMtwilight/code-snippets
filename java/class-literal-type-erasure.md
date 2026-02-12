# Classリテラルと型消去（Type Erasure）

## 概要
Javaでは実行時にジェネリクスの型情報が消える（型消去）ため、SpringのRestTemplateなどでレスポンスの変換先の型を指定する際には`.class`（クラスリテラル）を使って型情報を明示的に渡す必要がある。

## 使用場面
- `RestTemplate`や`TestRestTemplate`でレスポンスの変換先の型を指定するとき
- リフレクションで型情報が必要なとき
- フレームワークに「この型に変換して」と伝えるとき

## コード
```java
// クラスリテラルの基本
// Class<String>はString.classで取得できます
// Class<User>はUser.classで取得できます

// RestTemplateでの使用例
// String.class → JSONをそのまま文字列で受け取る
ResponseEntity<String> response = restTemplate.getForEntity(
    "/cashcards/99", String.class
);
// response.getBody() → "{\"id\":99,\"amount\":123.45}"

// CashCard.class → JSONをCashCardオブジェクトに変換
ResponseEntity<CashCard> response = restTemplate.getForEntity(
    "/cashcards/99", CashCard.class
);
// response.getBody() → CashCard{id=99, amount=123.45}
```

## 説明

### 型消去（Type Erasure）とは
Javaのジェネリクスはコンパイル時にのみ存在し、**実行時には型情報が消える**仕様。

```java
// コンパイル時：型がある
ResponseEntity<String> response = ...;

// 実行時（JVM上）：型が消えている
ResponseEntity response = ...;  // Stringという情報が消える
```

そのため、RestTemplateは実行時に「レスポンスを何に変換すべきか」を知ることができない。

### `.class` で型情報を渡す
`.class`はクラスリテラルと呼ばれ、クラス自体の情報を表す`Class`オブジェクトを取得する構文。

```java
restTemplate.getForEntity("/cashcards/99", String.class);
//                                         ^^^^^^^^^^^^
//               「レスポンスをStringとして受け取って」と明示的に伝えている
```

RestTemplateは渡された`Class`オブジェクトを使って：
1. HTTPレスポンスのボディを受け取る
2. `String.class` → そのまま文字列として返す
3. `CashCard.class` → JSONを`CashCard`オブジェクトに変換（デシリアライズ）して返す

### なぜジェネリクスだけでは不十分なのか
```java
// このメソッドシグネチャだけでは、実行時にTが何か分からない
public <T> ResponseEntity<T> getForEntity(String url, ???) {
    // Tの情報が消えているので、変換できない
}

// だから Class<T> を引数で受け取る
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType) {
    // responseType（例：String.class）を使ってボディを変換できる
}
```

## 参考
- [Oracle Java Tutorial - Type Erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html)
- [Oracle Java Tutorial - Class Literals](https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.8.2)

## 関連スニペット
- [ResponseEntity](../spring/rest-client/response-entity.md)
- [TestRestTemplate](../spring/test/test-rest-template.md)
- [RestTemplateとHTTPクライアント](../spring/rest-client/rest-template-http-client.md)

## 作成日
2026-02-13

## タグ
#java #generics #type-erasure #class-literal #reflection
