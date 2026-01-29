# Repository<T, ID>の型引数

## 概要
`Repository<T, ID>`はSpring Dataの中心的なマーカーインターフェースで、Tはエンティティ型、IDはその主キーの型を表す。

## 使用場面
- カスタムリポジトリインターフェースを定義する場合
- Spring Dataの自動実装機能を利用したい場合
- 特定のエンティティに対するデータアクセス層を作成する場合

## コード
```java
import org.springframework.data.repository.Repository;

// Repository<T, ID> の型引数
// T  = 管理対象のエンティティ（ドメインクラス）
// ID = そのエンティティのIDの型

// 例1: User エンティティ、ID は Long 型
public interface UserRepository extends Repository<User, Long> {
    List<User> findByName(String name);
    Optional<User> findByEmail(String email);
}

// 例2: Product エンティティ、ID は Long 型
public interface ProductRepository extends Repository<Product, Long> {
    List<Product> findByPriceLessThan(BigDecimal price);
}

// 例3: OrderItem エンティティ、複合キーの場合は埋め込みIDクラスを使用
public interface OrderItemRepository extends Repository<OrderItem, OrderItemId> {
    List<OrderItem> findByOrderId(Long orderId);
}
```

```java
// マーカーインターフェースとしての役割
// Repository を継承 = Spring に「私はリポジトリです」と宣言

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// ↑ アプリケーション起動時に Spring が...
// 1. @Repository または Repository を継承したインターフェースを探す
// 2. 見つけたインターフェースの実装クラスを自動生成
// 3. Bean として登録

@Service
public class UserService {
    private final UserRepository userRepository;

    // ↓ 自動生成された実装が注入される
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## 説明
### 型引数の意味
| 型引数 | 意味 | 例 |
|-------|------|-----|
| T | 管理対象のエンティティ | User, Product, Order |
| ID | エンティティの主キーの型 | Long, Integer, UUID, String |

### マーカーインターフェースとは
- **メソッドを持たないインターフェース**
- 「このクラスは〇〇です」という目印（マーク）を付ける役割
- Springがクラスパススキャン時にリポジトリを識別するために使用

### IDの型の選び方
```java
// Long（推奨）: 自動採番、十分な範囲
public interface UserRepository extends Repository<User, Long> {}

// UUID: 分散システム、セキュリティ重視
public interface TokenRepository extends Repository<Token, UUID> {}

// String: 自然キー（メールアドレスなど）
public interface AccountRepository extends Repository<Account, String> {}

// 複合キー: 中間テーブルなど
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
}
public interface OrderItemRepository extends Repository<OrderItem, OrderItemId> {}
```

## 参考
- [Spring Data Commons - Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)

## 関連スニペット
- [ドメインクラスとは](./domain-class.md)
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)
- [ジェネリクス<S extends T>](./generics-s-extends-t.md)

## 作成日
2026-01-29

## タグ
#spring #spring-data #repository #generics #marker-interface
