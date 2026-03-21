# REST Controller - 基本パターン（CRUD）

## 概要
`@RestController` と `@RequestMapping` を使った REST API コントローラーの基本パターン。GET / POST / PUT / PATCH / DELETE の 5 メソッドを DTO + Service 層と組み合わせて実装する。

## 使用場面
- リソース（例: TODO、ユーザー、商品）の CRUD API を設計する時
- Bean Validation（`@Valid`）でリクエストを検証したい時
- Service 層に処理を委譲し、Controller をシンプルに保ちたい時

## コード

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {

  private final TodoService todoService;

  // コンストラクタインジェクション
  public TodoController(TodoService todoService) {
    this.todoService = todoService;
  }

  // GET /api/todos?status=ALL|ACTIVE|COMPLETED
  @GetMapping
  public ResponseEntity<List<TodoResponse>> getAll(
      @RequestParam(name = "status", defaultValue = "ALL") String status) {
    return ResponseEntity.ok(todoService.findAll(status));
  }

  // POST /api/todos → 201 Created
  @PostMapping
  public ResponseEntity<TodoResponse> create(
      @Valid @RequestBody TodoRequest request) {
    TodoResponse created = todoService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
  }

  // PUT /api/todos/{id} → 200 OK
  @PutMapping("/{id}")
  public ResponseEntity<TodoResponse> update(
      @PathVariable("id") Long id,
      @Valid @RequestBody TodoRequest request) {
    TodoResponse updated = todoService.update(id, request);
    return ResponseEntity.ok(updated);
  }

  // PATCH /api/todos/{id}/toggle → 200 OK（部分更新）
  @PatchMapping("/{id}/toggle")
  public ResponseEntity<TodoResponse> toggle(@PathVariable("id") Long id) {
    TodoResponse toggled = todoService.toggleComplete(id);
    return ResponseEntity.ok(toggled);
  }

  // DELETE /api/todos/{id} → 204 No Content
  @DeleteMapping("/{id}")
  public ResponseEntity<Void> delete(@PathVariable("id") Long id) {
    todoService.delete(id);
    return ResponseEntity.noContent().build();
  }
}
```

## 説明

### HTTP メソッドと ResponseEntity の対応

| HTTP メソッド | 用途 | 正常時のステータス | ResponseEntity |
|---|---|---|---|
| `@GetMapping` | 一覧取得・単件取得 | 200 OK | `ResponseEntity.ok(body)` |
| `@PostMapping` | 新規作成 | 201 Created | `ResponseEntity.status(HttpStatus.CREATED).body(body)` |
| `@PutMapping` | 全体更新 | 200 OK | `ResponseEntity.ok(body)` |
| `@PatchMapping` | 部分更新 | 200 OK | `ResponseEntity.ok(body)` |
| `@DeleteMapping` | 削除 | 204 No Content | `ResponseEntity.noContent().build()` |

### @Valid でリクエストを検証

```java
@PostMapping
public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoRequest request) {
```

- `@Valid` を付けると `TodoRequest` の Bean Validation アノテーション（`@NotBlank`、`@Size` など）が実行される
- バリデーション失敗時は `MethodArgumentNotValidException` がスローされる
- `@RestControllerAdvice` で一括ハンドリングするのが定石

### @RequestParam のデフォルト値

```java
@RequestParam(name = "status", defaultValue = "ALL") String status
```

- クエリパラメータ未指定時に `"ALL"` が使われる
- `?status=ACTIVE` のように渡すと `"ACTIVE"` になる

### @PathVariable の明示的な name 指定

```java
@PathVariable("id") Long id
```

- メソッド引数名とパス変数名が一致する場合は省略可能だが、明示すると可読性が上がる

## DTO パターン

Entity をそのまま返すのではなく、DTO（Data Transfer Object）で変換して返す。

```
Request JSON → TodoRequest (DTO) → todoService.create() → Entity → TodoResponse (DTO) → Response JSON
```

**理由:**
- Entity のフィールド変更がレスポンス形式に影響しない
- 不要なフィールド（パスワードなど）を返さずに済む
- バリデーションロジックを DTO に閉じ込められる

## 参考
- [Spring Web MVC - Annotated Controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)
- [ResponseEntity](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)

## 関連スニペット
- [エラーハンドリング](./error-handling.md)
- [@WebMvcTest + MockMvc テスト](../test/controller-test.md)
- [CORS 設定](../config/cors-config.md)

## 作成日
2026-03-21

## タグ
#spring #rest-api #controller #restcontroller #requestmapping #responsebody #dto
