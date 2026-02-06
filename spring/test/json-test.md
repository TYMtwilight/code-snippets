# @JsonTest - JSONシリアル化/デシリアル化のスライステスト

## 概要
`@JsonTest`はSpring BootのスライステストアノテーションでJSON変換ロジックだけを軽量にテストできる。Jacksonフレームワークの設定とJSONテスト/パース機能を自動で提供する。

## 使用場面
- JavaオブジェクトがJSON契約通りにシリアル化されるか検証する時
- JSONが正しくJavaオブジェクトにデシリアル化されるか検証する時
- `@SpringBootTest`を使わず高速にJSON変換だけをテストしたい時

## コード

### 基本構成
```java
@JsonTest  // JSON関連のBeanだけをロード（軽量なスライステスト）
class CashCardJsonTest {

    @Autowired  // SpringがJacksonTesterインスタンスを自動生成・注入
    private JacksonTester<CashCard> json;

    @Test
    void shouldSerialize() throws IOException {
        CashCard card = new CashCard(99L, 123.45);

        // シリアル化: Java → JSON
        assertThat(json.write(card)).isStrictlyEqualToJson("expected.json");
        assertThat(json.write(card)).hasJsonPathNumberValue("@.id");
        assertThat(json.write(card)).extractingJsonPathNumberValue("@.id").isEqualTo(99);
        assertThat(json.write(card)).extractingJsonPathNumberValue("@.amount").isEqualTo(123.45);
    }

    @Test
    void shouldDeserialize() throws IOException {
        String expected = """
                {"id": 99, "amount": 123.45}
                """;

        // デシリアル化: JSON → Java
        assertThat(json.parseObject(expected).id()).isEqualTo(99L);
        assertThat(json.parseObject(expected).amount()).isEqualTo(123.45);
    }
}
```

### expected.json（テストリソース）
```json
{"id": 99, "amount": 123.45}
```

## 説明

### 3つの構成要素の役割

| 要素 | 役割 |
|------|------|
| **`@JsonTest`** | JSON関連Beanだけをロードするスライステスト環境を構築 |
| **`JacksonTester<T>`** | Jackson JSONライブラリのラッパー。`write()`でシリアル化、`parseObject()`でデシリアル化 |
| **`@Autowired`** | SpringのDI（依存性注入）で`JacksonTester`インスタンスを自動生成・フィールドに注入 |

### @JsonTestがロードするもの
- `JacksonTester` / `GsonTester` / `BasicJsonTester`
- `ObjectMapper`などのJackson設定
- カスタム`@JsonComponent`（定義している場合）

### @SpringBootTestとの違い

| 項目 | `@JsonTest` | `@SpringBootTest` |
|------|-------------|-------------------|
| ロード範囲 | JSON関連Beanのみ | アプリケーション全体 |
| 起動速度 | 高速 | 低速 |
| テスト対象 | シリアル化/デシリアル化 | 統合テスト全般 |
| HTTPリクエスト | 不可 | 可能 |

## 参考
- [Spring Boot - Auto-configured JSON Tests](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html#testing.spring-boot-applications.json-tests)
- [JacksonTester API](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/json/JacksonTester.html)

## 関連スニペット
- [シリアル化とデシリアル化](../serialization-deserialization.md)
- [API契約（API Contract）とJSON](../api-contract.md)
- [テスト駆動開発](./testing-first.md)

## 作成日
2026-02-06

## タグ
#spring #test #json #jackson #jsontest #slice-test #serialization #deserialization
