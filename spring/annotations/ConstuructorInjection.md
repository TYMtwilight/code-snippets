# コンストラクタインジェクションとBean生成順序

## 概要
Springコンテナが依存関係を解析してBeanを生成し、コンストラクタを通じて依存するBeanを注入する仕組みと、その生成順序を理解する

## 使用場面
- Controller → Service → Repository の依存関係を持つ多層アーキテクチャの構築
- Beanの初期化順序を理解する必要がある場合
- 循環依存の問題をデバッグする場合
- Spring起動時のBean生成フローを把握したい場合

## コード
```java
// 1. 依存関係なし - 最初に生成される
@Repository
public class UserRepository {
    
    public UserRepository() {
        System.out.println("1. UserRepository のインスタンスが生成されました");
    }
    
    public List<String> findAllUsers() {
        return List.of("田中太郎", "佐藤花子", "鈴木一郎");
    }
}

// 2. UserRepositoryに依存 - 次に生成される
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    // コンストラクタインジェクション（@Autowired省略可能）
    public UserService(UserRepository userRepository) {
        System.out.println("2. UserService のインスタンスが生成されました");
        System.out.println("   → UserRepository が注入されました");
        this.userRepository = userRepository;
    }
    
    public List<String> getAllUsers() {
        return userRepository.findAllUsers();
    }
}

// 3. UserServiceに依存 - 最後に生成される
@RestController
public class UserController {
    
    private final UserService userService;
    
    // コンストラクタインジェクション（@Autowired省略可能）
    public UserController(UserService userService) {
        System.out.println("3. UserController のインスタンスが生成されました");
        System.out.println("   → UserService が注入されました");
        this.userService = userService;
    }
    
    @GetMapping("/users")
    public List<String> getUsers() {
        return userService.getAllUsers();
    }
}
```

## 説明
**Bean生成の流れ**

1. **Springコンテナの起動**
   - アプリケーション起動時に、`@Component`、`@Service`、`@Repository`、`@Controller`などのアノテーションが付いたクラスをスキャン

2. **依存関係の解析**
   - 各クラスのコンストラクタを分析して依存関係を把握
   - 依存関係グラフを作成

3. **Bean生成順序の決定**
   - 依存関係の少ないものから順に生成
   - `UserRepository`（依存なし） → `UserService`（UserRepositoryに依存） → `UserController`（UserServiceに依存）

4. **インスタンス生成と注入**
```
Step 1: UserRepository userRepository = new UserRepository();
        → Springコンテナに登録

Step 2: UserService userService = new UserService(userRepository);
        → 既存のuserRepositoryを注入
        → Springコンテナに登録

Step 3: UserController userController = new UserController(userService);
        → 既存のuserServiceを注入
        → Springコンテナに登録
```

**重要なポイント**

- **Beanの生成**: Springコンテナが行う
- **コンストラクタインジェクション**: 既に生成済みのBeanをコンストラクタ引数として渡す
- **生成順序**: 依存関係が少ない（依存されている）Beanから先に生成される
- **シングルトン**: デフォルトではBeanは1つだけ生成され、複数の場所で同じインスタンスが再利用される

**コンストラクタインジェクションの利点**

1. **イミュータビリティ**: `final`フィールドを使用できる
2. **必須依存関係の明示**: コンストラクタ引数で依存関係が明確
3. **テスト容易性**: Springコンテナなしでもインスタンス化可能
4. **循環依存の検出**: 起動時にエラーとして検出できる

**実行結果（コンソール出力）**
```
1. UserRepository のインスタンスが生成されました
2. UserService のインスタンスが生成されました
   → UserRepository が注入されました
3. UserController のインスタンスが生成されました
   → UserService が注入されました
```

**よくある誤解**

❌ 「コンストラクタインジェクションでBeanが生成される」
✅ 「SpringコンテナがBeanを生成し、既存のBeanをコンストラクタ経由で注入する」

## 参考
- [Spring Framework公式ドキュメント - Bean Overview](https://docs.spring.io/spring-framework/reference/core/beans/definition.html)
- [Spring Framework公式ドキュメント - Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-autowire.html)
- [Baeldung - Constructor Injection in Spring](https://www.baeldung.com/constructor-injection-in-spring)

## 関連スニペット
- @Autowired - 依存性注入アノテーション


## 作成日
2025-01-27

## タグ
#spring #constructor-injection #bean #dependency-injection #ioc #lifecycle