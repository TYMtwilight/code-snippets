# @RestController

## 概要
REST APIのエンドポイントを定義するコントローラークラスに付けるアノテーション。HTTPリクエストを受け取り、JSONレスポンスを返す。

## 使用場面
- REST APIの実装
- JSONを返すWebサービス
- フロントエンド（React, Vue等）とのAPI連携

## コード
```java
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(defaultValue = "World") String name) {
        return new Greeting(counter.incrementAndGet(), template.formatted(name));
    }
}
```

## 説明

### @RestController

```java
@RestController
public class GreetingController { }
```

- このクラスがREST APIのコントローラーであることを示す
- 戻り値を自動的にJSONに変換（`@Controller` + `@ResponseBody`と同じ）
- `@ComponentScan`によって自動で検出・登録される

---

### @GetMapping

```java
@GetMapping("/greeting")
public Greeting greeting(...) { }
```

- `GET /greeting`へのリクエストをこのメソッドで処理
- 他のHTTPメソッド用：`@PostMapping`, `@PutMapping`, `@DeleteMapping`

---

### @RequestParam

```java
@RequestParam(defaultValue = "World") String name
```

**URLのクエリパラメータを受け取る**

| URL | name の値 |
|-----|----------|
| `/greeting` | `"World"`（デフォルト値） |
| `/greeting?name=Hiroki` | `"Hiroki"` |
| `/greeting?name=` | `""`（空文字） |

**オプション**:
```java
// デフォルト値あり（オプショナル）
@RequestParam(defaultValue = "World") String name

// 必須パラメータ（デフォルト）
@RequestParam String name  // なければ400エラー

// 明示的にオプション
@RequestParam(required = false) String name  // なければnull
```

---

### 処理の流れ

```
1. クライアント
   GET /greeting?name=Hiroki
       ↓
2. DispatcherServlet
   「/greeting → GreetingControllerだ！」
       ↓
3. GreetingController.greeting("Hiroki")
   counter.incrementAndGet() → 1
   template.formatted("Hiroki") → "Hello, Hiroki!"
   return new Greeting(1, "Hello, Hiroki!")
       ↓
4. Jackson（自動変換）
   Greeting → JSON
       ↓
5. レスポンス
   {"id": 1, "content": "Hello, Hiroki!"}
```

---

### よく使うアノテーション一覧

| アノテーション | 用途 |
|---------------|------|
| `@GetMapping` | データ取得（GET） |
| `@PostMapping` | データ作成（POST） |
| `@PutMapping` | データ更新（PUT） |
| `@DeleteMapping` | データ削除（DELETE） |
| `@RequestParam` | クエリパラメータ `?key=value` |
| `@PathVariable` | パス変数 `/users/{id}` |
| `@RequestBody` | リクエストボディ（JSON）|

---

### 実践例

```java
@RestController
public class UserController {

    // GET /users?status=active
    @GetMapping("/users")
    public List<User> getUsers(@RequestParam(defaultValue = "all") String status) {
        // ...
    }

    // GET /users/123
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // ...
    }

    // POST /users （JSONボディ）
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        // ...
    }

    // DELETE /users/123
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable Long id) {
        // ...
    }
}
```

## 参考
- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service)
- [Spring Web MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)

## 関連スニペット
- [@SpringBootApplication](./spring-boot-application.md)
- [Java Record](../java/record.md)
- [AtomicLong](../java/atomic-long.md)
- [書式指定子](../java/string-format-specifier.md)

## 作成日
2026-01-30

## タグ
#spring #rest-api #controller #annotation
