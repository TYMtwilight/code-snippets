# RestTemplate と HTTPクライアント

## 概要
HTTPクライアントとは、コードからHTTPリクエストを送信する仕組み。`RestTemplate`はSpringが提供するJava用HTTPクライアントで、外部APIの呼び出しに使用する。現在はメンテナンスモードであり、後継の`WebClient`や`RestClient`が推奨されている。

## 使用場面
- 外部のREST APIをJavaコードから呼び出すとき
- マイクロサービス間で通信するとき
- JSONレスポンスをJavaオブジェクトに変換して扱いたいとき

## コード
```java
RestTemplate restTemplate = new RestTemplate();

// GETリクエスト — データ取得
ResponseEntity<String> response = restTemplate.getForEntity(
    "https://api.example.com/users/1",
    String.class
);

// POSTリクエスト — データ送信
User newUser = new User("Taro", "taro@example.com");
ResponseEntity<User> created = restTemplate.postForEntity(
    "https://api.example.com/users",
    newUser,
    User.class
);
```

## 説明

### HTTPクライアントとは
HTTPリクエストを送信する側のプログラム。普段ブラウザがWebサイトにアクセスするとき、ブラウザがHTTPクライアントの役割を果たしている。

```
HTTPクライアント（リクエスト送信）  →  HTTPサーバー（レスポンス返却）
（ブラウザ、RestTemplate など）       （Spring Boot アプリなど）
```

### 身近なHTTPクライアントの例
| ツール/ライブラリ | 環境 |
|---|---|
| ブラウザ | 日常 |
| curl | コマンドライン |
| Postman | API開発 |
| RestTemplate | Java / Spring |
| fetch / axios | JavaScript |

### RestTemplateの主なメソッド
| メソッド | HTTPメソッド | 用途 |
|---|---|---|
| `getForEntity()` | GET | リソースの取得 |
| `getForObject()` | GET | リソースの取得（ボディのみ） |
| `postForEntity()` | POST | リソースの作成 |
| `put()` | PUT | リソースの更新 |
| `delete()` | DELETE | リソースの削除 |

### Spring HTTPクライアントの世代比較
| | RestTemplate | WebClient | RestClient |
|---|---|---|---|
| 導入時期 | Spring 3 | Spring 5 | Spring 6 |
| 処理方式 | 同期（ブロッキング） | 同期・非同期どちらも可 | 同期（ブロッキング） |
| 状態 | **メンテナンスモード** | 現役（リアクティブ向け） | **現在の推奨** |
| API設計 | 従来型 | リアクティブ | 流暢なメソッドチェーン |
| 学習コスト | 低い | やや高い | 低い |

RestTemplateは新機能追加はないが、シンプルな用途やテスト（TestRestTemplate）では今でも広く使われている。

## 参考
- [Spring Framework - REST Clients](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)
- [Spring Framework - RestTemplate](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-resttemplate)

## 関連スニペット
- [RestClient の基本](./rest-client-basics.md)
- [RESTクライアント/サーバーの概念](./rest-client-server-concept.md)
- [ResponseEntity](./response-entity.md)
- [TestRestTemplate](../test/test-rest-template.md)

## 作成日
2026-02-13

## タグ
#spring #rest-template #http-client #rest-api #web-client
