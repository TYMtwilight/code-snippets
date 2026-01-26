# @Autowired - 依存性注入アノテーション

## 概要
Springコンテナが管理するBeanを自動的に注入するためのアノテーションで、依存性注入（DI）を実現する最も基本的な方法

## 使用場面
- ServiceクラスをControllerに注入する
- RepositoryクラスをServiceに注入する
- 他のBeanを必要とするクラスで依存関係を解決する
- コンストラクタ、フィールド、セッターメソッドでの依存性注入

## コード
```java
// 1. コンストラクタインジェクション（推奨）
@RestController
public class UserController {
    
    private final UserService userService;
    
    @Autowired  // Spring 4.3以降、コンストラクタが1つの場合は省略可能
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

// 2. フィールドインジェクション（簡潔だが非推奨）
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User findById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}

// 3. セッターインジェクション（オプショナルな依存関係に使用）
@Service
public class EmailService {
    
    private MailSender mailSender;
    
    @Autowired
    public void setMailSender(MailSender mailSender) {
        this.mailSender = mailSender;
    }
}

// 複数の依存関係を注入
@Service
public class OrderService {
    
    private final UserRepository userRepository;
    private final ProductRepository productRepository;
    private final EmailService emailService;
    
    @Autowired
    public OrderService(UserRepository userRepository,
                       ProductRepository productRepository,
                       EmailService emailService) {
        this.userRepository = userRepository;
        this.productRepository = productRepository;
        this.emailService = emailService;
    }
}

// required属性の使用（必須ではない依存関係）
@Service
public class NotificationService {
    
    @Autowired(required = false)
    private SmsService smsService;  // SMSサービスがなくても起動可能
    
    public void notify(String message) {
        if (smsService != null) {
            smsService.send(message);
        }
    }
}
```

## 説明
**3つの注入方法の比較**

| 方法 | メリット | デメリット | 推奨度 |
|------|---------|-----------|--------|
| コンストラクタ | イミュータブル、テスト容易、必須依存関係が明確 | コードがやや長い | ★★★ 最推奨 |
| フィールド | コードが簡潔 | テスト困難、イミュータブルにできない | ★ 非推奨 |
| セッター | オプショナルな依存関係に適している | 依存関係が不明確になりやすい | ★★ 限定的に使用 |

**コンストラクタインジェクションが推奨される理由**
1. **イミュータビリティ**: `final`フィールドにできるため、オブジェクトが不変になる
2. **テスト容易性**: Springコンテナなしでインスタンス化しやすい
3. **必須依存関係の明示**: コンストラクタ引数で依存関係が一目瞭然
4. **循環依存の検出**: コンパイル時または起動時に循環依存を発見できる

**@Autowiredの動作**
- Springコンテナ内で型に一致するBeanを探して注入
- 同じ型のBeanが複数ある場合は`@Qualifier`で特定
- Beanが見つからない場合はエラー（`required=false`で回避可能）

**Spring 4.3以降の省略ルール**
```java
// コンストラクタが1つだけの場合、@Autowiredは省略可能
@RestController
public class UserController {
    
    private final UserService userService;
    
    // @Autowired不要
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

## 参考
- [Spring Framework公式ドキュメント - Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-autowire.html)
- [Spring Boot公式ガイド](https://spring.io/guides/gs/spring-boot/)
- [Baeldung - @Autowired](https://www.baeldung.com/spring-autowire)

## 関連スニペット
- @Component, @Service, @Repository - Beanとして登録するアノテーション
- @Qualifier - 同じ型の複数Beanから特定のものを選択
- @Primary - デフォルトで注入されるBeanを指定
- 依存性注入（DI）の基本概念

## 作成日
2025-01-27

## タグ
#spring #autowired #di #dependency-injection #bean #constructor-injection