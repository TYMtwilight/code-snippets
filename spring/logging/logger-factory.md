# Logger / LoggerFactory - ログ出力の基本

## 概要
SLF4Jを使ったログ出力の定型パターン。System.out.printlnの上位版として、日時やログレベルを自動で付与してくれる。

## 使用場面
- アプリケーションの動作状況を記録する
- デバッグ情報を出力する
- エラーや警告を記録する

## コード
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
public class MyApplication {

    // おまじない：クラスに1つLoggerを用意
    private static final Logger log = LoggerFactory.getLogger(MyApplication.class);

    public void someMethod() {
        log.info("情報メッセージ");
        log.debug("デバッグメッセージ");
        log.warn("警告メッセージ");
        log.error("エラーメッセージ");

        // 変数を埋め込む
        String name = "Taro";
        log.info("ユーザー {} がログインしました", name);

        // 例外をログに記録
        try {
            // ...
        } catch (Exception e) {
            log.error("処理中にエラーが発生しました", e);
        }
    }
}
```

## 説明
**各パーツの意味**
```java
private static final Logger log = LoggerFactory.getLogger(MyApplication.class);
```

| パーツ | 意味 |
|--------|------|
| `Logger log` | ログを出力するための道具 |
| `LoggerFactory.getLogger(...)` | Loggerを作ってくれる工場 |
| `MyApplication.class` | 「このクラス用のLoggerをください」 |
| `private static final` | クラスに1つあれば十分なので定数として定義 |

**なぜSystem.out.printlnではなくLoggerを使うか**
| 機能 | System.out | Logger |
|------|------------|--------|
| 日時 | 手動で付ける | 自動 |
| ログレベル | なし | INFO, DEBUG, ERROR等 |
| 出力先 | コンソールのみ | ファイル、外部サービス等 |
| 本番環境 | 不向き | 適切 |

**ログレベル**
| レベル | 用途 |
|--------|------|
| `trace` | 最も詳細なデバッグ情報 |
| `debug` | 開発時のデバッグ情報 |
| `info` | 通常の動作情報 |
| `warn` | 警告（動作は継続） |
| `error` | エラー（問題発生） |

**ログ出力例**
```
2024-01-31 10:00:00 INFO  MyApplication : ユーザー Taro がログインしました
```

## 参考
- [SLF4J公式](https://www.slf4j.org/)
- [Spring Boot Logging](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)

## 関連スニペット
- [ApplicationRunner](../annotations/application-runner.md)

## 作成日
2025-01-31

## タグ
#spring #logging #slf4j #logger #debug
