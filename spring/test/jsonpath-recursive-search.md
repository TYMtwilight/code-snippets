# JsonPath 再帰的検索（..演算子）

## 概要
JsonPathの`..`演算子による再帰的検索（recursive search）の仕組みを解説する。JSON構造の全階層を深く掘り下げて、該当する要素をすべて探し出す機能。

## 使用場面
- JSON構造のどの階層にあるかわからないデータを取得したいとき
- ネストが深いJSONから特定のフィールドを一括取得したいとき
- テストコードでレスポンス構造の変更に強い検証を書きたいとき

## コード

### 通常の検索 vs 再帰的検索

```json
{
  "store": {
    "book": [
      {
        "id": 1,
        "title": "吾輩は猫である",
        "author": { "id": 10, "name": "夏目漱石" }
      },
      {
        "id": 2,
        "title": "坊っちゃん",
        "author": { "id": 10, "name": "夏目漱石" }
      }
    ],
    "music": {
      "cd": { "id": 100, "title": "ベスト盤" }
    }
  }
}
```

```java
// 通常の検索（特定のパスのみ）
List<Integer> bookIds = ctx.read("$.store.book[*].id");
// 結果: [1, 2]  ← bookのidだけ

// 再帰的検索（全階層からidを探す）
JSONArray allIds = ctx.read("$..id");
// 結果: [1, 10, 2, 10, 100]
//       book  author  book  author  music.cd
//       のid  のid    のid  のid    のid
```

### 視覚的なイメージ

通常の検索（`$.store.book[*].id`）：
```
store
 └─ book ← ここだけ探す
     ├─ [0].id ✓
     └─ [1].id ✓
```

再帰的検索（`$..id`）：
```
store
 ├─ book
 │   ├─ [0]
 │   │   ├─ id ✓ ← 見つけた！
 │   │   └─ author
 │   │       └─ id ✓ ← ここも見つけた！
 │   └─ [1]
 │       ├─ id ✓ ← ここも！
 │       └─ author
 │           └─ id ✓ ← ここも！
 └─ music
     └─ cd
         └─ id ✓ ← ここも見つけた！
```

### 実用例: 深い階層のデータ

```json
{
  "company": {
    "departments": [
      {
        "name": "営業部",
        "employees": [
          {"id": 1, "name": "田中"},
          {"id": 2, "name": "佐藤"}
        ]
      },
      {
        "name": "開発部",
        "employees": [
          {"id": 3, "name": "鈴木"},
          {"id": 4, "name": "高橋"}
        ]
      }
    ]
  }
}
```

```java
// 再帰的検索: 全従業員のIDを一度に取得
JSONArray employeeIds = ctx.read("$..id");
// [1, 2, 3, 4]

// 通常の方法だと長いパスが必要
JSONArray ids = ctx.read("$.company.departments[*].employees[*].id");
```

### テストコードでの使用

```java
@Test
void shouldReturnAllCashCardsWhenListIsRequested() {
    ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);

    DocumentContext documentContext = JsonPath.parse(response.getBody());

    // 再帰的検索で全てのidを取得
    // レスポンスの構造が変わっても動作する
    JSONArray ids = documentContext.read("$..id");
    assertThat(ids).containsExactlyInAnyOrder(99, 100, 101);
}
```

## 説明
- `..`演算子は、JSON構造の**すべての階層**を深く掘り下げて要素を探す
- 通常の検索は「この部屋の本棚から探す」、再帰的検索は「家中のすべての部屋・クローゼット・引き出しの中まで探す」イメージ
- テストコードでは、レスポンス構造の変更に強い柔軟な検証が書ける
- ただし意図しない階層の値も取得するため、対象を絞りたい場合は通常のパス指定を使う

### 補足: プログラミングにおける「再帰」

再帰（recursion）とは、ある処理の中で自分自身を繰り返し呼び出すこと。

```java
// 階乗を計算する再帰関数
public int factorial(int n) {
    if (n <= 1) return 1;              // 基底条件（終了条件）
    return n * factorial(n - 1);       // 自分自身を呼び出す
}
// factorial(5) → 5 * 4 * 3 * 2 * 1 = 120
```

JsonPathの`..`も同様に、「階層構造を再帰的に掘り下げて探索する」という意味で「再帰的検索」と呼ばれる。

## 参考
- [Jayway JsonPath GitHub](https://github.com/json-path/JsonPath)

## 関連スニペット
- [Jayway JsonPath - JSONデータ抽出ライブラリ](jayway-jsonpath.md)
- [Spring Boot テスト基礎](springboottest-annotation.md)
- [TestRestTemplate](test-rest-template.md)

## 作成日
2026-02-17

## タグ
#spring #test #jsonpath #recursion #json
