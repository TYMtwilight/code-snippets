
# エラー解決のステップ

## 🎯 基本姿勢

- **エラーは成長の機会** — 恥ずかしいことじゃない、学びのチャンス
- **AI活用の黄金ルール** — 自分で5-10分考えてから、AIに相談
- **記録を残す** — 同じエラーで2度悩まない

---

## 📋 Step 1: 問題分析（10-15分）

### 1-1. エラーを再現可能にする

**再現手順を箇条書きにする**

#### バックエンド例（Spring Boot）

**環境**
- Spring Boot 3.2.1
- Java 17
- PostgreSQL 15

**再現手順**
1. アプリケーション起動: `./mvnw spring-boot:run`
2. Postmanで以下のリクエスト送信:
   - Method: POST
   - URL: `http://localhost:8080/api/users`
   - Body: `{"name": "Test", "email": "test@example.com"}`
3. → 500エラー発生
4. ログ確認: `NullPointerException at UserService.java:42`

**再現率**
- 毎回発生（100%）

#### フロントエンド例（Next.js）

**環境**
- Next.js 14.1.0
- Node.js 20.10.0
- Chrome 120

**再現手順**
1. 開発サーバー起動: `npm run dev`
2. ブラウザで `http://localhost:3000/users` にアクセス
3. 「新規ユーザー」ボタンをクリック
4. フォームに入力:
   - 名前: "Test User"
   - メール: "test@example.com"
5. 「送信」ボタンをクリック
6. → 画面が真っ白になる
7. コンソール: `TypeError: Cannot read properties of undefined`

**再現率**
- 毎回発生（ただしChromeのみ、Firefoxでは発生しない）

---

### 1-2. 情報を集める

- **エラーメッセージを読む**
  - スタックトレースの最後まで確認
  - "Caused by:"を辿って根本原因を探す
  - 自分のコード（パッケージ名）がある行に注目

- **ログを確認**
  - ERROR/WARN/INFOを時系列で追う
  - 必要ならログレベルをDEBUGに上げる

- **デバッガで変数確認**
  - null, 想定外の値がないか
  - ステップ実行で処理を追う

---

### 1-3. 分割統治法

**問題を小さく分解して、各部分を個別にテストする**

#### バックエンド例: Spring Batch処理が失敗

**大きな問題**
- バッチ処理が成功するのに、DBが更新されない

**分解**
```
Batch処理全体
├─ 1. Reader（データ読み込み）
├─ 2. Processor（データ加工）
├─ 3. Writer（データ書き込み）
└─ 4. Transaction（トランザクション管理）
```

**各部分の検証**

1. Readerのテスト:
```java
@Test
void testReader() {
    User user = reader.read();
    assertNotNull(user);  // → ✅成功
}
```

2. Processorのテスト:
```java
@Test
void testProcessor() {
    UserDTO output = processor.process(inputUser);
    assertEquals("TEST", output.getName());  // → ✅成功
}
```

3. Writerのテスト:
```java
@Test
void testWriter() {
    writer.write(Arrays.asList(dto));
    User saved = repository.findById(1);
    assertNotNull(saved);  // → ❌失敗！ここが原因
}
```

4. Writerをさらに分解:
   - DBコネクションは正常？ → ✅OK
   - SQLは実行されている？ → ✅OK
   - トランザクションはコミットされている？ → ❌されていない！

**原因特定**
- TransactionManagerのBean名が`txManager`で、Spring Batchが認識できなかった（`transactionManager`を期待）

---

#### フロントエンド例: ユーザー一覧ページで500エラー

**大きな問題**
- `/users`ページにアクセスすると画面が真っ白

**分解**
```
画面真っ白
├─ 1. ルーティング（App Router）
├─ 2. ページコンポーネント（page.tsx）
├─ 3. データフェッチ（fetch API）
├─ 4. APIルート（/api/users）
└─ 5. レンダリング（JSX）
```

**各部分の検証**

1. APIルートを直接確認:
```bash
curl http://localhost:3000/api/users
# → ✅成功: {"users": [...]}
```

2. ページコンポーネントにログ追加:
```typescript
export default async function UsersPage() {
  console.log('1. Component loaded');
  const res = await fetch('http://localhost:3000/api/users');
  console.log('2. Fetch done:', res.status);
  const data = await res.json();
  console.log('3. Data:', data);  // → undefined
}
```

**コンソール出力**
```
1. Component loaded
2. Fetch done: 200
3. Data: undefined  ← ❌ここで問題発見
```

3. fetchのレスポンスを詳細確認:
```typescript
const text = await res.text();
console.log('Response:', text);  // → 空文字列
```

4. Server Componentで絶対URLを使用 → ❌これが原因
   - 相対パス`/api/users`に変更 → ✅解決

**原因特定**
- Server Componentで`http://localhost:3000`を使うと、サーバー側で自分自身にfetchできない。相対パスを使うべきだった。

---

#### フロントエンド例2: フォーム送信でフリーズ

**大きな問題**
- フォーム送信ボタンを押すと画面がフリーズ

**分解**
```
ボタンクリック
├─ 1. イベントハンドラ実行
├─ 2. バリデーション
├─ 3. API呼び出し
├─ 4. レスポンス受信
└─ 5. 状態更新
```

**各部分の検証**

1. イベントハンドラにログ:
```typescript
const handleSubmit = async (e) => {
  console.log('1. Handler called');
  e.preventDefault();
  
  console.log('2. Validation');
  if (!name) return;
  
  console.log('3. API call');
  const res = await fetch('/api/users', {...});
  console.log('4. Response');  // ← ここに到達しない
}
```

**コンソール出力**
```
1. Handler called
2. Validation
3. API call
（フリーズ）
```

2. APIルート側を確認:
```typescript
export async function POST(req: Request) {
  const body = await req.json();
  const user = await createUser(body);  // ← ここで無限ループ？
}
```

3. createUser関数を確認:
```typescript
async function createUser(data) {
  // ...
  await createUser(data);  // ❌再帰呼び出し！
}
```

**原因特定**
- createUser関数内で自分自身を呼び出していた（無限ループ）
- → スタックオーバーフロー → フリーズ

---

## 💡 Step 2: 仮説（5分）

### 検証可能な仮説を立てる

**❌ 悪い仮説**
- 「何かがnullだから」
- 「設定が間違ってる」

**✅ 良い仮説**
- 「Repositoryが存在しないIDでnullを返している」
- 「commit-intervalが大きすぎてメモリ不足」
- 「Bean名が慣習と違うため注入失敗」

### 優先順位をつける
1. 最も可能性が高いもの
2. 最も簡単に確認できるもの
3. 影響範囲が小さいもの

---

### 仮説を立てられない時（即効対応）

#### 1. 詳しい人に聞く（最優先・最速 5-30分）

**社内Slack/Teamsで聞く場合**

```
こんにちは、○○で困っています🙏

【問題】
Spring Batchのジョブが成功するのに、DBが更新されません

【やったこと】
- Reader/Processor/Writerは個別テストOK
- ログにエラーなし
- トランザクション設定を確認中

【質問】
Spring Batchでトランザクションがコミットされない場合、
どこをチェックすればよいでしょうか？

詳細: [GitHub Issue URL]
```

**ポイント**:
- ✅ 何を試したか書く（丸投げしない）
- ✅ 具体的な質問をする
- ✅ Issue/コードへのリンクを貼る

---

#### 2. AIに効果的に聞く（24時間対応 2-10分）

**Claude/ChatGPTへのプロンプト**

```
以下のSpring Batchエラーについて、考えられる原因を3つ挙げて、
それぞれの検証方法を教えてください。

【問題】
Spring Batchジョブが正常終了するが、DBが更新されない

【環境】
- Spring Boot 3.2.1
- PostgreSQL 15

【コード】
[BatchConfiguration.javaを貼り付け]

【ログ】
[関連ログを貼り付け]

【試したこと】
- Reader/Processor/Writerは個別に動作確認済み
- ログにエラーなし
```

**ポイント**:
- ✅ 環境情報を明記
- ✅ コード全体を貼る
- ✅ 「3つ挙げて」など具体的に指示

---

#### 3. エラーメッセージで検索（5-15分）

**効果的な検索ワード**

❌ 悪い検索:
```
「Spring Batch エラー」
```

✅ 良い検索:
```
「Spring Batch TransactionManager bean not found」
「Spring Batch job completed but database not updated」
site:stackoverflow.com Spring Batch transaction not committed
```

**検索演算子を活用**:
```
# 完全一致
"Spring Batch" "transactionManager"

# サイト指定
site:stackoverflow.com [検索ワード]

# 期間指定
[検索ワード] after:2023-01-01
```

---

#### 4. 公式ドキュメントを「効率的に」読む（15分）

**時短読み方**:

1. **目次から関連章を探す（3分）**
```
Spring Batch Reference
├─ 3. Configuring a Job  ← これを読む
├─ 4. Configuring a Step
└─ ...
```

2. **その章の要約だけ読む（5分）**
   - 冒頭の概要
   - 図・コードサンプル
   - 注意事項（Note, Warning）

3. **Ctrl+F でキーワード検索（5分）**
   - "transaction"
   - "commit"
   - "transactionManager"

4. **AIに要約させる（2分）**
```
以下のドキュメントを要約して、
トランザクション設定に関する部分だけ抜き出してください。

[URLまたはドキュメント全文]
```

---

#### 5. GitHubで動くコードを探す（10分）

**検索方法**:
```
# GitHubで検索
filename:BatchConfiguration.java TransactionManager

# 動いているプロジェクトを見つける
org:spring-projects TransactionManager language:java
```

**手順**:
1. 動いているプロジェクトを見つける
2. 自分のコードと比較
3. 違いを確認して試す

---

#### 即効対応のまとめ

| 方法 | 時間 | 有効度 | いつ使う |
|-----|------|--------|---------|
| 詳しい人に聞く | 5-30分 | ⭐⭐⭐⭐⭐ | 最優先 |
| AIに聞く | 2-10分 | ⭐⭐⭐⭐ | すぐ試したい |
| エラー検索 | 5-15分 | ⭐⭐⭐⭐ | 既知のエラー |
| 公式ドキュメント | 15分 | ⭐⭐⭐⭐ | 仕様確認 |
| GitHub検索 | 10分 | ⭐⭐⭐ | 実装例 |

**推奨順序**:
1. 自分で5分考える（必須）
2. 詳しい人に聞く（最速）
3. AIに聞く（24時間対応）
4. エラーメッセージで検索
5. 公式ドキュメント

---

## 🔬 Step 3: 検証（10-20分）

### 3-1. 最小限の変更で検証
- **一度に1つだけ変更する**
- **ログやデバッグ出力を仕込む**
- **元に戻せるようにする** (git stash, コメントアウト)

### 3-2. 動いた後の確認（最重要！）

**必ず確認すること**:
1. **なぜ今の修正で動いたのか理由を説明できるか？**
2. **元のコードの何が問題だったのか？**
3. **公式ドキュメントで裏付けが取れるか？**
4. **再発防止策は？**

**❌ 悪い例**

「動いたからOK」

**✅ 良い例**

「Bean名を`transactionManager`に変更したら動いた。理由: Spring Batchはデフォルトでこの名前を探すため。公式ドキュメントで確認済み。→ 学習記録に残して次回から即解決できるようにする」

---

## 📝 記録の残し方

### daily-learningに記録

```markdown
### 解決したエラー: [エラー名]

#### 問題
[簡潔に]

#### 原因
[根本原因]

#### 解決策
[何をしたか]

#### 学んだこと
- [本質的な理解]
- [なぜそうなるか]

#### 再発防止
- [次回同じエラーに遭遇したときの対処法]
```

---

## 🔄 全体フロー

```
問題発生
  ↓
問題分析（10-15分）
  - 再現手順
  - 情報収集（ログ・デバッガ）
  - 分割
  ↓
自分で考える（5-10分）← 重要！
  ↓
【仮説が立たない場合】
  1. 詳しい人に聞く（最優先）
  2. AIに聞く
  3. エラー検索
  ↓
仮説（5分）
  - 検証可能な仮説を3-5個
  - 優先順位づけ
  ↓
検証（10-20分）
  - 最小限の変更
  - 1つずつ検証
  ↓
動いた後の確認（必須！）
  - なぜ動いたか理解
  - 公式で裏付け
  ↓
記録（5分）
  - daily-learning
```

---

## 🎯 重要ポイント

1. **自分で考えてからAIに相談**（5-10分ルール）
2. **一度に1つだけ変更する**
3. **動いた理由を必ず確認する**（偶然の成功を防ぐ）
4. **記録を残す**（次回から即解決）
5. **困ったら詳しい人に聞く**（最速の解決法）

---

**作成日**: 2026年1月28日  
**対象**: エラー解決の基本ステップ