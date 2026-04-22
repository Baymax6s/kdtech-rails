# kdtech-rails

Ruby on Rails + daisyUI で構築したフルスタックWebアプリケーション。

## 技術スタック

| カテゴリ | 技術 |
|---|---|
| バックエンド | Ruby 3.3.11 / Rails 8.1.3 |
| フロントエンド | Tailwind CSS + daisyUI |
| JavaScript | esbuild + Hotwire (Turbo / Stimulus) |
| データベース | SQLite3 (開発) / PostgreSQL (本番) |
| 本番環境 | Render |
| バージョン管理 | mise |

---

## mise とは

mise はRuby・Node.jsなど複数の言語ランタイムをバージョン管理するツールです。
`rbenv` や `nvm` を一本化したようなイメージです。

プロジェクトルートの `mise.toml` にバージョンが記載されており、
`mise install` を実行するとチーム全員が同じバージョンを使えます。

```toml
# mise.toml（プロジェクトで使うバージョンを固定）
[tools]
ruby = "3.3.11"
node = "24.15.0"
```

---

## 前提条件

### 1. mise のインストール

**macOS**
```bash
brew install mise
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc && source ~/.zshrc
```

**Linux / Windows (WSL)**
```bash
curl https://mise.run | sh
echo 'eval "$(mise activate bash)"' >> ~/.bashrc && source ~/.bashrc
```

インストール確認：
```bash
mise --version
# => mise 2024.x.x
```

### 2. Ruby / Node.js のインストール

リポジトリをクローン後、以下を実行するとmiseが `mise.toml` を読んで自動でバージョンを揃えます。

```bash
mise install
```

バージョン確認：
```bash
ruby --version   # => ruby 3.3.11
node --version   # => v24.15.0
```

**参照先の確認（重要）**

`which` コマンドでシステムのRuby/Node.jsではなく、miseが管理するものを参照しているか確認してください。

```bash
which ruby
# => /Users/<yourname>/.local/share/mise/installs/ruby/3.3.11/bin/ruby
#    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ miseのパスが出ればOK

which node
# => /Users/<yourname>/.local/share/mise/installs/node/24.15.0/bin/node
```

`/usr/bin/ruby` のようなシステムパスが出た場合はmiseのシェル設定が正しく読み込まれていません。
シェルの設定ファイル（`~/.zshrc` または `~/.bashrc`）に `eval "$(mise activate ...)"` が追記されているか確認してください。

### 3. Rails のインストール

mise管理のRubyに対してgemをインストールします（`sudo` は不要・使ってはいけません）。

```bash
gem install rails
mise reshim   # 新しいコマンドをmiseに認識させる

# シェルのコマンドキャッシュをクリアする（いずれか一つ）
source ~/.zshrc     # zsh の場合
source ~/.bashrc    # bash の場合
# または新しいターミナルを開く

rails --version  # => Rails 8.1.3
```

> **なぜ source が必要なのか**
> シェルは一度実行したコマンドのパスをキャッシュします。
> `mise reshim` でshimを作成しても、シェルが古いキャッシュを参照し続けるため
> `rails` が見つからないことがあります。
> `source ~/.zshrc` でキャッシュをリセットすることで新しいshimが認識されます。

参照先の確認：
```bash
which rails
# => /Users/<yourname>/.local/share/mise/shims/rails
#    miseのshim経由になっていればOK
```

> **miseのshimとは**
> miseはコマンドを直接実行せず、shim（中継ファイル）を経由してバージョンを切り替えます。
> gemで新しいコマンド（`rails` など）をインストールした後は `mise reshim` を実行して
> shimを更新する必要があります。

### 4. yarn のインストール

```bash
npm install -g yarn
mise reshim
source ~/.zshrc   # zsh の場合 / bash の場合は source ~/.bashrc
yarn --version   # => 1.22.x
```

参照先の確認：
```bash
which yarn
# => /Users/<yourname>/.local/share/mise/installs/node/24.15.0/bin/yarn
```

### 5. SQLite3

| OS | 対応 |
|---|---|
| macOS | プリインストール済み（対応不要） |
| Linux | `sudo apt install sqlite3 libsqlite3-dev` |
| Windows (WSL) | `sudo apt install sqlite3 libsqlite3-dev` |

確認コマンド：
```bash
sqlite3 --version
```

---

## セットアップ

```bash
# 1. リポジトリのクローン
git clone https://github.com/junhat6/kdtech-rails.git
cd kdtech-rails

# 2. mise でRuby/Node.jsバージョンを揃える
mise install

# 3. RailsのGemをインストール
bundle install

# 4. npmパッケージをインストール
yarn install

# 5. データベースの作成・マイグレーション
bin/rails db:create db:migrate
```

---

## 開発サーバーの起動

```bash
bin/dev
```

`bin/dev` は以下の3プロセスを同時に起動します。

| プロセス | 役割 |
|---|---|
| web | Rails サーバー（http://localhost:3000） |
| js | esbuild による JS の監視ビルド |
| css | PostCSS による CSS の監視ビルド |

JS・CSSの変更がブラウザにリアルタイム反映されるため、3つ同時起動が必要な構成です。

詳しい理由は [docs/why-bin-dev.md](docs/why-bin-dev.md) を参照してください。

ポートが競合する場合（他のRailsアプリを同時に起動しているときなど）：
```bash
bin/dev -p 3001
```

---

## テスト

```bash
bundle exec rspec
```

---

## デプロイ（Render）

本番環境はRenderを使用しています。
データベースはRenderのPostgreSQLサービスを使用します。
`DATABASE_URL` 環境変数で本番DBに接続します。

詳しい手順は [docs/deploy-to-render.md](docs/deploy-to-render.md) を参照してください。

```
wsl.exe --install Ubuntu-24.04
# 再起動

sudo apt update -y && sudo apt install -y curl
sudo install -dm 755 /etc/apt/keyrings
curl -fSs https://mise.jdx.dev/gpg-key.pub | sudo tee /etc/apt/keyrings/mise-archive-keyring.asc 1> /dev/null
echo "deb [signed-by=/etc/apt/keyrings/mise-archive-keyring.asc] https://mise.jdx.dev/deb stable main" | sudo tee /etc/apt/sources.list.d/mise.list
sudo apt update -y
sudo apt install -y mise

echo 'eval "$(mise activate bash)"' >> ~/.bashrc

sudo apt update
sudo apt install build-essential rustc libssl-dev libyaml-dev zlib1g-dev libgmp-dev


リポジトリのクローン
そこに移動
vscodeならwslでターミナルを起動する

Name: WSL
Id: ms-vscode-remote.remote-wsl
Description: Open any folder in the Windows Subsystem for Linux (WSL) and take advantage of Visual Studio Code's full feature set.
Version: 0.104.3
Publisher: Microsoft
VS Marketplace Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl

というvscode extensionを入れる
```