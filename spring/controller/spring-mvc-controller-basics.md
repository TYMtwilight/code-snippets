# Spring MVC 基本コントローラー

## 概要
Spring MVCでHTMLビューを返すコントローラーの基本形。Modelを使ってビューにデータを渡す。

## 使用場面
- Webページを表示するアプリケーション
- ThymeleafなどのテンプレートエンジンでHTMLを生成する場合

## コード
```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ViewController {

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("message", "Hello, Spring MVC!");
        return "index";  // templates/index.html を返す
    }
}
```

## 説明

### @Controller vs @RestController
| アノテーション | 戻り値 | 用途 |
|---|---|---|
| `@Controller` | ビュー名（String） | HTMLページ表示 |
| `@RestController` | JSON/文字列 | REST API |

### Modelの役割
- **スコープ**: 1リクエスト限り（Sessionではない）
- **Servletでの相当**: `request.setAttribute()`
- **役割**: DTOなどのデータを入れてビューに運ぶ「箱」

```java
model.addAttribute("key", value);
// ≒ request.setAttribute("key", value);
```

### HTTPメソッドのマッピング
| アノテーション | 用途 |
|---|---|
| `@GetMapping` | データ取得、ページ表示 |
| `@PostMapping` | データ送信、作成 |
| `@PutMapping` | データ更新 |
| `@DeleteMapping` | データ削除 |

### @RequestMapping
- クラスレベルで共通パスを指定する（**必須ではない**）
- メソッドに`@GetMapping`等があれば省略可能

## 参考
- [Spring MVC 公式ドキュメント](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Thymeleaf 公式](https://www.thymeleaf.org/)

## 関連スニペット
- REST APIコントローラー
- Thymeleafテンプレート基本形

## 作成日
2026-01-29

## タグ
#spring #spring-mvc #controller #thymeleaf #model
