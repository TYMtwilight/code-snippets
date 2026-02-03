# API契約（API Contract）とJSON

## 概要
APIプロバイダー（提供者）とコンシューマー（利用者）の間の「約束事」を定義し、テストコードで保証する仕組み。

## 使用場面
- REST APIを設計・開発する時
- フロントエンドとバックエンドのチームが分かれている時
- APIの変更が既存のクライアントを壊さないか確認したい時
- ドキュメントだけでなく、契約をコードで保証したい時

## コード

### API契約の構成要素
```
┌─────────────────┐         契約          ┌─────────────────┐
│  APIコンシューマー  │◄──────────────────►│  APIプロバイダー   │
│  （フロントエンド等） │   「こう使えば     │  （バックエンド）   │
│                 │    こう返す」       │                 │
└─────────────────┘                      └─────────────────┘
```

### 契約の例：CashCard API

| 項目 | 契約内容 |
|------|---------|
| エンドポイント | `GET /cashcards/{id}` |
| 成功時 | 200 OK + JSON本体を返す |
| 存在しない場合 | 404 Not Found を返す |
| 認証なし | 401 Unauthorized を返す |

### JSONの形式も契約の一部
```json
// コンシューマーは「この形式で返ってくる」と期待する
{
  "id": 99,
  "amount": 123.45,
  "owner": "sarah1"
}
```

### 契約違反の例
```json
// ❌ プロバイダーが勝手にフィールド名を変更
{
  "id": 99,
  "balance": 123.45,  // amount → balance に変更
  "owner": "sarah1"
}
// → コンシューマーのアプリが壊れる（amountを探してエラー）
```

### 契約テスト（Contract Test）
```java
@Test
void shouldReturnCashCardWithCorrectJsonStructure() {
    ResponseEntity<String> response =
        restTemplate.getForEntity("/cashcards/99", String.class);

    // 契約: 200を返す
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

    // 契約: JSONにこれらのフィールドがある
    DocumentContext json = JsonPath.parse(response.getBody());
    assertThat(json.read("$.id", Long.class)).isEqualTo(99);
    assertThat(json.read("$.amount", Double.class)).isEqualTo(123.45);
    assertThat(json.read("$.owner", String.class)).isEqualTo("sarah1");
}
```

## 説明

### API契約で決めるべきこと
1. **リクエスト**: コンシューマーはどのようなデータを送信するか
2. **レスポンス**: APIはどのようなデータをいつ返すか
3. **エラー処理**: 誤った使用や問題発生時に何を返すか

### なぜテストで契約を書くのか

**口約束・ドキュメントだけの問題：**
```
開発者A: 「amountフィールドで返すね」
開発者B: 「了解」
（3ヶ月後、Aが忘れてbalanceに変更...）
→ 本番でバグ発覚 💥
```

**テストで契約を書くメリット：**
- 誰かが`amount`を`balance`に変えたら → **テストが失敗**
- 契約違反が**自動で検出**される
- ドキュメントより**信頼できる**（コードは嘘をつかない）

### 用語まとめ
| 用語 | 意味 |
|------|------|
| **API契約** | 「このリクエストにはこのレスポンス」という約束 |
| **JSON** | その契約で使われるデータ形式 |
| **契約テスト** | 約束が守られているか自動で確認するテスト |

### 核心
> 「口約束やドキュメントだけでなく、**テストコードとして契約を書け**」
> → 契約違反は即座にテスト失敗として検出できる

## 参考
- [Spring Contract Testing](https://spring.io/projects/spring-cloud-contract)
- [Consumer-Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html)

## 関連スニペット
- [Spring MVC Controller基礎](./controller/spring-mvc-controller-basics.md)
- [RestController](./rest-controller.md)
- [RESTクライアント/サーバーの概念](./rest-client/rest-client-server-concept.md)

## 作成日
2026-02-04

## タグ
#spring #rest-api #api-contract #json #testing #contract-test
