# Pageable - ページネーションとソートのインターフェース

## 概要
`Pageable`は、ページネーション（ページ分割）とソート機能を提供するSpring Dataのインターフェース。大量データを小分けにして効率的に取得する際に使う。

## 使用場面
- 大量データを一括取得するとメモリ不足やレスポンス遅延が発生する場合
- 一覧画面でページ送りやソート機能を実装する場合
- REST APIでページネーション付きのレスポンスを返す場合

## コード

### 問題: 大量データの一括取得

```java
// 10万件のデータを一度に取得 → メモリ不足、レスポンス遅延
List<CashCard> allCards = cashCardRepository.findAll();
```

### 解決: PageRequestでページ分割

```java
// 20件ずつ取得
Page<CashCard> page = cashCardRepository.findAll(
    PageRequest.of(0, 20)  // 1ページ目、20件
);
```

### Repository定義

```java
public interface CashCardRepository extends CrudRepository<CashCard, Long> {
    Page<CashCard> findAll(Pageable pageable);

    // カスタムクエリでも使える
    Page<CashCard> findByOwner(String owner, Pageable pageable);
}
```

### PageRequestの作成方法

```java
// 基本形: ページ番号とサイズを指定
Pageable pageable = PageRequest.of(0, 20);

// ソート付き（昇順）
Pageable pageable = PageRequest.of(0, 20, Sort.by("amount").ascending());

// ソート付き（降順）
Pageable pageable = PageRequest.of(0, 20, Sort.by("amount").descending());

// 複数フィールドでソート
Pageable pageable = PageRequest.of(0, 20,
    Sort.by("amount").descending()
        .and(Sort.by("id").ascending())
);

// 短縮形
Pageable pageable = PageRequest.of(0, 20, Sort.Direction.DESC, "amount");
```

### Controllerでの利用（Springが自動でPageableを生成）

```java
@RestController
@RequestMapping("/cashcards")
public class CashCardController {

    @GetMapping
    public ResponseEntity<Page<CashCard>> findAll(Pageable pageable) {
        // Springがクエリパラメータから自動的にPageableを生成
        Page<CashCard> page = repository.findAll(pageable);
        return ResponseEntity.ok(page);
    }
}
```

**リクエスト例:**
```
GET /cashcards?page=0&size=10
GET /cashcards?page=1&size=20&sort=amount,desc
GET /cashcards?page=0&size=10&sort=amount,desc&sort=id,asc
```

### Pageオブジェクトから取得できる情報

```java
Page<CashCard> page = repository.findAll(PageRequest.of(0, 20));

// データ取得
List<CashCard> content = page.getContent();        // 実際のデータリスト

// ページ情報
int totalPages = page.getTotalPages();              // 総ページ数
long totalElements = page.getTotalElements();       // 総要素数
int number = page.getNumber();                      // 現在のページ番号（0始まり）
int size = page.getSize();                          // ページサイズ
int numberOfElements = page.getNumberOfElements();  // このページの要素数

// ページ状態
boolean hasNext = page.hasNext();           // 次のページがあるか
boolean hasPrevious = page.hasPrevious();   // 前のページがあるか
boolean isFirst = page.isFirst();           // 最初のページか
boolean isLast = page.isLast();             // 最後のページか
```

### レスポンス例

```json
{
  "content": [
    {"id": 99, "amount": 123.45},
    {"id": 100, "amount": 1.00}
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20
  },
  "totalPages": 5,
  "totalElements": 100,
  "last": false,
  "first": true,
  "numberOfElements": 20
}
```

### フィルタリング + ページネーション

```java
@GetMapping("/cashcards")
public ResponseEntity<Page<CashCard>> findByOwner(
    @RequestParam String owner,
    Pageable pageable
) {
    Page<CashCard> page = repository.findByOwner(owner, pageable);
    return ResponseEntity.ok(page);
}
// GET /cashcards?owner=sarah&page=0&size=10&sort=amount,desc
```

### テストコードでの使用

```java
@Test
void shouldReturnPageOfCashCards() {
    Pageable pageable = PageRequest.of(0, 10, Sort.by("amount").descending());
    Page<CashCard> page = repository.findAll(pageable);

    assertThat(page.getContent()).hasSize(10);
    assertThat(page.getTotalElements()).isEqualTo(100);
    assertThat(page.getTotalPages()).isEqualTo(10);
    assertThat(page.isFirst()).isTrue();
    assertThat(page.hasNext()).isTrue();

    // ソート順を確認
    List<CashCard> cards = page.getContent();
    assertThat(cards.get(0).getAmount())
        .isGreaterThan(cards.get(1).getAmount());
}
```

## 説明
- ページ番号は**0始まり**（`page=0`が1ページ目）
- `Pageable`をControllerの引数に宣言するだけで、Springがクエリパラメータ（`page`, `size`, `sort`）から自動生成する
- `Page`オブジェクトはデータ本体（`content`）とメタ情報（総ページ数、総件数など）を両方含む
- 複数ソートは`sort`パラメータを複数指定する（例: `sort=amount,desc&sort=id,asc`）

### ページ番号の考え方

```
データが100件、1ページ20件の場合:
page=0 → 1～20件目    page=1 → 21～40件目
page=2 → 41～60件目   page=3 → 61～80件目
page=4 → 81～100件目  totalPages = 5
```

### application.propertiesでのカスタマイズ

```properties
spring.data.web.pageable.default-page-size=20
spring.data.web.pageable.max-page-size=100
spring.data.web.pageable.one-indexed-parameters=true  # 1始まりに変更
```

## 参考
- [Spring Data - Paging and Sorting](https://docs.spring.io/spring-data/commons/reference/repositories/query-methods-details.html#repositories.special-parameters)

## 関連スニペット
- [Pageable and Page](pageable-and-page.md)
- [ページネーションURIとソートのベストプラクティス](pagination-uri-and-sorting-best-practices.md)
- [CrudRepository メソッド](crud-repository-methods.md)

## 作成日
2026-02-17

## タグ
#spring #data #pageable #pagination #sort
