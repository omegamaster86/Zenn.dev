---
title: "GRANT・ビュー・security definer"
---

## GRANT とは

**GRANT** は PostgreSQL が持つ「誰（ロール）が、どのオブジェクトに、どの操作をしてよいか」を付与する SQL です。権限を外すのが **REVOKE** です。RLS のポリシー作成より前に、オブジェクト単位で通れるかどうかが決まります。

対象になるのは主に次のとおりです。

| 種類 | よく使う権限 | 意味（ざっくり） |
| --- | --- | --- |
| テーブル・ビュー | `SELECT` `INSERT` `UPDATE` `DELETE` | 行の読み書き |
| シーケンス | `USAGE` `SELECT` `UPDATE` | シリアル列の採番 |
| 関数 | `EXECUTE` | RPC や RLS からの呼び出し |

Supabase からブラウザ経由で DB に届くリクエストは、だいたい次のロールになります（前提章の publishable key・JWT の説明と対応します）。

- **`anon`** — 未ログイン、または JWT が付かないリクエスト
- **`authenticated`** — ログイン後、ユーザーの JWT が付いたリクエスト
- **`service_role`** — secret key 用。**RLS を迂回する**のでクライアントには載せない

GRANT と RLS は別レイヤーです。両方そろって初めて意図どおり動きます。

| レイヤー | 決めること |
| --- | --- |
| GRANT / REVOKE | そのロールがテーブル・ビュー・関数に**到達できるか**、**どの操作**ができるか |
| RLS（ポリシー） | 到達できたうえで、**どの行**に触れるか |

混同しやすいパターンは次のとおりです。

- GRANT はあるが RLS で行が絞られる → 権限はあるのに 0 行（正常）
- RLS は正しいが GRANT が無い → オブジェクト自体に到達できずエラー
- RLS を有効にしたがポリシーが無い → デフォルト拒否（前章参照）

明示的に付ける例です。

```sql
-- authenticated だけが profiles を読める（行の絞り込みは別途 RLS）
grant select on table public.profiles to authenticated;

-- RLS ポリシー内から呼ぶヘルパー関数
grant execute on function public.is_org_member(uuid) to authenticated;
```

方針は **必要なロールに、必要な操作だけ** です。特に `public` スキーマは Data API の公開対象になりやすいので、まず権限を絞り、RLS で行を守る順で考えます。

## GRANT を絞る（RLS とは別のレイヤー）

既存のプロジェクトでは、`public` に作った新しいテーブルや関数に `anon` / `authenticated` / `service_role` への権限が自動で付きます。意図せず公開されるのを防ぐなら、ダッシュボードの Data API 設定で新しいテーブルの自動公開をオフにするか、既定の権限を剥奪します。

```sql
-- 新しく作るオブジェクトに自動で権限が付かないようにする（公式ドキュメントの手順）
alter default privileges for role postgres in schema public
  revoke all on tables from anon, authenticated, service_role;

alter default privileges for role postgres in schema public
  revoke all on sequences from anon, authenticated, service_role;

alter default privileges for role postgres in schema public
  revoke all on functions from anon, authenticated, service_role;

alter default privileges for role postgres
  revoke execute on functions from public;
```

:::message alert
最後の文は他のスキーマで `postgres` が作る関数（RLS から呼ぶヘルパー関数など）にも効きます。それらの関数には、呼び出すロールへ `execute` を個別に付与しないとポリシーが失敗します。以後はテーブルごとに必要な権限だけを明示的に GRANT してください。
:::

既存テーブルで、ブラウザから触らせたくないものは個別に剥奪します。

```sql
-- 監査ログはブラウザ（anon / authenticated）から一切触れないようにする
revoke all on table public.audit_logs from anon, authenticated;
```

## ビューは既定で RLS を迂回し得る

ビューは既定では作成者の権限で実行されるため、元テーブルの RLS が効かないことがあります。Postgres 15 以上では `security_invoker = true` を付けて、呼び出したユーザーの権限で実行させます。

```sql
-- ✅ 呼び出したユーザーの権限で実行され、orders の RLS が効く
create view public.my_orders
with (security_invoker = true)
as
  select id, user_id, total, created_at
  from public.orders;

-- 既存のビューに後から設定する場合
alter view public.my_orders set (security_invoker = true);
```

Postgres 14 以前では、`anon` / `authenticated` からビューの権限を外すか、ビューを公開しないスキーマに移します。

## security definer 関数は公開スキーマに置かない

`security definer` 関数は作成者の権限で動くため RLS を迂回できます。RLS は関数そのものには適用されないので、公開スキーマに置くと Data API の RPC から誰でも呼べる入口になり得ます。

- Data API で公開しないスキーマ（例：`private`）に置く。
- `execute` は必要なロールにだけ付与する。
- 権限を上げる処理の中でも、呼び出し元が誰で、その操作を許されているかを確認する。
- `set search_path = ''` を付け、テーブルはスキーマ名付きで参照する。

具体例は次章の toB の所属テーブルの例で示します。
