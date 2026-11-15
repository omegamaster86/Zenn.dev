---
title: "secret key の扱い"
---

secret key（`sb_secret_...`、旧 `service_role` キー）は **RLS を迂回**します。

- ブラウザ、モバイルアプリ、ソース管理、 `NEXT_PUBLIC_` 付きの環境変数には絶対に置かない（`NEXT_PUBLIC_` はブラウザ向けのバンドルに埋め込まれる）。
- サーバー側でも安易に使わない。ユーザー操作はユーザーの JWT と publishable key のクライアントで実行して RLS を効かせ、secret key は管理処理・バッチ・後述のサーバー集約など、自前で認可チェックを行う箇所に限定する。
- コンポーネントごとに別の secret key を使い、漏えい時はそのキーだけローテーションできるようにする。
- secret key を使うモジュールは `import 'server-only'` で保護し、`process.env` の参照を1か所（DAL など）に閉じ込める。

```ts
// lib/supabase/admin.ts
import 'server-only' // クライアントコンポーネントから import されるとビルドエラーになる
import { createClient } from '@supabase/supabase-js'

export function createAdminClient() {
  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SECRET_KEY!, // NEXT_PUBLIC_ を付けない
    {
      auth: {
        persistSession: false,
        autoRefreshToken: false,
      },
    },
  )
}
```

`server-only` は npm パッケージなので、`npm install server-only` で追加してから使います。