# SecurityFilterChain - HTTPリクエストへのセキュリティ処理の定義

## 概要
HTTPリクエストに対して「どのセキュリティ処理を、どの順番で適用するか」を定義したもの。`HttpSecurity` で設定を組み立て、`build()` で生成する。

## 使用場面
- 認証・認可のルールを設定するとき
- APIとWeb画面でセキュリティ設定を分けたいとき
- ログイン・ログアウトの挙動をカスタマイズするとき

## コード
```java
// 基本的な定義
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()  // 誰でもアクセス可
                .anyRequest().authenticated()               // それ以外は認証必須
            )
            .formLogin(form -> form
                .loginPage("/login").permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
            );

        return http.build();
    }
}
```

```java
// 複数チェーンを定義する（APIとWeb画面で設定を分ける）
@Bean
@Order(1)
public SecurityFilterChain apiChain(HttpSecurity http) throws Exception {
    http.securityMatcher("/api/**")   // /api/** にだけ適用
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .sessionManagement(s -> s.sessionCreationPolicy(STATELESS)); // JWTなどで使う
    return http.build();
}

@Bean
@Order(2)
public SecurityFilterChain webChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

## 説明

**リクエストの流れ**
```
リクエスト
    ↓
┌─────────────────────────────┐
│      SecurityFilterChain    │
│                             │
│  Filter1: 認証トークン確認   │
│      ↓                      │
│  Filter2: 認可チェック       │
│      ↓                      │
│  Filter3: セッション管理     │
│      ↓                      │
└─────────────────────────────┘
    ↓
コントローラー（処理本体）
```

**`HttpSecurity` との関係**

| | 役割 |
|---|---|
| `HttpSecurity` | ルールを設定するビルダー |
| `SecurityFilterChain` | `http.build()` で完成した設定の実体 |

`HttpSecurity` で「どう守るか」を組み立て、`build()` で `SecurityFilterChain` が完成する。

**複数チェーンの使い分け**

`@Order` で優先度を指定し、`securityMatcher()` で適用するURLパターンを絞る。マッチしたチェーンが使われ、残りは無視される。

```
リクエスト: /api/users
    ↓
@Order(1) apiChain: securityMatcher("/api/**") → マッチ → このチェーンを適用
@Order(2) webChain: 評価されない
```

## 参考
- [Spring Security Reference - Security Filter Chain](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-securityfilterchain)
- [Spring Security Reference - Multiple SecurityFilterChain](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html#_multiple_httpsecurity)

## 関連スニペット
- [@Configuration](../annotations/Configuration.md)

## 作成日
2026-02-20

## タグ
#spring #spring-security #security #filter #authentication #authorization
