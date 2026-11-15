---
title: "toB（membership）と MFA"
---

## toB：所属（membership）テーブルで企業単位に分離する

マルチテナントでは、「リクエストに含まれる企業 ID」ではなく、所属テーブル上の所属と役割を認可の根拠にします。

```sql
-- 所属テーブル：誰がどの企業にどの役割で所属しているか
create table public.memberships (
  org_id  uuid not null references public.organizations (id) on delete cascade,
  user_id uuid not null references auth.users (id) on delete cascade,
  role    text not null check (role in ('owner', 'admin', 'member')),
  primary key (org_id, user_id)
);

alter table public.memberships enable row level security;

-- 自分の所属だけ見える。INSERT / UPDATE / DELETE のポリシーは作らず、
-- 招待や権限変更はサーバー側で認可した上で行う
create policy "memberships: select own"
on public.memberships for select
to authenticated
using ((select auth.uid()) = user_id);

-- RLS から呼ぶヘルパー関数は公開しないスキーマに置く
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
    and m.user_id = (select auth.uid());
$$;

revoke all on function private.org_role(uuid) from public;
grant usage on schema private to authenticated;
grant execute on function private.org_role(uuid) to authenticated;

-- 企業のメンバーだけがプロジェクトを参照できる
alter table public.projects enable row level security;

create policy "projects: members can select"
on public.projects for select
to authenticated
using (private.org_role(org_id) is not null);

-- owner / admin だけが更新できる。with check で別企業への付け替えも防ぐ
create policy "projects: admins can update"
on public.projects for update
to authenticated
using (private.org_role(org_id) in ('owner', 'admin'))
with check (private.org_role(org_id) in ('owner', 'admin'));
```

`private` スキーマは Data API の公開スキーマに追加しないでください。ヘルパー関数を `security definer` にしているのは、ポリシーの中から RLS 付きの `memberships` を参照するときに、RLS が再帰的に評価されるのを避けるためです。

## 重要なテーブルは MFA（aal2）を要求する

MFA を済ませたセッションの JWT には `aal` クレームとして `aal2` が入ります。`as restrictive` のポリシーにすると、他の許可ポリシーがあっても MFA 未完了のアクセスを拒否できます。

```sql
-- MFA（aal2）を済ませたセッションだけ管理用テーブルに触れる
create policy "admin_settings: require mfa"
on public.admin_settings
as restrictive
to authenticated
using ((select auth.jwt() ->> 'aal') = 'aal2');
```

restrictive ポリシーは「絞る」だけで、単体ではアクセスを許可しません。別途、通常の（permissive な）ポリシーで誰が触れるかを定義してください。