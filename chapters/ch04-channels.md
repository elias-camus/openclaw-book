# 第4章　チャンネル連携 ― 外の世界とつなぐ

**この章で学ぶこと:**
- OpenClawが対応するチャンネル（メッセージングサービス）の概要
- **LINE連携の詳細手順**（日本ユーザー最重要）
- Discord・Telegram・Slackの接続手順
- 複数チャンネルの同時運用方法

**所要時間:** 約60〜75分

---

## 4-1. チャンネルとは何か

OpenClawにおける「チャンネル（Channel）」とは、AIエージェントとやりとりするためのインターフェースのことだ。Discord、Telegram、Slack、WhatsApp、Signalなど、普段使っているメッセージアプリをそのまま入力インターフェースとして使える。[REF-01]

対応チャンネルは執筆時点で20以上にのぼる:

**主要チャンネル:**
- **LINE**（日本国内シェアNo.1。本章で最重点解説）
- Discord / Slack / Microsoft Teams
- Telegram / Signal / WhatsApp
- iMessage（BlueBubbles経由）
- Google Chat / Mattermost / Matrix
- IRC / Nostr / Twitch チャットなど

一つのGatewayに複数のチャンネルを同時接続できる。「LINEでエージェントに指示を出し、Discordのサーバーにも同じエージェントが常駐する」といった使い方も可能だ。

> 💡 **日本ユーザーへのポイント:** 国内では圧倒的多数がLINEを使っている。「外出先からスマホでAIに指示を出す」用途には、Telegramより馴染みのあるLINEが圧倒的に使いやすい。まずLINE連携から始めよう。

---

## 4-2. LINE連携（日本ユーザー最重要）

LINEはLINE Messaging APIを通じてOpenClawと連携する。「外出先のスマホから自宅のAIに指示を送る」という、OpenClawの真骨頂を最も手軽に体験できるチャンネルだ。

```
仕組み:
[あなたのスマホ] → LINE → [LINE Messaging API]
      → [OpenClaw Gateway (Webhook)] → [AI処理]
      → 返答がLINEに戻る
```

> 画面で確認する内容: LINEアプリ上でAIエージェントのBotアカウントとのチャット画面が開いた状態です。ユーザーが「今日の天気は？」と送信すると、数秒後にBotから天気情報を含む返答メッセージが届きます。Botのアイコンと名前（設定したチャンネル名）がトーク画面の上部に表示されます。

### 必要なもの

- LINE アカウント（普段使いのものでOK）
- LINE Developersへの登録（無料）
- OpenClawがインストール済みのサーバー（Mac Mini or Lightsail）
- HTTPS対応のURLまたはngrok（後述）

### Step 1: LINEプラグインをインストール

LINEはOpenClawのプラグインとして提供されている。まずインストール:

```bash
$ openclaw plugins install @openclaw/line
```

### Step 2: LINE Developersでチャンネルを作成

1. https://developers.line.biz/console/ にアクセスし、普段のLINEアカウントでログイン

> 📌 **参照:** [LINE Developers Console](https://developers.line.biz/console/)
> 画面で確認する内容: LINE Developers Consoleのログインページが開きます。「LINEでログイン」ボタンをクリックし、普段使っているLINEアカウントのメールアドレスとパスワードで認証します。初回ログイン時は開発者アカウントの登録画面が表示されます。

2. 「プロバイダーを作成」をクリック → プロバイダー名を入力（例: `MyOpenClaw`）

3. 「新規チャンネル作成」→「Messaging API」を選択

> 📌 **参照:** [LINE Developers — チャンネル作成ドキュメント](https://developers.line.biz/ja/docs/messaging-api/getting-started/)
> 画面で確認する内容: プロバイダーを作成後、「チャンネルを作成する」をクリックすると、チャンネル種別の選択画面が表示されます。「Messaging API」のカードをクリックして選択します。次の画面でチャンネル名・説明・アイコン・業種を入力します。

4. 各項目を入力:
   - チャンネルアイコン: 任意（後から変更可能）
   - チャンネル名: 好きな名前（例: `私のAI秘書`）
   - チャンネル説明: 任意
   - 大業種・小業種: 任意

5. 利用規約に同意して「作成」

### Step 3: トークンとシークレットを取得

チャンネル作成後、「チャンネル設定」タブを開く:

- **チャンネルシークレット:** 「基本設定」タブに表示されている（`Channel secret`）
- **チャンネルアクセストークン（長期）:** 「Messaging API設定」タブ → 一番下の「チャンネルアクセストークン（長期）」の「発行」をクリック

> 📌 **参照:** [LINE Developers — チャンネルアクセストークンの発行](https://developers.line.biz/ja/docs/messaging-api/channel-access-tokens/)
> 画面で確認する内容: チャンネル設定ページの「Messaging API設定」タブを開き、最下部にスクロールすると「チャンネルアクセストークン（長期）」という項目があります。「発行」ボタンをクリックすると長い文字列のトークンが表示されます。コピーボタンで取得してください。

この2つをメモしておく。

### Step 4: Webhook URLの設定

LINE → OpenClawへのメッセージ転送のために「Webhook URL」を設定する。OpenClawはWebhookを受け取るサーバーとして機能する。

**Lightsailの場合（ドメインあり）:**

```
https://myagent.example.com/line/webhook
```

**Mac Miniの場合（ngrokで一時公開）:**

ngrok（エングロック）は自宅PCをインターネットに一時的に公開するツールだ:

```bash
# ngrokのインストール（Homebrewで）
$ brew install ngrok/ngrok/ngrok

# OpenClawのポートを公開
$ ngrok http 18789
```

> 画面で確認する内容: ngrokを起動すると、ターミナルにセッション情報が表示されます。`Forwarding` の行に `https://xxxx-xxx-xxx-xxx-xxx.ngrok-free.app -> http://localhost:18789` のような形式でURLが表示されます。この `https://` で始まるURLをコピーして使用します。

表示された `https://xxxx.ngrok.io` を使って:
```
https://xxxx.ngrok.io/line/webhook
```

LINE Developers の「Messaging API設定」→「Webhook URL」に貼り付け → 「検証」ボタンで確認。

> ⚠️ **ngrokの注意点:** 無料プランではURLが再起動のたびに変わる。Mac Miniを恒久的なサーバーにする場合は、固定ドメインを取得してLightsailと同様の設定をするか、ngrokの有料プランを使おう。

### Step 5: OpenClawの設定

`~/.openclaw/openclaw.json` に追記:

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "ここにチャンネルアクセストークン",
      "channelSecret": "ここにチャンネルシークレット",
      "dmPolicy": "pairing"
    }
  }
}
```

### Step 6: Gatewayを再起動してテスト

```bash
$ openclaw gateway restart
```

LINE Developersの「Messaging API設定」→「Webhookの利用」をオンにする。

LINEアプリで自分のBotアカウントを友だち追加して（QRコードは「Messaging API設定」に表示されている）、メッセージを送ってみよう。

> 📌 **参照:** [LINE Developers — 友だち追加方法](https://developers.line.biz/ja/docs/messaging-api/getting-started/#add-bot-as-friend)
> 画面で確認する内容: LINE Developers ConsoleのMessaging API設定タブ内に「QRコード」が表示されています。LINEアプリの「友だち追加」→「QRコード」でこのQRコードを読み取ると、作成したBotアカウントが友だちに追加されます。

「今日の天気は？」と送って返答が来れば成功だ。

### ペアリング（セキュリティ設定）

`dmPolicy: "pairing"` を設定した場合、最初にメッセージを送ると「ペアリングコード」が表示される。`openclaw pairing approve line <コード>` でそのユーザーを許可する仕組みだ。自分以外の人にBotを使わせたくない場合に有効だ。

```bash
$ openclaw pairing list line    # 承認待ち一覧
$ openclaw pairing approve line <CODE>  # 承認
```

---

### LINEの制限と注意事項

| プラン | 月間送信数 | 月額 |
|---|---|---|
| フリー | 200通 | 無料 |
| ライト | 5,000通 | 月5,000円 |
| スタンダード | 30,000通 | 月15,000円 |

個人利用の場合、月200通は少ないように見えるが、**「受信（エージェントへの指示）」はカウント対象外**で、**エージェントからの返信のみカウント**される。1日6〜7回の指示なら無料枠で収まる。

> 💡 **コスト節約のコツ:** LINEグループを活用するとカウントの仕方が変わり、節約できる場合がある。また「毎朝の定期通知」はCronジョブで設定しているため、そのカウントに注意して設計しよう。

---

### 📦 コラム: なぜ日本ではLINEが最適なのか

日本のLINE月間ユーザー数は約9,700万人（2024年時点）。スマートフォンを持つ日本人のほぼ全員が使っているといっても過言ではない。

TelegramやDiscordは開発者・ゲーマーには浸透しているが、家族・取引先・近所のお店など「ビジネス以外のつながり」を含めると、LINEに及ばない。

「外出先からAI秘書に指示を送る」という体験を最も自然にできるのが、すでに日常で使っているLINEだ。通知も来るし、グループも作れるし、既読もわかる。OpenClawをLINEと連携させることで、「AIエージェント」というハードルが一気に下がる。

---

## 4-3. Discord連携

Discordは開発者コミュニティでも人気が高く、OpenClawとの相性が抜群だ。サーバー（グループ）とDM（ダイレクトメッセージ）の両方に対応している。[REF-11]

### Step 1: Discord Botを作成する

1. https://discord.com/developers/applications にアクセスし、「New Application」をクリック

> 📌 **参照:** [Discord Developer Portal](https://discord.com/developers/applications)
> 画面で確認する内容: Discord Developer Portalを開くと、「Applications」の一覧画面が表示されます。右上の「New Application」ボタンをクリックしてアプリ名を入力し、「Create」ボタンで作成します。作成後、アプリの設定画面が開きます。

2. アプリ名を入力（例: `MyClaw`）して「Create」

3. 左メニューの「Bot」を選択し、「Add Bot」→「Yes, do it!」をクリック

> 📌 **参照:** [Discord Developer Portal — Bot設定](https://discord.com/developers/docs/topics/oauth2)
> 画面で確認する内容: 左メニューの「Bot」を開くと、Botの設定画面が表示されます。「Token」セクションにある「Reset Token」ボタンをクリックするとトークンが生成されます。**このトークンは一度しか表示されません。** 必ずコピーして安全な場所に保管してください。

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

> 📌 **参照:** [Discord Developer Portal — OAuth2](https://discord.com/developers/docs/topics/oauth2#bot-authorization-flow)
> 画面で確認する内容: 左メニューの「OAuth2」→「URL Generator」を開くと、ScopesとBot Permissionsのチェックボックス一覧が表示されます。`bot` にチェックを入れると下部にBot Permissionsの一覧が現れます。必要な権限にチェックを入れると、画面下部に招待URLが自動生成されます。

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

## 4-4. Telegram連携

Telegramは軽量で高速、APIも安定しており、個人利用のOpenClawに非常に向いている。[REF-12]

### Step 1: BotFatherでBotを作成する

1. Telegramアプリを開き、検索バーに「BotFather」と入力してチャットを開く
2. `/newbot` と送信する
3. Botの名前を入力（例: `My OpenClaw Bot`）
4. BotのユーザーID（username）を入力（例: `myopenclaw_bot`。`_bot`で終わる必要がある）

> 📌 **参照:** [Telegram BotFather 公式ドキュメント](https://core.telegram.org/bots/tutorial)
> 画面で確認する内容: TelegramでBotFatherを検索してチャットを開き、`/newbot` と送信します。Botの名前とusernameを設定すると、`Done! Congratulations on your new bot.` というメッセージとともに、`Use this token to access the HTTP API:` に続いて数字とハイフン・英数字からなるトークン文字列が表示されます。

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

## 4-5. Slack連携

SlackはビジネスチームでのBot活用に最適だ。

### Step 1: Slack Appを作成する

1. https://api.slack.com/apps にアクセスし、「Create New App」→「From scratch」を選択
2. アプリ名とワークスペースを設定

> 📌 **参照:** [Slack API — アプリ作成ページ](https://api.slack.com/apps)
> 画面で確認する内容: 「Create New App」をクリックして「From scratch」を選択すると、アプリ名とワークスペース選択のダイアログが表示されます。アプリ名（例: `MyClaw`）を入力し、インストール先のワークスペースをドロップダウンから選んで「Create App」をクリックします。

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

## 4-6. Signal連携

Signalは暗号化通信で知られる、セキュリティ重視のメッセージアプリだ。OpenClawとのSignal連携には、`signal-cli`を経由する方法が一般的だ。

> ⚠️ **注意:** SignalはAPIが公式には非公開のため、サードパーティツールを経由する。動作が不安定になる可能性があることを理解した上で使おう。

```bash
$ openclaw channels login signal
```

セットアップ手順はターミナルの指示に従う。電話番号を使った認証が必要になる。

---

## 4-7. WhatsApp連携

WhatsAppはQRコードをスキャンして接続する方式だ。

```bash
$ openclaw channels login whatsapp
```

ターミナルにQRコードが表示される。WhatsAppの「リンクされたデバイス」からスキャンするだけで接続完了だ。

> ⚠️ **注意:** WhatsAppの自動化はMetaの利用規約によりグレーゾーンな部分がある。個人的な自動化目的であれば問題になるケースは少ないが、大量メッセージの送信はアカウント停止のリスクがある。

---

## 4-8. 複数チャンネルの同時運用

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

## 4-9. グループチャットでの動作設定

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
- 日本ユーザーにはLINE連携が最も身近で使いやすい選択肢だ
- Discord・Telegram・Slackはそれぞれ開発者ポータルでBotを作成しトークンを取得する
- `openclaw.json`に複数チャンネルを設定することで同時運用できる
- グループチャットでは発言タイミングの設定が重要

次章では、使用するAIモデル（Claude、GPT-4など）の選び方と設定方法を解説する。

---

**参考リンク:**
- [REF-01] OpenClaw公式ドキュメント: https://docs.openclaw.ai
- [REF-11] Discord設定: https://docs.openclaw.ai/channels/discord.md
- [REF-12] Telegram設定: https://docs.openclaw.ai/channels/telegram.md
