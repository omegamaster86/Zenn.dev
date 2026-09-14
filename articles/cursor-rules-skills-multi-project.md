---
title: "Cursorのrules・skillsを複数PJで使い回す方法を整理したんじゃ【Local / Cloud 対応】"
emoji: "🌕"
type: "tech"
topics: ["cursor", "AI", "AI駆動開発", "Cursor Cloud", "Cursor Skills"]
published: false
# published_at: 2026-09-14 23:00
publication_name: "genai"
---

# 初めに

絵文字で毎回何を使うか迷うオメガマスターです。
ランダム絵文字みたいな機能つかないかな〜と密かに願っています。
Cursor で rules・skills・commands を使い始めた頃は「これ便利！」と思っていたのですが、PJ が増えると一気に地獄が始まりました。ローカルでは `/forge-mode（skillsやcommand）` がサクッと出るのに、Cloud Agent だと存在しない。各 PJ にコピーしたら更新が追いつかない。symlink で繋いだらローカルは天国、Cloud は無の境地…

正直、最初は「Cloud でも同じ `.cursor/` 読めるでしょ？」と甘く見ていました。甘かったです。原宿のチョコバナナより甘かったです…

今回は、**複数 PJ で rules / skills / commands を共有し、ローカルと Cloud の両方で使う**ための管理方法を、私が実際に試行錯誤した内容をベースに整理します。

実装例として私が使っている **omega**（`cursor-rules/omega`）を紹介しますが、omega じゃなくても同じ考え方で運用できます。omega はあくまで「うちの家のルール置き場」です。


:::message
この記事は 2026/09/14 時点の Cursor の挙動を前提に書いています。Cloud Agent 周りはアップデートで変わる可能性があるので、気になったら公式ドキュメントも確認してください。
あとomegaはオメガマスターのomegaです。決してFFのomegaでも時計のΩでもありません。
:::

# そもそもローカルと Cloud で読む場所が違うんじゃ

ここを理解しないと、あと全部ズレます。私もここで1週間くらい迷子になりました。

## ローカル Desktop が読むもの

| 種類 | 主な配置場所 | 備考 |
|------|-------------|------|
| rules | `プロジェクト/.cursor/rules/` | `global.mdc` など |
| skills | `プロジェクト/.cursor/skills/` | Agent Skills |
| commands | `プロジェクト/.cursor/commands/` | `/forge-mode` などの slash 入口 |
| agents | `プロジェクト/.cursor/agents/` | サブエージェント定義 |
| ユーザー共通 skills | `~/.cursor/skills/` | 全 PJ で使える |
| ローカル Plugin | `~/.cursor/plugins/local/` | 開発・個人利用向け |

ローカル Desktop は、**Mac の home ディレクトリ**と **開いている PJ の `.cursor/`** の両方を見られます。

## Cloud Agent が読むもの

Cloud Agent は **別 VM** で動きます。Mac の home は clone されません。

| 種類 | Cloud で読める？ | 条件 |
|------|-----------------|------|
| `プロジェクト/.cursor/`（commit 済み） | **読める** | GitHub から checkout された repo 内 |
| repo 外への絶対 symlink | **読めない** | `/Users/.../cursor-rules` などは壊れる |
| `~/.cursor/plugins/local/` | **届かない** | ローカル開発用。VM に sync されない |
| Customize から Install した Plugin | **読める可能性あり** | Marketplace / Team 経由の installed plugin |
| `~/.cursor/skills/` + Sync Skills | **skills だけ** | Settings → Agents で ON が必要 |
| `~/.cursor/rules/` | **自動適用されにくい** | workspace 側の rules を使うのが安全 |
| `.cursor/environment.json` の install | **読める** | Build 時に VM へ展開できる |

ここが本質です。覚えておくのはこれだけで OK です。

> **Cloud Agent が確実に読めるのは「GitHub 上の repo に含まれるもの」か「VM 起動時に install されるもの」だけ。**

ローカルで symlink 一本で繋いでいるだけでは、Cloud では動きません。…言いましたよね、甘かったって。

# 管理方式の比較（5パターン試してきた）

複数 PJ で共有する方法は、だいたい次の 5 パターンに分かれます。全部やりました。時間はかかりました。

## 1. 各 PJ にコピー（絶対止めよう、やっても２つまで）

```
your-app/
└── .cursor/
    ├── rules/
    ├── skills/
    └── commands/
```

| | ローカル | Cloud | commit 量 | 更新 |
|--|---------|-------|----------|------|
| 評価 | ○ | ○ | **多い** | 各 PJ で sync + commit |

確実ですが、PJ が増えるほど更新コストが爆発します。**「共有したい」なら最終手段**。3 PJ 目で諦めました。
あとコミットされるので、プライベートリポジトリ出ないと流出します。困ることはあまりないとは思いますが、他の人に見られるのが嫌な人はやめておきましょう。

## 2. repo 外への symlink（ローカル専用）

正本を `cursor-rules/omega` のような **別リポジトリ** に置き、各 PJ の `.cursor/` から **絶対パス symlink** で参照する方式です。`commands` だけじゃなく、agents / rules / skills もまとめてリンクします。

```text
cursor-rules/omega/          # 正本（1か所）
├── commands/
├── skills/
├── rules/
└── agents/

your-app/
└── .cursor/
    ├── commands/          -> /Users/you/cursor-rules/omega/commands
    ├── agents/            -> /Users/you/cursor-rules/omega/agents
    ├── rules/
    │   ├── global.mdc                    -> .../omega/rules/global.mdc
    │   ├── multi-agent-task-enforcement.mdc -> .../omega/rules/...
    │   └── forge-models.mdc              # 初回だけコピー（PJ 固有）
    └── skills/
        ├── forge-mode/    -> .../omega/skills/forge-mode
        └── verify-myapp/  # PJ 固有（リンクしない）
```

セットアップ例（omega の場合）:

```bash
/path/to/cursor-rules/omega/scripts/omega-link /path/to/your-app
```

手で `ln -s` するのではなく、スクリプトで一括リンクするのがポイントです。

| | ローカル | Cloud | commit 量 | 更新 |
|--|---------|-------|----------|------|
| 評価 | ○ | **×** | 不要 | 正本を直すだけ |

**ローカルで動く理由**: symlink 先の `/Users/you/cursor-rules/omega` が Mac 上に存在する。正本を直すだけで全 PJ に即反映されます。

**Cloud で壊れる理由**: Cloud Agent は GitHub の repo だけを VM に checkout する。Mac の home や repo 外のパスは存在しないため、symlink が宙に浮きます。

`.cursor/` を `.gitignore` していてもローカルでは問題ない一方、Cloud には `.cursor/` 自体が届かないので、この方式単体では Cloud 非対応です。絶対パス symlink を commit しても、他の人の Mac ではパスが違うので **チーム共有にも向きません**。

| 向いている | 向いていない |
|-----------|-------------|
| ローカル Desktop だけ使う | Cloud Agent も使う |
| 正本を1か所で更新したい | チームで同じパスを共有したい |
| commit を増やしたくない | Windows も混在（symlink 注意） |

ローカルだけなら最も楽。今もローカルはこれです。ただし Cloud では symlink 先が存在しないので **完全に非対応**。ここで初めて「あ、別マシンか」と気づきました。

## 3. repo 内 submodule + 相対 symlink

正本を **repo 内**（submodule）に置く方式です。方式2との違いは「symlink 先が repo の中にあるか外にあるか」だけですが、Cloud 対応の可否がここで分かれます。

```
your-app/
├── omega/              # git submodule
└── .cursor/
    └── commands -> ../omega/commands
```

| | ローカル | Cloud | commit 量 | 更新 |
|--|---------|-------|----------|------|
| 評価 | ○ | ○ | submodule bump が必要 | omega 更新のたび commit |

Cloud でも動きますが、**共有ルールを更新するたびに各 PJ で submodule を bump して commit** する必要があります。「commit を減らしたい」要件には向きません。

## 4. ローカル Plugin（`~/.cursor/plugins/local/`）

```
~/.cursor/plugins/local/my-rules -> /path/to/shared-rules
```

| | ローカル | Cloud | commit 量 | 更新 |
|--|---------|-------|----------|------|
| 評価 | ○ | **×** | 不要 | Plugin 側だけ |

[Plugins 公式ドキュメント](https://cursor.com/docs/plugins) でも、`~/.cursor/plugins/local` は **公開前のローカルテスト用** とされています。

Cloud Agent は別 VM で起動するため、**Mac の home ディレクトリは sync されません**（[Forum: Cloud Agents don't sync your laptop home directory](https://forum.cursor.com/t/cloud-agent-plugin-compatibility/167781)）。

> **個人ローカル Plugin は、ローカル Desktop 専用と考えるのが安全。**

Marketplace 公開は Cursor の審査が必要ですが、**ローカル Plugin なら許可は不要**です。自分用に置くだけで使えます。「Plugin 化したら Cursor に許可取らないと？」と一瞬ビビりましたが、公開しなければ大丈夫でした。

## 5. `environment.json` で Cloud Build 時に install（おすすめ）

```
your-app/
└── .cursor/
    ├── environment.json    # commit（1ファイル）
    └── install-omega.sh    # commit（1ファイル）
```

| | ローカル | Cloud | commit 量 | 更新 |
|--|---------|-------|----------|------|
| 評価 | △（別経路） | **◎** | **最小** | 正本更新は次回 Build で反映 |

Cloud VM 起動時に、install スクリプトで共有ルールを clone して配置します。Cursor スタッフも Forum でこの方式を推奨しています。

**ローカルは symlink、Cloud は environment.json** のハイブリッドが、現時点では一番バランスが良いです。いま私がこれで運用しています。

# 結論から言うと：ハイブリッド方式がいいんじゃ

長々と試した結果、私の要件はこうでした。

- 複数 PJ で同じ rules / skills / commands を使う
- ローカル Desktop でも Cloud Agent でも動かしたい
- 共有部分は **commit したくない**（または最小限）
- PJ ごとに少し設定を変えたい（モデル割当、動作確認レシピなど）

この場合の構成はこうです。

```
┌─────────────────────────────────────────┐
│  正本（1か所）                            │
│  cursor-rules/omega/                     │
│  ├── commands/                           │
│  ├── skills/                             │
│  ├── rules/                              │
│  └── agents/                             │
└─────────────────────────────────────────┘
         │                    │
         │ ローカル            │ Cloud
         ▼                    ▼
┌──────────────────┐  ┌──────────────────────────┐
│ omega-link       │  │ environment.json         │
│ （symlink）       │  │ + install-omega.sh       │
│ commit 不要       │  │ commit 2ファイルだけ       │
└──────────────────┘  └──────────────────────────┘
         │                    │
         ▼                    ▼
┌─────────────────────────────────────────┐
│  各 PJ リポジトリ                         │
│  .cursor/                                │
│  ├── commands      ← 共有（symlink/link） │
│  ├── skills/       ← 共有 + verify-* 固有  │
│  ├── rules/        ← 共有 + forge-models  │
│  ├── environment.json  ← Cloud 用のみ      │
│  └── install-omega.sh  ← Cloud 用のみ      │
└─────────────────────────────────────────┘
```

## 共有するもの vs PJ 固有のもの

| ファイル | 共有？ | 理由 |
|----------|--------|------|
| `commands/` | 共有 | slash コマンドの入口 |
| `skills/forge-mode/` など | 共有 | ワークフロー本体 |
| `rules/global.mdc` | 共有 | 常時適用ルール |
| `rules/forge-models.mdc` | **PJ 固有** | モデル割当は PJ ごとに変えたい |
| `skills/verify-<app>/` | **PJ 固有** | 動作確認レシピはアプリ依存 |
| `environment.json` | **PJ 固有** | Cloud Build フック |

**共有と固有を分ける**のが、複数 PJ 運用のコツです。全部を共有 repo に入れると、PJ 前提のコマンド（例: 特定の stg 環境への proxy 手順）が混ざって破綻します。`create-pr-video` で stg6 の proxy 前提が入ってた時、他 PJ で使おうとして詰みかけたのは内緒です。

# ローカル Desktop のセットアップ

omega の例です。正本は `cursor-rules/omega` にあります。ここはシンプルで気持ちいい。

## Step 1: symlink を張る

```bash
chmod +x /path/to/cursor-rules/omega/scripts/omega-link
/path/to/cursor-rules/omega/scripts/omega-link /path/to/your-project
```

`omega-link` が行うこと:

- `.cursor/commands` → `omega/commands` へ symlink
- `.cursor/agents` → `omega/agents` へ symlink
- `.cursor/rules/global.mdc` など → `omega/rules/` へ symlink
- `.cursor/skills/<共有>` → `omega/skills/` へ symlink
- `forge-models.mdc` は **初回のみコピー**（PJ ごとに編集可能）
- `verify-*` skill は **リンクしない**（PJ 固有）

## Step 2: 確認

Cursor Desktop を開き、slash コマンド（例: `/forge-mode`）が使えることを確認します。

## Step 3: 複数 PJ を一括リンク（任意）

`projects.txt` に PJ パスを書いて:

```bash
/path/to/cursor-rules/omega/scripts/omega-link-all
```

**commit は不要**です。`.cursor/` が gitignore されていても問題ありません。ここが symlink の良いところ。好き。

# Cloud Agent のセットアップ（ここが本番）

ローカルが楽な分、Cloud は一手間いります。でも commit は 2 ファイルだけなので、割に合います。

## Step 1: テンプレを PJ に置く

```bash
chmod +x /path/to/cursor-rules/omega/scripts/omega-cloud-init
/path/to/cursor-rules/omega/scripts/omega-cloud-init /path/to/your-project
```

生成されるファイル:

| ファイル | 内容 |
|----------|------|
| `.cursor/environment.json` | Cloud Build の install フック |
| `.cursor/install-omega.sh` | 正本 repo を clone して VM に展開 |

`environment.json` の例:

```json
{
  "user": "ubuntu",
  "install": "sh .cursor/install-omega.sh"
}
```

## Step 2: commit & push

```bash
git add .cursor/environment.json .cursor/install-omega.sh
git commit -m "chore: add Cloud Agent install hook for shared rules"
git push
```

**commit するのはこの 2 ファイルだけ**です。共有の skills / commands / rules 本体は commit しません。

## Step 3: Cloud Agent 環境を Rebuild

1. [Cloud Agents Dashboard](https://cursor.com/agents) または Desktop の Agents Window を開く
2. 対象 PJ の環境を選択
3. **Rebuild** を実行

Build 時に `install-omega.sh` が走り、VM 上で次が行われます。

- `cursor-rules` repo を clone
- `omega/scripts/install-omega-cloud.sh` を実行
- **commands / agents / rules** → workspace の `.cursor/` にリンク
- **skills** → workspace の `.cursor/skills/` と VM の `~/.cursor/skills/` の両方にリンク

skills を VM home にも置くのは、Cloud Agent が skills を home からも探索するためです。rules は workspace 側に置くのが安全です（`~/.cursor/rules` は Cloud で自動適用されにくい既知の挙動があります）。

## Step 4: 確認

Cloud Agent を起動し、`/forge-mode` などの slash コマンドが選べることを確認します。

## private repo の場合

正本 repo が private なら、Cloud Agent の **Secrets** に `GITHUB_TOKEN` を設定してください。`install-omega.sh` が clone 時に使います。

| 環境変数 | デフォルト | 用途 |
|----------|------------|------|
| `OMEGA_REPO` | `https://github.com/omegamaster86/cursor-rules.git` | 正本 repo URL |
| `OMEGA_REF` | `main` | ブランチ / tag |
| `GITHUB_TOKEN` | なし | private repo clone 用 |

# 更新時の運用

| 環境 | omega（共有ルール）を更新したとき |
|------|----------------------------------|
| ローカル Desktop | **何もしない**。symlink なので即反映 |
| Cloud Agent | **各 PJ の commit 不要**。次回 Rebuild で `OMEGA_REF` の最新を取得 |

skill を新規追加した場合は、ローカルでは `omega-link` を再実行してください（新しい skill ディレクトリの symlink を張るため）。

# やってはいけないこと（全部やった）

未来のオメガマスターのために、私が踏んだ地雷を書きます。

## 1. repo 外 symlink だけで Cloud も動くと思う

`/Users/you/cursor-rules/omega` への絶対 symlink は、ローカルでは動きますが Cloud では必ず壊れます。

## 2. `~/.cursor/plugins/local/` だけで Cloud もカバーする

ローカル Plugin は Desktop 専用です。Cloud には届きません。

Customize から user scope で Install した場合、installed plugin として Cloud に sync される可能性はありますが、**Marketplace 非公開の個人 plugin では保証が弱い**です。私も「Install したら Cloud でも読めるでしょ？」と思って検証しましたが、期待通りにはいきませんでした。確実性を取るなら `environment.json` 方式がよいです。

## 3. Sync Skills だけで全部カバーする

Settings → Agents → **Sync Skills for Cloud Agents** は、`~/.cursor/skills/` **だけ**を sync します。

- commands → 対象外
- rules → 対象外
- agents → 対象外

omega 全体を Cloud で使うなら、skills 同期だけでは足りません。

## 4. `.cursor/` を gitignore したまま Cloud を使う

Cloud は **commit されたファイルしか読めません**。Cloud 用の `environment.json` と `install-omega.sh` は必ず commit してください。

## 5. 共有 repo に PJ 固有の前提を入れる

`create-pr-video` のような「特定環境の proxy を立てる」コマンドは、共有 omega ではなく `verify-<app>/` skill に寄せるのがよいです。共有部分を汚さず、PJ 固有は repo に直接置きます。

# 方式の選び方（早見表）

| 要件 | おすすめ |
|------|----------|
| ローカルだけ、commit ゼロ | repo 外 symlink（`omega-link`） |
| ローカル + Cloud、commit 最小 | **symlink（ローカル）+ environment.json（Cloud）** |
| Cloud 完全確実、commit 多めでも OK | submodule + 相対 symlink |
| 組織全体で同じルールを配布 | Team Marketplace Plugin |
| 個人で commit ゼロ、Cloud 不要 | ローカル Plugin |

迷ったらこれでいいと思います。**「ローカルは symlink、Cloud は environment.json、PJ 固有は forge-models / verify-* だけ repo に置く」**。うちはこれです。

# セットアップ後の `.cursor/` の全体像

| パス | ローカル | Cloud | 種類 |
|------|----------|-------|------|
| `.cursor/commands/` | omega symlink | VM 上でリンク | 共有 |
| `.cursor/agents/` | 同上 | 同上 | 共有 |
| `.cursor/rules/global.mdc` | 同上 | 同上 | 共有 |
| `.cursor/rules/forge-models.mdc` | 初回コピー | 初回コピー | PJ 固有 |
| `.cursor/skills/<共有>/` | omega symlink | VM 上でリンク | 共有 |
| `.cursor/skills/verify-*/` | PJ 固有 | PJ 固有 | PJ 固有 |
| `.cursor/environment.json` | 使わない | commit 必須 | Cloud 用 |
| `.cursor/install-omega.sh` | 使わない | commit 必須 | Cloud 用 |

# 最後に

いかがでしたか？

rules・skills を複数 PJ で使い回すのは、最初は「コピーすればいいじゃん」で始まるんですが、PJ が増えると必ず破綻します。ローカルと Cloud で読む場所が違うのも、体感しないとピンと来ないところです。

私の場合はこうまとめています。

- **ローカル Desktop** → symlink で正本1か所、commit 不要
- **Cloud Agent** → `environment.json` で Build 時に install、commit は 2 ファイル
- **PJ 固有** → `forge-models.mdc` と `verify-*` だけ repo に置く
- **共有 repo に PJ 前提を入れない** → これ、地味に大事

omega はあくまで実装例です。自分の rules / skills セットでも、同じ「正本1か所 + ローカル symlink + Cloud install フック」の構造で運用できます。

Cloud Agent 周りはまだ進化中なので、もっと楽な公式のやり方が出てくれたら嬉しいですね。出てきたらまた記事書きます。

それでは、複数 PJ 運用、乗り切りましょう〜

# 参照

- [Cursor Plugins 公式ドキュメント](https://cursor.com/docs/plugins)
- [Cursor Skills 公式ドキュメント](https://cursor.com/docs/skills)
- [Cloud Agent Plugin Compatibility（Forum）](https://forum.cursor.com/t/cloud-agent-plugin-compatibility/167781)
- omega 正本: `cursor-rules/omega`（`omega/README.md` にセットアップ手順あり）
