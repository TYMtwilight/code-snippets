# @Profile - 環境ごとに動作を切り替える

## 概要
実行環境（開発、テスト、本番）によって有効にするBeanやコードを切り替えるアノテーション。

## 使用場面
- 開発環境と本番環境でDB接続先を切り替える
- テスト時に外部API呼び出しをスキップする
- 環境ごとに異なる設定を適用する

## コード
```java
// 開発環境でだけ使うBean
@Bean
@Profile("dev")
public DataSource devDataSource() {
    return new H2DataSource();  // ローカル用の軽量DB
}

// 本番環境でだけ使うBean
@Bean
@Profile("prod")
public DataSource prodDataSource() {
    return new PostgreSQLDataSource();  // 本番用DB
}

// テスト以外で動かす（! = 否定）
@Bean
@Profile("!test")
public ApplicationRunner run(RestClient.Builder builder) {
    // テスト時は外部APIを呼ばない
    return args -> {
        // 外部API呼び出し
    };
}

// 複数プロファイル指定
@Bean
@Profile({"dev", "test"})  // devまたはtestで有効
public MockService mockService() {
    return new MockService();
}
```

## 説明
**よく使う書き方**
```java
@Profile("dev")           // devのときだけ
@Profile("prod")          // prodのときだけ
@Profile("test")          // testのときだけ

@Profile("!test")         // test以外（! = NOT）
@Profile("!prod")         // prod以外

@Profile({"dev", "test"}) // devまたはtest
```

**プロファイルの指定方法**

方法1: application.properties
```properties
spring.profiles.active=dev
```

方法2: コマンドライン
```bash
./mvnw spring-boot:run -Dspring.profiles.active=prod
java -jar app.jar --spring.profiles.active=prod
```

方法3: 環境変数
```bash
export SPRING_PROFILES_ACTIVE=prod
```

**設定ファイルも分けられる**
```
application.properties          # 共通設定
application-dev.properties      # dev用
application-prod.properties     # prod用
application-test.properties     # test用
```

**図でまとめ**
```
┌─────────────────────────────────────────┐
│            あなたのアプリ                │
│                                         │
│  @Profile("dev")  → 開発時だけ有効      │
│  @Profile("prod") → 本番時だけ有効      │
│  @Profile("test") → テスト時だけ有効    │
│                                         │
└─────────────────────────────────────────┘
              ↑
              │ spring.profiles.active=???
              │
    ┌─────────┼─────────┐
    │         │         │
   dev       prod      test
  開発環境   本番環境   テスト
```

## 参考
- [Spring Boot Profiles](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles)
- [Spring @Profile](https://docs.spring.io/spring-framework/reference/core/beans/environment.html#beans-definition-profiles)

## 関連スニペット
- [@Bean](./Bean.md)
- [ApplicationRunner](./application-runner.md)

## 作成日
2025-01-31

## タグ
#spring #profile #environment #configuration #annotations
