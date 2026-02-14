# GetMapping findAll ページネーション付き一覧取得

## 概要
`@GetMapping`と`Pageable`パラメータを組み合わせて、ページネーション・ソート付きのGET一覧エンドポイントを実装するパターン。

## 使用場面
- REST APIで一覧取得エンドポイントを実装する場合
- 大量データをページ分割して返却する場合
- クライアントにソート順を指定させたい場合

## コード
```java
// 基本: CrudRepository.findAll() による単純な一覧取得
@GetMapping()
private ResponseEntity<Iterable<CashCard>> findAll() {
   return ResponseEntity.ok(cashCardRepository.findAll());
}
```

```java
// 推奨: ページネーション・ソート付きの一覧取得
@GetMapping
private ResponseEntity<List<CashCard>> findAll(Pageable pageable) {
   Page<CashCard> page = cashCardRepository.findAll(
           PageRequest.of(
                   pageable.getPageNumber(),
                   pageable.getPageSize(),
                   pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))));
   return ResponseEntity.ok(page.getContent());
}
```

```java
// リポジトリはPagingAndSortingRepositoryを継承（またはCrudRepositoryの拡張版）
public interface CashCardRepository
        extends CrudRepository<CashCard, Long>, PagingAndSortingRepository<CashCard, Long> {
}
```

## 説明
### 基本版 vs ページネーション版

| 観点 | 基本版 (`findAll()`) | ページネーション版 |
|------|---------------------|------------------|
| レスポンスサイズ | 無制限 | ページサイズで制御 |
| ソート | なし | 指定可能（デフォルト付き） |
| メモリ使用量 | 全件ロード | ページ分のみ |
| 本番適用 | ❌ 危険 | ✅ 推奨 |

### Pageableパラメータの自動解析
Spring MVCは`Pageable`引数を自動的にクエリパラメータから構築する：
- `page`: ページ番号（0始まり、デフォルト: 0）
- `size`: 1ページあたりの件数（デフォルト: 20）
- `sort`: ソートフィールドと方向（例: `sort=amount,desc`）

### getSortOr() によるデフォルトソート
`pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))` は、クライアントがsortパラメータを指定しなかった場合のデフォルトソートを設定する。pageやsizeと異なり、ソートのデフォルトをSpringが自動提供するのは不適切なため、明示的に指定する。

### page.getContent() でデータのみ返却
`Page`オブジェクト全体ではなく、`getContent()`でデータリスト部分のみを返却し、レスポンスのデータ契約をシンプルに保つ。

## 参考
- [Spring Data - PagingAndSortingRepository](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#repositories.paging-and-sorting)
- [Spring MVC - Pageable resolver](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#core.web.basic.paging-and-sorting)

## 関連スニペット
- [PageableとPage](../data/pageable-and-page.md)
- [CrudRepositoryの基本メソッド](../data/crud-repository-methods.md)
- [REST Controller GET with PathVariable](./rest-controller-get-with-path-variable.md)
- [ResponseEntity.created()](./response-entity-created.md)

## 作成日
2026-02-14

## タグ
#spring #controller #rest-api #pagination #get-mapping #findAll
