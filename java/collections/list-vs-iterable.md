# List と Iterable の違い

## 概要
Java における `Iterable<T>` と `List<T>` の継承関係・機能差・使い分けをまとめる。

## 使用場面
- メソッドの引数・戻り値の型を決めるとき
- Spring Data の `CrudRepository.findAll()` の戻り値を扱うとき

## コード

```java
// ---- 継承関係 ----
// Iterable<T>
//   └── Collection<T>
//         └── List<T>
//               └── ArrayList, LinkedList, ...

// ---- Iterable: for-each ループで回せることだけを保証 ----
Iterable<Item> iterable = itemRepository.findAll(); // CrudRepository の戻り値
for (Item item : iterable) {
    System.out.println(item);
}

// ---- List: 豊富な操作が使える ----
List<Item> list = new ArrayList<>();
list.get(0);        // インデックスアクセス ✅
list.size();        // サイズ取得 ✅
list.add(item);     // 追加 ✅
list.remove(0);     // 削除 ✅

// ---- Iterable → List への変換 ----
// 方法1: forEach でひとつずつ追加
List<Item> list1 = new ArrayList<>();
iterable.forEach(list1::add);

// 方法2: StreamSupport を使う
List<Item> list2 = StreamSupport
    .stream(iterable.spliterator(), false)
    .collect(Collectors.toList());
```

## 説明

| 操作 | Iterable | List |
|------|----------|------|
| for-each | ✅ | ✅ |
| インデックスアクセス | ❌ | ✅ |
| `size()` | ❌ | ✅ |
| `add()` / `remove()` | ❌ | ✅ |
| 用途 | 「回せる」最低限保証 | 順序付きコレクション |

- **`Iterable`** は `iterator()` メソッドのみを持つ最小限のインターフェース。DB やファイルなど一方向に流れるデータに使われる。
- **`List`** はランダムアクセス・サイズ取得・追加・削除など豊富な操作を提供する。
- **使い分け**: for-each だけなら引数・戻り値を `Iterable` にすると呼び出し側の柔軟性が上がる。インデックス操作や追加・削除が必要なら `List` を使う。

## 参考
- [Iterable (Java SE 21)](https://docs.oracle.com/en/java/docs/api/java.base/java/lang/Iterable.html)
- [List (Java SE 21)](https://docs.oracle.com/en/java/docs/api/java.base/java/util/List.html)
- [CrudRepository (Spring Data)](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)

## 関連スニペット
- [arrays-as-list.md](arrays-as-list.md)

## 作成日
2026-03-01

## タグ
#java #collections #list #iterable #spring-data #interface
