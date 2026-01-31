# RestClient retrieve() - HTTPリクエストの実行

## 概要
RestClientのチェーンで「実際にHTTPリクエストを送信して、レスポンスを取得する」メソッド。設定を確定してリクエストを実行するトリガー。

## 使用場面
- REST APIからデータを取得する
- RestClientのリクエスト設定を確定して送信する

## コード
```java
// 基本的な使い方
Quote quote = restClient
    .get()                    // GETリクエストを「準備」
    .uri("/api/random")       // URLを「設定」
    .retrieve()               // ← ここで実際に通信する！
    .body(Quote.class);       // ボディをQuoteに変換

// 文字列として取得
String json = restClient
    .get()
    .uri("/api/random")
    .retrieve()
    .body(String.class);

// エンティティ全体を取得（ヘッダー、ステータスコード含む）
ResponseEntity<Quote> response = restClient
    .get()
    .uri("/api/random")
    .retrieve()
    .toEntity(Quote.class);

int status = response.getStatusCode().value();  // 200
HttpHeaders headers = response.getHeaders();
Quote quote = response.getBody();
```

## 説明
**チェーンの役割**
```
.get()        → 「GETで」（準備）
.uri(...)     → 「このURLに」（準備）
.retrieve()   → 「アクセスして取ってくる」（実行！）←ここ
.body(...)    → 「中身をこの型で」（変換）
```

**retrieve() がないとどうなるか**
```java
// ①②だけでは、まだ何も起きていない（設定しただけ）
restClient.get().uri("/api/random")
// ↑ まだリクエストは送られていない

// ③ retrieve() で初めてリクエストが送られる
restClient.get().uri("/api/random").retrieve()
// ↑ ここで通信発生！
```

**retrieve() の後の選択肢**

| メソッド | 取得するもの |
|----------|------------|
| `.body(Quote.class)` | レスポンスボディをQuoteに変換 |
| `.body(String.class)` | レスポンスボディを文字列で取得 |
| `.toEntity(Quote.class)` | ヘッダーやステータスコードも含めて取得 |

**英語の意味**
| 英語 | 意味 |
|------|------|
| retrieve | 取ってくる、回収する |

```
         retrieve()
サーバー ←────────── リクエスト送信
         ─────────→ レスポンス受信
```

## 参考
- [Spring Framework - RestClient](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-restclient)

## 関連スニペット
- [RestClient基本](./rest-client-basics.md)
- [RESTクライアント/サーバーの概念](./rest-client-server-concept.md)

## 作成日
2025-01-31

## タグ
#spring #rest-client #http #retrieve #api
