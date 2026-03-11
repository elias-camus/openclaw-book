# 第5章　AIモデルの設定

**この章で学ぶこと:**
- OpenClawが対応するAIモデルの種類と特徴
- OpenAI・Anthropic・Amazon Bedrockの接続設定
- ローカルモデル（Ollama）でプライバシーを守る方法
- 用途・コストに応じたモデルの選び方

**所要時間:** 約30〜45分

---

## 5-1. OpenClawが対応するモデル一覧

OpenClawはAIモデルそのものは内蔵していない。外部のAIプロバイダーのAPIを呼び出す「橋渡し役」として機能する。そのため、どのAIプロバイダーのモデルを使うかは自分で選択できる。

執筆時点で対応している主なプロバイダーとモデル:

| プロバイダー | 主なモデル | 特徴 |
|---|---|---|
| Anthropic | Claude 3.5 Sonnet / Claude 3 Opus | 指示追従性が高く、長文処理が得意 |
| OpenAI | GPT-4o / GPT-4 Turbo / o3 | 最も広く使われる。マルチモーダル対応 |
| Amazon Bedrock | 上記を含む複数モデル | AWSユーザー向け。IAM認証で管理 |
| Google | Gemini 1.5 Pro / Flash | 大きなコンテキストウィンドウ |
| DeepSeek | DeepSeek-V3 / R1 | コスト効率が高い。中国発のモデル |
| Ollama（ローカル） | Llama 3 / Mistral / Gemma 等 | API費用ゼロ。完全プライベート |

---

## 5-2. OpenAI（ChatGPT）の設定

### APIキーの取得

1. https://platform.openai.com にアクセスし、アカウントを作成（またはログイン）
2. 右上のアカウントメニュー→「API keys」→「Create new secret key」

[📸 スクリーンショット: OpenAI PlatformのAPI Keysページ。新しいキーを作成している]

3. 発行されたキーをコピーする（`sk-` で始まる文字列）

> ⚠️ **注意:** APIキーは一度しか表示されない。必ずコピーして安全な場所に保管しよう。

### OpenClawへの設定

`~/.openclaw/openclaw.json` を編集:

```json
{
  "model": "openai/gpt-4o",
  "providers": {
    "openai": {
      "apiKey": "sk-YOUR_API_KEY_HERE"
    }
  }
}
```

または環境変数で渡す方法:

```bash
$ export OPENAI_API_KEY="sk-YOUR_API_KEY_HERE"
$ openclaw gateway
```

---

## 5-3. Anthropic（Claude）の設定

### APIキーの取得

1. https://console.anthropic.com にアクセスし、アカウントを作成
2. 「API Keys」→「Create Key」

```json
{
  "model": "anthropic/claude-3-5-sonnet-20241022",
  "providers": {
    "anthropic": {
      "apiKey": "sk-ant-YOUR_API_KEY_HERE"
    }
  }
}
```

---

## 5-4. Amazon Bedrock経由での設定

Amazon BedrockはAWSが提供するAIモデルの統合プラットフォームだ。ClaudeやLlamaなど複数のモデルをAWSアカウントで一括管理できる。AWSをすでに使っている企業ユーザーに向いている。

### IAM設定

AWSコンソールでBedrockへのアクセス権限を持つIAMユーザーまたはロールを作成し、認証情報（Access Key ID / Secret Access Key）を取得する。

```json
{
  "model": "amazon-bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0",
  "providers": {
    "amazon-bedrock": {
      "region": "us-east-1",
      "accessKeyId": "YOUR_AWS_ACCESS_KEY",
      "secretAccessKey": "YOUR_AWS_SECRET_KEY"
    }
  }
}
```

---

## 5-5. ローカルモデル（Ollama）との連携

Ollamaは、オープンソースのLLM（大規模言語モデル）をローカルで動かすためのツールだ。API費用がかからず、データが外部に出ないため、プライバシーを重視する場合に最適だ。

### Ollamaのインストール（Mac）

```bash
$ brew install ollama
$ ollama serve
```

### モデルのダウンロードと起動

```bash
# Llama 3（8B）をダウンロード
$ ollama pull llama3

# 動作確認
$ ollama run llama3
```

### OpenClawとの連携

```json
{
  "model": "ollama/llama3",
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434"
    }
  }
}
```

> 💡 **ポイント:** Ollamaは高性能なGPUがなくても動くが、Mac Mini M4のような高性能CPUがあると応答速度が大幅に上がる。小さいモデル（7B〜8B）ならMac Miniでも実用的な速度で動く。

---

## 5-6. モデルのコストと使い分け

APIを使う場合、費用はトークン単位で発生する。「トークン（token）」とはテキストを分割した単位で、日本語では1文字が1〜2トークン程度になる。

### 概算コスト（執筆時点・目安）

| モデル | 入力 (per 1M tokens) | 出力 (per 1M tokens) | 特徴 |
|---|---|---|---|
| GPT-4o | $2.50 | $10.00 | バランス型 |
| Claude 3.5 Sonnet | $3.00 | $15.00 | 高品質・指示追従性高 |
| DeepSeek-V3 | $0.27 | $1.10 | コスパ最強クラス |
| Gemini 1.5 Flash | $0.075 | $0.30 | 超低コスト |
| Ollama（ローカル）| $0 | $0 | 完全無料 |

> ⚠️ **注意:** 料金は頻繁に改定される。最新価格は各プロバイダーの公式サイトで確認すること。

### 個人利用の場合の目安

軽い日常利用（1日数十回の短い会話程度）なら月1,000〜3,000円で収まることが多い。Cronジョブで大量のタスクを自動処理する場合はコストが増える。まずはGPT-4o miniやGemini Flashで試してから、必要に応じてグレードアップするのが賢明だ。

---

## 5-7. 用途別モデル選択ガイド

| 用途 | おすすめモデル | 理由 |
|---|---|---|
| 日常会話・質問応答 | GPT-4o mini / Gemini Flash | 安くて速い |
| 長文生成・文章校正 | Claude 3.5 Sonnet | 長文品質が高い |
| コーディング・技術タスク | Claude 3.5 Sonnet / GPT-4o | コード生成精度が高い |
- コスト最優先 | DeepSeek-V3 | 圧倒的低コスト |
| プライバシー重視 | Ollama（ローカル） | データが外部に出ない |
| 企業・チーム利用 | Amazon Bedrock | AWS請求に一元化 |

### モデルフェイルオーバー

OpenClawはモデルの切り替え（フェイルオーバー）機能を持っている。メインモデルがエラーになった場合に自動でバックアップモデルに切り替えられる:

```json
{
  "model": "anthropic/claude-3-5-sonnet-20241022",
  "modelFallback": "openai/gpt-4o"
}
```

---

### 📦 コラム: DeepSeekとOpenClaw——中国での広がり

2026年初頭、OpenClawの爆発的な普及は中国でも起きた。中国のユーザーたちはDeepSeekモデルをOpenClawと組み合わせ、WeChatやFeishuなど国内の主要メッセージングアプリと連携するカスタム設定を公開した。[REF-24]

DeepSeekは性能面でGPT-4に匹敵しながら、価格が数分の一という破格のコストパフォーマンスを誇る。オープンソースのAIフレームワーク（OpenClaw）と低コストのオープンソースモデル（DeepSeek）の組み合わせが、AIエージェントの民主化を加速させている。

---

### 📦 コラム: APIキーの安全な管理

APIキーは「お金になる情報」だ。流出すると第三者に悪用され、多額の請求が来る可能性がある。基本的なルールを守ろう:

1. **Gitリポジトリに含めない** — `.gitignore`に`openclaw.json`を追加
2. **環境変数を使う** — ファイルに直書きせず`OPENAI_API_KEY`等の環境変数として管理
3. **権限を最小限に** — APIキーに使用量上限（Spending Limit）を設定しておく
4. **定期的にローテーション** — 3〜6ヶ月おきにキーを再発行する

---

## まとめ

- OpenClawはAPIキーを設定するだけでClaudeやGPT-4などの主要AIモデルに接続できる
- コスト重視ならDeepSeek・Gemini Flash、品質重視ならClaude 3.5 Sonnet
- プライバシーを最優先したい場合はOllamaでローカルモデルを動かす選択肢もある
- APIキーの管理は慎重に。使用量上限の設定を忘れずに

次章では、OpenClawがどのように「記憶」を持つのかを解説する。

---

**参考リンク:**
- [REF-01] OpenClaw公式ドキュメント: https://docs.openclaw.ai
- [REF-24] Wikipedia: OpenClaw（DeepSeekとの関係）
