# ReactiveCrudRepository - save()の戻り値を使う理由

## 概要
「保存したら、DBが勝手に値を追加・変更することがあるよ。だから返ってきたものを使ってね」という注意点。

## 使用場面
- エンティティを保存した後、そのエンティティを続けて使用するとき
- ID自動採番を使用しているとき
- `@Version`や監査フィールド（`@CreatedDate`等）を使用しているとき

## コード
```java
// 保存前
User user = new User();
user.setName("田中");
// user.getId() → null

// 保存
Mono<User> savedUser = repository.save(user);

// 保存後（返されたインスタンス）
savedUser.subscribe(u -> {
    u.getId();    // → 123（DBが自動で振った）
    u.getName();  // → "田中"
});
```

## 説明

### DBが自動で変更する値の例
```java
@Entity
public class User {
    @Id
    @GeneratedValue          // ← IDを自動生成
    private Long id;

    @Version                 // ← 保存時にインクリメント
    private Long version;

    @CreatedDate             // ← 作成日時を自動設定
    private LocalDateTime createdAt;

    @LastModifiedDate        // ← 更新日時を自動設定
    private LocalDateTime updatedAt;
}
```

### やりがちなミス
```java
User user = new User();
user.setName("田中");

repository.save(user);

// ❌ ダメ：元のuserにはIDがない
System.out.println(user.getId());  // → null

// ✅ 正しい：返されたインスタンスを使う
repository.save(user)
    .subscribe(saved -> {
        System.out.println(saved.getId());  // → 123
    });
```

### ポイント
**保存前のオブジェクトは古い。保存後に返ってきたものが正解。**

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)
- [楽観的ロック](./OptimisticLocking-楽観的ロック.md)

## 作成日
2026-02-02

## タグ
#spring #spring-data #reactive #save #jpa
