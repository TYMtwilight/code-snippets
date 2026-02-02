# ReactiveCrudRepository - IllegalArgumentException

## 概要
「nullを渡したらエラーになるよ」という注意点。引数自体がnullの場合、またはコレクション内にnullが含まれる場合に発生する。

## 使用場面
- `saveAll()`や`deleteAll()`など、複数のエンティティを扱うメソッドを使用するとき
- 入力値のバリデーションを考慮するとき

## コード
```java
// ケース1: 引数自体がnull
repository.saveAll(null);  // → IllegalArgumentException！

// ケース2: リストの中にnullが含まれる
List<User> users = Arrays.asList(
    new User("田中"),
    null,              // ← これがダメ
    new User("鈴木")
);
repository.saveAll(users);  // → IllegalArgumentException！

// ✅ 正しい使い方
List<User> users = Arrays.asList(
    new User("田中"),
    new User("鈴木")
);
repository.saveAll(users);

// ✅ 空リストもOK（エラーにならない）
repository.saveAll(Collections.emptyList());
```

## 説明

### 発生条件
1. **引数自体がnull**: `saveAll(null)`, `deleteAll(null)` など
2. **コレクション内にnullが含まれる**: `saveAll(Arrays.asList(user1, null, user2))`

### 対象メソッド
- `saveAll(Iterable<S> entities)`
- `saveAll(Publisher<S> entityStream)`
- `deleteAll(Iterable<? extends T> entities)`
- `deleteAll(Publisher<? extends T> entityStream)`
- `deleteAllById(Iterable<? extends ID> ids)`
- `findAllById(Iterable<ID> ids)`
- `findAllById(Publisher<ID> idStream)`

### ポイント
**nullは渡すな。空リストはOK。**

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)

## 作成日
2026-02-02

## タグ
#spring #spring-data #reactive #exception #validation
