# IoC/DI入門 - なぜ制御反転が必要なのか

## 概要
Spring Bootの核心的な仕組みであるIoC（制御反転）とDI（依存性注入）を、「なぜ必要か」という問題意識から理解する入門ガイド。

## 使用場面
- Spring Bootの仕組みを初めて学ぶ時
- 「IoCって何？」「DIって何？」と聞かれた時の説明
- 従来の方法とSpringの方法の違いを理解したい時

## コード

### 従来の方法（IoCなし）の問題点
```java
public class FamilyCashCardController {
    private CashCardRepository repository;

    public FamilyCashCardController() {
        // どのデータベースを使うか、コード内で決める必要がある
        if (System.getenv("ENV").equals("production")) {
            this.repository = new ProductionDatabaseRepository();
        } else {
            this.repository = new LocalDatabaseRepository();
        }
    }
}
```

**問題点：**
- 環境ごとの分岐が**コード内**に書かれている
- 新しい環境（テスト環境など）が増えるたびにコードを修正する必要がある
- テストしにくい（モックに差し替えにくい）

### Spring Bootを使った場合
```java
@RestController
public class FamilyCashCardController {
    private final CashCardRepository repository;

    // Springが自動的に適切なRepositoryを「注入」してくれる
    public FamilyCashCardController(CashCardRepository repository) {
        this.repository = repository;
    }
}
```

**設定ファイル（application.yml）で環境を切り替える：**
```yaml
# ローカル開発環境
spring:
  datasource:
    url: jdbc:h2:mem:testdb

# 本番環境（別ファイルで上書き）
spring:
  datasource:
    url: jdbc:mysql://production-server:3306/cashcard
```

**コントローラーのコードは一切変わらない！**

### 具体例：ファミリーキャッシュカードアプリ
開発環境では**H2メモリDB**、本番では**MySQL**を使いたいシナリオ

**1. インターフェースで抽象化**
```java
public interface CashCardRepository extends CrudRepository<CashCard, Long> {
    // メソッド定義
}
```

**2. Springが環境に応じて自動選択**
- ローカル → `spring.datasource.url=jdbc:h2:mem:testdb` → H2用の実装
- 本番 → `spring.datasource.url=jdbc:mysql://...` → MySQL用の実装

**3. コントローラーは何も知らなくていい**
```java
@RestController
public class CashCardController {
    // どのDBが使われるか知らない（知る必要がない）
    private final CashCardRepository repository;

    public CashCardController(CashCardRepository repository) {
        this.repository = repository; // Springが適切な実装を注入
    }
}
```

## 説明

### 用語の整理

#### IoC（制御反転）とDI（依存性注入）の関係
```
IoC（制御反転） ← より広い概念
  └── DI（依存性注入） ← IoCを実現する具体的な方法の1つ
```

| 用語 | 意味 |
|------|------|
| **IoC** | 「オブジェクトの生成・管理をアプリケーションコードではなく、外部（フレームワーク）に任せる」という**考え方** |
| **DI** | IoCを実現する**具体的な手法**。「必要なオブジェクトを外部から渡してもらう」 |

#### 「厳密には正しくない」について
- **Spring開発者の間では**: IoC ≒ DI として使われる
- **学術的には**: DIはIoCを実現する方法の1つに過ぎない
  - 他の方法：サービスロケーター、イベント駆動など

### 制御反転の本質

> **「Spring Boot では、アプリケーションコードに環境ごとの違いをハードコードせず、設定ファイルで切り替えられる」**

- **従来**: コードが「どの実装を使うか」を**制御**
- **Spring**: フレームワークが「どの実装を使うか」を**制御**（**反転**している）

### メリット
1. **環境依存のコードがなくなる**: if文で環境分岐しなくてよい
2. **設定変更だけで切り替え可能**: コードを変更せずにDB等を変更できる
3. **テストしやすい**: モックを簡単に注入できる
4. **関心の分離**: ビジネスロジックとインフラ設定が分離される

## 参考
- [Spring Framework Reference - IoC Container](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
- [Spring Boot Reference - Dependency Injection](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.spring-beans-and-dependency-injection)

## 関連スニペット
- [Spring IoC Container と Bean の基礎](./spring-ioc-container-and-bean%20.md)
- [Spring設計思想 - 設定による実装切り替え](./spring-provide-choice-at-every-level.md)
- [コンストラクタインジェクション](../annotations/ConstuructorInjection.md)

## 作成日
2026-02-03

## タグ
#spring #ioc #di #dependency-injection #introduction #beginner
