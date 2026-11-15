---
title: "前提：Supabase の API はブラウザから直接呼べる"
---

Supabase のクライアントは、publishable key（anon key）を使ってブラウザから Data API・Auth・Storage を直接呼び出します。publishable key は公開される前提の鍵なので、「鍵を知っている人だけがアクセスできる」という守り方はできません。

そのため、守りは次の3層で考えます。

| 層 | 役割 | 主な手段 |
| --- | --- | --- |
| DB（Supabase） | 誰がどの行・ファイルに触れるかを最終的に制限 | RLS、Storage ポリシー、GRANT の最小化 |
| アプリ（Next.js） | 各サーバー操作ごとに認証・認可 | Server Action / Route Handler 内での毎回の検証、DAL、入力検証 |
| 鍵・運用 | 鍵と運用者アカウントの保護、検知と対応 | secret key の管理、管理者 MFA、ログ監視、対応手順 |

目標は、**Next.js に不備があっても、他人のデータは DB 側で返らない**状態です。