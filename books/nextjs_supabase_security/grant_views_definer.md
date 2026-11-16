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

## security definer とは

PostgreSQL の関数には、**中の SQL を誰の権限で実行するか**を決めるモードがあります。ビューの `security_invoker`（前節）と同じ発想で、関数版を覚えると整理しやすいです。

| モード | 中の SQL が動く権限 | テーブルへの RLS |
| --- | --- | --- |
| **`security invoker`**（Postgres 15 以降の既定） | **呼び出したユーザー**（例：`authenticated`） | そのユーザー向けのポリシーがかかる |
| **`security definer`** | **関数の所有者**（マイグレーションなら多くは `postgres`） | 所有者の目で見える。所有者がテーブルオーナーだと RLS を通過し、行を広く読める |

イメージです。

- **invoker** — ユーザー本人が SQL を書いて実行しているのと同じ。GRANT と RLS の両方がそのユーザー基準。
- **definer** — 関数の中だけ一時的に「作成者の帽子」をかぶる。呼び出し元に `memberships` の SELECT を GRANT していなくても、関数の中からは読めることがある。

RLS のポリシー式の中で、別テーブル（所属テーブルなど）を参照したいときに `security definer` を使うことがあります。invoker のままだと「ポリシー評価 → テーブル参照 → またポリシー評価」となり、**再帰や期待外の拒否**が起きやすいからです。definer にすると、関数の中だけ作成者の権限で所属行を読み、**`auth.uid()` などで呼び出し元を絞った結果**を返せます。

一方で definer は**権限昇格の入口**になります。

- RLS は**関数本体**にはかからない（関数は行テーブルではない）。
- `public` に置き `grant execute ... to authenticated` すると、ブラウザから **Data API の RPC** で誰でも呼べる。
- 関数の中で `auth.uid()` を確認せずに秘密の行を返すと、RLS で守っていたデータが一気に漏れる。

安全に使うときの最低ラインは次のとおりです。

```sql
-- ✅ RLS ポリシーからだけ使うヘルパー（RPC には出さない）
create schema if not exists private;

create or replace function private.org_role(_org_id uuid)
returns text
language sql
stable
security definer
set search_path = ''
as $$
  select m.role
  from public.memberships m
  where m.org_id = _org_id
    and m.user_id = (select auth.uid());  -- 呼び出し元を必ず絞る
$$;

revoke all on function private.org_role(uuid) from public;
grant execute on function private.org_role(uuid) to authenticated;
```

```sql
-- ❌ public に definer を置き、中身で呼び出し元を検証しない
create function public.leak_all_profiles()
returns setof public.profiles
language sql
security definer
as $$ select * from public.profiles; $$;

grant execute on function public.leak_all_profiles() to authenticated;
-- → JWT を持つユーザーなら誰でも全行取得し得る
```

## security definer 関数は公開スキーマに置かない

運用ルールは次のとおりです（toB の所属チェックの流れは次章）。

- Data API で公開しないスキーマ（例：`private`）に置く。ダッシュボードの API 設定で `private` を公開スキーマに**追加しない**。
- `execute` は必要なロールにだけ付与する（`revoke ... from public` を忘れない）。
- definer の中でも、**`auth.uid()` や引数で「この操作を許すか」**を必ず確認する。作成者権限は「裏で参照するため」であって、無制限にデータを返すためではない。
- `set search_path = ''` を付け、テーブルは `public.memberships` のようにスキーマ名付きで参照する（search_path 乗っ取り対策）。

