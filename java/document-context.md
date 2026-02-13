# DocumentContext - JsonPathによるJSON解析

## 概要
`DocumentContext`は、JsonPathライブラリが提供するJSON解析用のインターフェース。JSON文字列を解析し、JSONPath式で特定のフィールドに簡単にアクセスできる。

## 使用場面
- テストでHTTPレスポンスのJSON内容を検証するとき
- JSON文字列から特定のフィールドを抽出したいとき
- ネストされたJSONや配列の要素にアクセスしたいとき

## コード
```java
// JSON文字列をDocumentContextに変換
DocumentContext documentContext = JsonPath.parse(response.getBody());

// JSONPath式でフィールドにアクセス
Number id = documentContext.read("$.id");
String name = documentContext.read("$.name");
Double amount = documentContext.read("$.amount");
```

## 説明

### JSONPath式の基本
`$`記号から始まるパス式を使ってJSONの要素にアクセスする:

| 式 | 意味 |
|---|---|
| `$.id` | ルート直下の`id`フィールド |
| `$.user.name` | ネストされたオブジェクトの`name` |
| `$.items[0]` | 配列の最初の要素 |
| `$.tags[*]` | 配列の全要素 |

### 型安全な取得
`read()`メソッドは戻り値の型を自動的に推論する:
```java
Number id = documentContext.read("$.id");       // 数値
String name = documentContext.read("$.name");    // 文字列
Double amount = documentContext.read("$.amount"); // Double
List<String> tags = documentContext.read("$.tags[*]"); // リスト
```

### ネストされた構造へのアクセス
```java
String city = documentContext.read("$.address.city");
```

### 実践例（CashCardテスト）
```java
// レスポンスのJSON: {"id": 99, "amount": 123.45}
DocumentContext documentContext = JsonPath.parse(response.getBody());

Number id = documentContext.read("$.id");
assertThat(id).isEqualTo(99);

Double amount = documentContext.read("$.amount");
assertThat(amount).isEqualTo(123.45);
```

### なぜDocumentContextを使うのか
```java
// ❌ 手動パースは面倒で読みづらい
String json = response.getBody();
// 文字列操作で値を取り出す...

// ✅ DocumentContextならシンプルで読みやすい
DocumentContext doc = JsonPath.parse(response.getBody());
Number id = doc.read("$.id");
```

## 参考
- [JsonPath GitHub](https://github.com/json-path/JsonPath)
- [JsonPath式リファレンス](https://github.com/json-path/JsonPath#path-examples)

## 関連スニペット
- [TestRestTemplate](../spring/test/test-rest-template.md)
- [SpringBootTest GETエンドポイントテスト](../spring/test/springboottest-get-endpoint.md)
- [JSONテスト](../spring/test/json-test.md)

## 作成日
2026-02-13

## タグ
#java #jsonpath #json #testing #document-context
