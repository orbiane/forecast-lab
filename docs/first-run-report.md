# 初回の試運転の報告（途中経過）

- 状態: **手順1で止まっています。** GitHub の Secrets に鍵が1つも登録されていないため、試運転（手順2）以降に進めていません
- 作業ブランチ: `setup/first-run`（`main` から作成）
- 日付: 2026-11-17

## したこと

1. `orbiane/forecast-lab` を `~/forecast-lab` にクローンした（フォルダは空だった）
2. `CLAUDE.md`、`SETUP.md`、`docs/sources.md`、ADR-001、ADR-002 を読んだ。あわせて、`uv.lock` が固定している `forecasting-tools` 0.3.5 の配布物（PyPI の wheel）を取り寄せ、中身を読んだ（実行はしていない）
3. 作業ブランチ `setup/first-run` を作った
4. このリポジトリの中だけで、コミットのメールを GitHub の noreply アドレスにした

```
$ git config --local user.email "$(gh api user --jq '"\(.id)+\(.login)@users.noreply.github.com"')"
$ git var GIT_AUTHOR_IDENT   # メール部分だけ抜き出して表示
email=<250697680+orbiane@users.noreply.github.com>
```

## 手順1: Secrets の確認 → 足りない

```
$ gh secret list -R orbiane/forecast-lab
（出力なし。exit=0）
$ gh variable list -R orbiane/forecast-lab
（出力なし。exit=0。BOT_ENABLED も未設定＝本番は止まっている）
```

`gh auth status` は `orbiane` でログイン済みなので、権限の問題ではなく、Secrets がまだ無いと判断した。

### 持ち主にお願いしたいこと（ブラウザだけで済みます）

`SETUP.md` の手順1〜4です。鍵の値はチャットやターミナルに貼らず、GitHub の画面に直接入れてください。

1. Metaculus: https://www.metaculus.com/futureeval/participate/ → ログイン → ボットを作る → 表示されたトークンをコピー
2. 参加フォーム: https://forms.gle/aQdYMq9Pisrf1v7d8 → 必須3問に答え、クレジットを申請
3. OpenRouter: https://openrouter.ai/keys → `Create Key` → キーをコピー（クレジットを待たないなら少額を入金）
4. GitHub: https://github.com/orbiane/forecast-lab → 上の `Settings` → 左の `Secrets and variables` → `Actions` → `Secrets` タブ → `New repository secret`
   - Name に `METACULUS_TOKEN`、Secret に手順1のトークン → `Add secret`
   - もう一度 `New repository secret` → Name に `OPENROUTER_API_KEY`、Secret に手順3のキー → `Add secret`
   - 名前は大文字・アンダースコアまで正確に。`Variables` タブの `BOT_ENABLED` はまだ作らないでください（試運転のあとでお聞きします）

済んだら「Secrets を入れた」とだけ伝えてください。続き（試運転→測定→文書の修正→本番の可否の質問）を進めます。

## 食い違った点・作業の前に指摘したいこと

1. **使われるモデルが文書の想定と違う見込み（重要）。** `docs/sources.md` は、`OPENROUTER_API_KEY` だけのときの既定を「予測 `claude-opus-5.5`、要約・読み取り `claude-haiku-5.5`、調査 `perplexity/sonar-pro`」としている（0.3.7 を読んだ結果）。しかし `uv.lock` が固定している 0.3.5 の `forecast_bot.py` 949行目〜の `_llm_config_defaults` では、`OPENROUTER_API_KEY` のときは次のとおり（`main.py` は `llms=` を指定していないので、この既定が使われるはず）
   - 予測役: `openrouter/openai/gpt-4o`
   - 要約役・読み取り役: `openrouter/openai/gpt-4o-mini`
   - 調査役: `openrouter/openai/gpt-4o-search-preview`
   - 実際に使われたモデルは試運転のログ（`is being served by provider` の行。0.3.5 の `general_llm.py` 426行目が出す）で確かめる。`docs/sources.md` の該当行はまだ書き換えていない（KICKOFF の手順4は、試運転で確かめたものだけを書き換える決まりのため）
   - これが本当なら、`docs/sources.md` の費用の推測（Opus 5.5 の価格で1問約0.4ドル）は前提が違う。モデルの選択は設計判断なので、持ち主に決めてもらう（下の「決めてほしいこと」）
2. **ログの検査語の出どころ。** KICKOFF の grep 対象のうち、`Reasoning for URL`・`with prediction`・`Found Research` は `main.py` の `logger.info`（`HIDE_FORECASTS_IN_LOGS=true` で止まる）、`First Rationale` は 0.3.5 の `log_report_summary`（811行目。隠す設定のときは呼ばれない）。`Total cost estimated` は `bot_helpers.py` 182行目、`Bot submitted` は `bot_helpers.py` 125行目の表示から出る。なので「0件」は意味のある検査になる。ただし `main.py` が止めるのは自分のロガーだけで、`forecasting_tools` や `litellm` のロガーは INFO のまま。ここから LLM の出力が漏れないかは、実際のログで確かめる（ADR-002 の「まだ怪しい点」と同じ）

## まだ怪しい点

- 上の1（モデルの食い違い）は、ソースを読んだだけで、実行では確かめていない
- `GITHUB_TOKEN` の push が60日ルールの「動き」に数えられるか（ADR-002 の要確認のまま）

## 持ち主に決めてほしいこと

- 今は Secrets の登録だけお願いします
- 試運転でモデルが `gpt-4o` 系だと確かめられたら、そのまま基準版として進めるか（ADR-001 は「見本をそのまま使う」）、文書が想定していた Claude 系に固定するか（`main.py` の変更になるので、別の作業で）を決めてください
