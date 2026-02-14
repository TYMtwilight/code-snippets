# GitHubデバッグIssue運用ガイド

## 🎯 デバッグIssueの目的

1. **思考整理** — エラーを言語化することで理解が深まる
2. **記録** — 解決プロセスが残り、後から参照できる
3. **進捗管理** — どこまで調査したか一目瞭然
4. **共有** — 他の人やAIに見せやすい

---

## 📋 Issueテンプレート作成

### .github/ISSUE_TEMPLATE/debug.yml

```yaml
name: 🐛 デバッグ用Issue
description: エラー解決のための調査・検証用Issue
title: "[Bug] "
labels: ["bug", "debug"]
body:
  - type: markdown
    attributes:
      value: |
        エラー解決のためのIssueです。調査しながら更新していきます。

  - type: textarea
    id: problem
    attributes:
      label: 🐛 問題
      description: 何が起きているか
      placeholder: |
        現象: ユーザー一覧ページにアクセスすると500エラー
        期待: ユーザー一覧が表示される
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: 📋 環境
      placeholder: |
        - OS: macOS Sonoma 14.2
        - Spring Boot: 3.2.1
        - Java: 17
    validations:
      required: true

  - type: textarea
    id: reproduce
    attributes:
      label: 📋 再現手順
      placeholder: |
        1. アプリケーション起動: `./mvnw spring-boot:run`
        2. Postmanでリクエスト送信
        3. → 500エラー発生
    validations:
      required: true

  - type: textarea
    id: error
    attributes:
      label: 📊 エラー情報
      description: エラーメッセージ、ログ、スタックトレース
      render: shell
    validations:
      required: true

  - type: checkboxes
    id: reproduce-rate
    attributes:
      label: 再現条件
      options:
        - label: 毎回発生する
        - label: 特定のデータで発生する
        - label: 特定の環境で発生する
        - label: ランダムに発生する
```

---

## 🔄 Issue運用フロー

### 1. エラー発生 → Issue作成（5分）

**Issueタイトル例**:
- `[Bug] Spring BatchでDBが更新されない`
- `[Bug] Next.js /usersページで画面真っ白`
- `[Bug] APIレスポンスが500エラー`

**初期状態（テンプレートに沿って記入）**:
```markdown
## 🐛 問題

**現象**
Spring Batchのジョブが正常終了するが、DBが更新されない

**期待する動作**
バッチ実行後、usersテーブルのstatusが更新される

## 📋 環境
- Spring Boot: 3.2.1
- Java: 17
- PostgreSQL: 15

## 📋 再現手順
1. `./mvnw spring-boot:run`
2. `curl -X POST http://localhost:8080/api/batch/run`
3. ログで "Job completed successfully" 確認
4. DBを確認: `SELECT * FROM users WHERE id = 1;`
→ statusが "PENDING" のまま

## 📊 エラー情報
なし（正常終了している）

## 再現条件
- [x] 毎回発生する
```

**ラベル設定**:
- `bug` - バグ
- `debug` - デバッグ中
- `backend` / `frontend` - 該当する方
- `priority: high` - 優先度（必要に応じて）

---

### 2. 問題分析 → Issueに追記（10-15分）

**コメントで分析結果を追加**:

```markdown
## 🔍 問題分析

### 情報収集

**デバッガで確認**
- `ItemProcessor.process()`: ✅正しく呼ばれている
- `outputData`: ✅変換後のデータが正しく生成されている
- `ItemWriter.write()`: ✅正しく呼ばれている

**ログ**
```
2026-01-28 19:30:15 INFO  Job: [name=updateUserStatusJob] launched
2026-01-28 19:30:15 INFO  Step: [updateUserStatusStep] executed in 120ms
2026-01-28 19:30:15 INFO  Job completed with status=COMPLETED
```

### 分割統治

**問題の分解**
```
Batch処理全体
├─ Reader: ✅動作OK（ログで確認）
├─ Processor: ✅動作OK（デバッガで確認）
├─ Writer: ✅write()は実行される
└─ Transaction: ❓コミットされていない？
```

**結論**: Reader/Processor/Writerは正常 → トランザクション周りを疑う
```

---

### 3. 仮説立案 → Issueに追記（5分）

```markdown
## 💡 仮説

| # | 仮説 | 可能性 | 検証コスト | 優先度 |
|---|------|--------|-----------|--------|
| 1 | TransactionManagerが正しく設定されていない | 高 | 低 | ⭐⭐⭐ |
| 2 | commit-intervalが大きすぎてメモリ上に保持 | 中 | 低 | ⭐⭐ |
| 3 | Writerの実装が空（何もしていない） | 低 | 低 | ⭐ |
| 4 | 別のDataSourceを見ている | 中 | 中 | ⭐⭐ |
```

---

### 4. 検証 → 各仮説ごとにコメント追加（10-20分）

**仮説1の検証**:

```markdown
## 🔬 検証: 仮説1

### TransactionManagerの設定確認

**確認内容**
BatchConfigurationクラスのBean定義を確認

**結果**
```java
@Bean
public PlatformTransactionManager txManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

❌ Bean名が`txManager`になっている
Spring Batchはデフォルトで`transactionManager`を探す

### 修正テスト

**変更内容**
```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

**結果**
✅ 成功！DBが更新された

**所感**
Bean名の慣習を守ることの重要性を理解した
```

---

### 5. 解決 → Issueを更新してクローズ（5分）

```markdown
## ✅ 解決策

### 原因
TransactionManagerのBean名が`txManager`だったため、Spring Batchが認識できず、トランザクションがコミットされていなかった。

### 対処方法
Bean名を`transactionManager`に変更

**修正コード**
```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

### なぜこれで解決したか
Spring Batchは慣習的に`transactionManager`という名前のBeanを探す設計。
カスタム名を使う場合は、`@Qualifier`または設定で明示的に指定が必要。

### 参考資料
- [Spring Batch Reference - Configuring a JobRepository](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#configuringJobRepository)

---

## 📝 学んだこと

### 技術的な学び
- Spring Batchは慣習（Convention over Configuration）を重視する
- フレームワークのデフォルト動作を理解することの重要性
- Bean命名規則を守るべき理由

### 再発防止策
- [x] チェックリストに追加: 「Bean名は慣習に従っているか？」
- [x] daily-learningに記録
- [ ] 次回プロジェクトのテンプレートに反映

---

## ⏱️ タイムログ

- 問題分析: 15分
- 仮説立案: 5分
- 検証: 10分
- 解決確認: 5分
- **合計: 35分**
```

**Issueをクローズ**:
- ラベルを`bug` → `resolved`に変更
- Closeボタンをクリック

---

## 📅 daily-learningへの転記（5分）

Issueが解決したら、daily-learningに要点を転記:

```markdown
# 2026-01-28

## 解決したエラー

### Spring Batch TransactionManager未設定

**Issue**: #42

**問題**
バッチ処理が正常終了するが、DBが更新されない

**原因**
TransactionManagerのBean名が`txManager`で、Spring Batchが認識できなかった

**解決策**
Bean名を`transactionManager`に変更

**学んだこと**
- Bean命名規則の重要性
- Spring Batchの慣習
- フレームワークのデフォルト動作を理解する重要性

**詳細**: [Issue #42](https://github.com/user/repo/issues/42)
```

---

## 🏷️ ラベル管理

### 推奨ラベル

**状態**:
- `debug` - デバッグ中（調査・検証中）
- `blocked` - 調査が行き詰まっている
- `resolved` - 解決済み

**カテゴリ**:
- `backend` - バックエンド関連
- `frontend` - フロントエンド関連
- `database` - データベース関連
- `infrastructure` - インフラ関連

**優先度**:
- `priority: high` - 高優先度（本番影響あり）
- `priority: medium` - 中優先度
- `priority: low` - 低優先度

**タイプ**:
- `bug` - バグ
- `investigation` - 調査のみ（バグかどうか不明）

---

## 📊 Issue一覧の活用

### Projects機能で管理

**カンバンボード作成**:
```
デバッグボード
├─ Todo（未着手）
├─ 分析中（問題分析・仮説立案）
├─ 検証中（仮説検証）
└─ 解決済み
```

**運用方法**:
1. Issueを作成 → Todoに自動配置
2. 分析開始 → 分析中に移動
3. 検証開始 → 検証中に移動
4. 解決 → 解決済みに移動してClose

---

## 🔍 過去Issueの検索

### よくあるエラーを即座に見つける

**検索例**:
```
# TransactionManager関連のエラー
is:issue label:backend TransactionManager

# 解決済みのSpring Batch問題
is:closed label:backend "Spring Batch"

# 自分が解決したエラー
is:closed author:@me label:resolved

# 未解決の高優先度バグ
is:open label:bug label:priority:high
```

---

## 💡 運用のコツ

### 1. Issue作成は即座に

```
エラー発生
  ↓
すぐIssue作成（5分）
  ↓
調査しながら更新
```

**メリット**:
- 思考が整理される
- 中断しても再開しやすい
- 進捗が可視化される

---

### 2. 調査中は細かくコメント追加

```markdown
## 調査中...

### 仮説1を検証中
Bean名を確認 → `txManager`になっている
変更して再実行中...

### 結果
✅ 動いた！これが原因だった
```

**メリット**:
- 試行錯誤の過程が残る
- 同じことを繰り返さない

---

### 3. 解決したら「なぜ」を必ず書く

❌ 悪い例:
```
Bean名を変更したら動いた
```

✅ 良い例:
```
Bean名を`transactionManager`に変更したら動いた。

理由: Spring Batchはデフォルトでこの名前のBeanを探す設計。
公式ドキュメントで確認済み。

参考: [URL]
```

---

### 4. 他の人が読んでも分かるように

**誰に向けて書くか**:
- 未来の自分（3ヶ月後に同じエラーに遭遇）
- チームメンバー
- AI（Claude/ChatGPT）に質問する時

**ポイント**:
- 専門用語は説明を添える
- コードは全体を貼る（抜粋NG）
- 環境情報は必ず記載

---

## 📈 効果測定

### 月次で振り返る

```markdown
## 2026年1月のデバッグ統計

**解決したIssue数**: 12個
**平均解決時間**: 45分 → 30分（改善！）
**よくあるエラーTOP3**:
1. TransactionManager設定ミス（3回）
2. Next.js fetch相対パス（2回）
3. N+1問題（2回）

**学び**:
- TransactionManagerは毎回チェックリストで確認するようにした
- Next.jsのfetchパターンをcode-snippetsに追加
```

---

## 🎯 実践例

### Issue #42の完全な流れ

**1. Issue作成（5分）**
```
タイトル: [Bug] Spring BatchでDBが更新されない
ラベル: bug, debug, backend
```

**2. 問題分析（15分）**
```
コメント追加:
- デバッガで各コンポーネント確認
- ログ収集
- 問題を分割
```

**3. 仮説立案（5分）**
```
コメント追加:
- 仮説テーブル作成
- 優先順位づけ
```

**4. 検証（10分）**
```
コメント追加:
- 仮説1の検証結果
- 修正内容
- 結果
```

**5. 解決（5分）**
```
コメント追加:
- 解決策
- なぜ動いたか
- 学んだこと
- 参考資料

Issueをクローズ
```

**6. daily-learning転記（5分）**
```
要点をdaily-learningに記録
Issue URLをリンク
```

**合計: 45分**

---

## ✅ まとめ

### デバッグIssueの価値

1. **思考整理** — 書くことで理解が深まる
2. **記録** — 過去の解決事例が財産になる
3. **効率化** — 同じエラーで悩まない
4. **成長可視化** — 解決時間が短くなる

### 運用のポイント

- ✅ エラー発生 → すぐIssue作成
- ✅ 調査しながらコメント追加
- ✅ 解決したら「なぜ」を書く
- ✅ daily-learningに転記

### 期待される効果

- **解決時間の短縮** — 45分 → 30分 → 15分
- **再発防止** — 同じエラーで悩まない
- **知識の蓄積** — チーム全体のレベルアップ
- **AI活用** — 過去Issueを元にAIが的確なアドバイス

---

**作成日**: 2026年1月28日  
**対象**: GitHubデバッグIssue運用ガイド