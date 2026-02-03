# シリアル化とデシリアル化 - API契約の核心

## 概要
Javaオブジェクト ↔ JSON の変換処理。API契約では「JSONの形式を定義する = シリアル化の形式を決める」ことを意味する。

## 使用場面
- REST APIでJSONを送受信する時
- API契約でデータ形式を定義する時
- Jacksonライブラリでオブジェクト変換する時

## コード

### シリアル化（Serialize）: Java → JSON
```java
// Javaオブジェクト
CashCard card = new CashCard(99L, 123.45, "sarah1");

// ↓ シリアル化（SpringがJacksonで自動変換）
```
```json
{"id": 99, "amount": 123.45, "owner": "sarah1"}
```

### デシリアル化（Deserialize）: JSON → Java
```json
{"id": 99, "amount": 123.45, "owner": "sarah1"}
```
```java
// ↓ デシリアル化（SpringがJacksonで自動変換）

CashCard card = new CashCard(99L, 123.45, "sarah1");
```

### API契約からコードへの変換

```
┌─────────────────────────────────────────────────────────┐
│  API契約書                                              │
│  ─────────                                              │
│  GET /cashcards/{id}                                   │
│  成功: 200 OK + {"id": 99, "amount": 123.45}          │
│  失敗: 404 Not Found                                   │
└─────────────────────────────────────────────────────────┘
                    ↓ 変換
┌─────────────────────────────────────────────────────────┐
│  APIプロバイダーの実装                                    │
└─────────────────────────────────────────────────────────┘
```
```java
@GetMapping("/cashcards/{id}")
public ResponseEntity<CashCard> findById(@PathVariable Long id) {
    return repository.findById(id)
        .map(ResponseEntity::ok)                    // 200 OK
        .orElse(ResponseEntity.notFound().build()); // 404
}
```
```
                    ↓ 変換
┌─────────────────────────────────────────────────────────┐
│  自動テスト                                              │
└─────────────────────────────────────────────────────────┘
```
```java
@Test
void shouldReturn200WhenCardExists() {
    // 契約通りの動作を検証
}

@Test
void shouldReturn404WhenCardNotFound() {
    // 契約通りの動作を検証
}
```

## 説明

### 用語の整理
| 用語 | 方向 | 意味 |
|------|------|------|
| **シリアル化** | Java → JSON | オブジェクトを送信可能な形式に変換 |
| **デシリアル化** | JSON → Java | 受信したデータをオブジェクトに復元 |

### API契約が重要な理由

API契約は以下を具体的に定義する：
1. **各コマンド**: エンドポイント（`GET /cashcards/{id}`）
2. **パラメータ**: パスパラメータ、クエリパラメータ、リクエストボディ
3. **シリアル化形式**: JSONのフィールド名、型、構造

### 契約駆動開発（Contract-First Development）

> **「API契約を先に決めれば、実装もテストも自然に決まる」**

```
契約を決める → 実装が決まる → テストが決まる
    ↓              ↓              ↓
  設計図        建物を建てる    検査する
```

| 成果物 | 役割 |
|--------|------|
| **API契約** | 設計図（何を作るか） |
| **APIプロバイダー** | 契約通りに動くControllerを実装 |
| **自動テスト** | 契約が守られているか検証 |

### Springでの自動変換
Spring Bootでは**Jackson**ライブラリが自動でシリアル化/デシリアル化を行う：
- `@RestController`のメソッド戻り値 → 自動でJSON化
- `@RequestBody`のパラメータ → 自動でJavaオブジェクト化

## 参考
- [Jackson Project](https://github.com/FasterXML/jackson)
- [Spring MVC - Message Converters](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config/message-converters.html)

## 関連スニペット
- [API契約（API Contract）とJSON](./api-contract.md)
- [Jacksonアノテーション](./rest-client/jackson-annotations.md)
- [RestController](./rest-controller.md)

## 作成日
2026-02-04

## タグ
#spring #serialization #deserialization #json #jackson #api-contract #contract-first
