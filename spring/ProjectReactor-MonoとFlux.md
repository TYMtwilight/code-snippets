# Project Reactor - Mono と Flux

## 概要
Springが採用しているリアクティブプログラミングのライブラリ。主に`Mono`と`Flux`の2つの型を提供する。

## 使用場面
- Spring WebFluxでリアクティブなAPIを構築するとき
- ReactiveCrudRepositoryを使用するとき
- 非同期処理を行うとき

## コード
```java
// Mono: 1件だけ返す（またはなし）
Mono<User> findById(Long id);        // 1人のユーザー or 見つからない

// Flux: 複数件を返す
Flux<User> findAll();                // 全ユーザー（0人以上）

// 使用例
repository.findById(1L)
    .subscribe(user -> System.out.println(user.getName()));

repository.findAll()
    .subscribe(user -> System.out.println(user.getName()));
```

## 説明

### Mono と Flux の違い
```
Mono<T>  → 0件 または 1件 のデータ
Flux<T>  → 0件 〜 N件 のデータ（複数）
```

### 従来の型との対応
| 従来（同期） | Project Reactor（リアクティブ） |
|-------------|-------------------------------|
| `T` | `Mono<T>` |
| `Optional<T>` | `Mono<T>` |
| `List<T>` | `Flux<T>` |
| `void` | `Mono<Void>` |

### なぜ専用の型が必要？
```java
// 普通の型だと「待たない」ができない
User user = findById(1);  // 結果が必要なので待つしかない

// Mono/Fluxは「後で届く約束」を表す
Mono<User> userMono = findById(1);  // まだデータはない、届く予定があるだけ
userMono.subscribe(user -> ...);     // 届いたときの処理を登録
```

### イメージ
`Mono`/`Flux`は **「今はないけど、後でデータが届く箱」** のようなもの。

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)
- [Project Reactor公式](https://projectreactor.io/)

## 関連スニペット
- [リアクティブパラダイム](./ReactiveParadigm-リアクティブパラダイム.md)
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)

## 作成日
2026-02-02

## タグ
#spring #reactive #webflux #project-reactor #mono #flux
