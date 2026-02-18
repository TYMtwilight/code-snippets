# 認証クッキー vs CSRFトークン

> **学習日:** 2026-02-19
> **情報源:** [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

---

## 一言でまとめると

クッキーは「ブラウザが勝手に送る」から偽造されやすい。
CSRFトークンは「正規ページを実際に受け取った者だけが知っている値」なので、外部から偽造できない。

---

## 認証クッキーの問題

認証クッキーは、**ブラウザが自動的に送信する**。

あるサイトにログインしてクッキーが保存されると、そのサイトへのリクエストが**どこから発生したかに関わらず**、ブラウザは自動でクッキーを付与する。

これがCSRFの根本的な問題。悪意あるサイトからのリクエストでも、ブラウザは正直にクッキーを送ってしまう。

---

## 認証クッキー vs CSRFトークン

| | 認証クッキー | CSRFトークン |
|---|---|---|
| 保存場所 | ブラウザが管理 | HTMLページ内（フォームの隠しフィールド等） |
| 送信タイミング | ブラウザが**自動で**送る | JavaScriptが**明示的に**送る |
| リクエストごとの値 | 同じ値が使い回される | **毎回異なる**値が生成される |
| 外部サイトから偽造 | 可能（CSRFの原因） | 不可（クロスオリジン制限で読み取れない） |

---

## なぜトークンで保護できるのか

**CSRFトークンなしの場合：**
1. 悪意あるサイトがリクエストを送る
2. ブラウザが自動でクッキーを付与
3. サーバーは本人のリクエストと区別できない → 攻撃成功

**CSRFトークンありの場合：**
1. サーバーがHTMLページを返す際に、そのページ内にCSRFトークンを埋め込む
2. 正規のリクエスト送信時、そのトークンも一緒に送る
3. サーバーはトークンを照合して正規のリクエストか判断する

悪意あるサイトは**被害者のページ内のトークンを読み取れない**（クロスオリジン制限のため）ので、正しいトークンを付与したリクエストを偽造できない。

---

## 関連

- **公式:** [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- **参考:** [MDN - SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- **関連概念:** SameSite クッキー属性, CORS, セッション管理
