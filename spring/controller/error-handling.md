# @RestControllerAdvice - 統一エラーハンドリング

## 概要
`@RestControllerAdvice` と `@ExceptionHandler` を使い、アプリケーション全体の例外を一箇所で捕捉して統一フォーマットの JSON エラーレスポンスを返すパターン。

## 使用場面
- バリデーションエラー（`@Valid`）を 400 Bad Request で返したい時
- 存在しないリソースへのアクセスを 404 Not Found で返したい時
- 各 Controller にエラーハンドリングを散在させたくない時

## コード

### GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

  // バリデーションエラー（@Valid で検出） → 400
  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<ErrorResponse> handleValidation(
      MethodArgumentNotValidException ex) {
    String message = ex.getBindingResult().getFieldErrors().stream()
        .map(FieldError::getDefaultMessage)
        .collect(Collectors.joining(", "));

    ErrorResponse error = new ErrorResponse(
        HttpStatus.BAD_REQUEST.value(),
        "Bad Request",
        message,
        LocalDateTime.now());
    return ResponseEntity.badRequest().body(error);
  }

  // リソース未検出（存在しない ID など） → 404
  @ExceptionHandler(RuntimeException.class)
  public ResponseEntity<ErrorResponse> handleNotFound(RuntimeException ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.NOT_FOUND.value(),
        "Not Found",
        ex.getMessage(),
        LocalDateTime.now());
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
  }
}
```

### ErrorResponse DTO

```java
public record ErrorResponse(
    int status,
    String error,
    String message,
    LocalDateTime timestamp
) {}
```

### リクエスト DTO（Bean Validation）

```java
public class TodoRequest {

  @NotBlank(message = "タイトルは必須です")
  @Size(max = 100, message = "タイトルは100文字以内で入力してください")
  private String title;

  // getter / setter
}
```

## 説明

### @RestControllerAdvice

- `@ControllerAdvice` + `@ResponseBody` の合成アノテーション
- `@Controller`（`@RestController` 含む）全体に横断的に適用される
- 個々の Controller にエラーハンドリングを書かなくて済む

### @ExceptionHandler の優先順位

同一クラス内に複数の `@ExceptionHandler` がある場合、**より具体的な例外クラス**が優先される。

```
MethodArgumentNotValidException  ← より具体的（先に検索される）
RuntimeException                 ← より汎用的（後に検索される）
Exception                        ← 最汎用（最後のフォールバック）
```

### エラーレスポンスの形式

```json
{
  "status": 400,
  "error": "Bad Request",
  "message": "タイトルは必須です",
  "timestamp": "2026-03-21T10:00:00"
}
```

### バリデーションエラーメッセージの結合

複数フィールドでバリデーションエラーが発生した場合、メッセージをカンマ区切りで結合する。

```java
String message = ex.getBindingResult().getFieldErrors().stream()
    .map(FieldError::getDefaultMessage)
    .collect(Collectors.joining(", "));
// 例: "タイトルは必須です, タイトルは100文字以内で入力してください"
```

## 発展

より細かく例外クラスを分けると管理しやすくなる。

```java
// カスタム例外を用意する場合
public class ResourceNotFoundException extends RuntimeException {
  public ResourceNotFoundException(String message) {
    super(message);
  }
}

// Service 側
throw new ResourceNotFoundException("TODO が見つかりません：id = " + id);

// Handler 側
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) { ... }
```

## 参考
- [Spring Web MVC - @ControllerAdvice](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-advice.html)
- [Bean Validation - 公式仕様](https://beanvalidation.org/)

## 関連スニペット
- [REST Controller 基本パターン](./rest-controller-basic.md)
- [@WebMvcTest + MockMvc テスト](../test/controller-test.md)

## 作成日
2026-03-21

## タグ
#spring #error-handling #restcontrolleradvice #exceptionhandler #validation #bean-validation
