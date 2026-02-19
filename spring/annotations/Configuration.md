# @Configuration - JavaでSpringの設定を書く

## 概要
「このクラスはBean定義を書く設定クラスだ」とSpringに伝えるアノテーション。XMLの設定ファイルの代わりに、Javaコードでアプリ設定を記述する。

## 使用場面
- `@Bean` メソッドをまとめた設定クラスを作るとき
- `@EnableWebSecurity` や `@EnableTransactionManagement` などの機能を有効化するとき
- 環境ごとに異なるBeanの設定を切り替えるとき

## コード
```java
// 基本的な使い方
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserServiceImpl();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

```java
// @Enable系アノテーションと組み合わせる
@Configuration
@EnableWebSecurity
@EnableTransactionManagement
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .build();
    }
}
```

```java
// @Configuration vs @Component で@Beanを使った場合の違い
@Configuration
public class ConfigA {
    @Bean
    public A a() { return new A(); }

    @Bean
    public B b() {
        return new B(a()); // ← a()を何度呼んでも同一インスタンスが返る（シングルトン保証）
    }
}

@Component
public class ConfigB {
    @Bean
    public A a() { return new A(); }

    @Bean
    public B b() {
        return new B(a()); // ← ここで new A() が実行されてしまう（別インスタンス）
    }
}
```

## 説明

**起動時の流れ**
```
アプリ起動
    ↓
Spring「@Configuration が付いたクラスを探そう」
    ↓
Spring「AppConfig があるな → 読み込もう」
    ↓
Spring「@Bean が付いたメソッドを全部呼び出してBeanに登録しよう」
    ↓
userService, passwordEncoder がDIコンテナに登録される
```

**@Component との違い**

| | `@Configuration` | `@Component` |
|---|---|---|
| 用途 | Bean定義を書く設定クラス | 汎用のコンポーネント登録 |
| `@Bean` メソッドを書く | 主目的 | できるが非推奨 |
| 内部でのメソッド呼び出し | CGLIBプロキシ経由でシングルトン保証 | 毎回 `new` が実行される |

`@Configuration` の内部では、CGLIBプロキシによって `@Bean` メソッドへの呼び出しがインターセプトされる。そのため同一クラス内で `@Bean` メソッドを呼び出しても、Springが管理するシングルトンインスタンスが返る。

**`@Enable〇〇` 系アノテーションは `@Configuration` クラスに付ける**
```java
// ❌ 通常のコンポーネントに付けても動作しない場合がある
@Component
@EnableWebSecurity
public class SomeComponent { ... }

// ✅ @Configuration クラスに付ける
@Configuration
@EnableWebSecurity
public class SecurityConfig { ... }
```

## 参考
- [Spring @Configuration Annotation](https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html)
- [Spring Boot Reference - Java-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html)

## 関連スニペット
- [@Bean](./Bean.md)
- [@Profile](./Profile.md)
- [コンストラクタインジェクション](./ConstuructorInjection.md)

## 作成日
2026-02-20

## タグ
#spring #configuration #bean #di #annotations #cglibproxy
