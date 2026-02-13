# TDD Red-Green-Refactor - テスト駆動開発サイクル

## 概要
TDD（テスト駆動開発）は、テストを先に書いてからプロダクションコードを実装する開発手法。Red-Green-Refactorの3ステップを繰り返してコードを段階的に成長させる。

## 使用場面
- REST APIのエンドポイントを新規実装するとき
- 仕様が明確で、テストで表現できるとき
- 安全にリファクタリングしたいとき

## コード

### Red: テストを書く → 失敗する
```java
@Test
void shouldReturnACashCardWhenDataIsSaved() {
    ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(response.getBody());
    Number id = documentContext.read("$.id");
    assertThat(id).isEqualTo(99);
}
```
この時点ではコントローラーが存在しないため、テストは失敗する（Red）。

### Green: 最小限のコードでテストを通す
```java
@RestController
@RequestMapping("/cashcards")
class CashCardController {
    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
        if (requestedId.equals(99L)) {
            CashCard cashCard = new CashCard(99L, 123.45);
            return ResponseEntity.ok(cashCard);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```
ハードコーディングでもテストが通ればOK（Green）。目的は「動くコード」を最速で得ること。

### Refactor: コードを改善する
```java
@RestController
@RequestMapping("/cashcards")
class CashCardController {
    private final CashCardRepository cashCardRepository;

    // コンストラクタインジェクション
    private CashCardController(CashCardRepository cashCardRepository) {
        this.cashCardRepository = cashCardRepository;
    }

    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
        Optional<CashCard> cashCardOptional = cashCardRepository.findById(requestedId);
        if (cashCardOptional.isPresent()) {
            return ResponseEntity.ok(cashCardOptional.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```
テストが通る状態を維持しながら、データベース接続などの本来の実装に置き換える。

## 説明

### 3ステップの意味

| ステップ | 状態 | やること |
|---------|------|---------|
| **Red** | テスト失敗 | 期待する振る舞いをテストとして書く |
| **Green** | テスト成功 | テストを通す最小限のコードを書く |
| **Refactor** | テスト成功を維持 | コードの品質を改善する |

### なぜハードコーディングから始めるのか
- テストが正しく書かれているか確認できる
- 一度に解決する問題を小さくできる
- 常に「動くコード」がある状態を保てる

### TDDの利点
- **設計が改善される**: テストしやすいコード = 疎結合なコード
- **回帰テストが自動的に蓄積される**: リファクタリング時の安全網になる
- **仕様がコードとして残る**: テストがドキュメントの役割を果たす

## 参考
- [Spring Academy - Spring Boot TDD](https://spring.academy/courses/building-a-rest-api-with-spring-boot)

## 関連スニペット
- [SpringBootTest GETエンドポイントテスト](springboottest-get-endpoint.md)
- [TestRestTemplate](test-rest-template.md)
- [DocumentContext](../../java/document-context.md)
- [テスティングファースト](testing-first.md)
- [ResponseEntity](../rest-client/response-entity.md)

## 作成日
2026-02-14

## タグ
#spring #tdd #testing #red-green-refactor #test-driven-development
