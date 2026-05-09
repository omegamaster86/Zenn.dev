---
title: "Stripe Projectを試してみたんじゃ"
emoji: "🌕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Stripe Project","AI","AI駆動開発"]
published: false
# published_at: 2026-05-08 09:30
publication_name: "genai"
---
# 初めに

### まず何がどうなる？

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


# 使用してみよう
### インストール
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