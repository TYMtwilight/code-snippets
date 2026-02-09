# @SpringBootTest - GETエンドポイントの統合テスト

## 概要
`@SpringBootTest`と`TestRestTemplate`を使い、GETエンドポイントに対する統合テストをテストファーストで書く方法。アプリケーション全体を起動し、実際のHTTPリクエストでレスポンスのステータスコードを検証する。

## 使用場面
- REST APIのGETエンドポイントをTDD（Red-Green-Refactor）で実装するとき
- エンドポイントが正しいHTTPステータスコードを返すことを検証したいとき
- Spring Bootアプリケーション全体を起動した統合テストを書くとき

## コード
```java
package example.cashcard;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CashCardApplicationTests {
    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void shouldReturnACashCardWhenDataIsSaved() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```

## 説明

### 主要なアノテーション・クラス

| 要素 | 役割 |
|------|------|
| `@SpringBootTest(webEnvironment = RANDOM_PORT)` | Spring Bootアプリを起動し、ランダムなポートでHTTPリクエストを受け付ける。ポート競合を回避できる |
| `@Autowired TestRestTemplate` | Spring DIによりテスト用HTTPクライアントが注入される。ローカルで起動したアプリに対してリクエストを送信できる |
| `restTemplate.getForEntity(url, type)` | 指定URLにHTTP GETリクエストを送り、`ResponseEntity`として結果を取得する |
| `ResponseEntity` | HTTPレスポンスのステータスコード、ヘッダー、ボディなどの情報を保持するSpringのオブジェクト |

### テストファーストの流れ

1. **Red（失敗）**: エンドポイント未実装の状態でテストを実行 → `404 NOT_FOUND`で失敗する
2. **Green（成功）**: エンドポイントを実装してテストをパスさせる
3. **Refactor（改善）**: コードを整理・改善する

未実装のエンドポイントにリクエストすると、Spring Webが自動的に`404 NOT_FOUND`を返す。これはフレームワークの正常な動作であり、エンドポイントのマッピングが存在しないことを意味する。

### `@Autowired`の注意点
`@Autowired`によるフィールドインジェクションは**テストコードでのみ**使用するのが推奨。本番コードではコンストラクタインジェクションを使用する。

### テスト実行
```bash
./gradlew test
```

## 参考
- [Spring Academy - Test First](https://spring.academy/courses/building-a-rest-api-with-spring-boot/lessons/test-first)
- [Spring Boot Testing - 公式ドキュメント](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)

## 関連スニペット
- [テスト駆動開発の概念](./testing-first.md)
- [JSON シリアル化/デシリアル化テスト](./json-test.md)
- [REST Controller](../rest-controller.md)
- [GETリクエストと@PathVariable](../controller/)

## 作成日
2025-02-09

## タグ
#spring #spring-boot #test #springboottest #tdd #rest-api #get #TestRestTemplate
