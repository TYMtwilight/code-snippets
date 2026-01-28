# BigDecimal - 高精度な10進数計算

## 概要
金融計算や精密な10進数演算に使用するJavaの数値型。浮動小数点の丸め誤差を回避できる。

## 使用場面
- 金額計算（消費税、割引、合計金額など）
- 科学技術計算で精度が必要な場合
- 浮動小数点の丸め誤差を許容できない計算

## コード
```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class BigDecimalExample {

    public static void main(String[] args) {
        // ===== 生成方法 =====
        // 文字列から生成（推奨）
        BigDecimal price = new BigDecimal("1000.50");

        // 整数から生成
        BigDecimal quantity = BigDecimal.valueOf(3);

        // NG: doubleから直接生成すると誤差が発生
        // BigDecimal bad = new BigDecimal(0.1); // 0.1000000000000000055511151231...

        // OK: doubleを使う場合はvalueOfを使用
        BigDecimal good = BigDecimal.valueOf(0.1); // 0.1

        // ===== 四則演算 =====
        BigDecimal a = new BigDecimal("100");
        BigDecimal b = new BigDecimal("3");

        // 加算
        BigDecimal sum = a.add(b);           // 103

        // 減算
        BigDecimal diff = a.subtract(b);     // 97

        // 乗算
        BigDecimal product = a.multiply(b);  // 300

        // 除算（スケールと丸めモードを指定）
        BigDecimal quotient = a.divide(b, 2, RoundingMode.HALF_UP);  // 33.33

        // ===== 比較 =====
        BigDecimal x = new BigDecimal("100.00");
        BigDecimal y = new BigDecimal("100");

        // equals: スケールも比較する（100.00 != 100）
        x.equals(y);      // false

        // compareTo: 数値として比較する（100.00 == 100）
        x.compareTo(y);   // 0 (等しい)

        // 比較結果: 負数 = 小さい, 0 = 等しい, 正数 = 大きい
        new BigDecimal("50").compareTo(new BigDecimal("100"));  // -1 (小さい)

        // ===== 丸め処理 =====
        BigDecimal value = new BigDecimal("123.456");

        // 小数点以下2桁に丸め
        BigDecimal rounded = value.setScale(2, RoundingMode.HALF_UP);  // 123.46

        // 主な丸めモード
        // HALF_UP: 四捨五入
        // HALF_DOWN: 五捨六入
        // CEILING: 正の無限大方向へ
        // FLOOR: 負の無限大方向へ
        // DOWN: ゼロ方向へ（切り捨て）
        // UP: ゼロから離れる方向へ（切り上げ）

        // ===== 定数 =====
        BigDecimal zero = BigDecimal.ZERO;   // 0
        BigDecimal one = BigDecimal.ONE;     // 1
        BigDecimal ten = BigDecimal.TEN;     // 10
    }

    // ===== 実用例：消費税計算 =====
    public static BigDecimal calculateTax(BigDecimal price, BigDecimal taxRate) {
        return price.multiply(taxRate).setScale(0, RoundingMode.DOWN);
    }

    public static BigDecimal calculateTotalWithTax(BigDecimal price) {
        BigDecimal taxRate = new BigDecimal("0.10");  // 10%
        BigDecimal tax = calculateTax(price, taxRate);
        return price.add(tax);
    }
}
```

## 説明
### なぜBigDecimalを使うのか
```java
// doubleの丸め誤差
double d1 = 0.1 + 0.2;
System.out.println(d1);  // 0.30000000000000004

// BigDecimalなら正確
BigDecimal b1 = new BigDecimal("0.1").add(new BigDecimal("0.2"));
System.out.println(b1);  // 0.3
```

### 重要なポイント
1. **文字列から生成する**: `new BigDecimal("0.1")` を使用（doubleリテラルを避ける）
2. **除算時は丸めモードを指定**: 指定しないと`ArithmeticException`が発生する可能性
3. **比較は`compareTo`を使う**: `equals`はスケールも比較するため意図しない結果になりやすい
4. **イミュータブル**: 演算結果は新しいオブジェクトとして返される

## 参考
- [BigDecimal (Java SE 17 & JDK 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/BigDecimal.html)
- [RoundingMode (Java SE 17 & JDK 17)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/RoundingMode.html)

## 関連スニペット
- [Math演算](../math/math-operations.md)
- [数値フォーマット](../format/number-format.md)

## 作成日
2026-01-29

## タグ
#java #bigdecimal #数値計算 #金融計算 #精度
