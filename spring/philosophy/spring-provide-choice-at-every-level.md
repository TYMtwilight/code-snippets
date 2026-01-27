# Spring設計思想：設定による実装切り替え

## 概要
Springの「Provide choice at every level」思想を実現する、設定ファイルだけでインフラ実装を切り替える方法。コードを変更せずに開発環境と本番環境で異なる技術スタックを使える。

## 使用場面
- 開発環境では軽量な技術、本番環境では高性能な技術を使い分けたい時
- プロジェクト初期は無料ツール、スケール後は有料サービスに移行したい時
- テスト環境と本番環境でインフラを分けたい時

## コード

### データベースの切り替え例
```java
// ビジネスロジック（環境に依存しない）
@Service
public class UserService {
    private final UserRepository repository;
    
    // コンストラクタインジェクション
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    public User saveUser(User user) {
        return repository.save(user);  // どのDBでも同じコード
    }
}
```
```yaml
# application-dev.yml（開発環境）
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver

# application-prod.yml（本番環境）
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/production
    driver-class-name: org.postgresql.Driver
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

### キャッシュの切り替え例
```java
// キャッシュを使いたいメソッド
@Service
public class ProductService {
    
    @Cacheable("products")
    public Product getProduct(Long id) {
        // 重い処理
        return productRepository.findByIdWithDetails(id);
    }
}
```
```yaml
# application-dev.yml
spring:
  cache:
    type: simple  # シンプルなメモリキャッシュ

# application-prod.yml
spring:
  cache:
    type: redis
    redis:
      host: redis.production.com
      port: 6379
```

### メール送信の切り替え例
```java
@Service
public class NotificationService {
    private final JavaMailSender mailSender;
    
    public NotificationService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }
    
    public void sendEmail(String to, String message) {
        SimpleMailMessage email = new SimpleMailMessage();
        email.setTo(to);
        email.setText(message);
        mailSender.send(email);
    }
}
```
```yaml
# application-dev.yml（開発：偽メールサーバー）
spring:
  mail:
    host: localhost
    port: 1025

# application-prod.yml（本番：SendGrid）
spring:
  mail:
    host: smtp.sendgrid.net
    port: 587
    username: apikey
    password: ${SENDGRID_API_KEY}
```

## 説明

### 核心原則
1. **ビジネスロジックはインターフェースに依存**：具象クラスではなく抽象（`UserRepository`, `JavaMailSender`）に依存する
2. **実装はSpringが注入**：DIコンテナが適切な実装を選んで注入してくれる
3. **どの実装を使うかは設定で決定**：`application.yml`や`@Profile`で環境ごとに切り替え

### メリット
- **段階的成長が可能**：小規模で始めて、必要に応じて高性能技術に移行
- **環境ごとに最適化**：開発は速度優先、本番は信頼性優先
- **テストが容易**：テスト用の軽量実装に簡単に切り替え可能
- **コスト管理**：初期は無料ツール、スケール後に有料サービスへ

### 実装の切り替えパターン
| レイヤー | 開発環境 | 本番環境 |
|---------|---------|---------|
| データベース | H2（インメモリ） | PostgreSQL（高性能） |
| キャッシュ | Simple（メモリ） | Redis（分散） |
| メール | ローカルSMTP | SendGrid（クラウド） |
| メッセージング | なし | RabbitMQ/Kafka |

### 注意点
- インターフェースに依存する設計が前提（疎結合）
- 環境変数やプロファイルの管理が重要
- 設定の誤りは実行時エラーになるため、起動時のログ確認が必須

## 参考
- [Spring Framework Overview - Design Philosophy](https://docs.spring.io/spring-framework/reference/overview.html#overview-philosophy)
- [Spring Boot - Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)
- [Spring - Profiles](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles)

## 関連スニペット
- [依存性注入（DI）の基本](./dependency-injection-basic.md)
- [Spring Profilesの使い方](./spring-profiles.md)
- [application.ymlの設定パターン](./application-yml-patterns.md)

## 作成日
2025-01-28

## タグ
#spring #design-philosophy #configuration #dependency-injection #profiles #infrastructure