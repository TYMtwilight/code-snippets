# Spring IoC Container と Bean の基礎

## 概要
Spring IoCコンテナ（ApplicationContext）がBeanのライフサイクル（生成・依存注入・管理）を担当する仕組み。開発者はビジネスロジックに集中でき、オブジェクト管理はSpringに任せられる。

## 使用場面
- Springアプリケーションの全ての場面（基礎中の基礎）
- サービス層、リポジトリ層、コントローラー層でのオブジェクト管理
- テスト時にモックを注入して依存を差し替えたい時
- 設定ファイルだけで実装を切り替えたい時（開発環境⇔本番環境）

## コード

### Bean定義の基本パターン
```java
// ❌ 従来の方法（DIなし）：自分で依存を管理
public class UserService {
    private UserRepository repository;
    
    public UserService() {
        // 自分でnewする（強く結合、テストしにくい）
        this.repository = new UserRepositoryImpl();
    }
}

// ✅ Springの方法（DI）：Springに依存管理を任せる
@Service  // ← これがBean定義のメタデータ
public class UserService {
    private final UserRepository repository;
    
    // コンストラクタインジェクション（推奨）
    public UserService(UserRepository repository) {
        this.repository = repository;  // Springが注入してくれる
    }
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Springが実装を自動生成してくれる
}
```

### 依存注入の3つの方法
```java
@Service
public class OrderService {
    
    // 方法1: コンストラクタインジェクション（最推奨）
    private final OrderRepository repository;
    private final PaymentService paymentService;
    
    public OrderService(OrderRepository repository, 
                       PaymentService paymentService) {
        this.repository = repository;
        this.paymentService = paymentService;
    }
    
    // 方法2: Setterインジェクション（オプショナルな依存に使う）
    private EmailService emailService;
    
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    // 方法3: フィールドインジェクション（非推奨：テストしにくい）
    // @Autowired
    // private LoggingService loggingService;
}
```

### テストでの活用
```java
// Beanはテスト時にモックへ差し替え可能
@Test
void testFindUser() {
    // モック（偽物）を作る
    UserRepository mockRepo = mock(UserRepository.class);
    when(mockRepo.findById(1L)).thenReturn(new User(1L, "Test User"));
    
    // モックを注入（本物のDBは不要）
    UserService service = new UserService(mockRepo);
    
    // テスト実行
    User user = service.findUser(1L);
    assertEquals("Test User", user.getName());
    
    // メリット：高速、安定、外部リソース不要
}
```

### ApplicationContextの自動起動
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        // ApplicationContextが自動で起動
        // - 全てのBeanを探す（@Service, @Repository等）
        // - 依存関係を解析
        // - Beanを作成
        // - 依存を注入
        SpringApplication.run(MyApplication.class, args);
    }
}
```

## 説明

### IoC（Inversion of Control：制御の反転）とは
- **従来**：オブジェクトが自分で依存を`new`して制御
- **Spring**：コンテナが依存を作成・注入して制御
- 制御権が「オブジェクト → コンテナ」に**反転**する

### DI（Dependency Injection：依存性注入）とは
- IoCを実現する具体的な手法
- オブジェクトは「何が必要か」を宣言するだけ（コンストラクタ引数等）
- 実際に作って渡すのはSpringコンテナの仕事

### Beanとは
- **定義**：Spring IoCコンテナが管理するオブジェクト
- **特徴**：
  1. Springが生成（自分で`new`しない）
  2. Springが依存を注入（アセンブル）
  3. Springがライフサイクルを管理
- **Beanにするもの**：`@Service`, `@Repository`, `@Controller`, `@Component`
- **Beanにしないもの**：エンティティ（`@Entity`）、DTO、値オブジェクト

### BeanFactory vs ApplicationContext
| 項目 | BeanFactory | ApplicationContext |
|------|-------------|-------------------|
| Bean管理 | ✅ 基本機能のみ | ✅ 基本機能 |
| AOP統合 | ❌ | ✅ |
| 国際化（i18n） | ❌ | ✅ |
| イベント機能 | ❌ | ✅ |
| Web機能 | ❌ | ✅（WebApplicationContext） |
| 実務での使用 | ほぼ使わない | **これを使う** |

**Spring Bootでは自動でApplicationContextが起動するので、普段は意識不要**

### コンストラクタインジェクションを推奨する理由
1. **テストしやすい**：`new UserService(mockRepo)`でモック注入可能
2. **不変性を保証**：`final`修飾子が使える
3. **依存が明確**：コンストラクタを見れば全ての依存が分かる
4. **依存過多に気づける**：パラメータが多すぎると設計を見直すサイン

### メタデータ（設定情報）
Springに「これはBeanです」「この依存が必要です」と教える情報

**3つの方法**：
1. アノテーション（主流）：`@Service`, `@Autowired`等
2. JavaConfig：`@Configuration` + `@Bean`
3. XML（レガシー）：`<bean>`タグ

## 参考
- [Spring Framework Reference - IoC Container](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
- [Spring Framework Overview - Design Philosophy](https://docs.spring.io/spring-framework/reference/overview.html#overview-philosophy)
- [Spring Boot Reference - Dependency Injection](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.spring-beans-and-dependency-injection)

## 関連スニペット
- [Spring設計思想：設定による実装切り替え](./spring-provide-choice-at-every-level.md)
- [依存性注入（DI）のベストプラクティス](./dependency-injection-best-practices.md)
- [Spring Beanのスコープとライフサイクル](./spring-bean-scopes.md)

## 作成日
2025-01-28

## タグ
#spring #ioc #di #dependency-injection #bean #application-context #core-concepts