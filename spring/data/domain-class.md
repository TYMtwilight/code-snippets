# ドメインクラスとは

## 概要
ドメインクラスは、ビジネスの対象となるデータを表すクラス（エンティティ）で、データベースのテーブルに対応するJavaクラス。

## 使用場面
- データベーステーブルをJavaオブジェクトとして扱いたい場合
- ビジネスロジックで扱うデータの構造を定義する場合
- ORMフレームワーク（JPA/Hibernate）でデータを永続化する場合

## コード
```java
import jakarta.persistence.*;

@Entity  // このクラスがエンティティ（ドメインクラス）であることを宣言
@Table(name = "users")  // 対応するテーブル名を指定
public class User {

    @Id  // 主キーを示す
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 自動採番
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(unique = true)
    private String email;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    // デフォルトコンストラクタ（JPA必須）
    protected User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }

    // getter/setter
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    // ...
}
```

```java
// 他のドメインクラスの例
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private BigDecimal price;
    private Integer stock;
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    private User user;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> items;
}
```

## 説明
### ドメインクラスの特徴
1. **`@Entity`アノテーション**: JPAにエンティティであることを伝える
2. **`@Id`フィールド**: 必ず主キーが必要
3. **デフォルトコンストラクタ**: JPAがインスタンス生成に使用（protected可）

### 命名規則
- クラス名 → テーブル名（デフォルト）
- フィールド名 → カラム名（デフォルト）
- 明示的に指定したい場合は`@Table`、`@Column`を使用

### ドメインクラスの例
| ドメインクラス | 対応するテーブル | 説明 |
|--------------|----------------|------|
| User | users | ユーザー情報 |
| Product | products | 商品情報 |
| Order | orders | 注文情報 |
| Category | categories | カテゴリ情報 |

## 参考
- [Jakarta Persistence (JPA) Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)

## 関連スニペット
- [Repository<T, ID>の型引数](./repository-type-parameters.md)
- [Spring Data Repositoryの抽象化](./repository-abstraction.md)

## 作成日
2026-01-29

## タグ
#spring #jpa #entity #domain #hibernate
