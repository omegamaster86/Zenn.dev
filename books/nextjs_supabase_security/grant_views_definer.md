---
title: "GRANT・ビュー・security definer"
---

## GRANT を絞る（RLS とは別のレイヤー）

GRANT は「そのロールがテーブル・ビュー・関数に到達できるか」、RLS は「どの行に触れるか」を決めます。両方を設定します。

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