# @SpringBootApplication

## 概要
Spring Bootアプリケーションのエントリーポイントに付けるアノテーション。`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`の3つをまとめた便利なアノテーション。

## 使用場面
- Spring Bootアプリケーションのメインクラス
- アプリケーションの起動ポイント

## コード
```java
package com.example.restservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestServiceApplication.class, args);
    }
}
```

## 説明

### @SpringBootApplication = 3つのアノテーションの合体

```java
// @SpringBootApplicationは以下と同じ
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class RestServiceApplication { }
```

---

### 1. @Configuration（従業員名簿）

**役割**: Bean（Springが管理するオブジェクト）の定義場所であることを示す

```java
@Configuration
public class AppConfig {

    @Bean  // Springが管理するオブジェクトとして登録
    public MyService myService() {
        return new MyService();
    }
}
```

**何が起こるか**:
- Springが起動時にBeanを作成
- 必要な場所に自動で注入（DI）

---

### 2. @EnableAutoConfiguration（自動設備導入）

**役割**: 入っているライブラリを見て、必要な設定を自動で行う

**例：spring-boot-starter-webが入っている場合**
```
Spring Boot：「あ、Webアプリ作りたいんだね！」
↓
自動で準備：
✓ Tomcatサーバー起動
✓ DispatcherServlet設定（URLとコントローラーの紐付け）
✓ JSON変換機能（Jackson）
✓ エラーハンドリング
```

**DispatcherServletとは**:
```
ブラウザ → HTTPリクエスト
    ↓
DispatcherServlet（交通整理係）
    ↓
/greeting → GreetingController
/users    → UserController
```

---

### 3. @ComponentScan（従業員スカウト）

**役割**: パッケージを探索して、Springが管理すべきクラスを自動で見つける

**探索するマーク**:
```java
@RestController  // コントローラー
@Service         // サービス
@Repository      // リポジトリ
@Component       // コンポーネント
@Configuration   // 設定クラス
```

**探索範囲**:
```
com.example.restservice/           ← @SpringBootApplicationがここ
├── controller/                    ← 探索される
│   └── GreetingController.java
├── service/                       ← 探索される
│   └── GreetingService.java
└── repository/                    ← 探索される
    └── UserRepository.java

com.example.other/                 ← 探索されない（別パッケージ）
```

---

### 起動時の流れ

```
1. アプリ起動
   ↓
2. @SpringBootApplicationを発見
   ↓
3. @ComponentScan
   「GreetingControllerを発見！登録！」
   ↓
4. @EnableAutoConfiguration
   「spring-webがある→Tomcat起動」
   ↓
5. 起動完了
   「http://localhost:8080 で待機中...」
```

### まとめ表

| アノテーション | 一言で言うと |
|---------------|-------------|
| @Configuration | 必要なもののリスト |
| @ComponentScan | 自動で探して登録 |
| @EnableAutoConfiguration | 全部自動で準備 |

## 参考
- [Spring Boot公式](https://spring.io/projects/spring-boot)
- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service)

## 関連スニペット
- [組み込みサーバー](./embedded-server.md)
- [REST Controller](./rest-controller.md)

## 作成日
2026-01-30

## タグ
#spring #spring-boot #annotation #configuration
