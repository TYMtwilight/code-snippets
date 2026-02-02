# Reactive Paradigm - リアクティブパラダイム

## 概要
「データが届いたら教えて、その間は別のことしてるから」というスタイル。非同期・ノンブロッキングでデータを扱う「待たない」プログラミングモデル。

## 使用場面
- 大量の同時接続を処理するAPI
- マイクロサービス間の通信
- リアルタイムデータ処理（チャット、通知など）

## コード
```java
// 従来: 結果が返るまで止まる
User user = repository.findById(1);  // ここで待機...
System.out.println(user.getName());

// リアクティブ: 「届いたらこれやって」と予約だけしておく
repository.findById(1)
    .subscribe(user -> System.out.println(user.getName()));
// すぐ次の処理へ進める
```

## 説明

### レストランで例えると
```
【従来の同期処理】
カウンターで注文 → 料理ができるまでその場で立って待つ → 受け取る
                   ↑ 何もできない

【リアクティブ】
注文して番号札をもらう → 席で別のことをする → 呼ばれたら取りに行く
                         ↑ 待ち時間を有効活用
```

### 従来の同期処理との比較
```
【同期（ブロッキング）】
スレッド: リクエスト → [待機中...] → レスポンス → 次の処理
          ↑ DBやAPIの応答を待つ間、スレッドが占有される

【リアクティブ（ノンブロッキング）】
スレッド: リクエスト → 他の処理へ → (データ到着時) → レスポンス処理
          ↑ 待機中も他の処理ができる
```

### 核となる概念
| 概念 | 説明 |
|------|------|
| **非同期** | 処理の完了を待たずに次へ進む |
| **ノンブロッキング** | スレッドを占有しない |
| **データストリーム** | データを流れとして扱う |
| **バックプレッシャー** | 受信側が処理速度を制御できる |

### メリット
- **少ないスレッドで高スループット**: I/O待ちでスレッドを無駄にしない
- **スケーラビリティ**: 大量の同時接続を効率的に処理
- **リソース効率**: メモリ・CPU使用量の削減

## 参考
- [ReactiveCrudRepository公式ドキュメント](https://spring.pleiades.io/spring-data/commons/docs/current/api/org/springframework/data/repository/reactive/ReactiveCrudRepository.html)

## 関連スニペット
- [Project Reactor型](./ProjectReactor-MonoとFlux.md)
- [ReactiveCrudRepository](./ReactiveCrudRepository.md)

## 作成日
2026-02-02

## タグ
#spring #reactive #webflux #async #non-blocking
