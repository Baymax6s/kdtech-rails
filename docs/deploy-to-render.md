# Render へのデプロイ手順

## 概要

本番環境は [Render](https://render.com) を使用しています。
開発環境は SQLite3 ですが、Renderのファイルシステムは**デプロイのたびにリセット**されるため、
本番環境では PostgreSQL を使います。

---

## 事前準備

### 1. Render アカウントの作成

[https://render.com](https://render.com) でアカウントを作成し、
GitHub アカウントと連携しておいてください。

### 2. Gemfile に pg gem を追加（初回のみ）

`pg` は PostgreSQL 用の Ruby ドライバです。本番環境でのみ使用します。

```ruby
# Gemfile
gem "sqlite3", ">= 2.1"  # 開発・テスト用（既存）

group :production do
  gem "pg"               # 本番用（追加）
end
```

追加後：
```bash
bundle install
```

### 3. database.yml の本番設定を変更（初回のみ）

`config/database.yml` の `production:` セクションを以下に変更します。
`DATABASE_URL` 環境変数（Render が自動で設定）を読み込む設定です。

```yaml
production:
  primary:
    url: <%= ENV["DATABASE_URL"] %>
```

> **なぜ DATABASE_URL か**
> Render は PostgreSQL を作成すると接続情報を `DATABASE_URL` という環境変数に自動でセットします。
> コードにDB接続情報を直書きせず環境変数経由にすることで、
> 接続情報をGitに含めずに済みます（セキュリティ上の基本原則）。

---

## デプロイ手順

### Step 1: PostgreSQL データベースを作成する

1. Render ダッシュボードで **New > PostgreSQL** を選択
2. 以下を設定：
   - **Name**: `kdtech-rails-db`（任意）
   - **Region**: `Singapore`（日本から近い）
   - **Plan**: `Free`
3. **Create Database** をクリック
4. 作成後、**Internal Database URL** をコピーしておく

### Step 2: Web Service を作成する

1. Render ダッシュボードで **New > Web Service** を選択
2. GitHub リポジトリ（`kdtech-rails`）を選択
3. 以下を設定：

| 項目 | 値 |
|---|---|
| Name | `kdtech-rails`（任意） |
| Region | `Singapore` |
| Branch | `main` |
| Runtime | `Ruby` |
| Build Command | `bin/render-build.sh` |
| Start Command | `bin/rails server -p $PORT` |
| Plan | `Free` |

### Step 3: 環境変数を設定する

Web Service の **Environment** タブで以下を追加します。

| キー | 値 | 説明 |
|---|---|---|
| `DATABASE_URL` | Step 1 でコピーした Internal Database URL | PostgreSQL 接続情報 |
| `RAILS_MASTER_KEY` | `config/master.key` の中身 | 暗号化キー |
| `RAILS_ENV` | `production` | Rails の動作環境 |

> **RAILS_MASTER_KEY とは**
> `config/credentials.yml.enc` を復号するためのキーです。
> `config/master.key` ファイルの中身をそのままコピーしてください。
> このファイルは `.gitignore` に含まれており、**Git にはコミットされません**。
> そのため Render に直接環境変数として渡す必要があります。

### Step 4: デプロイ

設定を保存すると自動的に初回デプロイが始まります。
ログを確認しながら完了を待ちます。

---

## bin/render-build.sh の中身

デプロイ時に Render が実行するビルドスクリプトです。

```bash
bundle install          # Gem をインストール
yarn install            # npm パッケージをインストール
bin/rails assets:precompile  # CSS/JS を本番用にビルド
bin/rails assets:clean       # 古いビルドファイルを削除
bin/rails db:migrate         # DB マイグレーションを実行
```

---

## 開発・本番のデータベース切り替えの仕組み

| 環境 | DB | 切り替え方法 |
|---|---|---|
| 開発（ローカル） | SQLite3 | `config/database.yml` の `development:` |
| 本番（Render） | PostgreSQL | `DATABASE_URL` 環境変数を参照 |

`DATABASE_URL` がセットされている環境（Render）では自動的に PostgreSQL が使われます。
ローカルでは `DATABASE_URL` をセットしないため SQLite3 が使われます。

---

## デプロイ後の確認

```
https://<サービス名>.onrender.com
```

Render ダッシュボードの **Logs** タブでエラーを確認できます。
