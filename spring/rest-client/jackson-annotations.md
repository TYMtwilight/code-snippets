# Jackson JSONアノテーション - @JsonIgnoreProperties, @JsonProperty

## 概要
JSONとJavaオブジェクトのマッピングを制御するアノテーション。不明なプロパティの無視やフィールド名の変換に使用する。

## 使用場面
- 外部APIのJSONレスポンスをJavaオブジェクトにマッピングする
- JSONキーとJavaフィールド名が異なる場合に対応する
- APIの変更に対して堅牢なコードを書く

## コード
```java
// 返ってくるJSON
// {
//    "type": "success",
//    "value": {
//       "id": 10,
//       "quote": "Really loving Spring Boot"
//    }
// }

// 外側のオブジェクト
@JsonIgnoreProperties(ignoreUnknown = true)
public record Quote(String type, Value value) { }

// 内側のオブジェクト（valueの中身）
@JsonIgnoreProperties(ignoreUnknown = true)
public record Value(Long id, String quote) { }

// フィールド名をJSONキーと違う名前にしたい場合
public record Quote(
    @JsonProperty("type") String kind,   // JSONの"type" → Javaのkind
    @JsonProperty("value") Value data    // JSONの"value" → Javaのdata
) { }
```

## 説明
**@JsonIgnoreProperties(ignoreUnknown = true)**

「知らないプロパティがあっても無視してエラーにしない」

```java
// APIが将来こう変わっても...
{
   "type": "success",
   "value": {...},
   "newField": "追加された項目"  // ← 新しく追加された
}

// Javaクラスに newField がなくてもエラーにならない
@JsonIgnoreProperties(ignoreUnknown = true)
public record Quote(String type, Value value) { }
// newField は無視される → アプリは動き続ける
```

**変数名 = JSONのキー名（デフォルト）**
```
JSON:  "type": "success"
          ↓ 名前が一致
Java:  String type
```

**@JsonProperty**

JSONキーとJavaフィールド名が違う場合に使用
```java
@JsonProperty("type") String kind  // JSONの"type"をkindに入れる
```

**入れ子構造のマッピング**
```
JSON                          Java
─────────────────────────────────────
{
  "type": "success"    →    Quote.type
  "value": {           →    Quote.value (Value型)
    "id": 10           →      Value.id
    "quote": "..."     →      Value.quote
  }
}
```

## 参考
- [Jackson Annotations](https://github.com/FasterXML/jackson-annotations)
- [Baeldung - Jackson Annotations](https://www.baeldung.com/jackson-annotations)

## 関連スニペット
- [RestClient基本](./rest-client-basics.md)

## 作成日
2025-01-31

## タグ
#spring #jackson #json #annotations #serialization
