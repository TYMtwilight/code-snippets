# ReactiveCrudRepository - 削除時の挙動

## 概要
`@Version`属性の有無によって、存在しないデータを削除しようとしたときの挙動が異なる。

## 使用場面
- 削除処理のエラーハンドリングを設計するとき
- 楽観的ロックを使用しているかどうかで挙動を理解するとき

## コード
```java
// @Versionなしのエンティティ
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}

// IDが999のユーザーは存在しないとする
repository.deleteById(999L);  // → エラーにならない（何も起きないだけ）

// --------------------------------------------------

// @Versionありのエンティティ
@Entity
public class Product {
    @Id
    private Long id;

    @Version
    private Long version;
}

// バージョン付きエンティティを削除
Product product = new Product();
product.setId(999L);
product.setVersion(1L);

repository.delete(product);  // → OptimisticLockingFailureException！
```

## 説明

### なぜ@Versionなしはエラーにならない？
削除の目的は「そのデータがない状態にすること」なので、**最初からなければ目的達成**という考え方。

### なぜ@Versionありはエラーになる？
`@Version`がある場合は「このバージョンのデータを削除して」という意味になる。
見つからないと「おかしい、誰かが先に変更した？」と判断してエラーになる。

### まとめ表
| 状況 | 存在しないデータの削除 |
|------|----------------------|
| `deleteById(id)` のみ | エラーなし |
| `@Version`なしエンティティ | エラーなし |
| `@Version`ありエンティティ | `OptimisticLockingFailureException` |

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [楽観的ロック](./OptimisticLocking-楽観的ロック.md)
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)

## 作成日
2026-02-02

## タグ
#spring #spring-data #reactive #delete #optimistic-locking
