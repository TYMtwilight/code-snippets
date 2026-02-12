# TestRestTemplate - テスト用HTTPクライアント

## 概要
テストコードから実際のHTTPリクエストを送信し、レスポンスを安全に検証するためのテスト専用HTTPクライアント。`@SpringBootTest(webEnvironment = RANDOM_PORT)` と組み合わせて使用する。

## 使用場面
- `@SpringBootTest`でエンドポイントの統合テストを書くとき
- HTTPステータスコード（404、500など）を含むエラーレスポンスを検証したいとき
- 実際のHTTP通信を伴うテストが必要なとき

## コード
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CashCardApplicationTests {
    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void shouldReturnACashCard() {
        // GETリクエスト
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }

    @Test
    void shouldReturn404ForUnknownId() {
        // 404でも例外にならず、ステータスコードを検証できる
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99999", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }
}
```

## 説明

### 主なメソッド
| メソッド | HTTPメソッド | 用途 |
|---|---|---|
| `getForEntity()` | GET | リソースの取得 |
| `postForEntity()` | POST | リソースの作成 |
| `put()` | PUT | リソースの更新 |
| `delete()` | DELETE | リソースの削除 |

### RestTemplateとの違い
| | RestTemplate | TestRestTemplate |
|---|---|---|
| 用途 | 本番コード | テストコード専用 |
| エラー時の動作 | **例外をスローする** | **例外をスローしない** |
| ベースURL | 自分で設定 | 自動で `localhost:ランダムポート` が設定される |

最も重要な違いは**例外をスローしない**点。RestTemplateは4xx/5xxレスポンスで例外を投げるが、TestRestTemplateはそのままレスポンスを返すため、ステータスコードをアサーションで検証できる。

### 動作の仕組み
1. `@SpringBootTest(RANDOM_PORT)` がランダムなポートでサーバーを起動
2. SpringがTestRestTemplateに起動したポートのベースURLを自動設定
3. テストコードから相対パス（`"/cashcards/99"`）でリクエストを送信
4. TestRestTemplateが `http://localhost:{ランダムポート}/cashcards/99` に変換して送信

## 参考
- [Spring Boot - TestRestTemplate 公式ドキュメント](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/web/client/TestRestTemplate.html)
- [Spring Academy - Test First](https://spring.academy/courses/building-a-rest-api-with-spring-boot/lessons/test-first)

## 関連スニペット
- [@SpringBootTest アノテーション](./springboottest-annotation.md)
- [GETエンドポイントの統合テスト](./springboottest-get-endpoint.md)
- [ResponseEntity](../rest-client/response-entity.md)
- [RestTemplateとHTTPクライアント](../rest-client/rest-template-http-client.md)

## 作成日
2026-02-13

## タグ
#spring #spring-boot #test #test-rest-template #http-client #integration-test
