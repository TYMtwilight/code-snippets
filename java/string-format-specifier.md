# 書式指定子（Format Specifier）

## 概要
文字列の中に変数を埋め込むための目印（`%s`, `%d`など）。`String.format()`や`.formatted()`で値を埋め込む。

## 使用場面
- メッセージテンプレートの作成
- ログ出力のフォーマット
- レポート・帳票の生成
- 数値の桁揃え表示

## コード
```java
// 基本的な使い方
String template = "Hello, %s!";
String result = template.formatted("Hiroki");
// result = "Hello, Hiroki!"

// 複数のパラメータ
String template = "Hello, %s! You are %d years old.";
String result = template.formatted("Hiroki", 38);
// result = "Hello, Hiroki! You are 38 years old."

// String.format()でも同じ結果
String result = String.format("Hello, %s!", "Hiroki");
```

## 説明

### 主な書式指定子

| 指定子 | 型 | 例 |
|--------|------|------|
| `%s` | 文字列（String） | `String.format("Name: %s", "Hiroki")` → `"Name: Hiroki"` |
| `%d` | 整数（decimal） | `String.format("Age: %d", 38)` → `"Age: 38"` |
| `%f` | 浮動小数点数 | `String.format("Price: %f", 99.99)` → `"Price: 99.990000"` |
| `%.2f` | 小数点以下2桁 | `String.format("Price: %.2f", 99.99)` → `"Price: 99.99"` |
| `%b` | 真偽値（boolean） | `String.format("Active: %b", true)` → `"Active: true"` |
| `%c` | 文字（character） | `String.format("Grade: %c", 'A')` → `"Grade: A"` |
| `%%` | %記号そのもの | `String.format("20%%")` → `"20%"` |

### 複数パラメータの対応関係

```java
String template = "Hello, %s! You are %d years old.";
//                        ↓              ↓
String result = template.formatted("Hiroki", 38);
// 順番に対応する
```

### 幅・桁数の指定

```java
// 整数
String.format("%5d", 42);       // → "   42"（5桁、右寄せ）
String.format("%-5d", 42);      // → "42   "（5桁、左寄せ）
String.format("%05d", 42);      // → "00042"（5桁、0埋め）

// 浮動小数点数
String.format("%.2f", 3.14159);     // → "3.14"（小数点以下2桁）
String.format("%8.2f", 3.14159);    // → "    3.14"（全体8桁）

// 文字列
String.format("%10s", "Hi");    // → "        Hi"（10桁、右寄せ）
String.format("%-10s", "Hi");   // → "Hi        "（10桁、左寄せ）
```

### 実践例：レシート

```java
String item1 = String.format("%-20s %6.2f円", "りんご", 150.0);
String item2 = String.format("%-20s %6.2f円", "バナナ", 98.5);
// 出力：
// りんご               150.00円
// バナナ                98.50円
```

### 注意点

```java
// ❌ 型が合わないとエラー
String.format("%d", "Hello");  // エラー！%dは整数用

// ❌ パラメータの数が合わないとエラー
String.format("Hello, %s and %s!", "A");  // エラー！足りない

// ✅ 正しい使い方
String.format("Hello, %s!", "Hiroki");  // OK
```

### 2つの書き方

```java
// 古い書き方（Java 1.5以降）
String result = String.format("Hello, %s!", "Hiroki");

// 新しい書き方（Java 15以降）
String result = "Hello, %s!".formatted("Hiroki");
```

## 参考
- [Formatter (Java SE 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Formatter.html)

## 関連スニペット
- [REST Controller](../spring/rest-controller.md)

## 作成日
2026-01-30

## タグ
#java #string #format #template
