# Spring Boot Formクラスでリクエストパラメータを受け取る

## 概要
Spring BootでFormクラス（DTO）を使ってPOSTリクエストのパラメータをまとめて受け取るパターン。

## 使用場面
- HTMLフォームからの送信データを受け取る場合
- リクエストパラメータが多い（3つ以上）場合
- パラメータをオブジェクトとして再利用したい場合

## コード

### Formクラス（DTO）
```java
package com.example.demo.RequestParamSample;

import lombok.Data;

@Data
public class UserForm {
    private String name;
    private String email;
    private Integer age;
}
```

### Controller
```java
package com.example.demo.RequestParamSample;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @PostMapping("/user/register")
    public String register(UserForm form) {
        return "登録完了 - " + form.getName() + "さん(" + form.getEmail() + ")";
    }
}
```

### メインクラス
```java
package com.example.demo.RequestParamSample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RequestParamSampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(RequestParamSampleApplication.class, args);
    }
}
```

## 説明
- `@Data`（Lombok）でGetter/Setter/toString等を自動生成
- Controllerのメソッド引数にFormクラスを指定するだけで自動バインド
- フィールド名とリクエストパラメータ名が一致すれば自動でマッピングされる
- `@ModelAttribute`は省略可能（Spring MVCが自動で適用）

## 参考
- [Spring MVC - @ModelAttribute](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/modelattrib-method-args.html)
- [Lombok @Data](https://projectlombok.org/features/Data)

## 関連スニペット
- @RequestParamで個別にパラメータを受け取る
- @RequestBodyでJSONを受け取る
- @Validatedでバリデーションを行う

## 作成日
2026-01-29

## タグ
#spring #spring-boot #controller #form #dto #lombok #rest-api
