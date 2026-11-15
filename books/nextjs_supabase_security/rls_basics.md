---
title: "RLS と Storage ポリシー"
---

公開スキーマ（特に `public`）の全テーブルで RLS を有効化します。ダッシュボードから作ったテーブルは既定で RLS が有効ですが、SQL Editor やマイグレーションで作ったテーブルは明示的に有効化が必要です。

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

良い例です。本人の行だけを扱い、UPDATE には `with check` も付けて、他人の `user_id` への書き換えを防ぎます。

```sql
-- ✅ 良い例：本人の行だけ参照・更新できる
alter table public.profiles enable row level security;

create policy "profiles: select own"
on public.profiles for select
to authenticated
using ((select auth.uid()) = user_id);

create policy "profiles: update own"
on public.profiles for update
to authenticated
using ((select auth.uid()) = user_id)
with check ((select auth.uid()) = user_id);
```

ポイント：

- 全テーブルを同じ `auth.uid()` パターンで済ませず、実際のアクセスモデル（本人だけ、同じ企業のメンバー、公開など）に合わせて書く。
- UPDATE には SELECT ポリシーも必要。無いとエラーにならず 0 件更新になるので、テストで気づきにくい。
- Storage も `storage.objects` へのポリシーで制御する。upsert には INSERT＋SELECT＋UPDATE が必要。