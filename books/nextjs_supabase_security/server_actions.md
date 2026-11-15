---
title: "Server Action とサーバー集約"
---

## Server Action ごとに認証・認可を再確認する

Server Action は直接 POST で呼べるため、**各アクションの中で**認証と、対象リソースへの権限（認可）を確認します。返り値も必要最小限に絞ります。

まず、`@supabase/ssr` でサーバー用のクライアントを作ります。

```ts
// lib/supabase/server.ts
import 'server-only'
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options),
            )
          } catch {
            // Server Component から呼ばれた場合は Cookie を書き込めないので無視する。
            // トークンの更新は Proxy（Next.js 15 以前は middleware）で行う
          }
        },
      },
    },
  )
}
```

Server Action の例です（入力検証に zod を使っています）。

```ts
// app/projects/actions.ts
'use server'

import { z } from 'zod'
import { createClient } from '@/lib/supabase/server'

const RenameProjectInput = z.object({
  projectId: z.string().uuid(),
  name: z.string().min(1).max(100),
})

export async function renameProject(input: unknown) {
  // 0. 入力検証：クライアントから来た値は信用しない
  const parsed = RenameProjectInput.safeParse(input)
  if (!parsed.success) return { ok: false, error: 'invalid_input' } as const

  const supabase = await createClient()

  // 1. 認証：Cookie の中身をそのまま信じず、Auth サーバーに確認する
  const {
    data: { user },
    error: authError,
  } = await supabase.auth.getUser()
  if (authError || !user) return { ok: false, error: 'unauthorized' } as const

  // 2. 認可：対象プロジェクトの企業に、必要な役割で所属しているか
  const { data: project } = await supabase
    .from('projects')
    .select('id, org_id')
    .eq('id', parsed.data.projectId)
    .maybeSingle()
  if (!project) return { ok: false, error: 'not_found' } as const

  const { data: membership } = await supabase
    .from('memberships')
    .select('role')
    .eq('org_id', project.org_id)
    .eq('user_id', user.id)
    .maybeSingle()
  if (!membership || !['owner', 'admin'].includes(membership.role)) {
    return { ok: false, error: 'forbidden' } as const
  }

  // 3. 更新：ユーザーの JWT のまま実行するので RLS も二重にかかる
  const { error: updateError } = await supabase
    .from('projects')
    .update({ name: parsed.data.name })
    .eq('id', project.id)
  if (updateError) return { ok: false, error: 'update_failed' } as const

  // 4. 返り値は必要最小限
  return { ok: true } as const
}
```

- `getSession()` は Cookie から読むだけで再検証しないため、サーバー側の認可には使わない。
- 署名を検証したクレームだけでよい場面（ページの保護など）では `getClaims()` も使える。Auth サーバー上の最新のユーザー情報で確認したいときは `getUser()` を使う（ネットワーク呼び出しが発生する）。

## ユーザーの JWT で呼ぶだけでは「集約」にならない

上の例はユーザーの JWT で Supabase を呼んでいるので、ブラウザも同じ JWT で同じテーブルを直接読み書きできます。アクション内のチェックは多層防御として有効ですが、「この操作はサーバー経由でしかできない」状態にはなっていません。

他者データの参照、一括エクスポート、権限変更、決済などの重要操作を本当にサーバーへ集約するには、**ブラウザのロールからそのテーブル・関数に届かないよう DB 側の権限を絞る**必要があります。

- 公開スキーマに置いたまま `anon` / `authenticated` の権限を剥奪し、サーバーからは secret key（`service_role`）でアクセスする。
- または Data API で公開しないスキーマに置き、サーバーから Postgres に直接接続（ORM など）してアクセスする。公開しないスキーマは secret key を使っても Data API からは届かない点に注意。

```sql
-- org_reports はブラウザからは一切読めない。サーバー（service_role）だけが読む
revoke all on table public.org_reports from anon, authenticated;
grant select on table public.org_reports to service_role;
```

```ts
// app/reports/actions.ts
'use server'

import { createClient } from '@/lib/supabase/server'
import { createAdminClient } from '@/lib/supabase/admin'

export async function exportOrgReports(orgId: string) {
  const supabase = await createClient()

  // 1. 認証
  const {
    data: { user },
  } = await supabase.auth.getUser()
  if (!user) throw new Error('unauthorized')

  // 2. 認可：引数の orgId を信じず、所属と役割を確認する
  const { data: membership } = await supabase
    .from('memberships')
    .select('role')
    .eq('org_id', orgId)
    .eq('user_id', user.id)
    .maybeSingle()
  if (membership?.role !== 'owner') throw new Error('forbidden')

  // 3. secret key は RLS を迂回するので、org_id の絞り込みは必須
  const admin = createAdminClient()
  const { data, error } = await admin
    .from('org_reports')
    .select('id, title, created_at')
    .eq('org_id', orgId)
    .limit(1000)
  if (error) throw error

  // 4. 重要操作は監査ログに残す
  await admin
    .from('audit_logs')
    .insert({ actor_id: user.id, org_id: orgId, action: 'export_org_reports' })

  return data
}
```

サーバー側では、毎回の権限検証に加えて、操作単位のレート制限や、重要操作での再認証（直近の MFA やパスワード確認など）も検討します。同じ考え方は Route Handler や Supabase Edge Functions でも使えます。

## Data API を使わないなら無効化する

アプリが Supabase のクライアントライブラリや REST / GraphQL のデータエンドポイントを使わない（サーバーから Postgres に直接つなぐだけ、など）なら、ダッシュボードで Data API 自体を無効化できます。無効化すると、GRANT や RLS の設定に関係なく自動生成の REST エンドポイントは応答しなくなります。
