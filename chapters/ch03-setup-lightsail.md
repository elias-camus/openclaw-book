# 第3章　環境構築 ― AWS Lightsail編

**この章で学ぶこと:**
- AWS LightsailにUbuntuインスタンスを立ててOpenClawを動かす方法
- systemdによる自動起動設定
- NginxとLet's EncryptでHTTPS化する手順

**所要時間:** 約60〜90分

---

## 3-1. AWS Lightsailとは

AWS Lightsail（ライトセイル）は、Amazon Web Servicesが提供するシンプルなVPS（仮想プライベートサーバー）サービスだ。「クラウドを使いたいが、EC2の設定は難しい」という人向けに、月額固定料金でサーバーを借りられる。[REF-26]

### 料金プラン（執筆時点での概算）

| プラン | 月額（目安） | メモリ | vCPU | SSD | 転送量 |
|---|---|---|---|---|---|
| Nano | $5 | 512MB | 2 | 20GB | 1TB |
| Micro | $10 | 1GB | 2 | 40GB | 2TB |
| Small | $20 | 2GB | 2 | 60GB | 3TB |

**OpenClaw用途には$10のMicroプランが最もバランスが良い。** $5のNanoプランでも動作はするが、複数チャンネルや重いタスクを走らせると不安定になる場合がある。

> 💡 **ポイント:** 料金は為替や改定によって変わるため、AWSの公式ページで最新情報を確認すること。執筆時点では月$10前後が一般的な目安だ。

---

## 3-2. AWSアカウントの作成

既にAWSアカウントを持っている場合はこの節を飛ばしてよい。

1. https://aws.amazon.com/jp/ にアクセスし、「AWSアカウントを作成」をクリック
2. メールアドレス・パスワード・アカウント名を入力する
3. 連絡先情報を入力する（個人利用の場合は「個人」を選択）
4. クレジットカード情報を入力する（無料枠あり、Lightsailは別途課金）
5. 本人確認（SMSまたは音声通話）を完了する

[📸 スクリーンショット: AWSアカウント作成画面のトップページ]

[📸 スクリーンショット: AWS管理コンソールのホーム画面]

---

## 3-3. Lightsailインスタンスの起動

1. AWS管理コンソールの検索バーに「Lightsail」と入力し、サービスを開く
2. 「インスタンスの作成」ボタンをクリックする

[📸 スクリーンショット: Lightsailのトップページ。「インスタンスの作成」ボタンが見える]

3. **インスタンスのロケーション:** 「東京」リージョンを選ぶ（日本から使う場合は遅延が少ない）
4. **プラットフォーム:** 「Linux/Unix」を選択
5. **ブループリント:** 「オペレーティングシステム(OS のみ)」→「Ubuntu 22.04 LTS」を選択

[📸 スクリーンショット: Lightsailインスタンス作成画面。UbuntuとOSのみを選択した状態]

6. **インスタンスプラン:** $10/月（Microプラン推奨）を選択
7. **インスタンス名:** 好きな名前をつける（例: `openclaw-server`）
8. 「インスタンスを作成」をクリック

数十秒〜1分ほどで「実行中」になる。

[📸 スクリーンショット: Lightsailのインスタンス一覧。作成したインスタンスが「実行中」になっている]

---

## 3-4. 静的IPの設定とSSH接続

### 静的IPの割り当て

デフォルトではインスタンスを再起動するたびにIPアドレスが変わる。静的IP（固定IP）を設定しておこう。

1. Lightsailのメニューから「ネットワーキング」を選ぶ
2. 「静的 IP の作成」をクリック
3. 先ほど作成したインスタンスに紐付ける

[📸 スクリーンショット: Lightsailのネットワーキング画面。静的IPをインスタンスにアタッチしている]

> 💡 **ポイント:** 静的IPはインスタンスに割り当てている間は無料。インスタンスを削除して静的IPだけ残すと課金される。不要になったら一緒に削除しよう。

### SSH接続

Lightsailのコンソールから直接ブラウザでSSH接続できる。インスタンスをクリック→「SSH を使用して接続する」ボタンを押すだけだ。

[📸 スクリーンショット: LightsailのSSHブラウザコンソール画面]

ローカルのターミナルからSSH接続する場合は、Lightsailのキーペア（.pemファイル）を使う:

```bash
$ chmod 400 ~/Downloads/openclaw-server.pem
$ ssh -i ~/Downloads/openclaw-server.pem ubuntu@<静的IPアドレス>
```

---

## 3-5. Ubuntuの初期設定

### パッケージの更新

サーバーに接続したら、まずパッケージを最新の状態にする:

```bash
$ sudo apt update && sudo apt upgrade -y
```

### ファイアウォール（ufw）の設定

UFW（Uncomplicated Firewall）を使って必要なポートだけを開ける:

```bash
# UFWを有効化
$ sudo ufw enable

# SSHを許可（これを忘れると締め出される！）
$ sudo ufw allow ssh

# OpenClawのポートを許可
$ sudo ufw allow 18789/tcp

# HTTPとHTTPSを許可（Nginx経由でアクセスする場合）
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp

# 設定確認
$ sudo ufw status
```

> ⚠️ **注意:** `sudo ufw enable`の前に必ず`sudo ufw allow ssh`を実行すること。順序を間違えるとSSH接続ができなくなる。

### タイムゾーンの設定

```bash
$ sudo timedatectl set-timezone Asia/Tokyo
$ timedatectl
```

---

## 3-6. Node.jsとOpenClawのインストール

### nvmのインストール

```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
$ source ~/.bashrc
```

### Node.js v22のインストール

```bash
$ nvm install 22
$ nvm use 22
$ nvm alias default 22

# 確認
$ node --version
$ npm --version
```

### OpenClawのインストール

```bash
$ npm install -g openclaw@latest
$ openclaw --version
```

### 初期設定

```bash
$ openclaw onboard
```

APIキーとエージェントの基本設定を対話形式で行う。

---

## 3-7. systemdによる常時起動設定

Ubuntuでは、systemdを使ってOpenClawをサービスとして登録できる。これでサーバー再起動後も自動的にOpenClawが立ち上がる。

### サービスファイルの作成

```bash
$ sudo nano /etc/systemd/system/openclaw.service
```

以下の内容を入力する（`ubuntu`の部分は実際のユーザー名に合わせる）:

```ini
[Unit]
Description=OpenClaw AI Agent Gateway
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/.nvm/versions/node/v22.13.0/bin/openclaw gateway
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=openclaw
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

> 💡 **ポイント:** `ExecStart`のパスはNode.jsのバージョンによって変わる。`which openclaw`コマンドで実際のパスを確認してから設定しよう。

### サービスの有効化と起動

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable openclaw
$ sudo systemctl start openclaw

# 状態確認
$ sudo systemctl status openclaw
```

`Active: active (running)`と表示されれば成功だ。

[📸 スクリーンショット: sudo systemctl status openclawの出力。active (running)と表示されている]

### ログの確認

```bash
$ sudo journalctl -u openclaw -f
```

---

## 3-8. ドメインとHTTPS設定

ブラウザのアドレスバーに毎回IPアドレスを入力するのは不便だ。独自ドメインを取得し、HTTPS化しておくと快適に使える。

### ドメインの設定

独自ドメイン（例: `myagent.example.com`）を取得したら、DNSのAレコードをLightsailの静的IPに向ける。DNSの反映には数分〜数時間かかる場合がある。

### Nginxのインストール

NginxをリバースプロキシとしてHTTPS接続を受け付け、OpenClaw（ポート18789）に転送する。

```bash
$ sudo apt install nginx -y
```

### Nginx設定ファイルの作成

```bash
$ sudo nano /etc/nginx/sites-available/openclaw
```

以下の内容を入力（`myagent.example.com`は実際のドメインに変更）:

```nginx
server {
    listen 80;
    server_name myagent.example.com;

    location / {
        proxy_pass http://localhost:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}
```

設定を有効化:

```bash
$ sudo ln -s /etc/nginx/sites-available/openclaw /etc/nginx/sites-enabled/
$ sudo nginx -t
$ sudo systemctl reload nginx
```

### Let's EncryptでHTTPS化

Certbotを使って無料のSSL証明書を取得する。[REF-27]

```bash
$ sudo apt install certbot python3-certbot-nginx -y
$ sudo certbot --nginx -d myagent.example.com
```

メールアドレスの入力と利用規約への同意を求められる。完了すると自動的にNginxの設定がHTTPS対応に更新される。

[📸 スクリーンショット: certbotの実行結果。「Congratulations! Your certificate and chain have been saved」と表示されている]

証明書は90日で期限切れになるが、Certbotが自動更新を設定してくれる:

```bash
$ sudo systemctl status certbot.timer
```

---

## 3-9. Mac Mini vs Lightsail 使い分けガイド

| シチュエーション | おすすめ |
|---|---|
| 既にMac Miniを持っている | Mac Mini |
| データを完全に手元で管理したい | Mac Mini |
| 停電・ネット障害が心配 | Lightsail |
| 外出先からも安定してアクセスしたい | Lightsail |
| 複数人で使うチームBotを立てたい | Lightsail |
| 初期費用をゼロに抑えたい | Lightsail |
| ローカルAIモデル（Ollama）も動かしたい | Mac Mini（性能が高いため）|

どちらを選んでも、OpenClaw本体の使い方は全く同じだ。この章以降の解説はMac Mini・Lightsailどちらにも共通して適用できる。

---

### 📦 コラム: Lightsailの注意点——バックアップとスナップショット

Lightsailインスタンスは突然の障害でデータが失われる可能性がゼロではない。OpenClawの設定ファイルやメモリファイルは定期的にバックアップしておこう。

Lightsailには「スナップショット」機能がある。コンソールから数クリックでインスタンス全体のバックアップが取れる。週1回程度のスナップショット取得を習慣にしておくと安心だ。

スナップショットは保存期間中、追加料金がかかる（執筆時点では月$0.05/GB程度）。古いスナップショットは定期的に削除しよう。

---

## まとめ

- AWS LightsailにUbuntuインスタンスを立て、OpenClawをインストールした
- systemdにサービス登録することで、サーバー再起動後も自動起動する
- NginxとLet's Encryptを組み合わせることで、独自ドメインでHTTPSアクセスできる
- Mac MiniとLightsailはどちらでもOpenClawを同等に使える

次の第4章では、DiscordやTelegramなどのメッセージングアプリとOpenClawを接続する方法を解説する。

---

**参考リンク:**
- [REF-01] OpenClaw公式ドキュメント: https://docs.openclaw.ai
- [REF-26] AWS Lightsail公式: https://docs.aws.amazon.com/lightsail/
- [REF-27] Let's Encrypt公式: https://letsencrypt.org/docs/
- [REF-28] Nginx公式: https://nginx.org/en/docs/
