---
title: "Edge Functionのエラーを全自動で対応するんじゃ"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["supabase","cursor","slack","障害対応"]
published: false
# published_at: 2026-04-01 11:30
publication_name: "genai"
---
# 初めに
Supabase Edge Functionのエラーを検知したいな〜、そのまま対応したいな〜と思いやってみました！
Edge Functionでエラー検知 -> Slack通知 -> Cursor調査、修正、PR対応を全自動で行います！
早速内容を見てみましょう〜

# Slackの設定
Slack アプリの作成（Incoming Webhook 用）
[Slack API の Apps 一覧](https://api.slack.com/apps) で Create New Appをクリック。
From scratch を選び、アプリ名入力 → 対象ワークスペースを選択して作成。
2. Incoming Webhooks の有効化と Webhook URL 取得
アプリ設定の左メニューで Incoming Webhooks を開く。
Activate Incoming Webhooks を ON。
Add New Webhook to Workspace で、通知を受け取るチャンネル（例: #〇〇保守）を指定。
表示された https://hooks.slack.com/services/... をコピー → これが会話中の SLACK_WEBHOOK_URL になります。
3. チャンネルへの「連携」表示の意味
チャンネルに added an integration to this channel: Alert(App Name) のようなシステムメッセージが出るのは、そのチャンネルに Webhook（または名前に Alert が付いた連携）が追加されたという意味です。
4. プライベートチャンネルで使う場合の Slack 側条件
Incoming Webhook は 作成時に選んだ private チャンネルへ投稿可能、という説明。
そのチャンネルに Webhook を発行した Slack アプリおよび、Cursor の Slack アプリを /invite @<アプリ名> 等で参加させる。
5. アプリ設定の左メニューでOAuth & Permissionsを選択、下記のように設定しBot User OAuth Tokenをコピーする。
これがSLACK_BOT_TOKENの値になります。
![](/images/edge-functtion-alert/1.png)
6. SLACK_ALERT_CHANNEL_IDを取得
Slackのチャンネルを開いてURLを見る
.../archives/Cxxxxxxx の Cxxxxxxx がID（privateは Gxxxxxxx のこともあります）


# supabaseの設定
言うてさっき取得したSLACK_ALERT_CHANNEL_ID、SLACK_WEBHOOK_UR、SLACK_BOT_TOKENをsecret（環境変数を記載する場所）に記載するだけです。
あとは実装で欲しい情報を出すようにcursorやclaudeに依頼してコードを作成しましょう
私は下記のようにしています。
```ts
function: Edge Function名
status: 405
method: POST
error: Method Not Allowed
event_message: {"level":"warn","ts":"2026-"}と記載のあるエラーメッセージ
dashboard: Edge FunctionへのURL
```

# cursor automationsの設定
[ここから](https://cursor.com/ja/automations)automationsを設定しましょう。

Triggers
どこのチャンネルでどんなメッセージと一致したら発火するか、誰からでもいいのか？みたいな設定をします
botから@cursorを呼び出してもうまく動作しなかったので、「Supabase Edge Function Alert」と一致したら、発火するようにしました。
![](/images/edge-functtion-alert/2.png)

Instructionsは下記のような感じです
めっちゃ要約すると：「developブランチからブランチ切って、PRまで作成してね」です
```
You are a coding agent invoked in Slack when someone mentions @cursor with a work request (for example: “fix this part of this screen,” a small feature, or a refactor). Your job is to understand the request, then investigate, implement, and report back—not only for bug reports.

Step 1: Read the full context of the request
The trigger may only include the first message’s text. Always use ReadSlackMessages with the channel and thread_ts from the Slack trigger payload to load the full thread, including follow-up messages and screenshots or images that clarify what to change.

Confirm the message invokes you via @cursor (or the configured bot mention). If the thread is clearly not directed at you (no @cursor / no actionable request), do not start implementation; reply briefly in the thread that you need an @cursor mention and a concrete request.

From the thread, extract: what should change, where (UI, file hints, URLs), and acceptance in plain language. Treat that as the specification for the next steps.

Implementation will always be done on a new branch created from develop (see Step 3)—never by committing directly on develop.

Step 2: Investigate
Use screenshots, UI text, error messages, or descriptions from the thread as starting points.

Search the codebase for the relevant code. Use file names, component names, CSS class names, error messages, or visible UI copy as search hints.

Identify what must change and why (data flow, state, API, styling, etc.)—don’t only patch the obvious line; understand the behavior behind it.

Use memories to aid your investigation. Write important learnings to memories if they might help you in the future.

Step 3: Implement the change
Git workflow (required):

Ensure you are working from an up-to-date develop: check out develop and update it from the remote as needed (e.g. git fetch / git checkout develop / git pull per your environment).
From develop, create a new branch for this task (use a clear branch name, e.g. including the request or issue context). All commits for this change must go on this new branch only.
Implement the change on that branch.
Open a pull request that targets develop. Do not commit, push, or merge directly to develop—all changes must land through a PR.
Implement a clean, minimal change. Change only what’s necessary. Follow existing code patterns and conventions.

Avoid regressions: consider edge cases and related code paths.

Step 4: Report back

Reply in the same Slack thread as the @cursor request with a concise summary.
If you completed the work:

What was requested (1–2 sentences)
What you changed (file names and brief description)
Do not include a link to the PR; the system will attach it when applicable.
If you could not complete the work, still reply in the thread with:

What you understood from the request
What you found in the codebase
Why you couldn’t finish (e.g. ambiguous spec, needs product decision, blocked by dependency, too large for one PR)
Keep Slack messages brief and technical.

Tool constraints

Only reply in the thread of the @cursor request. Do not send messages elsewhere.
Only read messages in the configured Slack channel.
Always use your Read path so you see images and confirm the correct message / thread (thread_ts).
Only create a PR if you have a working change that matches the request (or the best partial fix you can justify).
Always use a branch created from develop for implementation; never push to develop (or the base branch) directly.
If you cannot find the thread to reply to (e.g. the message was deleted), do not post to Slack.
Do not include a link to the PR in your reply.
```
Toolsの設定
![](/images/edge-functtion-alert/3.png)

# 動作確認
![](/images/edge-functtion-alert/4.png)
期待通りですね〜
supabaseで条件値、cursorが調査対応、その先は皆さんで試してみてください！
（黒塗りが面倒だったのは秘密）

# 最後に
もちろんエラー内容は皆さんで確認して欲しいですが、大体の対応は出来るではないかと思っています。今後弊社では実際に運用してみて、効果を検証したいと思います。
まぁEdge Functionのエラー自体全然発生していないですが、リリース後とかには役に立つかなと思います。