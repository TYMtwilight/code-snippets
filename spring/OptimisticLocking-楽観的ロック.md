# Optimistic Locking - 楽観的ロック

## 概要
「多分、他の人と同時に更新することはないだろう」という前提で動く排他制御の仕組み。バージョン番号で同時更新の衝突を検知する。

## 使用場面
- 複数ユーザーが同じデータを更新する可能性があるとき
- 同時更新の衝突が少ないと想定されるとき
- 軽量な排他制御が必要なとき

## コード
```java
@Entity
public class Product {
    @Id
    private Long id;

    private int stock;

    @Version  // ← これをつけるだけで楽観的ロックが適用される
    private Long version;
}
```

## 説明

### 問題：同時更新の衝突
```
Aさん: 在庫10を読む → 在庫を9に更新
Bさん:     在庫10を読む → 在庫を9に更新（Aの更新が消える！）
```

### 楽観的ロックの解決方法
バージョン番号で衝突を検知する。

```
① Aさん: 在庫10（version=1）を読む
② Bさん: 在庫10（version=1）を読む
③ Aさん: 更新「version=1のデータを9に変更」→ 成功、version=2になる
④ Bさん: 更新「version=1のデータを9に変更」→ 失敗！（もうversion=2だから）
```

### 衝突時の例外
`@Version`付きエンティティで衝突が発生すると`OptimisticLockingFailureException`がスローされる。

### 「悲観的ロック」との違い
| 楽観的ロック | 悲観的ロック |
|-------------|-------------|
| 衝突は少ないと想定 | 衝突は多いと想定 |
| 更新時にチェック | 読み取り時にロック |
| 軽量・高速 | 確実だが重い |

### ReactiveCrudRepositoryとの関係
`@Version`をつけたエンティティは、`ReactiveCrudRepository`で保存・削除時に自動的に楽観的ロックが適用される。

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)
- [削除時の挙動](./ReactiveCrudRepository-削除時の挙動.md)

## 作成日
2026-02-02

## タグ
#spring #spring-data #jpa #optimistic-locking #version #concurrency
