# SecurityFilterChain - HTTPリクエストへのセキュリティ処理の定義

## 概要

Webアプリへのリクエストが届いたとき、コントローラーに到達する前に「このページは誰でも見ていい？ログインが必要？」といったチェックを行う仕組み。

`SecurityFilterChain` はそのチェックのルールをまとめたもので、`HttpSecurity` を使ってルールを組み立て、最後に `http.build()` で完成させる。

```
ブラウザ → SecurityFilterChain（ルールに従ってチェック） → コントローラー
```

## 使用場面

- ログインが必要なページ・不要なページを分けたいとき
- ログイン・ログアウトの動きをカスタマイズしたいとき

## コード

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()  // /public/ 以下は誰でもアクセス可
                .anyRequest().authenticated()               // それ以外はログイン必須
            )
            .formLogin(form -> form
                .loginPage("/login").permitAll()            // ログインページのURLを指定
            )
            .logout(logout -> logout
                .logoutUrl("/logout")                       // ログアウトのURLを指定
            );

        return http.build();  // ここでSecurityFilterChainが完成する
    }
}
```

## 説明

### 処理の流れ

リクエストが来ると、コントローラーの前にセキュリティのチェックが入る。

```
ブラウザからリクエスト
    ↓
┌──────────────────────────────┐
│      SecurityFilterChain     │
│                              │
│  ① ログイン済みか確認         │
│  ② このURLにアクセスできるか  │
│  ③ セッションを管理           │
└──────────────────────────────┘
    ↓（問題なければ）
コントローラーで処理
```

### `authorizeHttpRequests` でURLごとにアクセス制限を設定する

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/public/**").permitAll()   // /public/ 以下は認証不要
    .requestMatchers("/admin/**").hasRole("ADMIN") // /admin/ は ADMIN ロールのみ
    .anyRequest().authenticated()               // 上記以外はログインが必要
)
```

上から順に評価されるので、**より具体的なパスを先に書く**。

### `formLogin` でログイン画面を設定する

```java
.formLogin(form -> form
    .loginPage("/login")          // ログイン画面のURL（デフォルトは /login）
    .defaultSuccessUrl("/home")   // ログイン成功後のリダイレクト先
    .permitAll()                  // ログイン画面自体は誰でもアクセス可にする
)
```

`permitAll()` を忘れると、ログイン画面にアクセスするためにもログインが必要という状態になるので注意。

### `logout` でログアウトを設定する

```java
.logout(logout -> logout
    .logoutUrl("/logout")               // ログアウトのURL（デフォルトは /logout）
    .logoutSuccessUrl("/login?logout")  // ログアウト後のリダイレクト先
)
```

### `HttpSecurity` との関係

| | 役割 |
|---|---|
| `HttpSecurity` | ルールを記述するビルダー（設定の組み立て役） |
| `SecurityFilterChain` | `http.build()` で完成した設定の実体 |

`HttpSecurity` で「どう守るか」を書き、`build()` を呼ぶことで `SecurityFilterChain` が完成してSpringに登録される。

## 参考

- [Spring Security Reference - Security Filter Chain](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-securityfilterchain)

## 関連スニペット

- [@Configuration](../annotations/Configuration.md)

## 作成日

2026-02-20

## タグ

#spring #spring-security #security #filter #authentication #authorization
