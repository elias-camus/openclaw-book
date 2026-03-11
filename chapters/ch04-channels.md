# 第4章　チャンネル連携 ― 外の世界とつなぐ

**この章で学ぶこと:**
- OpenClawが対応するチャンネル（メッセージングサービス）の概要
- Discord・Telegram・Slackの具体的な接続手順
- 複数チャンネルの同時運用方法

**所要時間:** 約45〜60分

---

## 4-1. チャンネルとは何か

OpenClawにおける「チャンネル（Channel）」とは、AIエージェントとやりとりするためのインターフェースのことだ。Discord、Telegram、Slack、WhatsApp、Signalなど、普段使っているメッセージアプリをそのまま入力インターフェースとして使える。[REF-01]

対応チャンネルは執筆時点で20以上にのぼる:

**主要チャンネル:**
- Discord / Slack / Microsoft Teams
- Telegram / Signal / WhatsApp
- LINE / iMessage（BlueBubbles経由）
- Google Chat / Mattermost / Matrix
- IRC / Nostr / Twitch チャットなど

一つのGatewayに複数のチャンネルを同時接続できる。「Discordでエージェントに指示を出し、Telegramに結果通知を受け取る」といった使い方も可能だ。

---

## 4-2. Discord連携

Discordは開発者コミュニティでも人気が高く、OpenClawとの相性が抜群だ。サーバー（グループ）とDM（ダイレクトメッセージ）の両方に対応している。[REF-11]

### Step 1: Discord Botを作成する

1. https://discord.com/developers/applications にアクセスし、「New Application」をクリック

[📸 スクリーンショット: Discord Developer Portalのトップ画面。「New Application」ボタンが見える]

2. アプリ名を入力（例: `MyClaw`）して「Create」

3. 左メニューの「Bot」を選択し、「Add Bot」→「Yes, do it!」をクリック

[📸 スクリーンショット: Discord BotページのTokenセクション]

4. 「Token」セクションの「Reset Token」をクリックし、トークンをコピーしておく
   （このトークンは一度しか表示されないので、必ずメモしておくこと）

### Step 2: BotをDiscordサーバーに招待する

1. 左メニューの「OAuth2」→「URL Generator」を選択
2. 「Scopes」で `bot` にチェック
3. 「Bot Permissions」で以下にチェック:
   - Send Messages
   - Read Message History
   - View Channels
   - Add Reactions（リアクション機能を使う場合）

4. 生成されたURLをブラウザで開き、Botを招待したいサーバーを選択

[📸 スクリーンショット: OAuth2 URL Generatorの画面。ScopesとPermissionsを設定している]

### Step 3: OpenClawにDiscordを設定する

```bash
$ openclaw channels login discord
```

表示される指示に従ってBotトークンを入力する。または `~/.openclaw/openclaw.json` を直接編集する:

```json
{
  "channels": {
    "discord": {
      "token": "YOUR_BOT_TOKEN_HERE"
    }
  }
}
```

設定後にGatewayを再起動すると、DiscordチャンネルでBotが応答するようになる。

---

> 💡 **ポイント:** DiscordのBotトークンは絶対にGitリポジトリや公開の場所に書かないこと。`openclaw.json`は`.gitignore`に追加しておこう。

---

## 4-3. Telegram連携

Telegramは軽量で高速、APIも安定しており、個人利用のOpenClawに非常に向いている。[REF-12]

### Step 1: BotFatherでBotを作成する

1. Telegramアプリを開き、検索バーに「BotFather」と入力してチャットを開く
2. `/newbot` と送信する
3. Botの名前を入力（例: `My OpenClaw Bot`）
4. BotのユーザーID（username）を入力（例: `myopenclaw_bot`。`_bot`で終わる必要がある）

[📸 スクリーンショット: TelegramのBotFatherとのチャット。/newbotを実行してトークンが発行された画面]

5. `HTTP API token` が発行される（例: `1234567890:ABCDEFGhijklmnopqrstuvwxyz`）

### Step 2: OpenClawにTelegramを設定する

```bash
$ openclaw channels login telegram
```

または `openclaw.json` に追記:

```json
{
  "channels": {
    "telegram": {
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

Gatewayを再起動して、Telegramで自分のBotにメッセージを送ると返答が来るはずだ。

---

### 📦 コラム: どのチャンネルを選ぶべきか——用途別おすすめ

| 用途 | おすすめチャンネル | 理由 |
|---|---|---|
| 個人利用・プライベート | Telegram / Signal | 軽量、プライバシー高い |
| 開発チームの共有Bot | Discord | スレッド管理、権限制御が優秀 |
| 社内業務Bot | Slack / Microsoft Teams | 既存ツールとの統合が容易 |
| 家族・友人との共有 | LINE / WhatsApp | 国内シェアが高い |
| パブリックBot | Discord | サーバーへの招待機能が使いやすい |

---

## 4-4. Slack連携

SlackはビジネスチームでのBot活用に最適だ。

### Step 1: Slack Appを作成する

1. https://api.slack.com/apps にアクセスし、「Create New App」→「From scratch」を選択
2. アプリ名とワークスペースを設定

[📸 スクリーンショット: Slack API管理画面のApp作成ダイアログ]

3. 「OAuth & Permissions」で以下のスコープを追加:
   - `chat:write`（メッセージ送信）
   - `channels:history`（チャンネル履歴の読み取り）
   - `channels:read`（チャンネル一覧の読み取り）
   - `im:history`（DM履歴）

4. 「Bot Token Scopes」設定後、「Install to Workspace」でトークンを取得

5. 「Event Subscriptions」→「Enable Events」をオンにし、Request URLにOpenClawのWebhook URLを入力

### Step 2: 設定ファイルに追記

```json
{
  "channels": {
    "slack": {
      "botToken": "xoxb-YOUR-BOT-TOKEN",
      "appToken": "xapp-YOUR-APP-TOKEN"
    }
  }
}
```

---

## 4-5. Signal連携

Signalは暗号化通信で知られる、セキュリティ重視のメッセージアプリだ。OpenClawとのSignal連携には、`signal-cli`を経由する方法が一般的だ。

> ⚠️ **注意:** SignalはAPIが公式には非公開のため、サードパーティツールを経由する。動作が不安定になる可能性があることを理解した上で使おう。

```bash
$ openclaw channels login signal
```

セットアップ手順はターミナルの指示に従う。電話番号を使った認証が必要になる。

---

## 4-6. WhatsApp連携

WhatsAppはQRコードをスキャンして接続する方式だ。

```bash
$ openclaw channels login whatsapp
```

ターミナルにQRコードが表示される。WhatsAppの「リンクされたデバイス」からスキャンするだけで接続完了だ。

> ⚠️ **注意:** WhatsAppの自動化はMetaの利用規約によりグレーゾーンな部分がある。個人的な自動化目的であれば問題になるケースは少ないが、大量メッセージの送信はアカウント停止のリスクがある。

---

## 4-7. 複数チャンネルの同時運用

`openclaw.json`に複数のチャンネルを並べることで、一つのGatewayが複数のサービスを同時に処理できる:

```json
{
  "channels": {
    "discord": {
      "token": "DISCORD_TOKEN"
    },
    "telegram": {
      "token": "TELEGRAM_TOKEN"
    },
    "slack": {
      "botToken": "SLACK_BOT_TOKEN",
      "appToken": "SLACK_APP_TOKEN"
    }
  }
}
```

Gatewayを再起動すれば、すべてのチャンネルが同時に稼働する。

---

## 4-8. グループチャットでの動作設定

グループチャット（Discordのサーバー、Slackのチャンネルなど）にBotを追加した場合、すべてのメッセージに反応すると邪魔になる。`AGENTS.md`に挙動を設定できる:

```markdown
## Group Chats

**Respond when:**
- Directly mentioned or asked a question
- You can add genuine value

**Stay silent when:**
- It's just casual banter
- Someone already answered
```

メンション（`@BotName`）されたときだけ反応させる設定が一般的だ。

---

### 📦 コラム: グループチャットでのエチケット——いつ発言していつ黙るか

グループチャットにAIエージェントを追加すると、すべての発言が見えている状態になる。これは便利でもあり、煩わしくもなりうる。

人間のグループチャット参加者が「すべてのメッセージに返信しない」のと同様に、エージェントも「発言するタイミングを選ぶ」ように設定しよう。基本ルールはシンプルだ——直接呼びかけられたときと、明らかに役に立てるときだけ発言する。

このさじ加減こそ、使いやすいBotと使いにくいBotの差になる。

---

## まとめ

- OpenClawは20以上のメッセージングサービスに対応している
- Discord・Telegram・Slackはそれぞれ開発者ポータルでBotを作成しトークンを取得する
- `openclaw.json`に複数チャンネルを設定することで同時運用できる
- グループチャットでは発言タイミングの設定が重要

次章では、使用するAIモデル（Claude、GPT-4など）の選び方と設定方法を解説する。

---

**参考リンク:**
- [REF-01] OpenClaw公式ドキュメント: https://docs.openclaw.ai
- [REF-11] Discord設定: https://docs.openclaw.ai/channels/discord.md
- [REF-12] Telegram設定: https://docs.openclaw.ai/channels/telegram.md
