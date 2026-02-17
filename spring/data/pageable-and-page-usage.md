# Pageable / PageRequest / Page - ページネーションとソート

## 概要
- **`Pageable`**: 「何ページ目を」「1ページ何件で」「どうソートして」取得するか、という **指定（仕様）** を表す *インターフェース*。
- **`PageRequest`**: `Pageable` の代表的な **実装クラス**。`PageRequest.of(...)` で具体的な `Pageable` インスタンスを作る。
- **`Page`**: `Pageable` の指定に従って取得された **結果**（データ本体 + メタ情報）を表す *インターフェース*。

関係性はこうです：

```

PageRequest (implements Pageable)  ---> Repository に渡す（要求）
---> Repository がDBから取得
Repository の戻り値が Page<T>      ---> 取得結果 + メタ情報

````

## 使用場面
- 大量データを一括取得するとメモリ不足やレスポンス遅延が発生する場合
- 一覧画面でページ送りやソート機能を実装する場合
- REST APIでページネーション付きのレスポンスを返す場合

---

## コード

### 問題: 大量データの一括取得
```java
// 10万件のデータを一度に取得 → メモリ不足、レスポンス遅延
List<CashCard> allCards = cashCardRepository.findAll();
````

### 解決: PageRequest（= Pageableの実体）でページ分割

```java
// PageRequest は Pageable の実装。
// 「0ページ目（1ページ目）を、20件で取得して」という要求を作るだけで、ここでは取得しない。
Pageable pageable = PageRequest.of(0, 20);

// 実際に取得するのは repository 側
Page<CashCard> page = cashCardRepository.findAll(pageable);
```

---

## Repository定義（重要な修正）

`findAll(Pageable)` は **`PagingAndSortingRepository`（または `JpaRepository`）** が提供する機能です。
そのため、`CrudRepository` のままだと `findAll(Pageable)` が使えません。

### 推奨: JpaRepository を継承（実務で一般的）

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CashCardRepository extends JpaRepository<CashCard, Long> {

    // JpaRepository には findAll(Pageable) が既に定義されているので、宣言は必須ではない
    // Page<CashCard> findAll(Pageable pageable);

    // カスタム検索 + Pageable
    Page<CashCard> findByOwner(String owner, Pageable pageable);
}
```

---

## PageRequest の作成方法（Pageable の具体化）

```java
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

// 基本形: ページ番号とサイズを指定（0始まり）
Pageable pageable = PageRequest.of(0, 20);

// ソート付き（昇順）
Pageable pageable2 = PageRequest.of(0, 20, Sort.by("amount").ascending());

// ソート付き（降順）
Pageable pageable3 = PageRequest.of(0, 20, Sort.by("amount").descending());

// 複数フィールドでソート
Pageable pageable4 = PageRequest.of(0, 20,
    Sort.by("amount").descending()
        .and(Sort.by("id").ascending())
);

// 短縮形
Pageable pageable5 = PageRequest.of(0, 20, Sort.Direction.DESC, "amount");
```

> `PageRequest.of(...)` は **PageRequest を作る**が、変数型を `Pageable` にしてもOK。
> 理由: `PageRequest implements Pageable` なので、`Pageable` として扱える。

---

## Controllerでの利用（Springが自動で Pageable を生成）

コントローラ引数に `Pageable` を置くと、Springがクエリパラメータから `Pageable` の実体（内部的には `PageRequest` など）を自動生成します。

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/cashcards")
public class CashCardController {

    private final CashCardRepository repository;

    public CashCardController(CashCardRepository repository) {
        this.repository = repository;
    }

    // GET /cashcards?page=0&size=10&sort=amount,desc
    @GetMapping
    public ResponseEntity<Page<CashCard>> findAll(Pageable pageable) {
        Page<CashCard> page = repository.findAll(pageable);
        return ResponseEntity.ok(page);
    }

    // GET /cashcards/by-owner?owner=sarah&page=0&size=10&sort=amount,desc
    @GetMapping("/by-owner")
    public ResponseEntity<Page<CashCard>> findByOwner(
            @RequestParam String owner,
            Pageable pageable
    ) {
        Page<CashCard> page = repository.findByOwner(owner, pageable);
        return ResponseEntity.ok(page);
    }
}
```

**リクエスト例:**

```
GET /cashcards?page=0&size=10
GET /cashcards?page=1&size=20&sort=amount,desc
GET /cashcards?page=0&size=10&sort=amount,desc&sort=id,asc
GET /cashcards/by-owner?owner=sarah&page=0&size=10&sort=amount,desc
```

---

## Page オブジェクトから取得できる情報

```java
Page<CashCard> page = repository.findAll(PageRequest.of(0, 20));

// データ取得
List<CashCard> content = page.getContent();        // 実データ

// ページ情報
int totalPages = page.getTotalPages();              // 総ページ数
long totalElements = page.getTotalElements();       // 総件数
int number = page.getNumber();                      // 現在ページ番号（0始まり）
int size = page.getSize();                          // ページサイズ
int numberOfElements = page.getNumberOfElements();  // このページに入っている件数

// ページ状態
boolean hasNext = page.hasNext();
boolean hasPrevious = page.hasPrevious();
boolean isFirst = page.isFirst();
boolean isLast = page.isLast();
```

---

## レスポンス例（概念）

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

---

## 説明（押さえどころ）

* ページ番号は **0始まり**（`page=0` が1ページ目）
* `Pageable` はインターフェースなので **単体では作れない**

  * 手で作るなら `PageRequest.of(...)`（= `Pageable` の具体インスタンス）
  * Controller引数なら Spring が自動生成（内部的には `PageRequest` 等）
* `Page` は **結果**（`content` + 総件数/総ページ数などのメタ情報）
* 複数ソートは `sort` パラメータを複数指定する
  例: `sort=amount,desc&sort=id,asc`

### ページ番号の考え方

```
データが100件、1ページ20件の場合:
page=0 → 1～20件目    page=1 → 21～40件目
page=2 → 41～60件目   page=3 → 61～80件目
page=4 → 81～100件目  totalPages = 5
```

---

## application.propertiesでのカスタマイズ

```properties
spring.data.web.pageable.default-page-size=20
spring.data.web.pageable.max-page-size=100
spring.data.web.pageable.one-indexed-parameters=true  # 1始まりに変更（page=1 が1ページ目）
```

---

## 参考

* Spring Data - Paging and Sorting

  * [https://docs.spring.io/spring-data/commons/reference/repositories/query-methods-details.html#repositories.special-parameters](https://docs.spring.io/spring-data/commons/reference/repositories/query-methods-details.html#repositories.special-parameters)

## 作成日

2026-02-17

## タグ

#spring #data #pageable #pagerequest #page #pagination #sort
