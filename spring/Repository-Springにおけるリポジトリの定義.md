# Repository - Springにおけるリポジトリの定義

## 概要
リポジトリはデータアクセス層を抽象化するインターフェース。ドメイン駆動設計（DDD）のリポジトリパターンに基づいている。

## 使用場面
- データベースへのCRUD操作を行うとき
- データアクセスの詳細を隠蔽したいとき
- ドメインロジックとデータ永続化を分離したいとき

## コード
```java
// インターフェースを定義するだけでOK
public interface UserRepository extends ReactiveCrudRepository<User, Long> {
    // 基本的なCRUDメソッドは自動で使える
    // カスタムメソッドも追加可能
    Flux<User> findByName(String name);
}
```

## 説明

### 基本的な役割
```
アプリケーション層  →  リポジトリ  →  データストア（DB等）
                    （データアクセスを抽象化）
```

- **永続化の詳細を隠蔽**: SQLやデータベース固有の操作を意識せずにエンティティを扱える
- **コレクションのように振る舞う**: エンティティの保存・取得・削除をシンプルなメソッドで提供

### Spring Dataのリポジトリ階層
```
Repository<T, ID>                    ← マーカーインターフェース（メソッドなし）
    │
    ├── CrudRepository<T, ID>        ← 同期的なCRUD操作
    │       │
    │       └── ListCrudRepository   ← Listを返すCRUD操作
    │
    └── ReactiveCrudRepository<T, ID> ← リアクティブなCRUD操作（Mono/Flux）
```

### ポイント
Springが実行時に実装クラスを自動生成してくれるため、インターフェースを定義するだけでデータアクセスが可能になる。

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)
- [CrudRepository](./CrudRepository.md)

## 作成日
2026-02-02

## タグ
#spring #spring-data #repository #ddd
