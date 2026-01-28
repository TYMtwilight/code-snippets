# Notifications API - ブラウザ通知

## 概要
Notifications APIは、ウェブページやアプリからシステムレベルの通知を表示するためのAPI。アプリケーションがバックグラウンド状態でもユーザーに情報を送信できる。

## 使用場面
- メールアプリで新着メッセージを通知
- チャットアプリでメッセージ受信を即時通知
- タスク管理アプリで期限切れを通知
- バックグラウンド処理の完了通知

## コード

### 許可状態の確認
```javascript
// 現在の許可状態を確認
if (Notification.permission === "granted") {
  // 通知可能
} else if (Notification.permission === "default") {
  // まだ許可を求められていない
} else if (Notification.permission === "denied") {
  // ユーザーが拒否した
}
```

### 許可をリクエスト
```javascript
// ユーザー操作（ボタンクリックなど）に応じて実行
Notification.requestPermission().then((permission) => {
  if (permission === "granted") {
    // 通知表示許可が得られた
    new Notification("許可されました！");
  }
});
```

### 通知の作成
```javascript
const notification = new Notification("新着メッセージ", {
  body: "田中さんからメッセージが届きました",
  icon: "/img/icon-128.png",
  tag: "message-notification"  // 同じタグで既存通知を置き換え
});
```

### イベントハンドリング
```javascript
const notification = new Notification("タイトル");

notification.addEventListener("click", () => {
  window.focus();  // ウィンドウにフォーカス
  notification.close();
});

notification.addEventListener("close", () => {
  console.log("通知が閉じられました");
});

notification.addEventListener("error", () => {
  console.log("通知エラー");
});
```

### タグによる通知の置き換え
```javascript
// 同じタグを持つ通知は、新しい通知で置き換わる
// 大量の通知でユーザーを煩わせないために使用
for (let i = 0; i < 10; i++) {
  new Notification(`メッセージ ${i}件`, {
    tag: "message-count"  // 1つの通知だけが表示される
  });
}
```

## 説明

### Notification.permission の値
| 値 | 説明 |
|---|---|
| `granted` | ユーザーが通知表示を許可 |
| `denied` | ユーザーが通知表示を拒否 |
| `default` | まだ許可を求めていない（deniedと同じ動作） |

### 重要な注意点
- **HTTPS必須**: セキュアコンテキスト（HTTPS）でのみ利用可能
- **ユーザー操作が必要**: 許可リクエストはボタンクリックなどのユーザー操作に応じて実行すべき（Chrome72+、Safari以降では無操作での許可要求を自動拒否）
- **自動で閉じる**: ほとんどのブラウザで数秒後に自動的に閉じる
- **Service Worker対応**: `ServiceWorkerRegistration.showNotification()`でバックグラウンド通知も可能

## 参考
- [MDN - Notifications API の使用](https://developer.mozilla.org/ja/docs/Web/API/Notifications_API/Using_the_Notifications_API)
- [MDN - Notification](https://developer.mozilla.org/ja/docs/Web/API/Notification)

## 関連スニペット
- （Service Worker関連のスニペットがあれば追加）

## 作成日
2026-01-28

## タグ
#javascript #web-api #notification #browser
