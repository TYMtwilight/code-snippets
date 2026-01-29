# Iterable<T>とは

## 概要
`Iterable<T>`は「繰り返し処理できるもの」を表すJavaの基本インターフェース。List、Set、CollectionなどはすべてIterableを実装している。

## 使用場面
- `findAll()`の戻り値を処理する場合
- for-each文で要素を順番に処理したい場合
- 様々なコレクション型を統一的に扱いたい場合

## コード
```java
// CrudRepository の findAll メソッド
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    Iterable<T> findAll();
    //  ↑ 「繰り返し処理できるもの」を返す
}
```

```java
// Iterable の使い方
@Service
public class UserService {
    private final UserRepository userRepository;

    // 基本: for-each で回す
    public void printAllUsers() {
        Iterable<User> users = userRepository.findAll();

        for (User user : users) {
            System.out.println(user.getName());
        }
    }

    // forEach メソッドを使う
    public void printAllUsers2() {
        userRepository.findAll()
            .forEach(user -> System.out.println(user.getName()));
    }

    // List に変換したい場合
    public List<User> getAllUsersAsList() {
        Iterable<User> iterable = userRepository.findAll();

        // 方法1: StreamSupport を使う
        List<User> list1 = StreamSupport
            .stream(iterable.spliterator(), false)
            .collect(Collectors.toList());

        // 方法2: 手動で追加
        List<User> list2 = new ArrayList<>();
        iterable.forEach(list2::add);

        return list1;
    }
}
```

```java
// Iterable の継承関係
//
// Iterable<T>        ← 最も基本（for-each 可能）
//     ↑
// Collection<T>      ← size(), add(), remove() など
//     ↑
// List<T> / Set<T>   ← 順序あり / 重複なし
//     ↑
// ArrayList / HashSet など

// なぜ CrudRepository は Iterable を返す？
// → 実装の自由度を高めるため
// → List でも Set でも返せる
```

```java
// 実務では JpaRepository を使うことが多い
public interface UserRepository extends JpaRepository<User, Long> {
    // JpaRepository の findAll() は List<T> を返す
    // ↓ オーバーライドされている
    // List<User> findAll();
}

@Service
public class UserService {
    private final UserRepository userRepository;

    public List<User> getAllUsers() {
        // JpaRepository なので直接 List が返る
        List<User> users = userRepository.findAll();
        return users;
    }
}
```

## 説明
### Iterableとは
- **Java標準のインターフェース**（`java.lang.Iterable`）
- **for-each文で使えるようにする**ための契約
- **iterator()メソッドを持つ**

### なぜIterableを使う？
| 理由 | 説明 |
|-----|------|
| 抽象化 | List, Set, 配列など様々な型を統一的に扱える |
| 柔軟性 | 実装がListでもSetでも、呼び出し側は同じコードで処理できる |
| 遅延評価 | 必要に応じて要素を生成することも可能 |

### CrudRepository vs JpaRepository
```java
// CrudRepository
Iterable<T> findAll();  // 抽象的

// JpaRepository (JPAに特化)
List<T> findAll();      // 具体的（使いやすい）
```

### 実務でのベストプラクティス
```java
// JpaRepository を継承するのが一般的
public interface UserRepository extends JpaRepository<User, Long> {
    // ほとんどのメソッドが List を返す
}
```

## 参考
- [Java Iterable JavaDoc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Iterable.html)
- [JpaRepository JavaDoc](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)

## 関連スニペット
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)
- [PageableとPage<T>](./pageable-and-page.md)

## 作成日
2026-01-29

## タグ
#java #iterable #collection #spring-data #for-each
