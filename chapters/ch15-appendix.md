# 付録

---

## 付録A　設定ファイルリファレンス（openclaw.json）

`~/.openclaw/openclaw.json` の主要な設定項目の完全リファレンス。

```json
{
  // ─── AIモデル設定 ───────────────────────────────────────
  "model": "anthropic/claude-sonnet-4-5",
  "modelFallback": "openai/gpt-4o-mini",

  // ─── プロバイダー別APIキー ────────────────────────────────
  "providers": {
    "anthropic": {
      "apiKey": "sk-ant-api03-..."
    },
    "openai": {
      "apiKey": "sk-proj-..."
    },
    "amazon-bedrock": {
      "region": "us-east-1",
      "accessKeyId": "AKIA...",
      "secretAccessKey": "..."
    },
    "google": {
      "apiKey": "AIza..."
    },
    "ollama": {
      "baseUrl": "http://localhost:11434"
    },
    "deepseek": {
      "apiKey": "sk-..."
    }
  },

  // ─── チャンネル設定 ────────────────────────────────────────
  "channels": {
    "discord": {
      "token": "MTxxxxxxxxx.xxxxxxx.xxxxxxxxxx"
    },
    "telegram": {
      "token": "1234567890:ABCdef..."
    },
    "slack": {
      "botToken": "xoxb-...",
      "appToken": "xapp-..."
    },
    "line": {
      "channelAccessToken": "...",
      "channelSecret": "...",
      "dmPolicy": "pairing"
    },
    "signal": {},
    "whatsapp": {}
  },

  // ─── 費用上限（月次アラート） ───────────────────────────────
  "budgetLimit": {
    "monthly": 3000,
    "notifyAt": 0.7
  },

  // ─── タイムゾーン・ロケール ────────────────────────────────
  "timezone": "Asia/Tokyo",
  "locale": "ja",

  // ─── ハートビート設定 ──────────────────────────────────────
  "heartbeat": {
    "intervalMs": 1800000
  },

  // ─── セキュリティ ─────────────────────────────────────────
  "security": {
    "allowedHosts": ["localhost", "127.0.0.1"]
  }
}
```

### 主要設定項目の説明

| キー | 説明 | 例 |
|---|---|---|
| `model` | メインで使うAIモデル | `"anthropic/claude-sonnet-4-5"` |
| `modelFallback` | メインが使えない時の代替 | `"openai/gpt-4o-mini"` |
| `timezone` | タイムゾーン | `"Asia/Tokyo"` |
| `budgetLimit.monthly` | 月間上限（円）| `3000` |
| `budgetLimit.notifyAt` | アラートを出す上限の割合 | `0.7`（70%で通知）|

---

## 付録B　よく使うコマンド完全リファレンス

### Gateway（ゲートウェイ）操作

```bash
# 起動・停止・管理
openclaw gateway              # フォアグラウンドで起動
openclaw gateway --background # バックグラウンド起動
openclaw gateway status       # 状態確認（起動中/停止中）
openclaw gateway restart      # 再起動
openclaw gateway stop         # 停止
openclaw gateway install-daemon  # 起動時自動実行に登録
openclaw gateway uninstall-daemon  # 自動起動を解除
openclaw gateway logs         # 最新ログを表示
openclaw gateway logs --follow # ログをリアルタイム表示（Ctrl+Cで終了）
```

### チャンネル管理

```bash
# 接続・切断
openclaw channels login discord
openclaw channels login telegram
openclaw channels login slack
openclaw channels login signal
openclaw channels list          # 接続中チャンネル一覧
openclaw channels logout discord  # 特定チャンネルを切断
openclaw channels status        # 全チャンネルの接続状態

# プラグイン（LINE等）
openclaw plugins install @openclaw/line
openclaw plugins list
openclaw plugins update @openclaw/line
openclaw plugins remove @openclaw/line
```

### スキル管理

```bash
# インストール・管理
openclaw skills list            # インストール済みスキル一覧
openclaw skills search <keyword> # Clawhubでスキル検索
openclaw skills install <name>  # スキルをインストール
openclaw skills update <name>   # スキルをアップデート
openclaw skills update --all    # 全スキルを一括アップデート
openclaw skills remove <name>   # スキルをアンインストール
openclaw skills info <name>     # スキルの詳細情報
```

### Cronジョブ管理

```bash
# ジョブの操作
openclaw cron list              # 登録済みジョブ一覧
openclaw cron run <job-id>      # ジョブを今すぐ手動実行
openclaw cron pause <job-id>    # ジョブを一時停止
openclaw cron resume <job-id>   # ジョブを再開
openclaw cron remove <job-id>   # ジョブを削除
openclaw cron logs <job-id>     # ジョブの実行ログ
openclaw cron runs <job-id>     # ジョブの実行履歴
```

### 設定管理

```bash
# 設定の確認・編集
openclaw config get             # 現在の設定を表示
openclaw config set model "anthropic/claude-haiku-4-5"  # 設定値を変更
openclaw config edit-soul       # SOUL.mdをエディタで開く
openclaw config path            # 設定ファイルの場所を表示

# アップデート・診断
openclaw update                 # 最新バージョンにアップデート
openclaw --version              # バージョン確認
openclaw doctor                 # 設定の問題をチェック
openclaw status                 # 全体状態のサマリー
```

### セッション管理

```bash
openclaw sessions list          # セッション一覧
openclaw sessions prune         # 古いセッションを削除
openclaw sessions history <key> # 特定セッションの履歴
```

---

## 付録C　トラブルシューティングQ&A（完全版）

### インストール・起動の問題

**Q: `openclaw: command not found` と表示される**

原因: インストールが完了していないか、PATHが通っていない

```bash
# 再インストール
npm install -g openclaw@latest

# PATHの確認
echo $PATH | grep -o '/usr/local/bin\|/opt/homebrew/bin'

# npmのglobalパスを確認
npm root -g
```

---

**Q: Gatewayが起動しない / すぐ止まる**

確認手順:
```bash
node --version          # v22以上か確認
openclaw doctor         # 設定エラーを確認
openclaw gateway logs   # エラーログを確認
```

よくある原因:
- Node.jsのバージョンが古い（v22以上が必要）
- `openclaw.json`の構文エラー（余分なカンマ等）
- ポート18789が別のプロセスに使われている

---

**Q: `Cannot find module` エラーが出る**

```bash
# キャッシュをクリアして再インストール
npm cache clean --force
npm install -g openclaw@latest
```

---

### チャンネル・接続の問題

**Q: Discordのメッセージに返答しない**

確認チェックリスト:
- [ ] Botトークンが正しいか（Discord Developer Portalで確認）
- [ ] BotにMessageContentIntentが有効か
- [ ] BotがサーバーとChannelに招待されているか
- [ ] `openclaw channels list`でdiscordが表示されるか

---

**Q: LINEのBotが返答しない**

確認チェックリスト:
- [ ] OpenClawが起動しているか（`openclaw gateway status`）
- [ ] Webhook URLが正しく設定されているか
- [ ] AWS Lightsailの場合、SSLが有効になっているか（https://が必要）
- [ ] ローカルの場合、ngrokが起動しているか
- [ ] ngrokを再起動した場合、新しいURLをLINE Developersに登録したか

---

**Q: Telegramのメッセージに返答しない**

```bash
# Botのtoken確認
curl "https://api.telegram.org/bot<YOUR_TOKEN>/getMe"
# {"ok":true,...} が返れば token は有効
```

---

### APIキー・費用の問題

**Q: `401 Unauthorized` エラーが返ってくる**

原因: APIキーが無効または期限切れ

対処:
1. Anthropic Console / OpenAI Platform でAPIキーの状態を確認
2. クレジットカードの支払いが通っているか確認
3. `openclaw config set anthropicApiKey "sk-ant-..."` で再設定

---

**Q: `429 Too Many Requests` エラーが出る**

原因: APIのレート制限に達している

対処:
- しばらく待つ（1〜5分）
- モデルを安い（軽い）ものに変更する
- Cronジョブの間隔を広げる

---

**Q: 月の費用が予想より高い**

確認・対処:
```bash
openclaw cron list    # 無駄なジョブを確認
openclaw status       # 使用量を確認
```
- 安価なモデル（claude-haiku / gpt-4o-mini）に切り替える
- 不要なCronジョブを削除する
- SOUL.mdが長すぎる場合は短縮する（毎回トークンを消費するため）

---

### メモリ・スキルの問題

**Q: エージェントが前回の会話を覚えていない**

原因: MEMORY.mdへの書き込みが設定されていない

対処: SOUL.mdに以下を追記:
```markdown
## 重要なことは必ずmemory/YYYY-MM-DD.mdに記録すること
セッションをまたいで覚えておきたい情報はMEMORY.mdに保存する
```

---

**Q: スキルをインストールしたが動かない**

```bash
openclaw gateway restart    # Gatewayを再起動
openclaw skills list        # スキルが認識されているか確認
openclaw doctor             # 問題がないか確認
```

---

**Q: ブラウザ操作（Browserスキル）が失敗する**

よくある原因:
- Cloudflare等のBot対策でブロックされている
- JavaScriptレンダリングが必要なページ
- 操作が速すぎる（timeoutを増やす）

対処: Browser RelayスキルをChrome拡張と組み合わせて使う（第9章参照）

---

## 付録D　モデル比較・選択ガイド（2026年3月時点）

### 用途別おすすめモデル

| 用途 | おすすめモデル | 理由 |
|---|---|---|
| 日常会話・日本語テキスト | Claude Sonnet / Haiku | 日本語の自然さが最高水準 |
| コーディング・技術作業 | Claude Sonnet / o3 | コード理解と生成が優秀 |
| 大量のテキスト処理 | Gemini 1.5 Pro | 100万トークンのコンテキスト |
| コスト最優先 | DeepSeek-V3 / Gemini Flash | API料金が非常に安い |
| プライバシー重視 | Llama 3（Ollama） | 完全ローカル・無料 |
| 高度な推論 | o3 / Claude Opus | 複雑な思考・分析タスク |
| Cronジョブ（軽い処理）| Claude Haiku / GPT-4o mini | 低コストで十分な品質 |

### プロバイダー別 費用感（2026年3月時点・概算）

> ⚠️ 料金は頻繁に変更されます。最新料金は各社の公式サイトを確認してください。

| モデル | 入力 (1M tokens) | 出力 (1M tokens) | 特徴 |
|---|---|---|---|
| Claude Haiku | 〜$0.25 | 〜$1.25 | 最安・最速 |
| Claude Sonnet | 〜$3 | 〜$15 | バランス型 |
| Claude Opus | 〜$15 | 〜$75 | 最高品質 |
| GPT-4o mini | 〜$0.15 | 〜$0.60 | OpenAI最安 |
| GPT-4o | 〜$2.50 | 〜$10 | 標準 |
| DeepSeek-V3 | 〜$0.27 | 〜$1.10 | 圧倒的コスパ |
| Gemini 1.5 Flash | 〜$0.075 | 〜$0.30 | Google最安 |
| Llama 3（Ollama）| 無料 | 無料 | 完全ローカル |

**月間費用の目安（個人・軽い利用）:**
- Claude Haiku中心: 300〜1,000円/月
- Claude Sonnet中心: 1,000〜5,000円/月
- DeepSeek中心: 200〜800円/月

---

## 付録E　推奨スキル一覧（Clawhub）

### 公式スキル（標準搭載）

| スキル名 | 機能 | 用途 |
|---|---|---|
| `weather` | 天気予報取得 | 毎朝の天気通知 |
| `discord` | Discord高度操作 | チャンネル管理・投稿 |
| `tmux` | ターミナルセッション操作 | 開発作業の自動化 |
| `healthcheck` | セキュリティ・システム監視 | 定期ヘルスチェック |
| `skill-creator` | スキルの自動生成 | 新スキル作成支援 |

### コミュニティ推奨スキル

| スキル名 | 機能 | 用途 |
|---|---|---|
| `github-monitor` | Issue/PR/Releaseの監視・通知 | 開発チーム向け |
| `notion` | Notionデータベース読み書き | タスク管理・ドキュメント |
| `google-calendar` | カレンダー操作 | スケジュール確認・追加 |
| `google-sheets` | スプレッドシート操作 | データ管理・集計 |
| `shopify` | ショップ情報取得 | EC管理 |
| `rss-reader` | RSSフィードの収集・要約 | ニュース収集 |
| `twitter` | X（Twitter）投稿・検索 | SNS運用 |
| `slack-advanced` | Slackの高度な操作 | チーム向け通知 |
| `aws-monitor` | AWSリソースの監視 | インフラ管理 |
| `sendgrid` | メール送信 | メルマガ・通知メール |

> ⚠️ Clawhubのスキルは第三者が作成しています。インストール前にソースコードを確認してください（第12章参照）。

### スキルのインストール例

```bash
# 天気スキル（公式）
openclaw skills install weather

# Notion連携
openclaw skills install notion

# GitHub監視
openclaw skills install github-monitor

# LINEプラグイン
openclaw plugins install @openclaw/line
```

---

## 付録F　SOUL.md テンプレート集

用途別のSOUL.mdテンプレートをコピーして使えます。

### 汎用テンプレート

```markdown
# SOUL.md

## 基本設定
あなたは[名前]です。[役割]として動作します。

## 性格・口調
- [例: タメ口・簡潔・親しみやすい]
- 余計な前置きをしない
- わからないことは「わかりません」と答える

## ユーザーについて
- 名前: [名前]
- 職業・活動: [説明]
- 重要な情報: [業種・よく使う情報等]

## よく使う情報
[繰り返し参照する情報をここに書く]

## 禁止事項
- 個人情報を外部に送信しない
- 削除・大量送信は確認なしに実行しない
```

### ビジネス向けカスタマーサポートテンプレート

```markdown
# SOUL.md

あなたは[会社名/店名]のカスタマーサポート担当[名前]です。

## 対応方針
- お客様の気持ちに共感する
- 問題解決を最優先に
- 解決できない場合は担当者へエスカレーション

## 会社/店舗情報
- 営業時間: [時間]
- 所在地/対応エリア: [説明]
- 連絡先: [メール等]

## FAQ
Q: [よくある質問1]
A: [回答]

Q: [よくある質問2]
A: [回答]

## エスカレーション基準
以下の場合は「担当者に確認してご連絡します」と答える:
- [金額・内容の基準]
- [法的・医療的相談]
```

---

## 付録G　Cron式 早見表

| やりたいこと | Cron式 |
|---|---|
| 毎分 | `* * * * *` |
| 5分おき | `*/5 * * * *` |
| 30分おき | `*/30 * * * *` |
| 毎時00分 | `0 * * * *` |
| 毎朝7時 | `0 7 * * *` |
| 毎朝9時（平日のみ）| `0 9 * * 1-5` |
| 毎週月曜朝8時 | `0 8 * * 1` |
| 毎週金曜夕方17時 | `0 17 * * 5` |
| 毎月1日朝9時 | `0 9 1 * *` |
| 毎月末（28日）夜22時 | `0 22 28 * *` |
| 毎日深夜0時 | `0 0 * * *` |
| 毎日昼12時と夜18時 | `0 12,18 * * *` |

**Cron式の構造:**
```
分   時   日   月   曜日
*    *    *    *    *

曜日: 0=日曜、1=月曜、2=火曜、3=水曜、4=木曜、5=金曜、6=土曜
```

---

## 付録H　参考文献・リソース一覧

### 公式リソース

| リソース | URL | 説明 |
|---|---|---|
| OpenClaw公式サイト | https://openclaw.ai | トップページ |
| 公式ドキュメント | https://docs.openclaw.ai | 全機能の詳細解説 |
| GitHub | https://github.com/openclaw/openclaw | ソースコード |
| Clawhub | https://clawhub.com | スキルマーケットプレイス |
| Discordコミュニティ | https://discord.com/invite/clawd | ユーザーコミュニティ |
| awesome-openclaw-skills | https://github.com/VoltAgent/awesome-openclaw-skills | おすすめスキルリスト |

### 設定関連サービス（公式）

| サービス | URL | 用途 |
|---|---|---|
| Anthropic Console | https://console.anthropic.com | ClaudeのAPIキー取得 |
| OpenAI Platform | https://platform.openai.com | GPT-4のAPIキー取得 |
| Google AI Studio | https://aistudio.google.com | GeminiのAPIキー取得 |
| LINE Developers | https://developers.line.biz | LINE Bot設定 |
| Discord Developer Portal | https://discord.com/developers | Discord Bot設定 |
| AWS Lightsail | https://lightsail.aws.amazon.com | クラウドサーバー |
| Homebrew | https://brew.sh | Macパッケージ管理 |
| Node.js | https://nodejs.org | JavaScript実行環境 |

### 本書で引用した参考文献（主要）

| ID | タイトル | URL |
|---|---|---|
| REF-01 | OpenClaw公式ドキュメント | https://docs.openclaw.ai |
| REF-02 | OpenClaw GitHub | https://github.com/openclaw/openclaw |
| REF-06 | Cronジョブドキュメント | https://docs.openclaw.ai/automation/cron-jobs.md |
| REF-09 | メモリ概念 | https://docs.openclaw.ai/concepts/memory.md |
| REF-14 | Clawhub | https://clawhub.com |
| REF-18 | 15 Powerful OpenClaw Use Cases | https://kanerika.com/blogs/openclaw-usecases/ |
| REF-22 | 25 OpenClaw Use Cases | https://www.hostinger.com/tutorials/openclaw-use-cases |
| REF-24 | Wikipedia: OpenClaw | https://en.wikipedia.org/wiki/OpenClaw |

完全な参考文献一覧は `references/REFERENCES.md` を参照のこと。

---

## 付録I　スクリーンショット撮影箇所リスト

本書には手順説明箇所に `[📸 スクリーンショット: ...]` というプレースホルダーが含まれています。実際の書籍版では以下の画面のスクリーンショットを挿入します。

**第2章（Mac Mini環境構築）:**
- Homebrewインストール後のターミナル表示
- Node.jsバージョン確認画面
- OpenClawインストール完了画面
- openclaw gatewayの起動画面

**第3章（AWS Lightsail）:**
- Lightsailのインスタンス作成画面
- セキュリティグループの設定画面
- SSL証明書設定完了の確認

**第4章（チャンネル連携）:**
- Discord Developer Portalのアプリ作成画面
  （https://discord.com/developers/applications）
- LINE Developers ConsoleのMessaging API設定画面
  （https://developers.line.biz/console）
- Telegram BotFatherとの会話画面

**第5章（AIモデル設定）:**
- Anthropic ConsoleのAPIキー生成画面
  （https://console.anthropic.com/settings/keys）
- OpenAI PlatformのAPIキー画面
  （https://platform.openai.com/api-keys）

> 📌 各サービスのUIは随時変更されます。最新の画面は各公式サイトをご参照ください。
