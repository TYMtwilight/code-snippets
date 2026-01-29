# CrudRepositoryの基本メソッド

## 概要
`CrudRepository<T, ID>`はRepositoryを拡張し、Create（作成）、Read（読み取り）、Update（更新）、Delete（削除）の基本操作を提供するインターフェース。

## 使用場面
- エンティティの基本的なCRUD操作を行う場合
- カスタムクエリを書かずに標準的なデータ操作をしたい場合
- シンプルなデータアクセス層を構築する場合

## コード
```java
// CrudRepository のインターフェース定義
public interface CrudRepository<T, ID> extends Repository<T, ID> {

    // === Create / Update ===
    <S extends T> S save(S entity);                    // 1件保存・更新
    <S extends T> Iterable<S> saveAll(Iterable<S> entities);  // 複数保存

    // === Read ===
    Optional<T> findById(ID id);                       // ID検索
    boolean existsById(ID id);                         // 存在チェック
    Iterable<T> findAll();                             // 全件取得
    Iterable<T> findAllById(Iterable<ID> ids);         // 複数ID検索
    long count();                                      // 件数カウント

    // === Delete ===
    void deleteById(ID id);                            // ID指定削除
    void delete(T entity);                             // エンティティ削除
    void deleteAllById(Iterable<? extends ID> ids);    // 複数ID削除
    void deleteAll(Iterable<? extends T> entities);    // 複数エンティティ削除
    void deleteAll();                                  // 全件削除
}
```

```java
// 実際の使用例
public interface UserRepository extends CrudRepository<User, Long> {
    // CrudRepository のメソッドが自動的に使える
    // 追加でカスタムメソッドも定義可能
    List<User> findByName(String name);
}

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // === Create ===
    public User createUser(String name, String email) {
        User user = new User(name, email);
        return userRepository.save(user);  // INSERT が実行される
    }

    // === Read ===
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }

    public List<User> getAllUsers() {
        List<User> users = new ArrayList<>();
        userRepository.findAll().forEach(users::add);
        return users;
    }

    public boolean userExists(Long id) {
        return userRepository.existsById(id);
    }

    public long getUserCount() {
        return userRepository.count();
    }

    // === Update ===
    public User updateUser(Long id, String newName) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
        user.setName(newName);
        return userRepository.save(user);  // UPDATE が実行される
    }

    // === Delete ===
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    public void deleteUsers(List<Long> ids) {
        userRepository.deleteAllById(ids);
    }
}
```

```java
// save() の動作: INSERT or UPDATE
@Transactional
public void saveExample() {
    // 新規エンティティ（IDがnull）→ INSERT
    User newUser = new User("Alice", "alice@example.com");
    userRepository.save(newUser);  // INSERT INTO users ...

    // 既存エンティティ（IDあり）→ UPDATE
    User existing = userRepository.findById(1L).get();
    existing.setName("Bob");
    userRepository.save(existing);  // UPDATE users SET name = 'Bob' WHERE id = 1
}
```

## 説明
### メソッド一覧
| 操作 | メソッド | 説明 |
|-----|---------|------|
| Create | `save(entity)` | エンティティを保存（IDがなければINSERT） |
| Create | `saveAll(entities)` | 複数エンティティを保存 |
| Read | `findById(id)` | IDで1件検索（Optional） |
| Read | `existsById(id)` | 存在チェック |
| Read | `findAll()` | 全件取得 |
| Read | `findAllById(ids)` | 複数IDで検索 |
| Read | `count()` | 件数カウント |
| Update | `save(entity)` | エンティティを更新（IDがあればUPDATE） |
| Delete | `deleteById(id)` | IDで削除 |
| Delete | `delete(entity)` | エンティティで削除 |
| Delete | `deleteAll()` | 全件削除 |

### save() の INSERT / UPDATE 判定
```
IDがnull → INSERT（新規作成）
IDがある → UPDATE（更新）
※ 正確には @Version や EntityInformation で判定
```

### CrudRepository vs JpaRepository
```java
// CrudRepository: 基本的なCRUD
// JpaRepository: CrudRepository + ページング + フラッシュ操作 + バッチ削除
public interface JpaRepository<T, ID>
    extends ListCrudRepository<T, ID>, ListPagingAndSortingRepository<T, ID> {

    void flush();
    <S extends T> S saveAndFlush(S entity);
    void deleteAllInBatch();
    // ...
}
```

## 参考
- [CrudRepository JavaDoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.core-concepts)

## 関連スニペット
- [Repository<T, ID>の型引数](./repository-type-parameters.md)
- [Optional<T>の使い方](./optional-usage.md)
- [PageableとPage<T>](./pageable-and-page.md)

## 作成日
2026-01-29

## タグ
#spring #spring-data #crud #repository #jpa
