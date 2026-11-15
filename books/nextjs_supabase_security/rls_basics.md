---
title: "RLS と Storage ポリシー"
---

公開スキーマ（特に `public`）の全テーブルで RLS を有効化します。ダッシュボードから作ったテーブルは既定で RLS が有効ですが、SQL Editor やマイグレーションで作ったテーブルは明示的に有効化が必要です。

RLS を有効にしただけでは、**そのロール向けのポリシーが1本も無い操作は拒否されます**（デフォルト拒否）。GRANT だけ残っていて「権限があるのに取れない」、またはポリシーを忘れて公開のまま、という混乱を避けるため、有効化とポリシー作成はセットで行います。

よくある悪い例です。

```sql
-- ❌ 悪い例1：ログインしていれば全員の行が読める
create policy "authenticated can read all"
on public.profiles for select
to authenticated
using (true);

-- ❌ 悪い例2：user_metadata はユーザー自身が書き換えられるので認可に使えない
create policy "admin can read all"
on public.profiles for select
to authenticated
using ((auth.jwt() -> 'user_metadata' ->> 'role') = 'admin');
```

ロールや管理者判定を JWT に載せる必要があるときは、ユーザーが編集できない **`app_metadata`（Auth Admin API でのみ更新）** を使うか、**所属テーブルなど DB 上の事実をポリシーで参照する**（toB 向けの具体例は次章）形にします。

良い例です。本人の行だけを扱い、INSERT / UPDATE には `with check` も付けて、他人の `user_id` へのなりすましや書き換えを防ぎます。

```sql
-- ✅ 良い例：本人の行だけ参照・作成・更新できる
alter table public.profiles enable row level security;

create policy "profiles: select own"
on public.profiles for select
to authenticated
using ((select auth.uid()) = user_id);

create policy "profiles: insert own"
on public.profiles for insert
to authenticated
with check ((select auth.uid()) = user_id);

create policy "profiles: update own"
on public.profiles for update
to authenticated
using ((select auth.uid()) = user_id)
with check ((select auth.uid()) = user_id);
```

未ログイン（`anon`）に公開してよい行だけ読ませたい場合は、`to anon` 用に別ポリシーを書き、`using` で公開条件を厳しく絞ります（例：フラグ列が true の行だけ）。それ以外のデータは `authenticated` 向けポリシーだけに任せます。

ポイント
- 全テーブルを同じ `auth.uid()` パターンで済ませず、実際のアクセスモデル（本人だけ、同じ企業のメンバー、公開など）に合わせて書く。
- INSERT・UPDATE には、操作種別ごとのポリシーに加え、参照用の SELECT ポリシーが必要なケースがある。UPDATE で SELECT ポリシーが無いとエラーにならず 0 件更新になるので、テストで気づきにくい。
- Storage も `storage.objects` へのポリシーで制御する。upsert には INSERT＋SELECT＋UPDATE が必要。