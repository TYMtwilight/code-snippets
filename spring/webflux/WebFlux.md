# Flux - リアクティブストリーム型（0個以上の要素）

## 概要
Spring WebFluxで使用されるリアクティブプログラミングの型で、0個以上の要素を非同期的にストリーム配信するProject Reactorのクラス

## 使用場面
- Spring WebFluxで複数のエンティティを非同期的に返すエンドポイント
- データベースから複数レコードを取得する非同期処理
- 無限ストリームやリアルタイムデータの配信
- リアクティブなデータパイプライン構築

## コード
```java
// 基本的なFluxの作成
Flux<String> flux = Flux.just("A", "B", "C");

// 空のFlux
Flux<String> empty = Flux.empty();

// 無限ストリーム
Flux<Long> infinite = Flux.interval(Duration.ofSeconds(1));

// WebFluxコントローラーでの使用例
@RestController
public class UserController {
    
    @Autowired
    private ReactiveUserRepository userRepository;
    
    // 複数のユーザーを非同期的に返す
    @GetMapping("/users")
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    // Fluxの変換・フィルタリング
    @GetMapping("/active-users")
    public Flux<User> getActiveUsers() {
        return userRepository.findAll()
            .filter(user -> user.isActive())
            .map(user -> new UserDTO(user));
    }
}

// MonoとFluxの使い分け
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return userRepository.findById(id); // 単一要素 → Mono
}

@GetMapping("/users")
public Flux<User> getUsers() {
    return userRepository.findAll(); // 複数要素 → Flux
}
```

## 説明
**MonoとFluxの違い**
- **Mono**: 0個または1個の要素（`Optional`や単一オブジェクトに相当）
- **Flux**: 0個以上の要素（`List`や`Stream`に相当）

**従来のSpring MVCとの比較**
```java
// 同期的（Spring MVC）
@GetMapping("/users")
public List<User> getUsers() {
    return userRepository.findAll(); // ブロッキング
}

// 非同期的（Spring WebFlux）
@GetMapping("/users")
public Flux<User> getUsers() {
    return userRepository.findAll(); // ノンブロッキング
}
```

**重要なポイント**
- Spring WebFluxとProject Reactorが必要（通常のSpring MVCでは使用しない）
- ノンブロッキングI/Oで高スループットを実現
- バックプレッシャー機能により、消費者のペースに合わせてデータを配信
- 学習曲線が急なため、まずは同期的なSpring MVCを習得してから取り組むのが推奨

## 参考
- [Project Reactor公式ドキュメント](https://projectreactor.io/docs/core/release/reference/)
- [Spring WebFlux公式ドキュメント](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Reactor Core API](https://projectreactor.io/docs/core/release/api/)

## 関連スニペット
- Mono - リアクティブストリーム型（0個または1個の要素）
- Spring WebFlux - リアクティブWebフレームワーク
- ReactiveRepository - リアクティブなデータアクセス

## 作成日
2025-01-27

## タグ
#spring #webflux #reactive #flux #project-reactor #async