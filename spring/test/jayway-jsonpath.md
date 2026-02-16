# Jayway JsonPath - JSONデータ抽出ライブラリ

## 概要
Jayway JsonPathは、JSON構造から特定のデータを抽出するためのJavaライブラリ。XPathのJSON版にあたり、Spring BootテストでAPIレスポンスを検証する際に活用される。

## 使用場面
- REST APIのJSONレスポンスをテストコードで検証するとき
- 複雑なJSON構造から特定のデータを抽出したいとき
- フィルタ条件を使ってJSONデータを絞り込みたいとき

## コード

### 基本的な使い方

```java
// JSONのパース
String json = "{\"name\":\"太郎\", \"age\":30}";
DocumentContext ctx = JsonPath.parse(json);

// データの読み取り
String name = ctx.read("$.name");  // "太郎"
Integer age = ctx.read("$.age");   // 30
```

### JsonPath記法一覧

| 記法 | 意味 | 例 |
|------|------|-----|
| `$` | ルート要素 | `$.name` |
| `.` | 子要素へのアクセス | `$.user.name` |
| `[]` | 配列要素へのアクセス | `$.items[0]` |
| `..` | 再帰的検索（深い階層も含む） | `$..id` |
| `*` | ワイルドカード | `$.items[*].name` |
| `[start:end]` | 配列のスライス | `$.items[0:2]` |

### パス指定の具体例

```json
{
  "store": {
    "book": [
      {"title": "吾輩は猫である", "price": 500, "id": 1},
      {"title": "坊っちゃん", "price": 400, "id": 2},
      {"title": "こころ", "price": 600, "id": 3}
    ],
    "bicycle": {
      "color": "red",
      "price": 19950
    }
  }
}
```

```java
// 基本的なパス
String title = ctx.read("$.store.book[0].title");  // "吾輩は猫である"

// 全ての本のタイトルを取得
List<String> titles = ctx.read("$.store.book[*].title");
// ["吾輩は猫である", "坊っちゃん", "こころ"]

// 再帰的検索 - 全てのpriceを取得
List<Integer> prices = ctx.read("$..price");
// [500, 400, 600, 19950]

// 配列の長さ
int bookCount = ctx.read("$.store.book.length()");  // 3

// 配列のスライス
List<Object> firstTwo = ctx.read("$.store.book[0:2]");
```

### フィルタ式

```java
// 価格が500円以上の本
List<Map<String, Object>> expensiveBooks =
    ctx.read("$.store.book[?(@.price >= 500)]");

// idが2の本
List<Map<String, Object>> book =
    ctx.read("$.store.book[?(@.id == 2)]");
```

#### フィルタ演算子

| 演算子 | 意味 |
|--------|------|
| `@` | 現在のノード |
| `==` | 等しい |
| `!=` | 等しくない |
| `<`, `<=`, `>`, `>=` | 比較 |
| `=~` | 正規表現マッチ |
| `&&` | AND |
| `\|\|` | OR |

### 返り値の型指定

```java
// 自動型推論
Number id = ctx.read("$.id");
Double amount = ctx.read("$.amount");

// 型を明示的に指定
String name = ctx.read("$.name", String.class);
List<String> items = ctx.read("$.items", new TypeRef<List<String>>(){});
```

### テストコードでの実践例

```java
@Test
void jsonPathExample() {
    String json = """
        {
            "cashCards": [
                {"id": 99, "amount": 123.45},
                {"id": 100, "amount": 1.00},
                {"id": 101, "amount": 150.00}
            ]
        }
        """;

    DocumentContext ctx = JsonPath.parse(json);

    // 配列の要素数
    int count = ctx.read("$.cashCards.length()");
    assertThat(count).isEqualTo(3);

    // 全てのIDを取得
    JSONArray ids = ctx.read("$.cashCards[*].id");
    assertThat(ids).containsExactlyInAnyOrder(99, 100, 101);

    // 再帰的に全てのamountを取得
    JSONArray amounts = ctx.read("$..amount");
    assertThat(amounts).containsExactlyInAnyOrder(123.45, 1.00, 150.00);

    // 金額が100以上のカード
    List<Map<String, Object>> highValue =
        ctx.read("$.cashCards[?(@.amount >= 100)]");
    assertThat(highValue).hasSize(2);
}
```

### データの更新

```java
DocumentContext ctx = JsonPath.parse(json);
ctx.set("$.name", "次郎");  // 値を更新
ctx.delete("$.age");         // フィールドを削除
String updatedJson = ctx.jsonString();
```

### 設定のカスタマイズ

```java
Configuration conf = Configuration.builder()
    .options(Option.DEFAULT_PATH_LEAF_TO_NULL)  // 存在しないパスはnullを返す
    .build();

DocumentContext ctx = JsonPath.using(conf).parse(json);
```

## 説明
- `$`がルート、`.`で階層を辿り、`[*]`で全要素、`..`で再帰的検索を行う
- フィルタ式`[?(@.field == value)]`で条件付き抽出が可能
- `read()`の戻り値は自動型推論されるが、`TypeRef`で明示指定もできる
- `set()` / `delete()`でJSON内のデータ更新も可能

## 参考
- [Jayway JsonPath GitHub](https://github.com/json-path/JsonPath)

## 関連スニペット
- [JsonPath 再帰的検索（..演算子）](jsonpath-recursive-search.md)
- [Spring Boot テスト基礎](springboottest-annotation.md)
- [TestRestTemplate](test-rest-template.md)

## 作成日
2026-02-17

## タグ
#spring #test #jsonpath #json #library
