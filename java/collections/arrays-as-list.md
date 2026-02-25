# Arrays.asList - 配列・可変長引数から固定サイズ List を作成

## 概要
`java.util.Arrays` の静的メソッド。配列や可変長引数から **固定サイズの List** を手軽に作成できる。

## 使用場面
- テスト用のダミーデータや初期値リストをインラインで作りたいとき
- 配列を List として扱いたいとき
- 小規模な定数リストをシンプルに定義したいとき

## コード
```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public class ArraysAsListExample {

    public static void main(String[] args) {

        // ===== 基本的な使い方 =====

        // 可変長引数から List を作成
        List<String> list = Arrays.asList("apple", "banana", "cherry");

        // 配列から List を作成
        String[] array = {"a", "b", "c"};
        List<String> list2 = Arrays.asList(array);

        // ===== 操作の可否 =====

        list.set(0, "x");   // OK: 既存要素の上書きは可能 → ["x", "banana", "cherry"]
        list.add("d");      // エラー！ UnsupportedOperationException（サイズ変更不可）
        list.remove(0);     // エラー！ UnsupportedOperationException（サイズ変更不可）

        // ===== 配列との連動 =====

        String[] src = {"A", "B", "C"};
        List<String> linked = Arrays.asList(src);
        src[0] = "Z";
        System.out.println(linked.get(0)); // "Z"（配列と List は同じバッキング配列を共有）

        // ===== 完全に可変な List が欲しい場合 =====

        // ArrayList でラップすることで add / remove が可能になる
        List<String> mutableList = new ArrayList<>(Arrays.asList("a", "b", "c"));
        mutableList.add("d");    // OK
        mutableList.remove("a"); // OK
    }

    // ===== Spring Boot サービス層でのよくある使用例 =====

    // テスト・ダミーデータを返す
    public List<String> getDummyItems() {
        return Arrays.asList("商品A", "商品B", "商品C");
    }
}
```

## 説明

### 返される List の性質

| 操作 | 結果 |
|------|------|
| `set()` による要素の上書き | OK |
| `add()` による追加 | `UnsupportedOperationException` |
| `remove()` による削除 | `UnsupportedOperationException` |
| `null` 要素の格納 | OK |
| 元配列との連動 | 元配列の変更が List に反映される |

### 可変 List が必要なとき
`Arrays.asList` が返す List はサイズ固定のため、要素の追加・削除が必要な場合は `ArrayList` でラップする。

```java
// 変更可能なリストが必要な場合
List<String> mutableList = new ArrayList<>(Arrays.asList("a", "b", "c"));
```

### Java 9 以降の代替手段
Java 9+ では `List.of()` が利用可能。こちらは `null` も `set()` も許容しない完全な不変リストを返す。

```java
// Java 9+: 完全不変リスト（null 不可、set() も不可）
List<String> immutable = List.of("a", "b", "c");
```

## 参考
- [Arrays.asList (Java SE 17 & JDK 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Arrays.html#asList(T...))
- [List.of (Java SE 17 & JDK 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html#of(E...))

## 関連スニペット
- [BigDecimal](../types/bigdecimal-basics.md)

## 作成日
2026-02-26

## タグ
#java #collections #list #arrays #asList #固定リスト
