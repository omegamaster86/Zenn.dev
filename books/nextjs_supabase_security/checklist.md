---
title: "チェックリスト・まとめ・参考リンク"
---

## 最初にやること

- 全公開テーブルの RLS と Storage ポリシーを整備し、ユーザー横断・企業横断のアクセスが拒否されることをテストする
- `anon` / `authenticated` への GRANT と既定の権限を見直し、不要なものを剥奪する
- 管理者（アプリの管理者と Supabase ダッシュボードの利用者）の MFA を必須化する
- secret key がブラウザ・`NEXT_PUBLIC_`・リポジトリに無いことを確認し、サーバーでの利用箇所を最小化する
- Server Action / Route Handler ごとに認証・認可を実装し、ビューと `security definer` 関数を点検する
- 重要操作は DB 側の権限を絞った上でサーバーに集約する
- ログ監視（ログイン失敗、拒否、大量取得）と、アカウント停止・セッション失効・鍵ローテーションの対応手順を用意する

## アクセステストで確認するケース

| ケース | 期待結果 |
| --- | --- |
| 匿名（未ログイン） | 公開データ以外は取得・更新できない |
| 本人 | 自分のデータのみ操作できる |
| 他人（同じ企業の別ユーザー） | 役割で許可された範囲以外は拒否 |
| 他社のユーザー | 企業 ID を書き換えても一切取得できない |
| 権限失効済み（退職・役割変更・停止後） | 失効後は拒否される（JWT 有効期限内の挙動も確認） |

テストは Next.js 経由だけでなく、**publishable key で Supabase の API を直接叩く経路**と Storage についても行います。

## まとめ

- Supabase の API はブラウザから直接呼べる。だから**DB 側（RLS・GRANT）を最後の砦**にする。
- RLS は全テーブルで有効化し、ビューは `security_invoker`、`security definer` 関数は公開しないスキーマへ。
- secret key は RLS を迂回する。`server-only` で閉じ込め、使う箇所では必ず自前で認可する。
- Server Action はアクションごとに認証・認可する。ユーザーの JWT で呼ぶだけでは集約にならないので、重要操作は DB 側の権限を絞ってからサーバーに寄せる。
- WAF は Next.js の前、ネットワーク制限は Postgres だけ、事前チェック関数は Data API だけ。それぞれ守れる範囲を把握して組み合わせる。
- 乗っ取り・セッション失効・大量取得は RLS だけでは防げない。MFA、監査ログ、対応手順まで含めて準備する。

## 参考リンク

- [Supabase: API keys](https://supabase.com/docs/guides/api/api-keys)
- [Supabase: Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supabase: Securing your API](https://supabase.com/docs/guides/api/securing-your-api)
- [Supabase: Hardening the Data API](https://supabase.com/docs/guides/database/hardening-data-api)
- [Supabase: Network Restrictions](https://supabase.com/docs/guides/platform/network-restrictions)
- [Supabase: Storage access control](https://supabase.com/docs/guides/storage/security/access-control)
- [Supabase: Creating a Supabase client for SSR](https://supabase.com/docs/guides/auth/server-side/nextjs)
- [Supabase: Auth rate limits](https://supabase.com/docs/guides/auth/rate-limits)
- [Supabase: CAPTCHA protection](https://supabase.com/docs/guides/auth/auth-captcha)
- [Supabase: Multi-factor authentication](https://supabase.com/docs/guides/auth/auth-mfa)
- [Supabase: User sessions](https://supabase.com/docs/guides/auth/sessions)
- [Supabase: Auth audit logs](https://supabase.com/docs/guides/auth/audit-logs)
- [Supabase: Log Drains](https://supabase.com/docs/guides/telemetry/log-drains)
- [Supabase: SAML SSO](https://supabase.com/docs/guides/auth/enterprise-sso/auth-sso-saml)
- [Supabase: Production checklist](https://supabase.com/docs/guides/deployment/going-into-prod)
- [Supabase: Product security](https://supabase.com/docs/guides/security/product-security)
- [Next.js: Data security](https://nextjs.org/docs/app/guides/data-security)
- [Supabase pricing](https://supabase.com/pricing)