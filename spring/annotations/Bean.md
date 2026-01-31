# @Bean - オブジェクトをSpringに管理させる

## 概要
メソッドの戻り値をSpringコンテナにBeanとして登録するアノテーション。Springが生成・管理するオブジェクトを定義する。

## 使用場面
- 外部ライブラリのクラスをBeanにする（自分で@Componentを付けられない場合）
- 初期化処理が必要なオブジェクトをBeanにする
- 設定値を使ってオブジェクトを生成する

## コード
```java
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    // 基本的な使い方
    @Bean
    public RestClient restClient() {
        return RestClient.builder()
            .baseUrl("http://localhost:8080")
            .build();
    }

    // 引数で他のBeanを受け取る（自動でDIされる）
    @Bean
    public UserService userService(UserRepository repository) {
        return new UserService(repository);
    }

    // Bean名を指定する
    @Bean("customClient")
    public RestClient anotherClient() {
        return RestClient.builder()
            .baseUrl("http://api.example.com")
            .build();
    }
}
```

## 説明
**@Bean の仕組み**
```
アプリ起動
    ↓
Spring「@Bean が付いたメソッドを探そう」
    ↓
Spring「restClient() に @Bean があるな」
    ↓
Spring「メソッドを呼び出して戻り値を取得しよう」
    ↓
restClient() 実行 → RestClient オブジェクトが返される
    ↓
Spring「このオブジェクトを 'restClient' という名前で管理しよう」
    ↓
他のクラスで @Autowired や コンストラクタ注入で使える
```

**なぜ@Beanが必要か**
```java
// ❌ 外部ライブラリのクラスには @Component を付けられない
@Component  // RestClientのソースコードは変更できない
public class RestClient { ... }

// ✅ @Bean でSpringに管理させる
@Bean
public RestClient restClient() {
    return RestClient.builder().build();
}
```

**@Component との違い**

どちらも**Beanを作る**という同じ目的。違いは「誰がインスタンスを作るか」。

| アノテーション | 付ける場所 | インスタンス生成 | 使い分け |
|--------------|-----------|----------------|---------|
| @Component | クラス定義 | **Springが** `new` する | 自分で作ったクラス |
| @Bean | メソッド | **自分で** `new` する | 外部ライブラリ、初期化が必要なもの |

```java
// @Component → Springが勝手に new MyService() する
@Component
public class MyService { ... }

// @Bean → 自分で new する（初期化処理を挟める）
@Bean
public RestClient restClient() {
    return RestClient.builder()      // ← 自分でビルド
        .baseUrl("http://...")
        .timeout(Duration.ofSeconds(30))
        .build();
}
```

**なぜ外部ライブラリには @Component を付けられないのか**

`@Component` はクラス定義の上に書く必要がある。
```java
@Component  // ← ソースコードを編集して書く
public class MyService { }
```

しかし外部ライブラリ（RestClient など）のソースコードはJARファイルの中にあり、編集できない。
```
あなたのプロジェクト
├── src/
│   └── MyService.java  ← 編集できる → @Component を付けられる
│
└── ライブラリ（JAR）
    └── RestClient.class  ← 編集できない → @Component を付けられない
```

だから `@Bean` を使って「このメソッドの戻り値をBeanにして」と伝える。

**選び方のフロー**
```
目的: Beanを作りたい
    │
    ├─ 自分のクラス？ → @Component（Springに任せる）
    │
    └─ 外部ライブラリ or 初期化が必要？ → @Bean（自分で作る）
```

**引数は自動でDIされる**
```java
@Bean
public UserService userService(UserRepository repo, PasswordEncoder encoder) {
    // repo と encoder は Spring が自動で渡してくれる
    return new UserService(repo, encoder);
}
```

**Bean名のルール**
```java
@Bean
public RestClient restClient() { }     // Bean名: "restClient"（メソッド名）

@Bean("myClient")
public RestClient restClient() { }     // Bean名: "myClient"（指定した名前）

@Bean(name = "myClient")
public RestClient restClient() { }     // 同じ意味
```

**図でまとめ**
```
┌─────────────────────────────────────────────────────┐
│                 Spring コンテナ                      │
│                                                     │
│   ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│   │ restClient  │  │ userService │  │ myService │  │
│   │  (Bean)     │  │   (Bean)    │  │  (Bean)   │  │
│   └─────────────┘  └─────────────┘  └───────────┘  │
│         ↑                ↑               ↑         │
│         │                │               │         │
│      @Bean            @Bean         @Component     │
│                                                     │
└─────────────────────────────────────────────────────┘
                          ↓
              必要な場所に自動で注入（DI）
```

## 参考
- [Spring @Bean Annotation](https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html)
- [Spring Boot Reference - Creating Your Own Auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)

## 関連スニペット
- [ApplicationRunner](./application-runner.md)
- [@Profile](./Profile.md)
- [@Autowired](./Autowired.md)
- [コンストラクタインジェクション](./ConstuructorInjection.md)

## 作成日
2025-01-31

## タグ
#spring #bean #di #dependency-injection #configuration #annotations
