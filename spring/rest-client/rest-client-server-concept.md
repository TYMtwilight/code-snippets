# RESTクライアント/サーバーの概念

## 概要
RESTの文脈では、HTTPリクエストを送る側がクライアント、受ける側がサーバー。バックエンドサーバーも別のAPIを呼ぶときは「クライアント」になる。

## 使用場面
- 外部APIを呼び出すバックエンドアプリケーションを開発する
- マイクロサービス間の通信を理解する
- APIの設計・利用関係を整理する

## コード
```java
// あなたのJavaアプリが「RESTクライアント」として外部APIを呼ぶ
@Bean
public ApplicationRunner run(RestClient.Builder builder) {
    RestClient restClient = builder.baseUrl("http://localhost:8080").build();
    return args -> {
        // 外部のREST APIにリクエストを送る = クライアントとして動作
        Quote quote = restClient
            .get().uri("/api/random")
            .retrieve()
            .body(Quote.class);
    };
}
```

## 説明
**従来のWebクライアント/サーバーのイメージ**
```
[ブラウザ/React/Next.js]  →  [Java/PHP サーバー]  →  [DB]
      クライアント                 サーバー
```

**RESTクライアント/サーバーは「相対的な関係」**
```
[ブラウザ] → [あなたのJavaアプリ] → [外部API]
              (RESTクライアント)    (RESTサーバー)
```

**同じアプリが両方の役割を持つこともある**
- ブラウザから見ると → サーバー
- 外部APIから見ると → クライアント

| 用語 | 意味 |
|------|------|
| Webクライアント | 主にブラウザやフロントエンドアプリ |
| RESTクライアント | REST APIを**呼び出す側**（バックエンドでもOK） |
| RESTサーバー | REST APIを**提供する側** |

**ポイント**: 「クライアント＝ブラウザ」という固定観念を外すと理解しやすい

## 参考
- [Spring Guide - Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest)

## 関連スニペット
- [RestClient基本](./rest-client-basics.md)
- [RestClient retrieve()](./rest-client-retrieve.md)

## 作成日
2025-01-31

## タグ
#spring #rest #rest-client #rest-server #api #概念
