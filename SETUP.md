# 持ち主の手作業（初回だけ）

ボットを動かすのに、持ち主にしかできない作業が5つある。どれもブラウザの画面操作だけで済む。所要はおよそ30分。2のクレジットの返事を待つ時間は含まない。

鍵（トークンやAPIキー）は、GitHub の Secrets 以外の場所（チャット、ファイル、メモ）に貼らない。

## 1. Metaculus のボット用アカウントとトークン

1. https://www.metaculus.com/futureeval/participate/ を開く
2. 人間用のアカウントで登録またはログインする
3. 設定ページに移ったら、ボットを作る（ボットは人間用とは別のアカウントになる）
4. 表示されたアクセストークンをコピーする → 手順3で `METACULUS_TOKEN` として貼る

## 2. 参加フォームとクレジットの申請

1. https://forms.gle/aQdYMq9Pisrf1v7d8 を開く
2. 最初の区切りの必須3問に答える（予測を出す前に全員必須）
3. 同じフォームで、LLM と検索のクレジットを申請する

今季はクレジットの配り方が厳しい。最初は約100ドル分で、追加は MiniBench の成績しだい。

## 3. OpenRouter のキー

見本ボットは、`OPENROUTER_API_KEY` が1つあれば、予測・調査・読み取りのすべてを OpenRouter 経由で動かす。

- クレジットが通ったら、その案内に従う（キーの受け取り方は通知を見て確かめる。`docs/sources.md` では要確認）
- 待たずに始めるなら、https://openrouter.ai/keys で自分のキーを作り、少額を入金する

代わりに `ANTHROPIC_API_KEY` を使うこともできる。ただし、その場合は調査役のモデルが変わり、費用の見積もりも変わる（`docs/sources.md` 参照）。まずは OpenRouter 1本を推す。

## 4. GitHub に鍵を登録する

1. https://github.com/orbiane/forecast-lab を開く
2. 上の `Settings` → 左の `Secrets and variables` → `Actions`
3. `Secrets` タブで `New repository secret` を押し、次の2つを登録する（名前は大文字で正確に）
   - `METACULUS_TOKEN` … 手順1のトークン
   - `OPENROUTER_API_KEY` … 手順3のキー

## 5. 試しに動かして、本番を始める

1. リポジトリ上の `Actions` タブ → 左の `Test Bot` → 右の `Run workflow` → 緑の `Run workflow`
2. 3〜5分で終わる。緑のチェックが付けば成功。赤なら、その実行を開いてログを見るか、Claude にリンクを渡す
3. 予測が届いたかを見る: Metaculus に人間用アカウントでログイン → 右上の名前 → `Settings` → `My Forecasting Bots` → `Switch to bot account` → 練習用の大会（bot-testing-area）の問題を開く。理由は `Private` のコメント欄にある
4. 本番を始める: `Settings` → `Secrets and variables` → `Actions` → `Variables` タブ → `New repository variable` で、名前 `BOT_ENABLED`、値 `true`
5. これで20分おきに、新しい問題に予測を出す

## コミットのメールアドレスを隠す（公開リポジトリなので）

GitHub の右上のアイコン → `Settings` → 左の `Emails` で、次の2つに印を付ける。

- `Keep my email addresses private` … GitHub の画面上の操作で、本物のアドレスを出さない
- `Block command line pushes that expose my email` … Mac から本物のアドレスが載ったコミットを push しようとしたら、GitHub が止める（安全網）

Mac の Claude Code 側では、このリポジトリのコミットに noreply アドレスを使うよう設定させる（`KICKOFF.md` の手順0）。

## 止め方

- `Variables` の `BOT_ENABLED` を `false` に書き換える。次の定時実行から止まる
- すぐに止めたいときは、`Actions` → `Forecast on new AI tournament questions` → 右上の `…` → `Disable workflow`
