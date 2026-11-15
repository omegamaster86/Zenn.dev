---
title: "WAF/CDN・レート制限"
---

## できること

- **WAF/CDN は Next.js の前に置く**。不審な通信や大量アクセスを、アプリに届く前に抑止できる。
- **Data API の事前チェック関数**：`pgrst.db_pre_request` に登録した Postgres 関数で、リクエストごとにヘッダーや IP を見てレート制限や追加の API キー確認を自作できる。公式ドキュメントに実装例がある。
- **Auth のレート制限と CAPTCHA**：サインアップやログインなど認証エンドポイントへの総当たり・bot 登録を抑える。
- **ネットワーク制限**：Postgres とプーラーに接続できる IP 範囲を制限できる。

```sql
-- 事前チェック関数の登録（関数本体は公式ドキュメントの例を参照）
alter role authenticator
  set pgrst.db_pre_request = 'public.check_request';

notify pgrst, 'reload config';
```

## できないこと・注意点

- **Next.js の前の WAF では、Supabase への直接アクセス（`*.supabase.co`）は守れない**。ブラウザから直接呼べる API は、DB 側の RLS・GRANT で守る。
- **ネットワーク制限は Postgres とプーラーだけ**に効く。Data API（PostgREST）、Auth、Storage などの HTTPS API や supabase-js 経由のアクセスには効かない。
- 事前チェック関数は **Data API（PostgREST）だけ**に効き、Realtime や Storage には効かない。また公式のIP単位レート制限の例は、GET / HEAD をカウントしない（書き込み系のみ）。読み取りの大量取得対策には別の手段が必要。
- Auth のレート制限と CAPTCHA は**認証エンドポイント向け**の保護で、全 API を守る WAF ではない。
- CDN や ISR を使う場合、セッション更新時の `Set-Cookie` を含むレスポンスがキャッシュされると、別のユーザーに他人のセッションが渡ってしまうおそれがある。`@supabase/ssr` の `setAll` が渡すキャッシュ制御ヘッダーを Proxy でレスポンスに付けるなど、公式ガイドのキャッシュの注意点に従う。