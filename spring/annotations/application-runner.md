# ApplicationRunner - アプリ起動時に実行する処理

## 概要
Springアプリケーション起動時に1回だけ実行したい処理を定義するインターフェース。初期化処理やデータ投入などに使用する。

## 使用場面
- アプリ起動時にAPIからデータを取得する
- 初期データをDBに投入する
- 外部サービスへの接続確認を行う

## コード
```java
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MyApplication {

    private static final Logger log = LoggerFactory.getLogger(MyApplication.class);

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    // 起動時に実行される処理
    @Bean
    public ApplicationRunner run(RestClient.Builder builder) {
        RestClient restClient = builder.baseUrl("http://localhost:8080").build();

        return args -> {
            // ここが起動時に実行される
            Quote quote = restClient
                .get().uri("/api/random")
                .retrieve()
                .body(Quote.class);
            log.info(quote.toString());
        };
    }
}
```

## 説明
**構造の理解**
```java
@Bean
public ApplicationRunner run(RestClient.Builder builder) {
    // ① 準備処理（Beanが作られるとき）
    RestClient restClient = builder.baseUrl("...").build();

    // ② 「起動時にやること」を返す
    return args -> {
        // ③ この中身はSpringが起動完了後に実行する
    };
}
```

**タイミング**
```
アプリ起動
    ↓
Spring「@Beanがあるな。runメソッドを呼ぼう」
    ↓
① RestClientを準備
    ↓
② ApplicationRunnerを返す（まだ中身は実行されない）
    ↓
Spring「準備完了。ApplicationRunnerを実行するぞ」
    ↓
③ API呼び出し、結果表示
```

**ラムダ式の展開**
```java
// ラムダ式
return args -> { /* 処理 */ };

// 同じ意味（展開形）
return new ApplicationRunner() {
    @Override
    public void run(ApplicationArguments args) {
        /* 処理 */
    }
};
```

**依存性注入（DI）**
```java
// 引数に書くだけでSpringが渡してくれる
public ApplicationRunner run(RestClient.Builder builder) {
    // builderを自分でnewしなくていい
}
```

**@Profile との組み合わせ**
```java
@Bean
@Profile("!test")  // テスト時は実行しない
public ApplicationRunner run(RestClient.Builder builder) {
    // ...
}
```

## 参考
- [Spring Boot - Application Events and Listeners](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)

## 関連スニペット
- [@Profile](./Profile.md)
- [@Bean](./Bean.md)
- [RestClient基本](../rest-client/rest-client-basics.md)
- [Logger/LoggerFactory](../logging/logger-factory.md)

## 作成日
2025-01-31

## タグ
#spring #application-runner #startup #initialization #bean
