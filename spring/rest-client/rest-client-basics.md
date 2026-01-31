# RestClient - Spring 6のHTTPクライアント

## 概要
Spring 6で導入された新しいHTTPクライアント。REST APIの呼び出しを1行で書け、JSONをJavaオブジェクトに自動変換できる。

## 使用場面
- 外部のREST APIを呼び出す
- マイクロサービス間の通信
- JSONレスポンスをJavaオブジェクトとして扱う

## コード
```java
// RestClientの作成
RestClient restClient = RestClient.builder()
    .baseUrl("http://localhost:8080")
    .build();

// GETリクエスト（Quoteオブジェクトとして取得）
Quote quote = restClient
    .get()
    .uri("/api/random")
    .retrieve()
    .body(Quote.class);

// パスパラメータ付き
Quote quote = restClient
    .get()
    .uri("/api/{id}", 1)  // /api/1 になる
    .retrieve()
    .body(Quote.class);

// クエリパラメータ付き
String result = restClient
    .get()
    .uri(uriBuilder -> uriBuilder
        .path("/api/search")
        .queryParam("q", "spring")
        .build())
    .retrieve()
    .body(String.class);

// POSTリクエスト
User created = restClient
    .post()
    .uri("/api/users")
    .contentType(MediaType.APPLICATION_JSON)
    .body(new User("Taro", "taro@example.com"))
    .retrieve()
    .body(User.class);
```

## 説明
**チェーンの流れ**
```java
restClient
    .get()                    // HTTPメソッドを指定（準備）
    .uri("/api/random")       // エンドポイントを指定（準備）
    .retrieve()               // 実際にリクエストを送信（実行）
    .body(Quote.class);       // レスポンスボディをQuoteに変換
```

**RestClient.Builderによる設定**
```java
// Springが自動で渡してくれるBuilderを使用
@Bean
public ApplicationRunner run(RestClient.Builder builder) {
    RestClient restClient = builder
        .baseUrl("http://localhost:8080")      // ベースURL
        .defaultHeader("Authorization", "...")  // 共通ヘッダー
        .build();
    // ...
}
```

**RestClientの利点**
| 特徴 | 説明 |
|------|------|
| 流暢なAPI | メソッドチェーンで読みやすい |
| 自動変換 | JSONをJavaオブジェクトに自動マッピング |
| Spring統合 | RestClient.Builderが自動設定される |
| モダン | RestTemplateの後継として推奨 |

## 参考
- [Spring Framework - RestClient](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-restclient)
- [Spring Guide - Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest)

## 関連スニペット
- [RESTクライアント/サーバーの概念](./rest-client-server-concept.md)
- [RestClient retrieve()](./rest-client-retrieve.md)
- [Jackson JSONアノテーション](./jackson-annotations.md)

## 作成日
2025-01-31

## タグ
#spring #rest-client #http #api #json
