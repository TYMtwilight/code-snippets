# @SpringBootTest - アノテーションとwebEnvironment

## 概要
`@SpringBootTest`はSpring Bootの**統合テスト**用アノテーション。テスト実行時にApplicationContextを起動し、本番に近い環境でテストできる。`webEnvironment`属性でWebサーバーの起動方法を制御する。

## 使用場面
- アプリケーション全体を起動した統合テストを書くとき
- 実際のHTTPリクエストでエンドポイントを検証したいとき
- `@Autowired`でBeanを注入してテストしたいとき

## 統合テストを自動化する意義

`@SpringBootTest`を用いた統合テストは、アプリケーション全体の結合状態を検証するだけでなく、リグレッションテストの基盤としても重要である。

従来、結合テストを手動で実施していた場合、修正のたびに同じ確認作業を繰り返す必要がある。しかし、統合テストをコード化しておけば、ビルド時やCI実行時に自動で再検証できる。

これにより以下の効果が得られる。

- 既存機能が壊れていないことを即座に確認できる
- 人的確認コストを削減できる
- 品質を担保したまま高速に改修できる
- 本番に近い構成での動作保証が可能になる


## コード
```java
// RANDOM_PORT：ランダムなポートで実サーバーを起動（最も一般的）
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class MyIntegrationTest {
    @Autowired
    TestRestTemplate restTemplate;  // 実HTTPリクエストを送信可能
}

// MOCK（デフォルト）：サーバーを起動しない
@SpringBootTest
class MyMockTest {
    @Autowired
    MockMvc mockMvc;  // モック環境でテスト
}

// DEFINED_PORT：設定ファイルのポートで起動
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
class MyDefinedPortTest { }

// NONE：Web環境なし
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
class MyNonWebTest { }
```

## 説明

### @SpringBootTestの役割
1. **ApplicationContextの起動** — 実際のSpring Bootアプリケーションと同様にBeanコンテナを起動する
2. **DIの有効化** — `@Autowired`によるBean注入が使えるようになる
3. **本番に近い環境** — コントローラ、サービス、リポジトリなど実際のBeanが組み合わさった状態でテストできる

### webEnvironment属性
テスト時にWebサーバーをどのように起動するかを指定するenum型の属性。

| 値 | 説明 | 使用するテストクライアント |
|---|---|---|
| `RANDOM_PORT` | ランダムなポートで実サーバーを起動 | `TestRestTemplate` |
| `DEFINED_PORT` | `application.properties`で定義したポートで起動 | `TestRestTemplate` |
| `MOCK`（デフォルト） | サーバーを起動せず、モック環境でテスト | `MockMvc` |
| `NONE` | Web環境なし | なし |

### RANDOM_PORTを選ぶ理由
- **ポート衝突を回避** — 空いているポートが自動で割り当てられる
- **本物のHTTP通信** — 実際にHTTPリクエスト/レスポンスが流れる
- **TestRestTemplate自動設定** — 起動したポートに自動接続してくれる

### スライステストとの違い
`@SpringBootTest`はアプリケーション**全体**を起動する。一方、スライステストは特定レイヤーのみ起動するため軽量。

| アノテーション | 対象レイヤー | 起動範囲 |
|---|---|---|
| `@SpringBootTest` | 全体 | 全てのBean |
| `@WebMvcTest` | コントローラ層 | Web関連のBeanのみ |
| `@JsonTest` | シリアル化/デシリアル化 | JSON関連のBeanのみ |
| `@DataJpaTest` | データ層 | JPA関連のBeanのみ |

## 参考
- [Spring Boot Testing - 公式ドキュメント](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- [Spring Academy - Test First](https://spring.academy/courses/building-a-rest-api-with-spring-boot/lessons/test-first)

## 関連スニペット
- [TestRestTemplate](./test-rest-template.md)
- [GETエンドポイントの統合テスト](./springboottest-get-endpoint.md)
- [JSONテスト](./json-test.md)
- [Spring IoCコンテナとBean](../philosophy/spring-ioc-container-and-bean%20.md)

## 作成日
2026-02-13

## タグ
#spring #spring-boot #test #springboottest #web-environment #integration-test
