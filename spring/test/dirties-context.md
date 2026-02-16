# @DirtiesContext - テストコンテキストのリセット

## 概要
`@DirtiesContext`は、Springのテストコンテキスト（ApplicationContext）を「汚れた」とマークし、再構築をトリガーするアノテーション。テスト間のデータ独立性を保証する。

## 使用場面
- テストがデータベースの状態を変更し、次のテストに影響を与える場合
- 各テストをクリーンな初期状態で実行したい場合
- `@Transactional`によるロールバックが使えないケース（REST API経由のテストなど）

## コード

### 背景: Springテストのコンテキスト再利用

Springはパフォーマンス最適化のため、テストコンテキストを再利用する。

```java
@SpringBootTest
class MyTest {
    @Test void test1() { /* ... */ }
    @Test void test2() { /* ... */ }
    @Test void test3() { /* ... */ }
}
// 3つのテストが同じApplicationContextを共有
// メリット: 高速（起動は1回だけ）
// デメリット: データベースなどの状態が引き継がれる
```

### 問題のケース

```java
@Test
void test1() {
    // ID=1のCashCardを作成
    restTemplate.postForEntity("/cashcards", new CashCard(null, 100.00), Void.class);
}

@Test
void test2() {
    // test1で作成したID=1がまだ残っている！
    ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);
    // 期待: 0件、実際: 1件
}
```

### クラスレベルで全テストをリセット

```java
@SpringBootTest
@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
class CashCardApplicationTests {

    @Test
    void shouldCreateANewCashCard() {
        CashCard newCashCard = new CashCard(null, 250.00);
        restTemplate.postForEntity("/cashcards", newCashCard, Void.class);
    }
    // ← ここでコンテキストがリセットされる

    @Test
    void shouldReturnAllCashCards() {
        // データベースは空の状態で開始（初期データのみ）
    }
}
```

### メソッドレベルで特定テストだけリセット

```java
@SpringBootTest
class CashCardApplicationTests {

    @Test
    void shouldGetCashCard() {
        // データを変更しない → リセット不要
    }

    @Test
    @DirtiesContext  // このテストだけリセット
    void shouldDeleteAllCashCards() {
        // 全削除するテスト → 次のテストに影響を与えないようリセット
    }

    @Test
    void anotherGetTest() {
        // クリーンな状態で実行される
    }
}
```

### 使用モード一覧

| モード | タイミング | 用途 |
|--------|-----------|------|
| `ClassMode.AFTER_EACH_TEST_METHOD` | 各テスト後 | データ変更が多い場合 |
| `ClassMode.BEFORE_EACH_TEST_METHOD` | 各テスト前 | 特定の初期状態が必要 |
| `ClassMode.AFTER_CLASS` | クラス全体の後 | 最後だけクリーンアップ |
| `ClassMode.BEFORE_CLASS` | クラス全体の前 | 最初だけリセット |
| `MethodMode.AFTER_METHOD`（デフォルト） | メソッド後 | 個別メソッドのリセット |
| `MethodMode.BEFORE_METHOD` | メソッド前 | 個別メソッドの事前リセット |

### パフォーマンスへの影響

```java
// 遅い: 毎回リセット → 4回のSpring起動
@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
class MyTest {
    @Test void test1() { }  // 起動 → テスト → シャットダウン
    @Test void test2() { }  // 起動 → テスト → シャットダウン
}

// 速い: コンテキスト再利用 → 1回のSpring起動
class MyTest {
    @Test void test1() { }  // 起動 → テスト
    @Test void test2() { }  // テスト（再利用）
}
```

### 代替案: @Transactional によるロールバック

```java
@SpringBootTest
@Transactional  // 各テスト後に自動ロールバック（コンテキスト再起動不要）
class MyTest {
    @Test
    void test1() {
        // データ変更してもロールバックされる
    }
}
```

## 説明
- Springはテスト高速化のためコンテキストを再利用するが、データベース状態が引き継がれる問題がある
- `@DirtiesContext`はコンテキストを破棄・再構築することでテスト間の独立性を保証する
- 毎回Spring起動が走るためパフォーマンスコストが大きい。必要最小限に使うのがベストプラクティス
- `@Transactional`で代替できるケースではそちらを優先する（REST API経由のテストでは使えない点に注意）

## 参考
- [Spring公式 - @DirtiesContext](https://docs.spring.io/spring-framework/reference/testing/annotations/integration-spring/annotation-dirtiescontext.html)

## 関連スニペット
- [Spring Boot テスト基礎](springboottest-annotation.md)
- [TestRestTemplate](test-rest-template.md)
- [テストファースト](testing-first.md)

## 作成日
2026-02-17

## タグ
#spring #test #annotation #dirtiescontext #context
