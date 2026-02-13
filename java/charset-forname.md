# Charset.forName()

## 概要
`java.nio.charset.Charset` クラスの静的メソッドで、文字セット名（文字列）から `Charset` オブジェクトを取得する。

## 使用場面
- 設定ファイルやHTTPヘッダから文字セット名を動的に受け取る場合
- `StandardCharsets` に定義されていない文字セット（`Shift_JIS` など）を使う場合

## コード
```java
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

// 文字セット名を指定して取得（大文字小文字を区別しない）
Charset utf8 = Charset.forName("UTF-8");
Charset sjis = Charset.forName("Shift_JIS");
Charset iso8859 = Charset.forName("ISO-8859-1");
```

### よく使う文字セット名
```java
Charset.forName("UTF-8")       // 最も一般的
Charset.forName("UTF-16")
Charset.forName("US-ASCII")
Charset.forName("ISO-8859-1")
Charset.forName("Shift_JIS")   // 日本語
Charset.forName("EUC-JP")      // 日本語
```

### StandardCharsets との比較
```java
// Charset.forName() — 文字列指定（タイポしてもコンパイルは通る）
Charset cs1 = Charset.forName("UTF-8");

// StandardCharsets — 定数（コンパイル時にチェックされる、推奨）
Charset cs2 = StandardCharsets.UTF_8;
```

### 動的に文字セットを受け取る例
```java
// HTTPレスポンスのContent-Typeから文字セットを取得
String charsetName = "Shift_JIS"; // 動的に決まる値
Charset charset = Charset.forName(charsetName);
byte[] bytes = responseBody.getBytes();
String text = new String(bytes, charset);
```

## 説明
- 引数にはIANA登録名またはエイリアスを指定する
- 不正な名前を渡すと `IllegalCharsetNameException`、未サポートの場合は `UnsupportedCharsetException` がスローされる
- `StandardCharsets` で定義されている定数: `UTF_8`, `UTF_16`, `UTF_16BE`, `UTF_16LE`, `US_ASCII`, `ISO_8859_1`
- 標準的な文字セット（UTF-8など）にはタイプセーフな `StandardCharsets` の使用が推奨される

## 参考
- [Charset - Java公式ドキュメント](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/charset/Charset.html)
- [StandardCharsets - Java公式ドキュメント](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/charset/StandardCharsets.html)

## 関連スニペット
- [String.format() 書式指定子](./string-format-specifier.md)

## 作成日
2026-02-13

## タグ
#java #charset #encoding #nio
