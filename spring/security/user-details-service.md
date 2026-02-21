# UserDetailsService - Spring Securityの認証ユーザー取得インターフェース

## 概要

Spring Security が認証時にユーザー情報を取得するためのインターフェース。実装を差し替えることで、DB・メモリ・外部APIなど任意のソースからユーザーを取得できる。

## 使用場面

- 本番環境でDBからユーザー情報を取得して認証するとき
- テスト環境でDBを使わずコード内に直接書いた固定ユーザーで認証するとき
- LDAP・外部APIなど独自のユーザー取得ロジックを組み込むとき

## コード

```java
// インターフェースの定義（Spring Security 側）
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

```java
// 本番用：DBからユーザーを取得する実装
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("ユーザーが見つかりません: " + username);
        }
        return user; // UserインターフェースがUserDetailsを実装していれば直接返せる
    }
}
```

```java
// テスト用：メモリ上のユーザーで認証する実装
@Service
public class TestUserDetailsService implements UserDetailsService {

    private final PasswordEncoder passwordEncoder;

    public TestUserDetailsService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if ("sarah1".equals(username)) {
            return User.builder()
                .username("sarah1")
                .password(passwordEncoder.encode("abc123"))
                .roles("USER")
                .build();
        }
        throw new UsernameNotFoundException("ユーザーが見つかりません: " + username);
    }
}
```

## 説明

### 1. Spring Securityの認証コンポーネント全体像

「ログインする」という処理は、実際には複数のコンポーネントが連携して実現されている。各コンポーネントは担当する責務が明確に分かれている。

```
[ブラウザ]
    │  POST /login  (username=sarah1, password=abc123)
    ↓
SecurityFilterChain（フィルターの列）
    │  リクエストが順番に通過するフィルターの集まり
    │  CSRF検証・セッション確認・認証処理など各フィルターが直列に並んでいる
    ↓
UsernamePasswordAuthenticationFilter
    │  フォームのusernameとpasswordを取り出して認証処理に渡す
    │  Spring Securityが自動で作るフィルター
    ↓
AuthenticationManager
    │  「誰か認証できる人いますか？」と呼びかける窓口
    │  複数のProviderを束ねることができる
    ↓
DaoAuthenticationProvider（AuthenticationProviderの実装）
    │  実際に「このユーザーは本物か？」を確かめる担当
    │  ユーザー情報を取得して、パスワードを照合する
    ↓
UserDetailsService  ← ★ここだけ自分で実装する
    │  「このusernameのユーザー情報をください」という依頼に答える
    │  DBからでも、メモリからでも、どこから取ってもいい
    ↓
UserDetails
    │  取得されたユーザー情報（username, ハッシュ済みpassword, roles）
    │  DaoAuthenticationProviderがパスワード照合に使う
    ↓
[認証成功 or 401]
```

このうち、アプリ開発者が実装するのは `UserDetailsService` だけでよい。それ以外はSpring Securityが提供する。

---

### 2. UserDetailsServiceが担う唯一の責務

`UserDetailsService` のメソッドは1つだけ。

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

このメソッドの責務は「usernameに対応するユーザー情報を返すこと」のみ。パスワードが合っているかの判定はここでは行わない。それは呼び出し元の `DaoAuthenticationProvider` の仕事。

つまり分担はこうなる。

| コンポーネント | 責務 |
|---|---|
| `UserDetailsService` | ユーザー情報をどこから取得するか |
| `DaoAuthenticationProvider` | 取得した情報でパスワードを照合するか |
| `UsernamePasswordAuthenticationFilter` | HTTPリクエストからusername/passwordを取り出すか |

---

### 3. なぜインターフェースになっているのか

`DaoAuthenticationProvider` の内部は以下のようになっている（概略）。

```java
public class DaoAuthenticationProvider {

    // 具体的な実装クラスではなく、インターフェース型で持つ
    private UserDetailsService userDetailsService;

    public Authentication authenticate(String username, String rawPassword) {
        // UserDetailsServiceに「このusernameのユーザー情報をくれ」と依頼するだけ
        UserDetails user = userDetailsService.loadUserByUsername(username);

        // 返ってきたUserDetailsを使ってパスワードを照合する
        if (passwordEncoder.matches(rawPassword, user.getPassword())) {
            return 認証成功オブジェクト;
        }
        throw new BadCredentialsException("パスワードが違う");
    }
}
```

`DaoAuthenticationProvider` は `UserDetailsService` 型（インターフェース型）でフィールドを持っている。実際に何のクラスが入っているかは関知しない。`loadUserByUsername` を呼べばユーザー情報が返ってくる、それだけを信頼する。

この設計のおかげで、`DaoAuthenticationProvider` のコードを一切変えずに、ユーザー情報の取得先だけを差し替えられる。

```
DaoAuthenticationProvider が持つ UserDetailsService の実体

  本番環境 → MyUserDetailsService（DB参照）
  テスト環境 → InMemoryUserDetailsManager（メモリ参照）
  大企業 → LdapUserDetailsService（LDAPサーバー参照）

  どれを差し込んでも DaoAuthenticationProvider は同じコードで動く
```

---

### 4. Spring Bootによる自動接続

Spring Bootには「よく使うBean同士を自動でつなぐ」仕組み（Auto-configuration）がある。`UserDetailsService` もその対象で、DIコンテナに登録するだけで `DaoAuthenticationProvider` に自動で接続される。設定コードは不要。

```java
// @Service を付けてDIコンテナに登録するだけ
// Spring BootがDaoAuthenticationProviderに自動で渡してくれる
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("ユーザーが見つかりません: " + username);
        }
        return user; // UserインターフェースがUserDetailsを実装していれば直接返せる
    }
}
```

ただし、`UserDetailsService` のBeanが複数存在すると、Spring Bootが「どちらを使えばいいか」判断できずに起動時エラーになる。本番とテストで切り替える場合は `@Profile` で環境ごとに1つだけ有効にする。

```java
@Bean
@Profile("!test") // "test"プロファイル以外（本番・開発）で有効
public UserDetailsService productionUsers() {
    // DBから取得する実装を返す
}

@Bean
@Profile("test") // "test"プロファイルのときだけ有効
public UserDetailsService testOnlyUsers() {
    // メモリ上の固定ユーザーを返す
}
```

---

### 5. UserDetailsとは何か

`UserDetailsService` が返す `UserDetails` は、認証に必要なユーザー情報を持つインターフェース。

```java
public interface UserDetails {
    String getUsername();
    String getPassword();           // ハッシュ済みパスワード
    Collection<? extends GrantedAuthority> getAuthorities(); // ロール（ROLE_ADMIN など）
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

DBのエンティティクラスに `implements UserDetails` を付けてこのインターフェースを実装すると、`loadUserByUsername` からエンティティをそのまま返せる。

---

### 6. LDAPとは（大企業のユーザー管理）

LDAP（Lightweight Directory Access Protocol）は会社の「社員名簿サーバー」にアクセスするための規格。

大企業では社内システムが多数あり、それぞれがユーザー情報を持つと管理が煩雑になる。退職者が出たときに全システムから削除する必要があるなど。LDAPで一元管理することで、退職時にLDAPから削除するだけで全システムのアクセスを無効にできる。

```
❌ アプリごとにユーザー管理（管理が大変）
社内システムA：ユーザーDB
社内システムB：ユーザーDB
社内システムC：ユーザーDB

✅ LDAPで一元管理
LDAPサーバー（一つだけ）
    ↑
社内システムA ─┤
社内システムB ─┤← 全部ここを参照
社内システムC ─┘
```

Spring SecurityはLDAPにも対応しており、DBの代わりにLDAPサーバーを参照する `UserDetailsService` 実装が用意されている。アプリ側のコードはほぼ変わらず、`loadUserByUsername` の中でDBを呼ぶかLDAPを呼ぶかが違うだけ。

## 参考

- [Spring Security Reference - UserDetailsService](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details-service.html)
- [Spring Security Reference - InMemoryUserDetailsManager](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html)

## 関連スニペット

- [SecurityFilterChain](./security-filter-chain.md)
- [@Configuration](../annotations/Configuration.md)

## 作成日

2026-02-20

## タグ

#spring #spring-security #authentication #user-details-service #di #interface
