# REST Controller - GETリクエストと@PathVariable

## 概要
`@RestController`でREST APIコントローラーを定義し、`@GetMapping`と`@PathVariable`でパス変数を使ったGETエンドポイントを実装する。`ResponseEntity`でHTTPステータスコードとレスポンスボディを制御する。

## 使用場面
- 特定リソースをIDで取得するGETエンドポイントを実装する時
- HTTPステータスコード（200 OK / 404 Not Found等）を明示的に制御したい時
- RESTful なURIパターン（`/resources/{id}`）でAPIを設計する時

## コード

```java
@RestController
class CashCardController {

    @GetMapping("/cashcards/{requestedId}")
    private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
        CashCard cashCard = /* リポジトリからCashCardを取得 */;
        return ResponseEntity.ok(cashCard); // 200 OK + ボディ
    }
}
```

## 説明

### @RestController
```java
@RestController
class CashCardController { }
```
- クラスをREST Controllerとして宣言し、Spring Webに注入される
- Spring Webがルーティングを行い、適切なハンドラメソッドにリクエストを振り分ける

### @GetMapping とURIパス
```java
@GetMapping("/cashcards/{requestedId}")
```
- HTTP GETリクエストのみをこのメソッドにルーティングする
- `{requestedId}`はパス変数のプレースホルダー

### @PathVariable
```java
private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
```
- URIパスの`{requestedId}`部分の値をメソッド引数に注入する
- パラメータ名がプレースホルダー名と一致することでSpringが自動的にバインドする

| リクエスト | `requestedId`の値 |
|-----------|-------------------|
| `GET /cashcards/99` | `99` |
| `GET /cashcards/42` | `42` |

### ResponseEntity
```java
return ResponseEntity.ok(cashCard); // 200 OK
```
- HTTPレスポンスのステータスコードとボディを明示的に制御するクラス
- `ResponseEntity.ok(body)` → 200 OK + ボディ
- `ResponseEntity.notFound().build()` → 404 Not Found（ボディなし）

### リクエストの流れ

```
GET /cashcards/99
    ↓
Spring Web（ルーティング）
    ↓ @GetMapping("/cashcards/{requestedId}") にマッチ
CashCardController.findById(99)
    ↓ @PathVariable で 99 を注入
ResponseEntity.ok(cashCard)
    ↓ Jackson が CashCard → JSON に自動変換
200 OK + {"id": 99, "amount": 123.45}
```

## 参考
- [Spring Academy - Implementing GET](https://spring.academy/courses/building-a-rest-api-with-spring-boot/lessons/implementing-get)
- [Spring Web MVC - Annotated Controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)

## 関連スニペット
- [@RestController](../rest-controller.md)
- [シリアル化とデシリアル化](../serialization-deserialization.md)
- [Spring Data Repository](../repository/)
- [テスト駆動開発](../test/testing-first.md)

## 作成日
2026-02-08

## タグ
#spring #rest-api #controller #getmapping #pathvariable #response-entity
