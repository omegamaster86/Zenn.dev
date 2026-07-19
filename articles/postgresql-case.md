---
title: "PostgreSQLのCASEに関して調べてみたんじゃ"
emoji: "⏳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PostgreSQL", "SQL", "CASE"]
published: false
# published_at: 2026-05-31 18:30
publication_name: "genai"
---

# はじめに
いや〜分割キーボード流行ってきましたね〜
皆さんも是非分割キーボードデビューをしてみてください！
さて今回は『達人に学ぶSQL徹底指南書 第2版』の第1章「CASE式のススメ」を読んでみて、PostgreSQL でどう使えるか整理してみました。

# 本題
## CASE式は「文」ではなく「式」
よく「CASE文」と呼ばれますが、正確には **CASE式** らしいです。

そして式なので、評価結果は **1つのスカラ値** になります。だから `SELECT` だけでなく、`WHERE`・`GROUP BY`・`ORDER BY`・`UPDATE` の `SET` など、**式が書ける場所ならどこでも使えます**。

```sql
-- SELECT 句での分岐
SELECT
  name,
  CASE status
    WHEN 'active'   THEN '有効'
    WHEN 'inactive' THEN '無効'
    ELSE '不明'
  END AS status_label
FROM users;
```

### 2種類の書き方

| 種類 | 構文 | 使いどころ |
| --- | --- | --- |
| **単純CASE** | `CASE 式 WHEN 値 THEN ...` | 同じ式を複数の値と比較するとき |
| **検索CASE** | `CASE WHEN 条件 THEN ...` | 条件がバラバラのとき |

```sql
-- 単純CASE: status という列を各値と比較
CASE status
  WHEN 'active' THEN '有効'
  ELSE '無効'
END

-- 検索CASE: 条件式を自由に書ける
CASE
  WHEN score >= 90 THEN 'A'
  WHEN score >= 70 THEN 'B'
  ELSE 'C'
END
```

## ユースケース1: コード値の変換（ラベル付け）
いちばん基本的な使い方。アプリ側でやるより、レポート用クエリでは SQL 側でやったほうが楽なことが多いです。

```sql
SELECT
  order_id,
  CASE payment_method
    WHEN 'credit' THEN 'クレジットカード'
    WHEN 'bank'   THEN '銀行振込'
    WHEN 'cash'   THEN '現金'
    ELSE 'その他'
  END AS payment_label
FROM orders;
```

## ユースケース2: GROUP BY で柔軟な集計
『達人に学ぶSQL徹底指南書』でも強調されているパターンです。固定のカラムでなく、**集約の切り口を CASE で動的に決める** ときに威力を発揮します。

orders テーブル

| id | user_id | amount | ordered_at |
| --- | --- | --- | --- |
| 1 | 1 | 1000 | 2025-01-15 |
| 2 | 1 | 500 | 2025-02-20 |
| 3 | 2 | 3000 | 2025-01-10 |
| 4 | 2 | 800 | 2025-03-01 |

```sql
-- 金額帯ごとの件数を集計
SELECT
  CASE
    WHEN amount < 1000  THEN '〜999'
    WHEN amount < 3000  THEN '1000〜2999'
    ELSE '3000〜'
  END AS amount_range,
  COUNT(*) AS order_count
FROM orders
GROUP BY 1
ORDER BY 1;
```

`GROUP BY` に CASE 式をそのまま書いてもよいですが、PostgreSQL では `GROUP BY 1`（SELECT リストの1番目）も使えます。
なぜこの書き方をしたか？
普通の GROUP BY は、既存の列で集計します。
```sql
-- user_id 列がそのまま集計キーになる
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id;
```
でも金額帯はテーブルに列がないので、CASEで集計の切り口をその場で作る必要があります。

## ユースケース3: 行持ち → 列持ち（ピボット）
集約関数の中に CASE を入れると、**条件ごとの件数・合計を横に並べる** ピボットができます。HAVING 句を使わずに1クエリでまとめられるのが CASE 式の強みです。

```sql
-- 支払い方法ごとの件数を横に並べる
SELECT
  user_id,
  COUNT(CASE WHEN payment_method = 'credit' THEN 1 END) AS credit_count,
  COUNT(CASE WHEN payment_method = 'bank'   THEN 1 END) AS bank_count,
  COUNT(CASE WHEN payment_method = 'cash'   THEN 1 END) AS cash_count
FROM orders
GROUP BY user_id;
```

`COUNT(CASE WHEN ... THEN 1 END)` は、条件に合わない行は `NULL` になるのでカウントされません。本書でも「集約関数の中に CASE を入れる」パターンとして紹介されています。

## ユースケース4: UPDATE で条件分岐
1回の UPDATE で、行ごとに異なる値をセットできます。複数の UPDATE 文を書くより、意図が1箇所にまとまります。

```sql
UPDATE products
SET price = CASE
  WHEN category = 'book'  THEN price * 1.10  -- 本は10%値上げ
  WHEN category = 'food'  THEN price * 1.05  -- 食品は5%値上げ
  ELSE price
END
WHERE discontinued = false;
```

`ELSE` を忘れると、該当しない行は `NULL` になってしまうので注意です。

## SELECT の CASE と WHERE、どっちを使う？
ユースケースを見て「`SELECT` の `CASE` でも絞れそうだけど、`WHERE` と何が違うの？」と思った方へ。私も同じことを思いました。調べたところ、**行を減らしたいなら `WHERE`、行は残して値を変えたいなら `SELECT` の `CASE`** です。どちらが速いというより、**やりたいことが違います**。

### まず認識合わせ
`SELECT` の `CASE` は基本的に **絞り込み（フィルタ）ではありません**。

```sql
-- SELECT の CASE: 全行返る。表示する値だけ変わる（ユースケース1と同じ考え方）
SELECT
  name,
  CASE WHEN status = 'active' THEN '有効' ELSE '無効' END AS label
FROM users;

-- WHERE: 条件に合わない行は結果から消える
SELECT name
FROM users
WHERE status = 'active';
```

上の `CASE` は「無効」ユーザーも行として返ります。`WHERE` は `active` だけ残します。

### 使い分け早見表

| 目的 | 使うもの |
| --- | --- |
| 不要な行を結果から除外したい | **`WHERE`** |
| 全行（または多くの行）を見せつつ、列の値を条件で変えたい | **`SELECT` の `CASE`** |
| 集計の切り口を変えたい | **`GROUP BY` + `CASE`**（ユースケース2） |
| 条件別の件数を横に並べたい | **集約関数 + `CASE` / `FILTER`**（ユースケース3） |

### `WHERE` を使うとき
- レポート対象を限定する（例: 直近1年の注文だけ）
- インデックスを効かせて件数を減らしたい
- APIや画面に渡す行数そのものを減らしたい

```sql
SELECT *
FROM orders
WHERE ordered_at >= CURRENT_DATE - INTERVAL '1 year';
```

単純に行を絞るだけなら、わざわざ `CASE` を使う必要はありません。

### `WHERE` に `CASE` を使うとき（やや上級）
比較対象そのものを行ごとに変えたいときだけです。頻度は低めです。

```sql
-- カテゴリごとに「高い」とみなす基準が違う
SELECT *
FROM products
WHERE price > CASE category
  WHEN 'book' THEN 1000
  WHEN 'food' THEN 500
  ELSE 300
END;
```

これは「絞る」のではなく、**行ごとに閾値が違う条件で絞る**パターンです。

### 集計では `WHERE` だけでは足りない
`WHERE` で先に絞ると、**他の条件の集計ができなくなります**（ユースケース3の話）。

```sql
-- ❌ credit だけに絞ると bank / cash の件数が取れない
SELECT COUNT(*) FROM orders WHERE payment_method = 'credit';

-- ✅ 1クエリで横並び集計
SELECT
  user_id,
  COUNT(*) FILTER (WHERE payment_method = 'credit') AS credit_count,
  COUNT(*) FILTER (WHERE payment_method = 'bank')   AS bank_count
FROM orders
GROUP BY user_id;
```

### よくあるアンチパターン

```sql
-- ❌ 見た目は絞ってるようだが、行は全部返る（非効率・紛らわしい）
SELECT *
FROM users
WHERE CASE WHEN status = 'active' THEN true ELSE false END;

-- ✅ こう書く
SELECT *
FROM users
WHERE status = 'active';
```

```sql
-- 「active 以外は NULL にする」だけなら行は残る
SELECT CASE WHEN status = 'active' THEN name END AS active_name
FROM users;

-- 本当に active だけ欲しいなら WHERE のほうが明確でインデックスも効きやすい
SELECT name FROM users WHERE status = 'active';
```

### 判断のフロー

```
行数を減らしたい？
  ├─ Yes → WHERE（可能なら CASE は使わない）
  └─ No  → 値を変えたい？
            ├─ 行ごとの表示・計算 → SELECT の CASE
            └─ 集計を条件別に分けたい → 集約関数 + CASE / FILTER
```

パフォーマンスの目安としては、**早い段階（`WHERE`）で行を減らすほうが一般的に有利**です。`SELECT` の `CASE` は残った行に対する計算なので、フィルタの代替にはなりません。

## PostgreSQL ならではの話

### FILTER 句との使い分け
PostgreSQL 9.4 以降は、集約関数に `FILTER` 句を使えます。CASE を包むより読みやすいことが多いです。

```sql
-- CASE を使う書き方
COUNT(CASE WHEN status = 'active' THEN 1 END)

-- FILTER を使う書き方（PostgreSQL）
COUNT(*) FILTER (WHERE status = 'active')
```

| 観点 | CASE | FILTER |
| --- | --- | --- |
| 可搬性 | 標準SQL、どのDBでも動く | PostgreSQL 固有 |
| 可読性 | 条件が複雑だと長くなりがち | 集計の条件分岐はシンプル |
| 表現力 | `SELECT` 全体で使える | 集約関数の直後のみ |

「本番は PostgreSQL 固定」なら `FILTER`、「他DBも意識する」なら CASE、という切り分けが現実的です。

### COALESCE / NULLIF との関係
CASE でよく書いていたパターンは、専用関数のほうが短いこともあります。

```sql
-- NULL を別の値に置き換え
COALESCE(nickname, name, '名無し')
-- 同等: CASE WHEN nickname IS NOT NULL THEN nickname WHEN name IS NOT NULL THEN name ELSE '名無し' END

-- 2つの値が同じなら NULL にする
NULLIF(column_a, column_b)
-- 同等: CASE WHEN column_a = column_b THEN NULL ELSE column_a END
```

ただし、**複数条件の分岐** は CASE のほうが向いています。COALESCE は「NULL なら次の値」という連鎖に特化した関数です。

### 型に注意
CASE の各 `THEN` 句は **同じデータ型** になる必要があります。型がバラバラだとエラーになるか、暗黙のキャストが入ります。

```sql
-- ❌ 型が揃っていないとエラーになりやすい
CASE
  WHEN flag THEN 1
  ELSE 'なし'
END

-- ✅ 明示的にキャスト
CASE
  WHEN flag THEN 1::text
  ELSE 'なし'
END
```

## CASE を使うときの注意

1. **評価順序**: 上から順に評価され、最初にマッチした分岐だけが返る（`IF / ELSIF / ELSE` と同じ）
2. **ELSE を省略すると NULL**: どの条件にも当てはまらない行は `NULL` になる
3. **ネストは深くしすぎない**: 3段以上になるなら、サブクエリや CTE で分けたほうが読みやすいことが多い
4. **パフォーマンス**: 単純な分岐なら問題になりにくいが、巨大テーブルの `GROUP BY` + 複雑な CASE はインデックス設計とセットで考える

## まとめ
- CASE は「文」ではなく「式」。`SELECT` 以外の句でも使える
- **行を減らすなら `WHERE`、値を変えるなら `SELECT` の `CASE`** — 役割が違う
- ラベル変換・柔軟な集計・ピボット・条件付き UPDATE に強い
- PostgreSQL では集計の条件分岐は `FILTER` 句も選択肢

## 参考
- ミック著『達人に学ぶSQL徹底指南書 第2版 初級者で終わりたくないあなたへ』（翔泳社, 2018）— 第1章「CASE式のススメ」
- PostgreSQL 公式ドキュメント — Conditional Expressions  
  https://www.postgresql.org/docs/current/functions-conditional.html
