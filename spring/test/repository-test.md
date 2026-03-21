# @DataJpaTest - Repository テンプレート

## 概要
`@DataJpaTest` を使い、インメモリ DB（H2）上でカスタムクエリや JPA の挙動を検証するテストパターン。`@CreatedDate` / `@LastModifiedDate` などの Entity ライフサイクルも確認できる。

## 使用場面
- カスタムクエリメソッド（`findByXxx`、`findAllByOrderByXxx`）が正しく動くことを検証する時
- `@CreatedDate` / `@LastModifiedDate` が保存時に自動セットされることを確認する時
- DB スキーマとエンティティのマッピングを検証する時

## コード

```java
@DataJpaTest
class TodoRepositoryTest {

  @Autowired
  private TodoRepository todoRepository;

  // --------------------------------------------------
  // カスタムクエリ：全件取得（降順）
  // --------------------------------------------------

  @Test
  void 全件取得_データがない場合は空リストを返す() {
    List<Todo> todos = todoRepository.findAllByOrderByCreatedAtDesc();

    assertThat(todos).isEmpty();
  }

  @Test
  void 全件取得_複数件を作成日時の降順で返す() {
    // Given
    Todo todo1 = new Todo();
    todo1.setTitle("最初の TODO");
    todoRepository.save(todo1);

    Todo todo2 = new Todo();
    todo2.setTitle("2番目の TODO");
    todoRepository.save(todo2);

    // When
    List<Todo> todos = todoRepository.findAllByOrderByCreatedAtDesc();

    // Then
    assertThat(todos).hasSize(2);
    assertThat(todos.get(0).getTitle()).isEqualTo("2番目の TODO"); // 降順
    assertThat(todos.get(1).getTitle()).isEqualTo("最初の TODO");
  }

  // --------------------------------------------------
  // カスタムクエリ：完了状態でフィルタリング
  // --------------------------------------------------

  @Test
  void 未完了のTODOだけ取得できる() {
    // Given
    Todo active = new Todo();
    active.setTitle("未完了タスク");
    active.setCompleted(false);
    todoRepository.save(active);

    Todo completed = new Todo();
    completed.setTitle("完了済みタスク");
    completed.setCompleted(true);
    todoRepository.save(completed);

    // When
    List<Todo> activeTodos = todoRepository.findByCompleted(false);

    // Then
    assertThat(activeTodos).hasSize(1);
    assertThat(activeTodos.get(0).getTitle()).isEqualTo("未完了タスク");
    assertThat(activeTodos.get(0).getCompleted()).isFalse();
  }

  // --------------------------------------------------
  // Entity ライフサイクル（@CreatedDate / @LastModifiedDate）
  // --------------------------------------------------

  @Test
  void 保存時にcreatedAtとupdatedAtが自動設定される() {
    // Given
    Todo todo = new Todo();
    todo.setTitle("日時テスト");

    LocalDateTime beforeSave = LocalDateTime.now();

    // When
    Todo saved = todoRepository.save(todo);

    LocalDateTime afterSave = LocalDateTime.now();

    // Then
    assertThat(saved.getId()).isNotNull();
    assertThat(saved.getCreatedAt())
        .isNotNull()
        .isAfterOrEqualTo(beforeSave)
        .isBeforeOrEqualTo(afterSave);
    assertThat(saved.getUpdatedAt())
        .isNotNull()
        .isAfterOrEqualTo(beforeSave)
        .isBeforeOrEqualTo(afterSave);
    assertThat(saved.getCompleted()).isFalse(); // デフォルト値の確認
  }
}
```

## 説明

### @DataJpaTest の特徴

| 項目 | 内容 |
|---|---|
| 起動範囲 | JPA 関連コンポーネントのみ（Controller / Service は起動しない）|
| DB | インメモリ DB（H2）を自動使用 |
| トランザクション | 各テストはロールバックされる（テスト間の分離が保証される）|
| 用途 | カスタムクエリ・Entity のマッピング検証 |

### @Autowired で Repository を注入

```java
@Autowired
private TodoRepository todoRepository;
```

- `@DataJpaTest` がコンテキストを起動するため、`@Autowired` で注入可能
- Service テストの `@Mock` と異なり、実際の JPA 実装が動く

### テストの独立性

- 各 `@Test` メソッドはトランザクション内で実行され、テスト終了時に自動ロールバックされる
- テスト間でデータが残らないため、`@BeforeEach` で削除する必要がない

### Entity のデフォルト値を検証

```java
assertThat(saved.getCompleted()).isFalse(); // Entity の @Column(columnDefinition="boolean default false")
assertThat(saved.getId()).isNotNull();       // ID が自動採番されている
assertThat(saved.getCreatedAt()).isNotNull();// @CreatedDate が設定されている
```

### カスタムクエリの命名規則

Spring Data JPA のメソッド名クエリ（Query Derivation）:

```java
// findBy + フィールド名
List<Todo> findByCompleted(Boolean completed);

// findAllBy + フィールド名 + OrderBy + フィールド名 + Desc
List<Todo> findAllByOrderByCreatedAtDesc();
```

## 参考
- [Spring Data JPA - Query Derivation](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Spring Boot - @DataJpaTest](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html)

## 関連スニペット
- [Mockito - Service テスト](./service-test.md)
- [@WebMvcTest + MockMvc テスト](./controller-test.md)

## 作成日
2026-03-21

## タグ
#spring #test #datajpatest #repository-test #jpa #h2 #query-derivation
