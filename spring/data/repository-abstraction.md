# Spring Data Repositoryの抽象化

## 概要
Spring Data Repositoryは、データアクセスレイヤーの定型コードを削減し、永続ストア（DB）の違いを抽象化する仕組み。

## 使用場面
- 異なるデータベースを統一的なインターフェースで操作したい場合
- データアクセス層のボイラープレートコードを削減したい場合
- MySQLからPostgreSQLへの移行など、DBを切り替える可能性がある場合

## コード
```java
// どのDBを使っても同じインターフェースで操作できる
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
    List<User> findByEmailContaining(String email);
}

// MySQL でも PostgreSQL でも MongoDB でも
// 同じ UserRepository インターフェースが使える
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }
}
```

## 説明
### 抽象化しているもの
- **永続ストア（DB）の違い**: MySQL、PostgreSQL、MongoDB、Redisなど
- **データアクセスの定型コード**: JDBC接続、SQLの組み立て、ResultSetの変換など

### メリット
1. **DBの切り替えが容易**: 設定変更だけでDBを切り替え可能
2. **コード量の削減**: CRUDの基本操作を自動実装
3. **統一されたAPI**: どのDBでも同じメソッドで操作

## 参考
- [Spring Data Commons - Reference Documentation](https://docs.spring.io/spring-data/commons/docs/current/reference/html/)
- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)

## 関連スニペット
- [ドメインクラスとは](./domain-class.md)
- [Repository<T, ID>の型引数](./repository-type-parameters.md)
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)

## 作成日
2026-01-29

## タグ
#spring #spring-data #repository #jpa #abstraction
