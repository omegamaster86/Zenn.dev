---
title: "Stripe Projectを試してみたんじゃ"
emoji: "🏃‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Stripe","Stripe Project","AI","AI駆動開発","CLI"]
published: true
published_at:2026-05-10 11:30
publication_name: "genai"
---
# 初めに

Stripe Project（Stripe CLI の `projects` プラグインとワークフロー）を入れると、**サードパーティのアカウント作成からローカルの環境変数まで**の進め方が変わります。ここでは「実際の開発で手が触れる部分」に絞って対比します。

| 開発の場面 | 使用前（Stripe Project を使わない） | 使用後（Stripe Project を使う） |
| --- | --- | --- |
| プロバイダのアカウント | 利用するサービスごとにサインアップし、ダッシュボードで API キーやプロジェクトを用意することが多い | `stripe projects link <provider>` や `stripe projects add …` で、Stripe アカウントとプロバイダ側アカウントの紐づけ・リソース作成を CLI から進められる（既存アカウントを使うかどうかもその流れの中で決められる） |
| ローカルの環境変数（`.env`） | README にキー名だけ書いて、各自がコンソールから値をコピペして `.env` を埋める。初回セットアップが属人化しやすい | サービスを追加・変更したタイミングで CLI が資格情報を取りに行き、ローカルの `.env` に反映される。手元を最新に揃えたいときは `stripe projects env --pull`（新しいマシンや clone 直後など） |
| シークレットの置き場 | チームによってはパスワードマネージャやチャットでキーを共有するなど、運用にばらつき | 取得した資格情報は `.projects/vault/vault.json` に暗号化してキャッシュされ、Stripe Secret Store と連携する（ドキュメント上のモデル）。`.env` はローカル開発用のプレーンテキストのまま |
| Git に載せるもの | `.env` の誤コミットリスク。`.gitignore` をプロジェクトごとに整える必要 | `stripe projects init` 時に `.gitignore` へ vault や `.env` を入れる流れ。コミット対象は主に `.projects/state.json` など「プロジェクトの状態」で、チームで同じ土台を共有しやすい |
| チーム開発・マシンを増やしたとき | 「誰がどのキーを発行したか」「どこから取ればよいか」を都度説明しがち | リポジトリを pull / clone したあと、メンバーが自分のプロバイダ紐づけを済ませ、`env --pull` で自分用の資格情報を手元に引く、という手順に寄せられる（※ `state.local.json` はチーム共有の前提になるなど、公式ドキュメントのファイル単位のルールに従う） |
| 本番・ステージングの環境変数 | ホスティング先のダッシュボードや CI のシークレットに、別途キーを登録する（今どきも同様） | **Projects が本番ホストに自動で `.env` を流し込むわけではない**（ドキュメント明記）。ローカルで揃った変数名・値を、従来どおりホスト側やプロバイダの手順に沿って設定する必要がある |
| エージェントとの開発（任意） | Stripe 連携のドキュメントをブラウザで行き来しながら実装指示を書くことが多い | `stripe-projects` スキルで Projects のコマンドと手順に寄せた提案がしやすく、`skills-lock.json` でスキル構成もプロジェクトに残せる |

※ プロビジョニングや有料プランの変更では、Stripe 側でログイン・請求まわり（`stripe projects billing add` など）を整えてから進める必要があります。環境は dev / staging / production で **別プロジェクトに分ける** のが推奨されています。


# 設定してみよう
```
brew install stripe/stripe-cli/stripe && stripe plugin install projects
```
Stripe にログイン（未ログインなら）
```
stripe login
```
Projects は「請求情報を Stripe 経由でまとめる」前提の機能なので、アカウント・請求まわりは Stripe 側の案内に従う必要があります。

公式にあったskillsをインストール
```
npx skills add https://docs.stripe.com
```
実行するとこんな感じの選択画面が表示されます。
![](/images/stripe-project/1.png)

| スキル | 意味 | 選ぶか |
| --- | --- | --- |
| stripe-projects | Projects / CLI まわりのガイド | 選ぶ（これがメイン） |
| stripe-best-practices | Stripe 連携の一般的なベストプラクティス | 任意（Stripe の実装もするならおすすめ） |
| upgrade-stripe | API バージョンや SDK のアップグレード手順 | 今回の「Projects を試す」だけなら不要。バージョン上げを検討するときに入れればよい |

Stripeの機能は必要としないので、stripe-projectsを選択していきます。 
選択するにはスペースをクリックする必要があるようですね。
![](/images/stripe-project/2.png)
選択して、Enterをクリックするとこんな選択肢が表示されます。
![](/images/stripe-project/3.png)
32もエージェントがあるんだと思う一方でcursorがなかったので、納得がいきません。激おこです。
何も選ばずにEnterをクリックすると、どこまで適応させるか選択することができます。
今回はProjectを選択しておきましょう。
![](/images/stripe-project/4.png)

その後、skills-lock.jsonが作成されます。

projects プラグインをアップグレードする必要がある場合は、以下のコマンドを実行します。
```
stripe plugin upgrade projects
```
:::message
現在の Stripe CLI バージョンが Projects プラグインをサポートしていない場合は、Stripe CLI をアップグレードしてください。
:::
入っているか確認する場合次のコマンドを実行
```
stripe plugin list
```

プロジェクト初期化（ルートで）
```
stripe projects init
```
公開してはいけない系があったので、スクショは控えますね…
その後こんなファイルが作成されています。
![](/images/stripe-project/5.png)
![](/images/stripe-project/6.png)

使えるプロバイダを見てみましょう。
```
stripe projects catalog
```
実行結果の一例です。カタログの内容はアップデートで変わります。

### 利用できるプロバイダー一覧
#### HOSTING

| サービス | プラン | 概要 |
| --- | --- | --- |
| cloudflare/workers | Free | Workers のビルド・デプロイ・スケール |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |
| huggingface/platform | Free & Paid | Hugging Face プラットフォームへのアクセス |
| netlify/project | Free | Netlify がホストするサイト |
| railway/hosting | Free | GitHub リポジトリから Railway にデプロイ |
| render/static-site | Free | 静的サイト向け無料ホスティング |
| render/web-service | Free & Paid | マネージド Web サービス |
| vercel/project | Free & Paid | Vercel にデプロイしたアプリケーション |

#### DATABASE

| サービス | プラン | 概要 |
| --- | --- | --- |
| cloudflare/d1 | Free | サーバーレス SQL（D1） |
| cloudflare/hyperdrive | Free | Workers から既存 DB への接続 |
| flyio/mpg | Paid | Fly.io のマネージド PostgreSQL |
| neon/postgres | Free | 認証統合済みの Neon Postgres |
| planetscale/mysql | Paid | マネージド MySQL |
| planetscale/postgresql | Paid | マネージド PostgreSQL |
| railway/mongo | Free | Railway 上の MongoDB |
| railway/postgres | Free | Railway 上の PostgreSQL |
| railway/redis | Free | Railway 上の Redis（キャッシュ） |
| render/postgres | Free & Paid | Render のマネージド PostgreSQL |
| supabase/project | Free | DB・認証などを含む Supabase プロジェクト |
| turso/database | Free & Paid | SQLite ベースの Turso DB |
| upstash/redis | Free & Paid | serverless Redis |
| upstash/vector | Free & Paid | serverless Vector |

#### AI

| サービス | プラン | 概要 |
| --- | --- | --- |
| algolia/application | Free & Paid | Algolia の検索・ディスカバリー |
| amplitude/analytics | Free | Amplitude のプロダクトアナリティクス |
| cloudflare/workers-ai | Free | Workers 上で AI モデルを実行 |
| elevenlabs/tts | Free | ElevenLabs の Text-to-Speech API |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |
| huggingface/bucket | Free & Paid | Hugging Face のストレージバケット |
| huggingface/platform | Free & Paid | Hugging Face プラットフォームへのアクセス |
| openrouter/api | Free & Paid | OpenRouter でモデルを比較・利用 |
| posthog/analytics | Free | PostHog のプロダクトアナリティクス |
| upstash/vector | Free & Paid | serverless Vector |

#### ANALYTICS

| サービス | プラン | 概要 |
| --- | --- | --- |
| amplitude/analytics | Free | Amplitude のプロダクトアナリティクス |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |
| mixpanel/analytics | Free | Mixpanel のプロダクトアナリティクス |
| posthog/analytics | Free | PostHog のプロダクトアナリティクス |

#### COMMUNICATIONS

| サービス | プラン | 概要 |
| --- | --- | --- |
| agentmail/api | Free & Paid | AgentMail API（受信トレイの作成など） |
| upstash/qstash | Free & Paid | Upstash QStash（メッセージング） |

#### QUEUE

| サービス | プラン | 概要 |
| --- | --- | --- |
| cloudflare/queues | Free | メッセージの送受信 |
| inngest/app | Free & Paid | Inngest アプリのデプロイ |

#### BROWSER

| サービス | プラン | 概要 |
| --- | --- | --- |
| browserbase/project | Free & Paid | Browserbase プロジェクト（ブラウザ自動化） |
| cloudflare/browser-run | Free | Cloudflare 上のヘッドレスブラウザ |

#### AUTH

| サービス | プラン | 概要 |
| --- | --- | --- |
| auth0/client | Free & Paid | Auth0（モバイル・Web・IoT クライアント） |
| clerk/auth | Free & Paid | Clerk 認証 |
| neon/postgres | Free | 認証統合済み Neon Postgres |
| privy/app | Free & Paid | Privy（認証・ウォレット） |
| supabase/project | Free | DB・認証などを含む Supabase プロジェクト |
| workos/auth | Free & Paid | WorkOS AuthKit |

#### CACHE

| サービス | プラン | 概要 |
| --- | --- | --- |
| cloudflare/kv | Free | Cloudflare KV |
| railway/redis | Free | Railway 上の Redis |
| upstash/redis | Free & Paid | serverless Redis |

#### CDN

| サービス | プラン | 概要 |
| --- | --- | --- |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |

#### CI

| サービス | プラン | 概要 |
| --- | --- | --- |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |

#### FEATURE_FLAGS

| サービス | プラン | 概要 |
| --- | --- | --- |
| amplitude/analytics | Free | Amplitude（フィーチャーフラグ用途も） |
| posthog/analytics | Free | PostHog（フィーチャーフラグ用途も） |

#### OBSERVABILITY

| サービス | プラン | 概要 |
| --- | --- | --- |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |
| sentry/project | Free | Sentry（エラー追跡など） |

#### PAYMENTS

| サービス | プラン | 概要 |
| --- | --- | --- |
| privy/app | Free & Paid | Privy（認証・ウォレット） |

#### SEARCH

| サービス | プラン | 概要 |
| --- | --- | --- |
| algolia/application | Free & Paid | Algolia の検索・ディスカバリー |
| firecrawl/api | Free | Firecrawl API（検索・スクレイピング） |
| upstash/search | Free & Paid | Upstash の全文検索 |

#### STORAGE

| サービス | プラン | 概要 |
| --- | --- | --- |
| gitlab/project | Free | CI/CD 付き GitLab プロジェクト |
| huggingface/bucket | Free & Paid | Hugging Face のストレージバケット |
| railway/bucket | Free | Railway の S3 互換オブジェクトストレージ |
| supabase/project | Free | DB・ストレージ等を含む Supabase プロジェクト |

おい…
めっちゃあるやん！流石Stripeだな…
今回私が使用したいvercelとsupabseがあるので、早速追加して行きましょう。
```
stripe projects add vercel/project
stripe projects add supabase/project
```
`stripe projects add vercel/project`を実行後こんな感じになります。
![](/images/stripe-project/7.png)
![](/images/stripe-project/8.png)
```
stripe projects status
```
で実際に接続できているか確認できます。
![](/images/stripe-project/10.png)
```
stripe projects env
```
では「Stripe Projects が把握している環境変数（認証情報の束）の一覧」が CLI で表示されている状態です。
![](/images/stripe-project/11.png)

`stripe projects env --pull` について
Next steps にある env --pull は、Vault とローカルの .env などを Projects の正として同期・取得するためのコマンドです。別マシンにした／.env を消した／最新に再取得したいときなどに使うイメージです。すでに add のときに .env に注入済みなら、必須ではないことも多いですが、「手元のファイルを確実に揃えたい」ときに実行します。

:::message
`stripe projects add vercel/project`を実行すると、ルートの.envにシークレットだった値が記載されています。
:::
ちょっとこのままsupabseの設定もして行きます。ほぼ同じ流れになったので、割愛します。
もろもろ終了するとこんな感じにファイルも作成されています。
![](/images/stripe-project/12.png)

あとは実際にweb上で確認してみましょう。
![](/images/stripe-project/13.png)
![](/images/stripe-project/14.png)
実際にPJは作成されていますが、どちらの環境変数にも.envに記載されている値は記載されていません。
実際に公式ドキュメントにも「stripe projects env --pull は、ローカル開発用に認証情報をローカルの .env ファイルに書き込みます。本番環境のホストには環境変数を書き込みません。
本番環境で同じ認証情報を使用するには、それらをホストの環境変数設定に追加してください。Stripe Projects ではこの手順は自動化されません。」
と記載があります。

`stripe projects env --pull`で取得できる値は`stripe projects add vercel/project`で作成した値だけなので、リモートの環境変数で追加したものは取得できないようです。
つまり`stripe projects env --pull`を実行しても下記の環境変数しか取得できません。
```
supabase-plan
    SUPABASE_ORG_SLUG=hav••••••••
    SUPABASE_PLAN=••••••••

supabase-project
    SUPABASE_DB_PASS=Xg••••••••
    SUPABASE_DB_URL=post••••••••
    SUPABASE_POOLER_URL=post••••••••
    SUPABASE_PROJECT_REF=yqz••••••••
    SUPABASE_PROJECT_URL=http••••••••

vercel-plan
    VERCEL_PLAN=••••••••
    VERCEL_TEAM_ID=team••••••••
    VERCEL_TEAM_URL=http••••••••

vercel-project
    VERCEL_ORG_ID=team••••••••
    VERCEL_PROJECT_ID=prj_••••••••
    VERCEL_PROJECT_LINK={"or••••••••
    VERCEL_PROJECT_URL=http••••••••
    VERCEL_TOKEN=vca_••••••••
```

# 最後に
気になった方はぜひ使用してみてください。
正直、リモートの環境変数を自動で反映してくれるとよかったのですが、環境変数の共通化だけが目的なら使わなくても良さそうですね…

とはいえ、**次のようなときは Stripe Projects を検討する価値がある**と感じました。
- **連携先の料金や請求の入口を Stripe 側に寄せて把握したい**とき。各プロバイダの請求画面を個別に追いかける運用と比べ、誰がどの環境でどのプランに乗っているかを整理しやすい（`stripe projects billing add` など、請求まわりは公式の案内に従う前提）。
- **dev / staging / production を別プロジェクトに分け、環境と課金の境界も切りたい**とき。公式でも環境ごとの分離が推奨されています。
一方で、**ホスティング先の環境変数を Projects が自動で書き換えてほしい**、あるいは **既存のホスト側のシークレットストアだけを真実の源泉にしたい**だけなら、別の仕組みの方が主役になりやすいです。

公式はこちらに貼っておきます。
https://docs.stripe.com/projects