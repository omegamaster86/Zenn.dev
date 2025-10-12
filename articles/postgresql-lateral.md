---
title: "PostgreSQLのLATERALに関して"
emoji: "⏳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PostgreSQL", "LATERAL"]
published: false
# published_at: 2025-10-05 17:00
---

# はじめに
いや〜いきなり寒くなりましたね〜
そろそろ鍋の季節ですが、皆さんは何鍋派ですか？
さて今回は`LATERAL`という聞き慣れない文法を見つけたので、使い方を調べてみました〜

# 本題

`LEFT JOIN LATERAL` は PostgreSQL の強力な機能です。

```jsx
SELECT * 
FROM users u
LEFT JOIN LATERAL (
  SELECT * FROM orders o 
  WHERE o.user_id = u.id  -- 左側の u を参照できる!
  LIMIT 5
) o ON TRUE
```

- **左側のテーブルの各行を参照できる**サブクエリ
- 左側の各行ごとにサブクエリが実行される
- `ON TRUE` は常に結合（サブクエリ内で条件指定済みのため）

### LATERAL の重要な特徴
特徴1: 左側のテーブルを参照できる

```jsx
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

### 特徴2: 行ごとに動的に実行
左側の各行に対して、サブクエリが個別に実行されます。

## LEFT JOIN LATERAL のユースケース

| ユースケース | 説明 |
| --- | --- |
| **Top-N**  | **クエリ**各グループの上位N件を取得 |
| 複雑な集計 | 行ごとに異なる条件で集計 |
| 相関サブクエリの最適化 | 従来の相関サブクエリを効率化 |
| 動的な結合条件 | 左側の値に基づいて動的に結合 |

usersテーブル

| id | name |
| --- | --- |
| 1 | 一龍 |
| 2 | 二郎 |
| 3 | 三虎 |

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

```jsx
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