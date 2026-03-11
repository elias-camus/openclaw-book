# 付録

---

## 付録A　設定ファイルリファレンス（openclaw.json）

`~/.openclaw/openclaw.json`の主要な設定項目一覧。

```json
{
  // AIモデルの指定
  "model": "anthropic/claude-3-5-sonnet-20241022",
  "modelFallback": "openai/gpt-4o",

  // プロバイダー設定
  "providers": {
    "anthropic": { "apiKey": "sk-ant-..." },
    "openai": { "apiKey": "sk-..." },
    "amazon-bedrock": {
      "region": "us-east-1",
      "accessKeyId": "...",
      "secretAccessKey": "..."
    },
    "ollama": { "baseUrl": "http://localhost:11434" }
  },

  // チャンネル設定
  "channels": {
    "discord": { "token": "..." },
    "telegram": { "token": "..." },
    "slack": { "botToken": "xoxb-...", "appToken": "xapp-..." },
    "whatsapp": {}
  },

  // タイムゾーン
  "timezone": "Asia/Tokyo"
}
```

---

## 付録B　よく使うコマンド一覧

```bash
# Gatewayの操作
openclaw gateway              # 起動
openclaw gateway status       # 状態確認
openclaw gateway restart      # 再起動
openclaw gateway stop         # 停止
openclaw gateway install-daemon  # デーモン登録

# チャンネル管理
openclaw channels login discord
openclaw channels login telegram
openclaw channels list
openclaw channels logout discord

# スキル管理
openclaw skills list
openclaw skills search <keyword>
openclaw skills install <name>
openclaw skills update <name>
openclaw skills remove <name>

# Cronジョブ管理
openclaw cron list
openclaw cron run <job-id>
openclaw cron remove <job-id>

# セッション管理
openclaw sessions list
openclaw sessions prune       # 古いセッション削除

# 診断・メンテナンス
openclaw doctor               # 設定の問題チェック
openclaw status               # 全体状態確認
openclaw update               # アップデート
openclaw --version            # バージョン確認
```

---

## 付録C　トラブルシューティングQ&A

**Q: Gatewayが起動しない**  
A: `node --version`でNode.js v22以上かを確認。`openclaw doctor`で設定を診断する。

**Q: チャンネルからのメッセージに返答しない**  
A: Botトークンが正しいか確認。`openclaw channels list`でチャンネルが認識されているか確認。

**Q: APIエラーが返ってくる**  
A: APIキーの有効期限・使用量上限を確認。`openclaw.json`のモデル名が正しいか確認。

**Q: Cronジョブが実行されていない**  
A: `openclaw cron list`でジョブが登録されているか確認。systemd/launchdの状態確認。

**Q: メモリファイルが大きくなりすぎた**  
A: `memory/`ディレクトリの古いファイルをアーカイブ。`MEMORY.md`の古い情報を整理・削除。

**Q: ブラウザ操作が失敗する**  
A: 対象サイトのCloudflare等のBot対策によりブロックされている可能性がある。

**Q: スキルをインストールしたが動かない**  
A: Gatewayを再起動する。`openclaw skills list`でスキルが認識されているか確認。

---

## 付録D　AIモデル比較表（執筆時点）

| モデル | プロバイダー | 強み | コスト感 |
|---|---|---|---|
| Claude 3.5 Sonnet | Anthropic | 指示追従・長文・コード | 中〜高 |
| Claude 3 Opus | Anthropic | 高品質・複雑な推論 | 高 |
| GPT-4o | OpenAI | バランス・マルチモーダル | 中 |
| GPT-4o mini | OpenAI | 速度・コスパ | 低 |
| o3 | OpenAI | 高度な推論 | 高 |
| DeepSeek-V3 | DeepSeek | 超コスパ | 非常に低 |
| Gemini 1.5 Pro | Google | 大コンテキスト | 中 |
| Gemini 1.5 Flash | Google | 超低コスト | 非常に低 |
| Llama 3 (Ollama) | Meta/ローカル | 無料・プライベート | 0円 |

---

## 付録E　推奨スキル・ツール一覧

### 公式スキル（標準搭載）
| スキル名 | 機能 |
|---|---|
| weather | 天気予報取得 |
| discord | Discord高度操作 |
| tmux | ターミナルセッション操作 |
| healthcheck | セキュリティ監視 |
| skill-creator | スキル自動生成 |

### コミュニティ推奨スキル（Clawhub）
| スキル名 | 機能 |
|---|---|
| github-monitor | GitHubのIssue/PR監視 |
| notion | Notionデータベース操作 |
| google-calendar | Googleカレンダー連携 |
| shopify | ショップ情報取得 |
| rss-reader | RSSフィードの収集・要約 |

---

## 付録F　参考文献一覧

本書で引用・参照した主要なリソース。

| ID | タイトル | URL |
|---|---|---|
| REF-01 | OpenClaw公式ドキュメント | https://docs.openclaw.ai |
| REF-02 | OpenClaw GitHub | https://github.com/openclaw/openclaw |
| REF-06 | Cronジョブドキュメント | https://docs.openclaw.ai/automation/cron-jobs.md |
| REF-07 | Cron vs Heartbeat | https://docs.openclaw.ai/automation/cron-vs-heartbeat.md |
| REF-08 | マルチエージェント | https://docs.openclaw.ai/concepts/multi-agent.md |
| REF-09 | メモリ概念 | https://docs.openclaw.ai/concepts/memory.md |
| REF-10 | Gatewayアーキテクチャ | https://docs.openclaw.ai/concepts/architecture.md |
| REF-14 | Clawhub | https://clawhub.com |
| REF-15 | awesome-openclaw-skills | https://github.com/VoltAgent/awesome-openclaw-skills |
| REF-17 | OpenClaw解説（Medium/Milvus） | https://milvusio.medium.com/... |
| REF-18 | 15 Powerful OpenClaw Use Cases | https://kanerika.com/blogs/openclaw-usecases/ |
| REF-19 | 25+ OpenClaw Use Cases | https://forwardfuture.ai/p/... |
| REF-20 | OpenClaw for Small Business | https://aisoftwaresystems.com/blog/openclaw-101... |
| REF-21 | OpenClaw Business Use Cases | https://contabo.com/blog/openclaw-use-cases... |
| REF-22 | 25 OpenClaw Use Cases | https://www.hostinger.com/tutorials/openclaw-use-cases |
| REF-23 | Moltbot on Emergent | https://emergent.sh/tutorial/moltbot-on-emergent |
| REF-24 | Wikipedia: OpenClaw | https://en.wikipedia.org/wiki/OpenClaw |
| REF-26 | AWS Lightsail | https://docs.aws.amazon.com/lightsail/ |
| REF-27 | Let's Encrypt | https://letsencrypt.org/docs/ |
| REF-29 | Node.js | https://nodejs.org/en/download/ |
| REF-30 | Homebrew | https://brew.sh |

完全な参考文献一覧は `references/REFERENCES.md` を参照のこと。
