# CORS 設定 - WebMvcConfigurer

## 概要
`WebMvcConfigurer` を実装して Spring MVC の CORS（Cross-Origin Resource Sharing）設定を行うパターン。フロントエンド（例: Vite dev server の localhost:5173）から バックエンド API にアクセスできるようにする。

## 使用場面
- フロントエンド（React/Vue など）とバックエンド（Spring Boot）を別ポートで開発する時
- 特定のオリジン・メソッドだけ許可して CORS を制限したい時
- `@CrossOrigin` を各 Controller に書かずに、一元管理したい時

## コード

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addCorsMappings(@NonNull CorsRegistry registry) {
    registry.addMapping("/api/**")           // 対象エンドポイント
        .allowedOrigins("http://localhost:5173") // 許可するオリジン
        .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
        .allowedHeaders("*");                // 全ヘッダー許可
  }
}
```

## 説明

### CORS が必要な理由

ブラウザのセキュリティポリシー（Same-Origin Policy）により、異なるオリジン（ホスト・ポート・スキームが違う）へのリクエストはデフォルトでブロックされる。

```
フロントエンド: http://localhost:5173  (Vite)
バックエンド:   http://localhost:8080  (Spring Boot)
            ↑ ポートが違う → 別オリジン → CORS 設定が必要
```

### 設定パラメータ

| メソッド | 説明 | 例 |
|---|---|---|
| `addMapping` | 許可するエンドポイントのパターン | `"/api/**"` |
| `allowedOrigins` | 許可するオリジン | `"http://localhost:5173"` |
| `allowedMethods` | 許可する HTTP メソッド | `"GET", "POST"` など |
| `allowedHeaders` | 許可するリクエストヘッダー | `"*"` で全許可 |
| `allowCredentials` | Cookie / 認証情報の送信を許可 | `true`（JWT Cookie 使用時など）|

### 本番環境への対応

開発時は `localhost` を許可するが、本番ではデプロイ先の URL に変更する。
`@Value` や環境変数で切り替えるのが一般的。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Value("${cors.allowed-origins}")
  private String allowedOrigins;

  @Override
  public void addCorsMappings(@NonNull CorsRegistry registry) {
    registry.addMapping("/api/**")
        .allowedOrigins(allowedOrigins) // 環境変数から取得
        .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
        .allowedHeaders("*");
  }
}
```

```yaml
# application-dev.yml
cors:
  allowed-origins: http://localhost:5173

# application-prod.yml
cors:
  allowed-origins: https://your-app.example.com
```

### Controller ごとに設定する方法（小規模向け）

一元管理せず、Controller や各エンドポイントに直接設定することも可能。

```java
@RestController
@CrossOrigin(origins = "http://localhost:5173") // このコントローラーだけ許可
@RequestMapping("/api/todos")
public class TodoController { ... }
```

## 参考
- [Spring Web MVC - CORS](https://docs.spring.io/spring-framework/reference/web/webmvc-cors.html)
- [MDN - Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/ja/docs/Web/HTTP/CORS)

## 関連スニペット
- [REST Controller 基本パターン](../controller/rest-controller-basic.md)

## 作成日
2026-03-21

## タグ
#spring #cors #webmvcconfigurer #configuration #security #localhost
