# ResponseEntity.created() - リソース作成レスポンスの便利メソッド

## 概要
REST APIでリソース作成時に `201 Created` レスポンスと `Location` ヘッダーを返す際、Springの `ResponseEntity.created()` を使って簡潔に記述する方法。

## 使用場面
- `@PostMapping` でリソースを新規作成し、作成先URIをクライアントに返すとき
- RESTful APIのベストプラクティスに従った `201 Created` レスポンスを返すとき

## コード
```java
// ✅ Spring提供の便利メソッド（推奨）
return ResponseEntity
        .created(uriOfCashCard)
        .build();

// ❌ 手動で全部指定する方法（冗長）
return ResponseEntity
        .status(HttpStatus.CREATED)
        .header(HttpHeaders.LOCATION,
                uriOfCashCard.toASCIIString())
        .build();
```

## 説明
### `created()` メソッドが自動で行うこと
- ステータスコードを `201 Created` に設定
- `Location` ヘッダーにURIを追加
- URIの形式チェック（バリデーション）

### 手動指定に対する利点
- **コードが短い**: 1行で済む
- **ミスが減る**: ステータスコードやヘッダー名の間違いを防げる
- **URI検証**: URIが正しい形式かチェックしてくれる
- **意図が明確**: `created` という名前で何をしているか一目瞭然

### 実際の使用例
```java
@PostMapping
public ResponseEntity<Void> createCashCard(@RequestBody CashCard newCard) {
    CashCard saved = repository.save(newCard);

    // 作成したリソースのURI: /cashcards/123
    URI location = URI.create("/cashcards/" + saved.getId());

    // 便利メソッドで一発
    return ResponseEntity.created(location).build();
}
```

### レスポンス例
```
HTTP/1.1 201 Created
Location: /cashcards/123
```

## 参考
- [ResponseEntity - Spring公式ドキュメント](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)

## 関連スニペット
- [REST Controller GET with PathVariable](rest-controller-get-with-path-variable.md)
- [REST Controller基本](../rest-controller.md)
- [API Contract](../api-contract.md)

## 作成日
2026-02-14

## タグ
#spring #controller #rest-api #response-entity #post #201-created
