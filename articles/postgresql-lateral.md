---
title: "PostgreSQLのLATERALに関して調べてみたんじゃ"
emoji: "⏳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PostgreSQL", "LATERAL"]
published: true
published_at: 2025-10-12 20:00
---

# はじめに
いや〜いきなり寒くなりましたね〜
そろそろ鍋の季節ですが、皆さんは何鍋派ですか？
さて今回は`LATERAL`という聞き慣れない文法を見つけたので、使い方を調べてみました〜

# 本題
`LEFT JOIN LATERAL` は PostgreSQL の強力な機能です。

```sql
SELECT * 
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o 
  WHERE o.user_id = u.id  -- 左側の u を参照できる!
  ORDER BY o.created_at DESC  -- 最新順に並べる（ORDER BY なしの LIMIT は不定）
  LIMIT 5
) o ON TRUE
```

- **左側のテーブルの各行を参照できる**サブクエリ
- 左側の各行ごとにサブクエリが評価される
- `ON TRUE` は常に結合（サブクエリ内で条件指定済みのため）

### LATERAL の重要な特徴
特徴1: 左側のテーブルを参照できる

```sql
-- ❌ 通常のサブクエリ（エラー）
SELECT *
FROM users u
LEFT JOIN (
  SELECT * FROM orders 
  WHERE user_id = u.id  -- u が見えない!
) o ON TRUE

-- ✅ LATERAL（正常動作）
SELECT *
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders 
  WHERE user_id = u.id  -- u を参照できる!
) o ON TRUE
```

### 特徴2: 行ごとに評価される
左側の各行に対して、サブクエリが個別に評価されます（ただし、PostgreSQLの最適化により実際の実行計画は状況によって異なります）。

## LEFT JOIN LATERAL のユースケース

| ユースケース | 説明 |
| --- | --- |
| **Top-N**  | 各グループの上位N件を取得 |
| 複雑な集計 | 行ごとに異なる条件で集計 |
| 相関サブクエリの最適化 | 従来の相関サブクエリを効率化 |
| 動的な結合条件 | 左側の値に基づいて動的に結合 |

usersテーブル

| id | name |
| --- | --- |
| 1 | 太郎 |
| 2 | 花子 |
| 3 | 次郎 |

orders テーブル

| id | user_id | product | amount |
| --- | --- | --- | --- |
| 1 | 1 | 本 | 1000 |
| 2 | 1 | ペン | 500 |
| 3 | 1 | ノート | 300 |
| 4 | 1 | 消しゴム | 100 |
| 5 | 1 | 定規 | 200 |
| 6 | 1 | 鉛筆 | 150 |
| 7 | 1 | はさみ | 400 |
| 8 | 2 | 本 | 2000 |
| 9 | 2 | ペン | 600 |

実行結果

| user_id | name | order_id | product | amount |
| --- | --- | --- | --- | --- |
| 1 | 太郎 | 1 | 本 | 1000 |
| 1 | 太郎 | 2 | ペン | 500 |
| 1 | 太郎 | 3 | ノート | 300 |
| 1 | 太郎 | 4 | 消しゴム | 100 |
| 1 | 太郎 | 5 | 定規 | 200 |
| 2 | 花子 | 8 | 本 | 2000 |
| 2 | 花子 | 9 | ペン | 600 |
| 3 | 次郎 | NULL | NULL | NULL |

## ポイント

### 1. **LIMIT 5 の効果**

- 太郎は7件の注文があるが、**5件のみ**取得される（6件目と7件目は除外）
- 花子は2件なので全て取得される

### 2. **LEFT JOIN の効果**

- 次郎は注文がないが、**ユーザー情報は表示される**（注文列は NULL）
- `INNER JOIN` なら次郎は結果に含まれない

### 3. **処理の流れ**

```
1. ユーザー「太郎」を取得
   → LATERAL サブクエリ: 太郎の注文を5件取得
   → 結果: 太郎の行が5行できる

2. ユーザー「花子」を取得
   → LATERAL サブクエリ: 花子の注文を2件取得
   → 結果: 花子の行が2行できる

3. ユーザー「次郎」を取得
   → LATERAL サブクエリ: 次郎の注文0件
   → 結果: 次郎の行が1行（注文情報はNULL）
```

これで **「各ユーザーの最新5件の注文」** を取得できます！

> 注: 上のクエリは `orders` に `created_at` 列がある前提です。最新を厳密に取りたい場合は、必ずサブクエリ内で `ORDER BY created_at DESC` と `LIMIT` を組み合わせてください（`ORDER BY` がない `LIMIT` は任意の5件になります）。

## 補足: 見落としやすいポイント

### ORDER BY なしの LIMIT は不定
「最新N件」を取りたいのに `ORDER BY` を書かないと、返る行は不定です。必ずソートキーを明示しましょう。

```sql
-- 悪い例（不定の5件）
SELECT *
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o
  WHERE o.user_id = u.id
  LIMIT 5
) o ON TRUE;

-- 良い例（created_at の降順で最新5件）
SELECT *
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o
  WHERE o.user_id = u.id
  ORDER BY o.created_at DESC
  LIMIT 5
) o ON TRUE;
```

### LEFT JOIN LATERAL と CROSS JOIN LATERAL の違い
- `LEFT JOIN LATERAL`: 右側（サブクエリ）が0行でも左側の行を残す（右側はNULL）。上記の次郎の部分です。
- `CROSS JOIN LATERAL`: 右側が0行だと結果にも現れない（必須関係）。

```sql
-- 右側が0行でもユーザーを残したい
SELECT u.*, o.*
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o WHERE o.user_id = u.id ORDER BY o.created_at DESC LIMIT 5
) o ON TRUE;

-- 右側が0行なら除外したい
SELECT u.*, o.*
FROM users u
CROSS JOIN LATERAL (
  SELECT * FROM orders o WHERE o.user_id = u.id ORDER BY o.created_at DESC LIMIT 5
) o;
```

### ON 句と WHERE 句の使い分け（LEFT を潰さない）
`LEFT JOIN` では、結合後の `WHERE o.id IS NOT NULL` のような条件を書くと、実質的に `INNER JOIN` 化してしまいます。左外部結合の性質を残したい場合は、可能な限り条件を LATERAL サブクエリ内に入れるか、`ON` 句で表現し、外側の `WHERE` で右側の列に非NULL制約を置かないようにしましょう。

## 代替アプローチ

### 1) ウィンドウ関数（ROW_NUMBER）
N件取りたいだけならウィンドウ関数もシンプルで強力です。

```sql
WITH ranked AS (
  SELECT
    u.id  AS user_id,
    u.name,
    o.id  AS order_id,
    o.product,
    o.amount,
    ROW_NUMBER() OVER (
      PARTITION BY u.id
      ORDER BY o.created_at DESC
    ) AS rn
  FROM users u
  LEFT JOIN orders o ON o.user_id = u.id
)
SELECT *
FROM ranked
WHERE rn <= 5
ORDER BY user_id, rn;
```

長所: 1回のスキャンで全体を処理しやすい。短所: 書き味はやや長い。用途やデータ量次第で `LATERAL` と使い分け。

### 2) DISTINCT ON（最新1件の取得に最適）
PostgreSQL 独自の `DISTINCT ON` は「各グループの最新1件」を短く書けます（N>1は苦手）。

```sql
-- LATERALを使う場合（最新1件）
SELECT u.id AS user_id, u.name, o.*
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o
  WHERE o.user_id = u.id
  ORDER BY o.created_at DESC
  LIMIT 1
) o ON TRUE;

-- DISTINCT ONを使う場合（LATERALなし）
SELECT DISTINCT ON (u.id) u.id, u.name, o.*
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
ORDER BY u.id, o.created_at DESC NULLS LAST;
```

DISTINCT ON を使う場合、ORDER BY の最初のカラムは DISTINCT ON で指定したカラムと一致させる必要があります。また、LEFT JOIN で注文がないユーザーも含める場合は `NULLS LAST` を指定すると安全です。

## 性能上の注意とインデックス
- `LATERAL` は左側の行ごとに右側サブクエリが評価されるため、左側の件数が多いと負荷が高くなる可能性があります。
- `orders(user_id, created_at DESC)` の複合インデックスを作ると、`WHERE user_id = ? ORDER BY created_at DESC LIMIT N` が効率的に走りやすくなります。

```sql
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_user_created_at_desc
  ON orders (user_id, created_at DESC);
```

- 大量データで N がそこそこ大きい場合は、ウィンドウ関数でまとめて処理してから結合するほうが速いケースもあります（実測・EXPLAINで判断）。

## 応用: JSON で子テーブルを束ねて返す
各ユーザーに「最新5件の注文一覧（JSON配列）」をぶら下げたい場合。

```sql
SELECT u.id, u.name, o.recent_orders
FROM users u
LEFT JOIN LATERAL (
  SELECT json_agg(o ORDER BY o.created_at DESC) AS recent_orders
  FROM (
    SELECT * FROM orders o2
    WHERE o2.user_id = u.id
    ORDER BY o2.created_at DESC
    LIMIT 5
  ) o
) o ON TRUE;
```

## 参考
PostgreSQL 公式ドキュメント
https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-LATERAL

Top-N per group（PostgreSQL Wiki）
https://wiki.postgresql.org/wiki/Least_(N)_per_Group