# Mockito - Service テンプレート

## 概要
`@ExtendWith(MockitoExtension.class)` と `@Mock` / `@InjectMocks` を使い、Repository をモック化して Service のビジネスロジックのみを検証するテストパターン。Spring コンテキストも DB も起動しない。

## 使用場面
- Service 層のビジネスロジック（分岐・計算・変換）を単体テストする時
- DB アクセスを伴わず高速にテストしたい時
- 存在しない ID が渡された時に例外がスローされることを検証する時

## コード

```java
@ExtendWith(MockitoExtension.class)
class TodoServiceTest {

  @Mock
  private TodoRepository todoRepository; // Repository をモック化

  @InjectMocks
  private TodoServiceImpl todoService;   // テスト対象（モックが注入される）

  // --- テストデータ作成ヘルパー ---
  private Todo createTodo(Long id, String title, Boolean completed) {
    Todo todo = new Todo();
    todo.setId(id);
    todo.setTitle(title);
    todo.setCompleted(completed);
    return todo;
  }

  // --------------------------------------------------
  // 正常系：一覧取得
  // --------------------------------------------------

  @Test
  void findAll_ALLで全件取得できる() {
    // Given
    List<Todo> todos = List.of(
        createTodo(1L, "タスク1", false),
        createTodo(2L, "タスク2", true));
    when(todoRepository.findAllByOrderByCreatedAtDesc()).thenReturn(todos);

    // When
    List<TodoResponse> result = todoService.findAll("ALL");

    // Then
    assertThat(result).hasSize(2);
    verify(todoRepository).findAllByOrderByCreatedAtDesc();
  }

  // --------------------------------------------------
  // 正常系：作成
  // --------------------------------------------------

  @Test
  void create_新しいTODOを作成できる() {
    // Given
    TodoRequest request = new TodoRequest();
    request.setTitle("新しいタスク");
    Todo saved = createTodo(1L, "新しいタスク", false);
    when(todoRepository.save(any(Todo.class))).thenReturn(saved);

    // When
    TodoResponse result = todoService.create(request);

    // Then
    assertThat(result.getTitle()).isEqualTo("新しいタスク");
    assertThat(result.getCompleted()).isFalse();
    verify(todoRepository).save(any(Todo.class));
  }

  // --------------------------------------------------
  // 正常系：更新
  // --------------------------------------------------

  @Test
  void update_タイトルを更新できる() {
    // Given
    Todo existing = createTodo(1L, "古いタイトル", false);
    when(todoRepository.findById(1L)).thenReturn(Optional.of(existing));
    when(todoRepository.save(any(Todo.class))).thenReturn(existing);

    TodoRequest request = new TodoRequest();
    request.setTitle("新しいタイトル");

    // When
    TodoResponse result = todoService.update(1L, request);

    // Then
    assertThat(result.getTitle()).isEqualTo("新しいタイトル");
    verify(todoRepository).findById(1L);
    verify(todoRepository).save(existing);
  }

  // --------------------------------------------------
  // 異常系：存在しない ID
  // --------------------------------------------------

  @Test
  void update_存在しないIDで例外が発生する() {
    // Given
    when(todoRepository.findById(999L)).thenReturn(Optional.empty());
    TodoRequest request = new TodoRequest();
    request.setTitle("更新");

    // When & Then
    assertThatThrownBy(() -> todoService.update(999L, request))
        .isInstanceOf(RuntimeException.class)
        .hasMessageContaining("TODO が見つかりません");
  }

  // --------------------------------------------------
  // 異常系：null ID
  // --------------------------------------------------

  @Test
  void update_idがnullのときIllegalArgumentExceptionをスローする() {
    TodoRequest request = new TodoRequest();
    request.setTitle("タイトル");

    assertThrows(IllegalArgumentException.class, () -> todoService.update(null, request));
    verify(todoRepository, never()).findById(any());
    verify(todoRepository, never()).save(any());
  }

  // --------------------------------------------------
  // 正常系：トグル
  // --------------------------------------------------

  @Test
  void toggleComplete_falseからtrueに切り替わる() {
    // Given
    Todo todo = createTodo(1L, "タスク", false);
    when(todoRepository.findById(1L)).thenReturn(Optional.of(todo));
    when(todoRepository.save(any(Todo.class))).thenReturn(todo);

    // When
    todoService.toggleComplete(1L);

    // Then
    assertThat(todo.getCompleted()).isTrue();
  }

  // --------------------------------------------------
  // 正常系：削除
  // --------------------------------------------------

  @Test
  void delete_存在するTODOを削除できる() {
    // Given
    when(todoRepository.existsById(1L)).thenReturn(true);
    doNothing().when(todoRepository).deleteById(1L);

    // When
    todoService.delete(1L);

    // Then
    verify(todoRepository).existsById(1L);
    verify(todoRepository).deleteById(1L);
  }

  @Test
  void delete_存在しないIDで例外が発生する() {
    when(todoRepository.existsById(999L)).thenReturn(false);

    assertThatThrownBy(() -> todoService.delete(999L))
        .isInstanceOf(RuntimeException.class)
        .hasMessageContaining("TODO が見つかりません");
  }
}
```

## 説明

### アノテーション早見表

| アノテーション | 役割 |
|---|---|
| `@ExtendWith(MockitoExtension.class)` | JUnit 5 で Mockito を有効化 |
| `@Mock` | モックオブジェクトを作成（Spring 非依存）|
| `@InjectMocks` | テスト対象クラスを作成し、`@Mock` を注入 |

### Mockito 主要メソッド

```java
// 戻り値のあるメソッドのモック
when(repository.findById(1L)).thenReturn(Optional.of(entity));
when(repository.save(any(Entity.class))).thenReturn(entity);

// 例外のスロー
when(repository.findById(999L)).thenReturn(Optional.empty());
// → Service 側で空の Optional から例外をスローさせる

// void メソッドのモック
doNothing().when(repository).deleteById(1L);
doThrow(new RuntimeException("error")).when(repository).method(arg);

// 呼ばれたことを検証
verify(repository).findById(1L);
verify(repository).save(entity);
verify(repository, never()).deleteById(any()); // 呼ばれていないことを検証
```

### 例外テストのパターン

```java
// パターン 1: assertThatThrownBy（AssertJ）- メッセージも検証できる
assertThatThrownBy(() -> service.update(999L, request))
    .isInstanceOf(RuntimeException.class)
    .hasMessageContaining("TODO が見つかりません");

// パターン 2: assertThrows（JUnit 5）- 例外クラスだけ検証
assertThrows(IllegalArgumentException.class, () -> service.update(null, request));
```

### テストの構造（Given-When-Then）

```java
@Test
void メソッド名_状況_期待結果() {
  // Given（前提条件）：モックの振る舞いを設定
  when(repository.findById(1L)).thenReturn(Optional.of(entity));

  // When（実行）：テスト対象を呼び出す
  Result result = service.doSomething(1L);

  // Then（検証）：結果を検証
  assertThat(result.getTitle()).isEqualTo("期待値");
  verify(repository).findById(1L); // 呼び出されたことも検証
}
```

## 参考
- [Mockito - 公式サイト](https://site.mockito.org/)
- [AssertJ - 公式ドキュメント](https://assertj.github.io/doc/)

## 関連スニペット
- [@WebMvcTest + MockMvc テスト](./controller-test.md)
- [@DataJpaTest - Repository テスト](./repository-test.md)

## 作成日
2026-03-21

## タグ
#spring #test #mockito #service-test #unit-test #given-when-then #assertj
