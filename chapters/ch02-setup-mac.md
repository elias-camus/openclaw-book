# 第2章　環境構築 ― Mac Mini編

**この章で学ぶこと:**
- Mac Miniを自宅AIサーバーとして設定する方法
- Homebrew・Node.js・OpenClawのインストール手順
- 常時起動（デーモン化）の設定方法

**所要時間:** 約60〜90分

---

## 2-1. Mac Miniを自宅サーバーとして使う理由

「サーバー」と聞くと、企業のデータセンターにある大型機器を想像するかもしれない。しかし個人でOpenClawを動かすには、そんな大袈裟なものは必要ない。手元にあるMac Miniで十分だ。

Mac Miniが自宅サーバーとして優れている理由を整理しておこう。

| 比較軸 | Mac Mini（M4） | AWS Lightsail（月$10プラン）|
|---|---|---|
| 初期費用 | 約92,800円〜 | 0円 |
| 月額費用 | 電気代のみ（約400〜800円/月） | 約1,500円/月 |
| 性能 | 非常に高い（M4チップ） | 標準的（vCPU 2〜4）|
| データの場所 | 完全に手元 | AWSクラウド上 |
| 管理の手間 | macOSなので直感的 | SSH必要 |
| 停電・ネット障害 | リスクあり | 基本なし |

**3年間の総コスト比較（概算）:**
- Mac Mini: 92,800円 + 電気代（約28,800円）= 約121,600円
- Lightsail: 月1,500円 × 36ヶ月 = 54,000円

コストだけで見ればLightsailが有利だが、Mac Miniには「データが完全に手元にある」「性能が圧倒的に高い」「既に持っている場合は追加費用ゼロ」という強みがある。

> 💡 **ポイント:** 既にMac Miniを持っている場合、または「データを自分の手元で管理したい」という方はMac Mini編を選ぼう。クラウド派・外出先からのアクセスを重視する方は第3章（Lightsail編）へ。

---

## 2-2. macOSの初期設定

OpenClawを常時起動させるためには、まずmacOSの設定を見直しておく必要がある。特に重要なのは「スリープの無効化」と「自動ログイン」だ。

### スリープを無効化する

Mac Miniがスリープするとエージェントが止まってしまう。サーバーとして使う場合はスリープを無効にする。

1. Apple メニュー（左上の🍎）→「システム設定」を開く
2. 左メニューから「ロック画面」を選ぶ
3. 「スリープ」の設定を「しない」に変更する

> 📌 **参照:** [Apple サポート — Macのスリープと省エネルギーの設定](https://support.apple.com/ja-jp/guide/mac-help/mchle41a6ccd/mac)
> 画面で確認する内容: 左メニューの「ロック画面」を開くと、「ディスプレイがオフのとき、または画面ロック後にスリープさせない」というトグルが表示されます。これをオンにするか、スリープ開始時間を「しない」に設定します。

4. 同じく「省エネルギー」→「Power Nap」もオフにしておくと安心だ

### リモートログインを有効化する（任意）

外出先からSSH接続でMac Miniを操作したい場合は有効にしておく。

1. 「システム設定」→「一般」→「共有」を開く
2. 「リモートログイン」をオンにする

> 📌 **参照:** [Apple サポート — Macでのリモートログインの設定](https://support.apple.com/ja-jp/guide/mac-help/mchlp1066/mac)
> 画面で確認する内容: 「システム設定」→「一般」→「共有」を開くと、「リモートログイン」という項目があります。トグルをオンにすると、SSH経由での接続が可能になります。有効化後、接続用のSSHコマンド（例: `ssh username@192.168.x.x`）が画面下部に表示されます。

---

## 2-3. Homebrewのインストール

Homebrew（ホームブリュー）は、macOSで使える定番のパッケージ管理ツールだ。「ソフトウェアをコマンド一つでインストール・管理できる仕組み」と理解すれば十分だ。[REF-30]

ターミナルを開く（Spotlight検索で「ターミナル」と入力するか、アプリケーション→ユーティリティ→ターミナル）。

> 画面で確認する内容: ターミナル.appを起動すると、黒または白の背景にコマンド入力欄（プロンプト）が表示されます。通常は `username@hostname ~ %` のような文字列が行頭に表示されています。この状態でコマンドを入力できます。

以下のコマンドをそのままコピー&ペーストして実行する:

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール中にパスワードの入力を求められる。Macのログインパスワードを入力してEnterキーを押す（入力中は画面に何も表示されないが、正常だ）。

> 📌 **参照:** [Homebrew 公式サイト](https://brew.sh)
> 画面で確認する内容: インストールが正常に完了すると、ターミナルに `==> Installation successful!` と表示されます。続けて `Next steps:` という見出しで、PATH設定のための追加コマンドが案内されます。この指示に従ってPATHを設定してください。

インストールが完了したら、以下のコマンドで確認する:

```bash
$ brew --version
```

`Homebrew 4.x.x`のようにバージョンが表示されれば成功だ。

> 💡 **ポイント:** Apple Siliconチップ（M1/M2/M3/M4）のMacでは、Homebrewは `/opt/homebrew/` にインストールされる。インストール完了後に表示される「Next steps」の指示に従ってPATHを設定しておこう。

---

## 2-4. Node.jsのインストール

OpenClawはNode.js上で動作する。ここではnvm（Node Version Manager）を使ってインストールする。nvmを使うと複数バージョンのNode.jsを切り替えられるため、他のプロジェクトとの干渉を防げる。[REF-29]

### nvmのインストール

```bash
$ brew install nvm
```

インストール後、設定ファイルにnvmのパスを追加する。使っているシェルに応じてファイルが異なる（macOS Catalina以降はzshが標準）:

```bash
$ echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
$ echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
$ source ~/.zshrc
```

### Node.js v22のインストール

OpenClawはNode.js v22以上を必要とする。

```bash
$ nvm install 22
$ nvm use 22
$ nvm alias default 22
```

インストールを確認する:

```bash
$ node --version
# v22.x.x と表示されればOK

$ npm --version
# 10.x.x と表示されればOK
```

> 画面で確認する内容: `node --version` を実行すると `v22.x.x`、`npm --version` を実行すると `10.x.x` のようにバージョン番号だけが1行で表示されます。これが確認できればNode.jsのインストールは完了です。

---

## 2-5. OpenClawのインストール

Node.jsが準備できたら、いよいよOpenClaw本体をインストールする。[REF-01]

```bash
$ npm install -g openclaw@latest
```

インストールが完了したら確認:

```bash
$ openclaw --version
```

バージョン番号が表示されれば成功だ。

### 初期設定ウィザード（onboard）

OpenClawには初期設定を対話形式でサポートする`onboard`コマンドがある:

```bash
$ openclaw onboard
```

> 📌 **参照:** [OpenClaw 公式ドキュメント](https://docs.openclaw.ai)
> 画面で確認する内容: `openclaw onboard` を実行すると、ターミナルに対話形式の質問が順番に表示されます。「Select AI provider」「Enter your API key」「Set agent name」といったプロンプトが出るので、矢印キーや入力で回答します。設定が完了すると「Onboarding complete!」のようなメッセージが表示されます。

ウィザードでは以下の設定を行う:

1. **AIモデルの選択:** OpenAI / Anthropic / Amazon Bedrock から選ぶ
2. **APIキーの入力:** 選択したプロバイダーのAPIキーを貼り付ける
3. **エージェント名の設定:** 好きな名前をつけられる（後から変更可能）

> 💡 **ポイント:** APIキーは `~/.openclaw/openclaw.json` に保存される。このファイルをGitリポジトリに含めないよう注意すること。`.gitignore`に追加しておくと安全だ。

---

## 2-6. 初回起動と動作確認

設定が完了したら、Gatewayを起動してみよう。

```bash
$ openclaw gateway
```

> 📌 **参照:** [OpenClaw 公式ドキュメント — Gateway](https://docs.openclaw.ai)
> 画面で確認する内容: `openclaw gateway` を実行すると、`Gateway started` または `Listening on port 18789` のようなメッセージがターミナルに表示されます。エラーが表示されず、カーソルが止まった状態（待機中）になれば起動成功です。

起動後、ブラウザで以下のURLにアクセスするとWeb Control UIが開く:

```
http://localhost:18789
```

> 画面で確認する内容: ブラウザで `http://localhost:18789` を開くと、OpenClawのWeb Control UIが表示されます。画面中央にチャット入力欄があり、上部にセッション名やモデル名が表示されています。入力欄にメッセージを打ってAIが返答すれば、正常に動作しています。

Web Control UIのチャット欄に何か入力してAIが返答すれば、OpenClawは正常に動作している。

---

## 2-7. 常時起動設定（daemonモード）

このままでは、ターミナルを閉じるとGatewayも止まってしまう。Mac Miniを起動したら自動的にOpenClawも起動するよう、daemonとして登録する。

OpenClawには`--install-daemon`オプションが用意されている:

```bash
$ openclaw onboard --install-daemon
```

または個別に登録する場合:

```bash
$ openclaw gateway install-daemon
```

> 📌 **参照:** [OpenClaw 公式ドキュメント](https://docs.openclaw.ai)
> 画面で確認する内容: `openclaw gateway install-daemon` を実行すると、`Daemon installed successfully` または `Service registered` のようなメッセージが表示されます。これでlaunchdへの登録が完了し、Mac再起動後も自動的にGatewayが起動するようになります。

これでmacOSのlaunchd（ローンチディー）というシステムサービス管理機能にOpenClawが登録される。Mac再起動後も自動でGatewayが立ち上がる。

### daemonの確認・管理

```bash
# 状態確認
$ openclaw gateway status

# 停止
$ openclaw gateway stop

# 再起動
$ openclaw gateway restart
```

---

## 2-8. ローカルネットワークからのアクセス設定

同じWi-Fiに繋がっているスマートフォンやノートPCからもWeb Control UIにアクセスしたい場合、Mac MiniのローカルIPアドレスを確認しておこう。

```bash
$ ipconfig getifaddr en0
# 例: 192.168.1.10
```

この場合、同じネットワーク内の他のデバイスから `http://192.168.1.10:18789` でアクセスできる。

> 💡 **ポイント:** インターネット（外部）からのアクセスを許可したい場合は、ルーターのポートフォワーディング設定とHTTPS化が必要になる。セキュリティリスクが増すため、第12章を参照してから慎重に設定しよう。

---

## 2-9. よくあるエラーとトラブルシューティング

### ❌ `npm install -g openclaw` で Permission denied が出る

原因: システムのnpmディレクトリへの書き込み権限がない  
対処: nvmでインストールしたNode.jsを使っていれば発生しないはず。`which node`でパスを確認し、nvm管理のNodeが使われているか確認する:

```bash
$ which node
# /Users/username/.nvm/versions/node/v22.x.x/bin/node と表示されるべき
```

### ❌ `openclaw gateway` でポートが使用中というエラーが出る

```
Error: Port 18789 is already in use
```

対処: 既に別のGatewayプロセスが起動している可能性がある:

```bash
$ lsof -i :18789
$ kill -9 <PID>
```

または別ポートを指定して起動:

```bash
$ openclaw gateway --port 19000
```

### ❌ APIキーが無効と言われる

対処: `~/.openclaw/openclaw.json`を開き、APIキーが正しく入力されているか確認する。スペースや改行が混入していないかチェックしよう。

```bash
$ cat ~/.openclaw/openclaw.json
```

### ❌ Web Control UIが開かない

対処: Gatewayが正常に起動しているか確認:

```bash
$ openclaw gateway status
```

`running`と表示されていれば、ブラウザのキャッシュクリアを試みる。

---

### 📦 コラム: Mac Mini M4 vs クラウド——どっちがお得？

2024年末に登場したMac Mini M4は、コンパクトな筐体に驚異的な性能を詰め込んだマシンだ。OpenClawを動かすだけなら完全にオーバースペックとも言えるが、余ったリソースでローカルAIモデル（Ollamaなど）も同時に動かせるという利点がある。

電力消費はアイドル時で約10W程度と非常に省エネだ。月24時間×30日フル稼働しても電気代は100〜200円程度に収まる計算になる。

クラウドと自宅サーバー、どちらが「お得」かは使い方次第だ。ただ「自分のデータは自分のマシンの中にある」という安心感は、金額では測れないかもしれない。

---

## まとめ

- Mac MiniをOpenClawサーバーとして使うには、スリープ無効化・Homebrew・Node.js・OpenClawの順でセットアップする
- `openclaw onboard`で対話形式の初期設定ができる
- `openclaw gateway`でAIエージェントが起動し、`http://localhost:18789`でWeb UIにアクセスできる
- `--install-daemon`でMac起動時の自動立ち上げを設定できる

第3章では、クラウド（AWS Lightsail）上でOpenClawを動かす方法を解説する。

---

**参考リンク:**
- [REF-01] OpenClaw公式ドキュメント: https://docs.openclaw.ai
- [REF-29] Node.js公式: https://nodejs.org/en/download/
- [REF-30] Homebrew公式: https://brew.sh
