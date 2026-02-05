# Spring Data Repository 自動検出ルール

## 概要

Spring BootがRepositoryインターフェースを自動的に検出し、実装クラスを生成・登録する仕組みとそのルール

## 使用場面

- Repositoryインターフェースを定義する際のパッケージ構成を決めるとき
- `@EnableJpaRepositories`でスキャン範囲をカスタマイズするとき
- 複数のデータソースを扱う際にRepositoryを分離するとき

## コード

```java
// 基本パターン：@SpringBootApplicationと同じパッケージ以下
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// com.example.demo.repository.UserRepository
// → 自動検出される
public interface UserRepository extends JpaRepository<User, Long> {
}

// カスタマイズパターン
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.demo.repositories",
    repositoryImplementationPostfix = "Impl"
)
public class JpaConfig {
}

// カスタム実装を追加する場合
public interface UserRepositoryCustom {
    List<User> customQuery();
}

// 実装クラス名は「元のインターフェース名 + Impl」
public class UserRepositoryImpl implements UserRepositoryCustom {
    @Override
    public List<User> customQuery() {
        // カスタム実装
    }
}

// Spring Dataが生成した実装とカスタム実装が統合される
public interface UserRepository
    extends JpaRepository<User, Long>, UserRepositoryCustom {
}
```

## 説明

### 自動検出の仕組み

1. **デフォルトのスキャン範囲**
   - `@SpringBootApplication`が付いたクラスのパッケージとそのサブパッケージ
   - `Repository`インターフェース（またはその継承インターフェース）を検出

2. **検出対象の条件**
   - `Repository<T, ID>`、`CrudRepository<T, ID>`、`JpaRepository<T, ID>`などを継承
   - または`@RepositoryDefinition`アノテーション付き

3. **実装クラスの生成**
   - Spring Dataがプロキシベースの実装を自動生成
   - 実行時に動的に作成され、Springコンテナに登録

### カスタマイズオプション

- **basePackages**: スキャン対象パッケージを明示的に指定
- **repositoryImplementationPostfix**: カスタム実装クラスの接尾辞（デフォルト: "Impl"）
- **includeFilters/excludeFilters**: 検出対象の細かい制御

### カスタム実装の統合ルール

- インターフェース名が`UserRepository`の場合、`UserRepositoryImpl`がカスタム実装として認識される
- Spring Dataの自動生成実装とカスタム実装が1つのBeanに統合される
- カスタム実装は`@PersistenceContext`や`@Autowired`で`EntityManager`等を注入可能

## 参考

- [Spring Data Commons - Defining Repository Interfaces](https://docs.spring.io/spring-data/commons/reference/repositories/definition.html)
- [Spring Data JPA - Repository Query Keywords](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)

## 関連スニペット

- [Repository基本パターン](./repository-basic-pattern.md)
- [カスタムRepository実装](./custom-repository-implementation.md)
- [Spring Data JPAクエリメソッド命名規則](./query-method-naming.md)

## 作成日

2025-02-05

## タグ

#spring-data #repository #auto-detection #component-scan #jpa
