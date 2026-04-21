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

## 前提条件

### 1. mise のインストール

mise はRuby・Node.jsのバージョンを一元管理するツールです。

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

### 2. Ruby / Node.js のインストール

リポジトリをクローン後、以下を実行するとmiseが自動でバージョンを揃えます。

```bash
mise install
```

確認コマンド：
```bash
ruby --version   # => ruby 3.3.11
node --version   # => v24.15.0
```

### 3. Rails のインストール

```bash
gem install rails
mise reshim
rails --version  # => Rails 8.1.3
```

### 4. yarn のインストール

```bash
npm install -g yarn
mise reshim
yarn --version   # => 1.22.x
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

ポートが競合する場合：
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
