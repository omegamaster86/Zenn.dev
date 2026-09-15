---
title: "複数検証環境へのデプロイ運用を GitHub Actions で整理したんじゃ"
emoji: "🚀"
type: "tech"
topics: ["GitHubActions", "GCP", "CloudRun", "CI/CD", "Next.js"]
published: false
# published_at: 2026-07-26 12:30
publication_name: "genai"
---

# 初めに

いや〜もう9月も半ばですね〜
今年の夏は去年ほど暑くなくてよかったなと思っているオメガマスターです〜

今回担当しているプロジェクトでは、検証環境が **develop1〜６ / staging1〜６ / production** と複数並立しています。そんなにいる？って意見はまともです実際は月日が経つにつれて現在はcdevelop5、６ / staging5、６ / production**の5つのみ動いております。
今までやったことある環境分けはVercelの環境で、**develop,uat,production**の３つの環境分けだったので、かなり新鮮でした。
今回は実際に運用している**手動トリガー + GitHub Environments** という方式で、任意のブランチを任意の環境に載せる設計について解説していきたいと思います。

:::message
対象は ntv-contech/frontend です。backend も同じパターン（`workflow_dispatch` + Environments）で動いています。
:::

# やりたいこと

やりたいこと自体はシンプルです。

1. 複数の検証環境（dev5 / dev6 / staging6 など）を並行運用する
2. feature ブランチ・統合ブランチ・release ブランチを、目的に応じて好きな環境にデプロイする
3. 環境ごとに API URL や Cloud Run サービス名を切り替える
4. 間違ったブランチが本番に行く事故を防ぐ

# そもそも push では自動デプロイしないんじゃ

ここを理解しないと、あと全部ズレます。私も最初ここで1回くらい迷子になりました。

## 全体の流れ

```mermaid
flowchart LR
  A[ブランチを選ぶ] --> B[Actions で CD を手動実行]
  B --> C[デプロイ先環境を選択]
  C --> D[Docker ビルド]
  D --> E[Artifact Registry に push]
  E --> F[Cloud Run にデプロイ]
```

ポイントは次の2つです。

1. **push では自動デプロイしない** — `workflow_dispatch` で手動実行
2. **環境ごとの設定は GitHub Environments に集約** — Variables / Secrets で切り替え

「ブランチ名 = 環境」という自動マッピングは **ありません**。
任意のブランチを、手動で選んだ環境にデプロイする設計です。

:::message
Vercel や Netlify の「main に merge したら自動デプロイ」とは違う方式です。複数検証環境 + 統合ブランチ運用では、この手動方式はかなり一般的です。
:::

# ワークフローの定義

`.github/workflows/deploy.yml` のトリガー部分はこうなっています。

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment for deployment'
        required: true
        type: environment
```

`type: environment` を指定すると、GitHub Actions の実行画面で **Environment のドロップダウン** が表示されます。

登録されている環境の例:

| GitHub Environment | 用途（概略） |
| --- | --- |
| `develop5` | dev5 検証環境 |
| `develop6` | dev6 検証環境 |
| `staging6` | stg6 統合検証環境 |
| `production` | 本番環境 |

# ジョブ構成（2段階）

デプロイは2段階のジョブで実行されます。

## Job 1: Build and Push

```yaml
build-and-push:
  runs-on: ubuntu-latest
  environment: ${{ inputs.environment }}  # ← 環境ごとの vars/secrets を参照
```

処理の流れ:

1. 選択されたブランチを checkout
2. GCP 認証（`GCP_SERVICE_ACCOUNT_KEY`）
3. 環境ごとの build-arg で Docker イメージをビルド
4. Artifact Registry に push（タグは commit の short SHA）

Next.js の `NEXT_PUBLIC_*` など、ビルド時に埋め込む値は **環境ごとの Variables** から渡します。

```yaml
docker build \
  --build-arg NEXT_PUBLIC_API_BASE_URL=${{ vars.NEXT_PUBLIC_API_BASE_URL }} \
  --build-arg NEXT_PUBLIC_APP_URL=${{ vars.NEXT_PUBLIC_APP_URL }} \
  # ... 他の build-arg
  -t "$IMAGE_TAG" .
```

## Job 2: Deploy to Cloud Run

```yaml
deploy:
  needs: build-and-push
  environment: ${{ inputs.environment }}
```

`gcloud run deploy` で、環境ごとに定義された Cloud Run サービスへ反映します。

```yaml
gcloud run deploy ${{ vars.CLOUD_RUN_SERVICE_NAME }} \
  --image=.../frontend:${{ needs.build-and-push.outputs.sha_short }} \
  --region=${{ vars.GCP_REGION }} \
  --memory=${{ vars.MEMORY }} \
  --cpu=${{ vars.CPU }} \
  # ...
```

環境ごとに **別の Cloud Run サービス**（例: `contech-dev5-run-frontend` / `contech-dev6-run-frontend`）にデプロイされるイメージです。

# GitHub Environments の役割

各 Environment には **Variables** と **Secrets** が紐づいています。

| 種類 | 例 |
| --- | --- |
| Variables | `CLOUD_RUN_SERVICE_NAME`, `NEXT_PUBLIC_API_BASE_URL`, `GCP_PROJECT_ID`, `GCP_REGION` |
| Secrets | `GCP_SERVICE_ACCOUNT_KEY`, `KEYCLOAK_CLIENT_SECRET`, `INTERNAL_TOKEN` |

ワークフロー内では `vars.*` / `secrets.*` で参照します。
ジョブに `environment: ${{ inputs.environment }}` を付けることで、選択した環境の値が自動的に解決されます。

## 設定場所

GitHub リポジトリ → **Settings** → **Environments** → 対象環境 → **Environment variables** / **Environment secrets**

ここに環境差分を全部寄せるのがポイントです。ワークフロー YAML に dev5 用・dev6 用の値をベタ書きしない。

# 実際のデプロイ手順

1. GitHub → **Actions** → **CD - Frontend Deployment**
2. **Run workflow** をクリック
3. **Branch**: デプロイしたいブランチを選択（例: `stg6/integration-202609-v1`）
4. **Environment**: デプロイ先を選択（例: `staging6`）
5. **Run workflow** で実行

実行時に選ぶのはこの2つです。

| 選ぶもの | 意味 |
| --- | --- |
| Branch | どのブランチのコードをデプロイするか |
| Environment | どの Cloud Run 環境に載せるか |

:::message
PR をマージしただけではデプロイされません。「今 dev6 に何が載っているか」は Actions の実行履歴を見る必要があります。
:::

# ブランチと環境の対応（運用ルール）

ブランチ名と環境の自動マッピングは **ありません**。チームの運用ルールで決めています。

実際のデプロイ例:

| ブランチ | デプロイ先 Environment |
| --- | --- |
| `stg6/integration-202609-v1` | `staging6` |
| `feature/streaming-progress-episode-count` | `develop6` |
| `release-202609-v3` | `develop5` または `production` |

統合ブランチ（`stg6/integration-*`）を staging6 に載せて動作確認し、feature ブランチは develop6 で個別検証する、といった運用が典型的です。

統合 PR には `do-not-merge` ラベルが付いていて、マージ禁止の使い捨てブランチとして動作確認に使われます。この運用と手動デプロイは相性がいいです。

# 他のデプロイ方式との比較

| 方式 | よくある場面 | 本プロジェクト |
| --- | --- | --- |
| push/merge で自動デプロイ | SaaS、小〜中規模、CI/CD が成熟したチーム | 使っていない |
| 手動 + 環境選択 | 複数検証環境、統合ブランチ運用 | **採用** |
| GitOps (Argo CD 等) | K8s、大規模インフラ | 使っていない |
| タグ/リリースで本番のみ自動 | 本番は自動、検証は手動 | 一部あり |

## メリット

- **任意ブランチを任意環境に載せられる** — 統合ブランチや feature ブランチを柔軟に検証できる
- **事故が少ない** — push 連動だと「間違ったブランチが本番に行く」リスクがある
- **複数 feature の統合テスト向き** — `stg6/integration-*` のような統合ブランチ運用と相性が良い
- **環境ごとの設定が GitHub に集約** — コードとインフラ設定の対応が明確

## デメリット

- 毎回人手が必要（「デプロイして」がボトルネックになりやすい）
- ブランチと環境の対応は運用ルール依存（自動化されていない）
- PR マージ ≠ デプロイなので、「今 dev6 に何が載っているか」は Actions の履歴を確認する必要がある

:::message alert
「古い」方式というより **制御重視** の設計です。スタートアップの自動デプロイ一択とは違いますが、複数検証環境 + 統合ブランチ運用では十分一般的です。
:::

# よくある進化パターン

運用が成熟するにつれ、次のような改善がよく行われます。うちも検討中です。

1. **develop6 への push は自動**、staging / production は手動 or 承認
2. **デプロイ状況の可視化** — README や Slack に「今 dev6 はこのコミット」を通知
3. **ブランチ名で環境を推論**（例: `stg6/*` → staging6）— ただし運用が複雑になるので慎重に

# まとめ

- 本プロジェクトのデプロイは **手動 `workflow_dispatch` + GitHub Environments** で実現している
- ブランチと環境の対応は **運用ルール** で管理（自動マッピングなし）
- 複数検証環境 + 統合ブランチ運用では、この方式は一般的かつ実用的
- 環境差分は GitHub Environments の Variables / Secrets で切り替える

「push したら自動で行くんでしょ？」と思っていた頃の自分に教えてあげたい内容でした。
同じように複数検証環境を運用している方の参考になれば嬉しいです。

# 参考

- [GitHub Environments ドキュメント](https://docs.github.com/ja/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [workflow_dispatch イベント](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch)
- [Google Cloud Run デプロイ](https://cloud.google.com/run/docs/deploying)
