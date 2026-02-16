# ResponseEntity - HTTPレスポンス全体を表すオブジェクト

## 概要
`ResponseEntity<T>`はHTTPレスポンスの「ステータスコード」「ヘッダー」「ボディ」をまとめて保持するSpringのクラス。ジェネリクス`<T>`でボディの型を指定する。

## 使用場面
- REST APIのレスポンスをステータスコードやヘッダーも含めて扱いたいとき
- テストでHTTPレスポンスの各部分を検証したいとき
- コントローラからステータスコードを明示的に返したいとき

## コード
```java
// テストでの使用例：レスポンスの取得と検証
ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);

response.getStatusCode();  // → 200 OK（ステータスコード）
response.getHeaders();     // → Content-Type: application/json 等（ヘッダー）
response.getBody();        // → "{\"id\":99,\"amount\":123.45}"（ボディ）

// ステータスコードの検証
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

// ボディの検証
String body = response.getBody();
assertThat(body).isNotBlank();
```

```java
// コントローラでの使用例：レスポンスの構築
@GetMapping("/cashcards/{id}")
public ResponseEntity<CashCard> findById(@PathVariable Long id) {
    CashCard card = repository.findById(id);
    if (card != null) {
        return ResponseEntity.ok(card);           // 200 OK + ボディ
    }
    return ResponseEntity.notFound().build();      // 404 Not Found
}
```

## 説明

### HTTPレスポンスの構成とResponseEntityの対応
```
HTTP/1.1 200 OK                    ← getStatusCode()
Content-Type: application/json     ← getHeaders()

{"id": 99, "amount": 123.45}      ← getBody()
```

`ResponseEntity`はこの3つの要素すべてを保持する。

### ジェネリクス `<T>` が指定するもの
ボディの型のみを指定する。ステータスコードとヘッダーは常に同じ型。

```java
ResponseEntity<String>       // ボディが String
ResponseEntity<CashCard>     // ボディが CashCard オブジェクト
ResponseEntity<List<User>>   // ボディが User のリスト
```

### 主なメソッド
| メソッド | 戻り値 | 説明 |
|---|---|---|
| `getStatusCode()` | `HttpStatusCode` | HTTPステータスコード |
| `getHeaders()` | `HttpHeaders` | レスポンスヘッダー |
| `getBody()` | `T` | レスポンスボディ（ジェネリクスで指定した型） |

### getForEntity と getForObject の違い
`RestTemplate`でGETリクエストを送る際、目的に応じて使い分ける。

| メソッド | 戻り値 | 用途 |
|---|---|---|
| `getForEntity` | `ResponseEntity<T>` | ステータスコード・ヘッダー・ボディすべてが必要なとき |
| `getForObject` | `T` | ボディだけが必要なとき |

```java
// レスポンス全体を取得（テストで多用）
ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);

// ボディだけ取得
String body = restTemplate.getForObject("/cashcards", String.class);
```

テストではステータスコードも検証したいので`getForEntity`を使うことが多い。

### ResponseEntity の静的ファクトリメソッド（コントローラ向け）
| メソッド | ステータスコード |
|---|---|
| `ResponseEntity.ok(body)` | 200 OK |
| `ResponseEntity.created(uri).build()` | 201 Created |
| `ResponseEntity.noContent().build()` | 204 No Content |
| `ResponseEntity.notFound().build()` | 404 Not Found |
- 静的ファクトリメソッドというのは、クラスに属するメソッド（つまりstaticメソッド）で、新しいオブジェクトを生成して返す役割を持ちます。コンストラクタの代わりに使うことが多く、メソッド名で意味を明示できるのが利点です。

- ResponseEntity.notFound()は、HTTPステータス404（Not Found）を示すレスポンスを構築するための静的ファクトリメソッドです。これを呼ぶと、ステータスが404のレスポンスが準備され、その後.build()を呼ぶことで完成したレスポンスを返せます。

## 参考
- [Spring Framework - ResponseEntity](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)

## 関連スニペット
- [TestRestTemplate](../test/test-rest-template.md)
- [RestTemplateとHTTPクライアント](./rest-template-http-client.md)
- [Classリテラルと型消去](../../java/class-literal-type-erasure.md)
- [REST Controller](../rest-controller.md)

## 作成日
2026-02-13

## タグ
#spring #response-entity #http #rest-api #generics
