# @WebMvcTest + MockMvc - Controller テンプレート

## 概要
`@WebMvcTest` で Controller 層だけを起動し、`MockMvc` で HTTP リクエスト/レスポンスを検証するテストパターン。Service 層は `@MockitoBean` でモック化する。

## 使用場面
- Controller が正しい HTTP ステータスコードを返すことを検証する時
- バリデーションエラー（`@Valid`）が正しく 400 を返すことを確認する時
- Service 層を起動せず、高速に Controller のみをテストしたい時

## コード

```java
@WebMvcTest(TodoController.class)
public class TodoControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockitoBean                      // Service をモック化（Spring Boot 3.4+）
  private TodoService todoService;

  @Autowired
  private ObjectMapper objectMapper; // オブジェクト → JSON 変換に使用

  // --- テストデータ作成ヘルパー ---
  private TodoResponse createResponse(Long id, String title, Boolean completed) {
    return new TodoResponse(id, title, completed, LocalDateTime.now(), LocalDateTime.now());
  }

  // --------------------------------------------------
  // GET - 一覧取得
  // --------------------------------------------------

  @Test
  void GET_全件取得_200を返す() throws Exception {
    // Given
    List<TodoResponse> todos = List.of(
        createResponse(1L, "タスク1", false),
        createResponse(2L, "タスク2", true));
    when(todoService.findAll("ALL")).thenReturn(todos);

    // When & Then
    mockMvc.perform(get("/api/todos"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$", hasSize(2)))
        .andExpect(jsonPath("$[0].title").value("タスク1"));
  }

  @Test
  void GET_クエリパラメータ付きで200を返す() throws Exception {
    when(todoService.findAll("ACTIVE"))
        .thenReturn(List.of(createResponse(1L, "未完了タスク", false)));

    mockMvc.perform(get("/api/todos").param("status", "ACTIVE"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$", hasSize(1)))
        .andExpect(jsonPath("$[0].completed").value(false));
  }

  // --------------------------------------------------
  // POST - 作成
  // --------------------------------------------------

  @Test
  void POST_正常なリクエストで201を返す() throws Exception {
    // Given
    TodoRequest request = new TodoRequest();
    request.setTitle("新しいタスク");
    when(todoService.create(any(TodoRequest.class)))
        .thenReturn(createResponse(1L, "新しいタスク", false));

    // When & Then
    mockMvc.perform(post("/api/todos")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.id").value(1))
        .andExpect(jsonPath("$.title").value("新しいタスク"))
        .andExpect(jsonPath("$.completed").value(false));
  }

  @Test
  void POST_タイトル空で400を返す() throws Exception {
    TodoRequest request = new TodoRequest();
    request.setTitle(""); // バリデーション違反

    mockMvc.perform(post("/api/todos")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.status").value(400));
  }

  // --------------------------------------------------
  // PUT - 更新
  // --------------------------------------------------

  @Test
  void PUT_正常な更新で200を返す() throws Exception {
    TodoRequest request = new TodoRequest();
    request.setTitle("更新後タイトル");
    when(todoService.update(eq(1L), any(TodoRequest.class)))
        .thenReturn(createResponse(1L, "更新後タイトル", false));

    mockMvc.perform(put("/api/todos/1")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.title").value("更新後タイトル"));
  }

  @Test
  void PUT_存在しないIDで404を返す() throws Exception {
    TodoRequest request = new TodoRequest();
    request.setTitle("更新");
    when(todoService.update(eq(999L), any(TodoRequest.class)))
        .thenThrow(new RuntimeException("TODO が見つかりません：id = 999"));

    mockMvc.perform(put("/api/todos/999")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.status").value(404));
  }

  // --------------------------------------------------
  // PATCH - 部分更新（トグル）
  // --------------------------------------------------

  @Test
  void PATCH_トグルで200を返す() throws Exception {
    when(todoService.toggleComplete(1L))
        .thenReturn(createResponse(1L, "タスク", true));

    mockMvc.perform(patch("/api/todos/1/toggle"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.completed").value(true));
  }

  // --------------------------------------------------
  // DELETE - 削除
  // --------------------------------------------------

  @Test
  void DELETE_正常な削除で204を返す() throws Exception {
    doNothing().when(todoService).delete(1L);

    mockMvc.perform(delete("/api/todos/1"))
        .andExpect(status().isNoContent());
  }

  @Test
  void DELETE_存在しないIDで404を返す() throws Exception {
    doThrow(new RuntimeException("TODO が見つかりません：id = 999"))
        .when(todoService).delete(999L);

    mockMvc.perform(delete("/api/todos/999"))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.status").value(404));
  }
}
```

## 説明

### @WebMvcTest の特徴

| 項目 | 内容 |
|---|---|
| 起動範囲 | Controller 層のみ（Service / Repository は起動しない）|
| 速度 | `@SpringBootTest` より高速 |
| 用途 | HTTP リクエスト/レスポンスの検証 |
| DB | 起動しない |

### MockMvc メソッド早見表

```java
// HTTP メソッド
mockMvc.perform(get("/path"))
mockMvc.perform(post("/path"))
mockMvc.perform(put("/path/1"))
mockMvc.perform(patch("/path/1/action"))
mockMvc.perform(delete("/path/1"))

// リクエスト設定
.param("key", "value")                          // クエリパラメータ
.contentType(MediaType.APPLICATION_JSON)        // Content-Type ヘッダー
.content(objectMapper.writeValueAsString(obj)) // リクエストボディ（JSON）

// レスポンス検証
.andExpect(status().isOk())          // 200
.andExpect(status().isCreated())     // 201
.andExpect(status().isNoContent())   // 204
.andExpect(status().isBadRequest())  // 400
.andExpect(status().isNotFound())    // 404
.andExpect(jsonPath("$.field").value("値"))
.andExpect(jsonPath("$", hasSize(2)))
```

### @MockitoBean（Spring Boot 3.4+）

- Spring Boot 3.4 以降は `@MockBean`（deprecated）の代わりに `@MockitoBean` を使用する
- Spring コンテキストに Bean として登録されたモックを注入する

```java
@MockitoBean
private TodoService todoService;

// モックの振る舞いを設定
when(todoService.findAll("ALL")).thenReturn(todos);
doNothing().when(todoService).delete(1L);
doThrow(new RuntimeException("...")).when(todoService).delete(999L);
```

### void メソッドのモック

```java
// void メソッドは when().thenReturn() が使えない
doNothing().when(todoService).delete(1L);
doThrow(new RuntimeException("エラー")).when(todoService).delete(999L);
```

## 参考
- [Spring Boot - Testing Web Layer](https://spring.io/guides/gs/testing-web/)
- [MockMvc - 公式ドキュメント](https://docs.spring.io/spring-framework/reference/testing/spring-mvc-test-framework.html)

## 関連スニペット
- [REST Controller 基本パターン](../controller/rest-controller-basic.md)
- [エラーハンドリング](../controller/error-handling.md)
- [Mockito - Service テスト](./service-test.md)

## 作成日
2026-03-21

## タグ
#spring #test #webmvctest #mockmvc #mockitobean #controller-test #junit5
