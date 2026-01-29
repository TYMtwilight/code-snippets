# PageableとPage<T>

## 概要
`Pageable`はページングリクエスト（何ページ目、何件、ソート順）を表し、`Page<T>`はページングレスポンス（データ＋ページ情報）を表すSpring Data独自の型。

## 使用場面
- 大量のデータを分割して取得したい場合
- 一覧画面でページネーションを実装する場合
- APIでページング機能を提供する場合

## コード
```java
// PagingAndSortingRepository の定義
public interface PagingAndSortingRepository<T, ID> extends Repository<T, ID> {
    Iterable<T> findAll(Sort sort);         // ソートのみ
    Page<T> findAll(Pageable pageable);     // ページング + ソート
}
```

```java
// Pageable の作成方法
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

// 基本: ページ番号とサイズ
Pageable pageable1 = PageRequest.of(0, 10);  // 0ページ目、10件

// ソート付き
Pageable pageable2 = PageRequest.of(0, 10, Sort.by("name"));  // name昇順

// ソート方向指定
Pageable pageable3 = PageRequest.of(0, 10, Sort.by("name").ascending());
Pageable pageable4 = PageRequest.of(0, 10, Sort.by("createdAt").descending());

// 複数ソート条件
Pageable pageable5 = PageRequest.of(0, 10,
    Sort.by("status").ascending()
        .and(Sort.by("createdAt").descending())
);

// Sort.Direction を使う方法
Pageable pageable6 = PageRequest.of(0, 10, Sort.Direction.DESC, "createdAt");
```

```java
// Page<T> の使い方
@Service
public class UserService {
    private final UserRepository userRepository;

    public Page<User> getUsers(int pageNumber, int pageSize) {
        Pageable pageable = PageRequest.of(pageNumber, pageSize, Sort.by("name"));
        Page<User> page = userRepository.findAll(pageable);

        // === データ取得 ===
        List<User> users = page.getContent();      // 現在ページのデータ

        // === ページ情報 ===
        int totalPages = page.getTotalPages();     // 総ページ数
        long totalElements = page.getTotalElements(); // 全件数
        int currentPage = page.getNumber();        // 現在ページ番号（0始まり）
        int size = page.getSize();                 // 1ページあたりの件数
        int numberOfElements = page.getNumberOfElements(); // 現在ページの件数

        // === ナビゲーション ===
        boolean hasNext = page.hasNext();          // 次ページあり？
        boolean hasPrevious = page.hasPrevious();  // 前ページあり？
        boolean isFirst = page.isFirst();          // 最初のページ？
        boolean isLast = page.isLast();            // 最後のページ？

        return page;
    }
}
```

```java
// 実務での使用例: REST API
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // 方法1: パラメータで受け取る
    @GetMapping
    public Page<User> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String direction
    ) {
        Sort sort = direction.equalsIgnoreCase("desc")
            ? Sort.by(sortBy).descending()
            : Sort.by(sortBy).ascending();
        Pageable pageable = PageRequest.of(page, size, sort);
        return userRepository.findAll(pageable);
    }

    // 方法2: Pageable を直接受け取る（Spring が自動変換）
    @GetMapping("/v2")
    public Page<User> getUsersV2(Pageable pageable) {
        // GET /api/users/v2?page=0&size=10&sort=name,asc
        return userRepository.findAll(pageable);
    }
}
```

```java
// カスタムクエリでのページング
public interface UserRepository extends JpaRepository<User, Long> {

    // メソッド名クエリ + ページング
    Page<User> findByStatus(String status, Pageable pageable);

    // @Query + ページング
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    Page<User> findByEmailDomain(@Param("domain") String domain, Pageable pageable);

    // ネイティブクエリ + ページング（countQuery が必要）
    @Query(
        value = "SELECT * FROM users WHERE status = :status",
        countQuery = "SELECT count(*) FROM users WHERE status = :status",
        nativeQuery = true
    )
    Page<User> findByStatusNative(@Param("status") String status, Pageable pageable);
}
```

```java
// Page<T> のJSON出力例
// GET /api/users?page=0&size=2
{
    "content": [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"}
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 2,
        "sort": {"sorted": true, "unsorted": false}
    },
    "totalPages": 5,
    "totalElements": 10,
    "last": false,
    "first": true,
    "numberOfElements": 2,
    "size": 2,
    "number": 0,
    "empty": false
}
```

## 説明
### Pageable と Page の関係
```
Pageable（リクエスト）        Page<T>（レスポンス）
┌─────────────────┐         ┌──────────────────────┐
│ page: 1         │         │ content: [...]       │
│ size: 10        │  ────>  │ totalPages: 5        │
│ sort: name,asc  │         │ totalElements: 50    │
└─────────────────┘         │ hasNext: true        │
                            └──────────────────────┘
```

### 主要クラス・インターフェース
| 型 | 役割 | 主なメソッド/プロパティ |
|----|------|----------------------|
| `Pageable` | リクエスト | page, size, sort |
| `PageRequest` | Pageable の実装 | of(page, size, sort) |
| `Page<T>` | レスポンス | content, totalPages, hasNext |
| `Sort` | ソート条件 | by(property), ascending(), descending() |

### Slice vs Page
```java
// Page: 総件数を取得（COUNT クエリが実行される）
Page<User> page = userRepository.findAll(pageable);
page.getTotalElements();  // 全件数（追加クエリ必要）

// Slice: 総件数を取得しない（パフォーマンス良い）
Slice<User> slice = userRepository.findByStatus("active", pageable);
slice.hasNext();  // 次ページの有無のみ判定
// slice.getTotalElements();  // ❌ メソッドがない
```

## 参考
- [Spring Data Commons - Paging and Sorting](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#repositories.paging-and-sorting)
- [Pageable JavaDoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Pageable.html)
- [Page JavaDoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html)

## 関連スニペット
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)
- [Spring MVC 基本コントローラー](../controller/spring-mvc-controller-basics.md)

## 作成日
2026-01-29

## タグ
#spring #spring-data #pagination #pageable #page #rest-api
