# ページネーションURIパラメータとソートのベストプラクティス

## 概要
Spring DataのページネーションにおけるURIクエリパラメータの構成方法と、明示的なソート指定が重要である理由を解説する。

## 使用場面
- REST APIのページネーションURIを設計する場合
- クライアントにページネーション仕様を共有する場合
- ソート未指定時のデフォルト動作を理解・設定する場合

## コード
```
# URIクエリパラメータの段階的な構成例

# 2ページ目を取得（0始まり）
/cashcards?page=1

# 1ページあたり3件
/cashcards?page=1&size=3

# amount順でソート
/cashcards?page=1&size=3&sort=amount

# 降順（カンマの前後にスペースを入れない！）
/cashcards?page=1&size=3&sort=amount,desc
```

```java
// サーバー側: getSortOr() でデフォルトソートを設定
pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))

// PageRequest.of() でソート条件を明示的に構築
PageRequest.of(
    1,   // ページ番号（0始まり、この場合2ページ目）
    10,  // ページサイズ（最後のページは件数が少ない場合あり）
    Sort.by(new Sort.Order(Sort.Direction.DESC, "amount"))
);
```

## 説明
### URIパラメータ仕様

| パラメータ | 説明 | デフォルト値 | 例 |
|-----------|------|------------|-----|
| `page` | ページ番号（0始まり） | 0 | `page=1` |
| `size` | 1ページあたりの件数 | 20 | `size=3` |
| `sort` | ソートフィールド,方向 | なし | `sort=amount,desc` |

**注意**: `sort=amount,desc` のカンマ前後にスペースを入れてはいけない。

### なぜ明示的なソートが重要か

ソートを指定しない「未ソート」のページネーションは以下の問題を引き起こす：

1. **予測不能な順序**: 新しいレコードを追加した際、どのページに表示されるか予測できない
2. **認知負荷**: 開発者やユーザーにとって、ランダムに見える順序は理解しにくい
3. **将来のリスク**: Spring・Java・DBのバージョンアップで「暗黙の順序」が変わる可能性がある

```
# 未ソートの場合: 新規データがどこに入るか不明
ページ1: [$0.19, $1000.00, $50.00]
ページ2: [$20.00, $10.00]
→ $42.00を追加 → どのページに入る？ 不明

# amount降順でソート: 新規データの位置が明確
ページ1: [$1000.00, $50.00, $42.00]  ← $42.00はここに入る
ページ2: [$20.00, $10.00, $0.19]
```

### Springのデフォルト値について
- `page`と`size`はSpringがデフォルト値（0と20）を提供する → 妥当
- `sort`はSpringがデフォルト値を提供しない → アプリケーション側で`getSortOr()`を使い明示的に設定すべき

## 参考
- [Spring Data Web Support - Paging and Sorting](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#core.web.basic.paging-and-sorting)

## 関連スニペット
- [PageableとPage](./pageable-and-page.md)
- [GetMapping findAll ページネーション付き](../controller/get-mapping-find-all-pagination.md)

## 作成日
2026-02-14

## タグ
#spring #spring-data #pagination #sorting #rest-api #best-practices #uri
