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
  // まだ許可も拒否もしていない → requestPermission() で許可を求められる
} else if (Notification.permission === "denied") {
  // ユーザーが拒否した → requestPermission() を呼んでも許可ダイアログは出ない
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

### サンプル：許可リクエストから通知表示まで一連の流れ
```javascript
// 許可状態に応じて処理を分岐し、通知を表示するユーティリティ関数
async function showNotification(title, options = {}) {
  // 既に拒否されている場合は何もしない
  if (Notification.permission === "denied") {
    console.warn("通知がブロックされています。ブラウザの設定を確認してください。");
    return;
  }

  // まだ許可を求めていない場合はリクエストする
  if (Notification.permission === "default") {
    const permission = await Notification.requestPermission();
    if (permission !== "granted") return;
  }

  // 通知を表示
  const notification = new Notification(title, options);

  // クリックしたらウィンドウにフォーカスして通知を閉じる
  notification.addEventListener("click", () => {
    window.focus();
    notification.close();
  });
}

// 使用例：ボタンクリックで通知を出す
document.getElementById("notifyBtn").addEventListener("click", () => {
  showNotification("タスク完了", {
    body: "レポートのエクスポートが完了しました",
    icon: "/img/icon-128.png",
    tag: "task-complete"
  });
});
```

### サンプル：ポモドーロタイマーの終了通知
```javascript
// タイマー終了時に通知を出す例
function startPomodoroTimer(minutes) {
  const ms = minutes * 60 * 1000;

  setTimeout(async () => {
    await showNotification("⏰ ポモドーロ終了！", {
      body: `${minutes}分経過しました。休憩しましょう。`,
      icon: "/img/tomato.png",
      tag: "pomodoro"  // 複数タイマーが重ならないようにtagを統一
    });
  }, ms);
}

startPomodoroTimer(25);
```

## 説明

### Notification.permission の値
| 値 | 説明 |
|---|---|
| `granted` | ユーザーが通知表示を許可 |
| `denied` | ユーザーが通知表示を拒否（`requestPermission()` を呼んでもダイアログは出ない） |
| `default` | まだ許可も拒否もしていない（`requestPermission()` でダイアログを出せる） |

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