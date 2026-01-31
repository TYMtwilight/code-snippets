# curl - コマンドラインHTTPクライアント

## 概要
コマンドラインからHTTPリクエストを送るためのツール。REST APIのテストやデバッグに使用する。

## 使用場面
- REST APIの動作確認をブラウザを開かずに行う
- スクリプトでAPIを自動呼び出しする
- ヘッダーやHTTPメソッドを細かく指定してテストする

## コード
```bash
# 基本：GETリクエスト
curl https://example.com/api/users

# POSTリクエスト（JSONデータ送信）
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"Taro"}' \
  https://example.com/api/users

# 認証ヘッダー付き
curl -H "Authorization: Bearer token123" \
  https://example.com/api/protected

# レスポンスヘッダーも表示
curl -i https://example.com/api/users

# レスポンスを整形して表示（jqと組み合わせ）
curl https://example.com/api/users | jq
```

## 説明
**ブラウザ vs curl**

| 方法 | 特徴 |
|------|------|
| ブラウザ | URLを入力してEnter → 結果が画面に表示される |
| curl | ターミナルでコマンドを打つ → 結果がテキストで表示される |

**よく使うオプション**

| オプション | 意味 |
|------------|------|
| `-X POST` | HTTPメソッドを指定 |
| `-H "..."` | ヘッダーを追加 |
| `-d '...'` | リクエストボディを送信 |
| `-i` | レスポンスヘッダーを表示 |
| `-v` | 詳細なデバッグ情報を表示 |

**なぜ開発者はcurlを使うのか**
- 手軽：ブラウザを開かなくていい
- 自動化：スクリプトに組み込める
- 細かい制御：ヘッダーやメソッドを自由に指定できる

**プログラムからAPI呼び出しへ**
curlは「人間が手動で確認する」には便利だが、プログラムから使うにはコードで呼び出したい。
→ SpringのRestClientやRestTemplateを使う理由

## 参考
- [curl公式サイト](https://curl.se/)
- [curl manページ](https://curl.se/docs/manpage.html)

## 関連スニペット
- [RestClient基本](../spring/rest-client/rest-client-basics.md)

## 作成日
2025-01-31

## タグ
#curl #http #cli #api #tools
