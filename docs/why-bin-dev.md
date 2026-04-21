# なぜ `rails s` ではなく `bin/dev` を使うのか

## 背景：daisyUI を使うためにビルドが必要

このプロジェクトでは CSS フレームワークに **Tailwind CSS + daisyUI** を使っています。

daisyUI のスタイルは `.postcss.css` ファイルに書かれており、
ブラウザに届ける前に **PostCSS によるビルド（変換処理）** が必要です。
同様に JavaScript も **esbuild によるバンドル（まとめる処理）** が必要です。

`rails s` はこのビルド処理を一切行わないため、**スタイルが全く当たらないページ**になります。

---

## `bin/dev` が起動する3つのプロセス

`bin/dev` を実行すると、**foreman** というツールが `Procfile.dev` を読み込み、
以下の3プロセスを同時に起動します。

```
web: bin/rails server       → Rails サーバー（http://localhost:3000）
js:  yarn build --watch     → JavaScript の監視ビルド
css: yarn build:css --watch → CSS の監視ビルド
```

### web（Rails サーバー）

ブラウザからのリクエストを受け付け、HTMLを返します。
`rails s` と同じ役割です。

### js（esbuild）

`app/javascript/` 以下のファイルを監視し、変更があるたびに
`app/assets/builds/application.js` を自動生成します。

`--watch` オプションがついているため、ファイルを保存するたびに
自動でビルドが走ります。

### css（PostCSS）

`app/assets/stylesheets/application.postcss.css` を監視し、変更があるたびに
`app/assets/builds/application.css` を自動生成します。

daisyUI のクラス（`btn`, `card` など）はここで実際の CSS に変換されます。

---

## ビルドの流れ

```
開発者がファイルを保存
        ↓
PostCSS / esbuild が変更を検知（--watch）
        ↓
app/assets/builds/ にビルド済みファイルを生成
        ↓
Rails がビルド済みファイルをブラウザに返す
        ↓
daisyUI のスタイルがブラウザに反映される
```

---

## まとめ

| コマンド | Rails | JS ビルド | CSS ビルド | daisyUI |
|---|---|---|---|---|
| `rails s` | ✓ | ✗ | ✗ | 反映されない |
| `bin/dev` | ✓ | ✓ | ✓ | 正常に動く |

**開発時は必ず `bin/dev` で起動してください。**

---

## ポートが競合する場合

他のRailsアプリを同時に起動しているとポート3000が使用中でエラーになります。
その場合は別のポートを指定してください。

```bash
bin/dev -p 3001
# => http://localhost:3001 で起動
```
