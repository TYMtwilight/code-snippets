# Optional<T>の使い方

## 概要
`Optional<T>`は、値が存在しない可能性を型で表現するコンテナクラス。findByIdの結果がnullかもしれないことを明示し、NullPointerExceptionを防ぐ。

## 使用場面
- データベースからIDで検索する場合（結果が存在しない可能性がある）
- nullチェック忘れによるNullPointerExceptionを防ぎたい場合
- 値の有無に応じた処理を明示的に記述したい場合

## コード
```java
// CrudRepository の findById メソッド
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    Optional<T> findById(ID id);
    // ↑ 結果がnullかもしれないことを型で表現
}
```

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    // パターン1: isPresent() で存在チェック
    public void pattern1(Long id) {
        Optional<User> optionalUser = userRepository.findById(id);

        if (optionalUser.isPresent()) {
            User user = optionalUser.get();
            System.out.println(user.getName());
        } else {
            System.out.println("User not found");
        }
    }

    // パターン2: orElse() でデフォルト値
    public User pattern2(Long id) {
        return userRepository.findById(id)
            .orElse(new User("Guest", "guest@example.com"));
    }

    // パターン3: orElseThrow() で例外をスロー（最も一般的）
    public User pattern3(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    // パターン4: orElseGet() で遅延評価
    public User pattern4(Long id) {
        return userRepository.findById(id)
            .orElseGet(() -> createDefaultUser());  // 必要な時だけ実行
    }

    // パターン5: ifPresent() で存在時のみ処理
    public void pattern5(Long id) {
        userRepository.findById(id)
            .ifPresent(user -> sendWelcomeEmail(user));
    }

    // パターン6: ifPresentOrElse() で存在/不在で分岐（Java 9+）
    public void pattern6(Long id) {
        userRepository.findById(id)
            .ifPresentOrElse(
                user -> System.out.println("Found: " + user.getName()),
                () -> System.out.println("Not found")
            );
    }

    // パターン7: map() で値を変換
    public Optional<String> pattern7(Long id) {
        return userRepository.findById(id)
            .map(User::getName);  // Optional<User> → Optional<String>
    }

    // パターン8: filter() で条件フィルタ
    public Optional<User> pattern8(Long id) {
        return userRepository.findById(id)
            .filter(user -> user.isActive());  // アクティブなユーザーのみ
    }
}
```

```java
// ❌ アンチパターン: get() を直接呼ぶ
public User badExample(Long id) {
    Optional<User> optional = userRepository.findById(id);
    return optional.get();  // ❌ NoSuchElementException の可能性
}

// ❌ アンチパターン: Optional を引数や戻り値以外で使う
public void badExample2(Optional<User> user) {  // ❌ 引数に Optional は避ける
    // ...
}
```

## 説明
### Optional のメリット
1. **nullチェック忘れを防止**: コンパイル時に値の有無を意識させる
2. **意図の明確化**: 「値がないかもしれない」ことを型で表現
3. **流暢なAPI**: map, filter, orElse などでチェーン処理

### 主要メソッド一覧
| メソッド | 説明 | 用途 |
|---------|------|------|
| `isPresent()` | 値が存在するか | 条件分岐 |
| `isEmpty()` | 値が存在しないか（Java 11+） | 条件分岐 |
| `get()` | 値を取得（非推奨） | - |
| `orElse(T)` | 値がなければデフォルト値 | デフォルト値が軽量な場合 |
| `orElseGet(Supplier)` | 値がなければ生成 | デフォルト値が重い場合 |
| `orElseThrow(Supplier)` | 値がなければ例外 | 存在必須の場合 |
| `ifPresent(Consumer)` | 存在時のみ処理 | 副作用のある処理 |
| `map(Function)` | 値を変換 | 型変換 |
| `filter(Predicate)` | 条件でフィルタ | 追加条件 |

## 参考
- [Java Optional JavaDoc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Optional.html)
- [Tired of Null Pointer Exceptions? Consider Using Java SE 8's Optional!](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)

## 関連スニペット
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)
- [Repository<T, ID>の型引数](./repository-type-parameters.md)

## 作成日
2026-01-29

## タグ
#java #optional #null-safety #spring-data #best-practices
