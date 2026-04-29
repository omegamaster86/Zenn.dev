---
title: "Edge Functionのエラーを@cursorでお片付けするんじゃ"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["supabase","cursor","slack","障害対応"]
published: false
# published_at: 2026-04-01 11:30
publication_name: "genai"
---
# 初めに
Supabase Edge Functionのエラーを検知したいな〜、そのまま対応したいな〜と思いやってみました！
早速内容を見てみましょう〜

# Edge Function -> Slack通知 -> Cursor調査 Runbook

このドキュメントは、Supabase Edge Function の異常（非2xx）を Slack に通知し、通知を起点に Cursor で原因調査までつなぐための社内運用手順書です。

## 1. 目的と適用範囲

### 1-1. 目的

- Edge Function 障害の検知を早める
- Slack から該当関数へ即時に遷移できる状態を作る
- 通知メッセージから Cursor に調査を依頼し、初動を標準化する

### 1-2. 適用範囲

- Supabase Edge Functions（本リポジトリ）
- 通知先 Slack チャンネル（例: `#alert`）
- Cursor Slack アプリ利用時の調査フロー

## 2. 現在の実装概要

### 2-1. 主要ファイル

- 共通通知ヘルパー: `supabase/functions/_shared/slack-alert.ts`
- 適用中関数: `supabase/functions/get-surveys-results-for-admin/index.ts`

### 2-2. 通知対象（2026-04時点）

`get-surveys-results-for-admin` において、次のステータスで通知されます。

- 405: HTTPメソッド不正
- 400: `targetNow` 未指定 / `page` 不正
- 401 / 403: 管理者認証失敗
- 500: 予期しない例外

### 2-3. 通知送信方式の優先順位

`notifySlackOnNon2xx(...)` は次の順で送信方式を選択します。

1. `SLACK_BOT_TOKEN` + `SLACK_ALERT_CHANNEL_ID` がある場合  
   -> Slack Web API (`chat.postMessage`)
2. 上記がない場合  
   -> Incoming Webhook (`SLACK_WEBHOOK_URL`)
3. どちらも無い場合  
   -> 送信スキップ（`skipped`）

## 3. 事前準備（Secrets / Slack設定）

## 3-1. 必須 Secrets

最小構成:

- `SLACK_WEBHOOK_URL`

推奨構成（運用しやすい）:

- `SLACK_BOT_TOKEN`
- `SLACK_ALERT_CHANNEL_ID`
- `CURSOR_SLACK_MEMBER_ID`（例: `U0123456789`）

任意:

- `CURSOR_SLACK_MENTION`

## 3-2. Slack側の前提条件

- 通知先チャンネルに対象アプリ（Webhook元アプリ / Bot / Cursorアプリ）が参加している
- private channel の場合は `/invite @AppName` が必要
- Cursorアプリをメンション起動したい場合、Cursorアプリが同じチャンネルに参加済みであること

## 3-3. Secret設定コマンド例

```bash
# Webhook方式
supabase secrets set SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."

# Bot方式
supabase secrets set SLACK_BOT_TOKEN="xoxb-..."
supabase secrets set SLACK_ALERT_CHANNEL_ID="C0123456789"

# Cursorメンション
supabase secrets set CURSOR_SLACK_MEMBER_ID="U0123456789"
```

## 4. 実装標準（Edge Function側）

## 4-1. 組み込み位置

以下の方針で統一します。

- `logger.LoggingEnd(...)` の直後
- 実際に非2xxレスポンスを返す直前
- 例外処理（`catch`）では 500 通知を送る

## 4-2. 実装テンプレート

```ts
import { notifySlackOnNon2xx } from "../_shared/slack-alert.ts";

const functionName = "your-edge-function";

const notifyIfNon2xx = async (
  status: number,
  errorMessage: string,
  details?: Record<string, unknown>,
) => {
  const notifyResult = await notifySlackOnNon2xx({
    functionName,
    request: req,
    status,
    errorMessage,
    details,
  });
  if (notifyResult === "failed") {
    logger.LoggingWarn("slack_notify_failed", { status, errorMessage });
  }
};
```

## 4-3. 実装時の注意

- 通知失敗で本処理を失敗させない（監視は補助、業務処理が主）
- `details` は必要最小限（PIIや秘匿値を入れない）
- 既存ログ出力構造（`LoggingStart/End`）を崩さない

## 5. Slack通知フォーマット仕様

## 5-1. 標準出力項目

- `function`
- `status`
- `method`
- `path`
- `error`
- `event_message`（JSON文字列）
- `dashboard`（推奨。関数画面リンク）
- `details`（重複キー除去後に残る情報のみ）

## 5-2. 重複除外ルール

`details` から次のキーは自動除外されます。

- `status`
- `method`
- `path`
- `error`
- `errorMessage`

## 5-3. 生成される Dashboard URL

`request.url` のホストが `<project-ref>.supabase.co` の場合、次を生成します。

`https://supabase.com/dashboard/project/<project-ref>/functions/<function-name>`

## 6. 障害発生時の運用フロー（実運用）

1. Slack `#alert` 通知を確認
2. `function` / `status` / `error` / `event_message` を読む
3. `dashboard` リンクから Supabase Function 画面を開く
4. Function Logs で同時刻の `requestId` / error を確認
5. 必要に応じて通知スレッドで Cursor に調査依頼
6. 暫定対応（再実行 / ロールバック / feature flag）を判断
7. 恒久対応のIssue化と再発防止策を記録

## 7. Cursor調査依頼テンプレート（Slack）

## 7-1. 手動依頼テンプレート（スレッド推奨）

```text
@Cursor
Supabase Edge Functionの異常を調査してください。
function: get-surveys-results-for-admin
status: 500
path: /get-surveys-results-for-admin
dashboard: https://supabase.com/dashboard/project/<project-ref>/functions/get-surveys-results-for-admin
依頼内容: 再現条件、根本原因、影響範囲、修正案、暫定回避策を提示してください。
```

## 7-2. 自動メンション条件

- `CURSOR_SLACK_MEMBER_ID` が有効な場合は `<@...>` を自動付与
- 無い場合は `CURSOR_SLACK_MENTION`、さらに無い場合は `@Cursor` を使用

## 8. 動作確認手順（検証用）

## 8-1. デプロイ

```bash
supabase functions deploy get-surveys-results-for-admin --project-ref <project-ref>
```

## 8-2. 405を意図的に発生

```bash
curl -i -X POST "https://<project-ref>.supabase.co/functions/v1/get-surveys-results-for-admin"
```

期待値:

- HTTP 405
- Slack通知が1件到達
- 通知本文に `function`, `status`, `dashboard` が含まれる

## 8-3. 400を意図的に発生（`targetNow` 欠落）

```bash
curl -i -H "Authorization: Bearer <token>" -H "apikey: <token>" \
  "https://<project-ref>.supabase.co/functions/v1/get-surveys-results-for-admin"
```

期待値:

- HTTP 400（`targetNow is required`）
- Slack通知が到達

## 9. トラブルシュート

## 9-1. 通知が来ない

確認順:

1. 関数が最新コードで再デプロイ済みか
2. Secrets がデプロイ先プロジェクトに設定済みか
3. チャンネルID / Webhook URL が正しいか
4. private channel にアプリが参加しているか
5. Function Logs に `slack_notify_failed` が出ていないか

## 9-2. 通知内容が古い

- 旧バージョン関数が稼働している可能性が高い
- 再デプロイ後に 405 を1回送ってフォーマットを確認

## 9-3. `@Cursor` が反応しない

- `CURSOR_SLACK_MEMBER_ID` が誤っている
- Cursorアプリがチャンネル未参加
- plain text で `@cursor` と書いており実メンション扱いになっていない

## 10. セキュリティ運用

- `SLACK_WEBHOOK_URL` / `SLACK_BOT_TOKEN` は必ず Secrets 管理
- チャット・スクリーンショットでトークン露出時は即ローテーション
- `details` に個人情報・機密値を入れない
- 必要に応じて通知チャンネルを環境別に分離（dev/stg/prod）

## 11. 変更履歴

- 2026-04-29: 初版作成（Edge Function -> Slack -> Cursor 運用手順を独立ドキュメント化）
