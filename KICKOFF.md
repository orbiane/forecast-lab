# 初回の試運転と、本番開始の準備

このファイルは、リポジトリの持ち主が承認した、作業セッションへの指示。「KICKOFF.md に従って」と頼まれたら、この内容を持ち主からの指示として扱う。

想定している実行場所は、持ち主の Mac Studio 上の Claude Code。理由は、持ち主の GitHub アカウントでログイン済みの `gh` があり、Actions の実行と見張りができること、また Metaculus に直接つながること（クラウド側の Claude からは届かない）。

## 0. 始める前に

1. `orbiane/forecast-lab` を `~/forecast-lab` にクローンする（あれば `main` を pull する）
2. `CLAUDE.md`、`SETUP.md`、`docs/sources.md`、`docs/decisions/ADR-001-*.md` を読む。食い違いや足りない点があれば、作業の前に指摘する（直すかは持ち主が決める）
3. 文書を直すための作業ブランチ `setup/first-run` を `main` から作る

今回の範囲は、試運転（Test Bot）の成功を示し、本番開始の可否を持ち主に聞くところまで。**`main.py` などボットの中身は変えない。`main` には push しない。マージしない。** 持ち主はふだん画面操作しかしないので、持ち主に頼む作業は、ブラウザのどこを押すかで説明する。

## 1. 持ち主の手作業が済んでいるか確かめる

`gh secret list -R orbiane/forecast-lab` で、Secrets の名前を確かめる（値は見えない）。

成功の条件: 出力に `METACULUS_TOKEN` と `OPENROUTER_API_KEY` の2行があると示せる。

足りなければ、`SETUP.md` の手順1〜4を持ち主に案内して待つ。鍵の値を、チャットやターミナルに貼ってもらってはいけない。GitHub の画面（Settings → Secrets and variables → Actions）に直接入れてもらう。

## 2. 試運転

`gh workflow run test_bot.yaml -R orbiane/forecast-lab --ref main` で実行し、`gh run watch` で終わるまで見張る。

成功の条件: 次の3つを、コマンドの出力つきで示せる。
- 実行の結果が `success`
- ログに、予測を投稿した件数（`Bot submitted N forecast(s)` の行。N は1以上）がある
- ログに出ている、実際に使われたモデル名（予測役・調査役・読み取り役）

失敗したら、ログの該当部分を読んで原因を書く。鍵の名前の間違いのような持ち主側の設定なら、直し方を画面操作で案内する。コードの問題なら直さずに止まり、報告する。試運転は合わせて3回まで。

## 3. 時間と費用を測る

- Actions の課金対象の時間: `gh api repos/orbiane/forecast-lab/actions/runs/<run_id>/timing` の値
- LLM の費用: ログに費用やトークン数が出ていれば、それを使う。出ていなければ、持ち主に OpenRouter の Activity 画面（https://openrouter.ai/activity）を見てもらい、試運転の分の金額を教えてもらう

成功の条件: 「試運転で〇問、Actions〇分、LLM〇ドル（どこで数えたか）」を示せる。そこから、毎時実行した場合の月の Actions 時間と、月の LLM 費用の見込みを出せる（見込みは計算式も書く）。

## 4. 文書を実測で直す

`docs/sources.md` の「要確認」「推測」の行のうち、手順2〜3で確かめたもの（使われたモデル名、1問あたりの費用など）を「生データで確認」に書き換える。`setup/first-run` に commit して push する。

成功の条件: `git diff main..setup/first-run -- docs/sources.md` に、書き換えた行と根拠（どの実行のどのログか）が出る。

## 5. 本番を始めるか、持ち主に聞いて止まる

手順3の見込みを添えて、持ち主に「`BOT_ENABLED` を `true` にして毎時の本番を始めてよいか」を聞く。費用の上限はまだ決まっていない（ADR-001 の「保留」）ので、見込みの月額を必ず書く。

持ち主がこのセッションで「はい」と答えたら、`gh variable set BOT_ENABLED --body true -R orbiane/forecast-lab` を実行し、`gh variable list -R orbiane/forecast-lab` で設定を示す。そのあと次の定時実行（毎時23分 UTC）を1回見届け、スキップではなく実行されたことを示す。

## 今回の約束

- 試運転は合わせて3回まで。失敗しても連打しない
- 本番の大会への予測は、持ち主が `BOT_ENABLED` を承認するまで出さない（手動で `Forecast on new AI tournament questions` を実行しない）
- 鍵の値を表示・記録・コミットしない
- 公開・非公開の切り替えはしない（持ち主の判断待ち）
- push してよいのは作業ブランチ `setup/first-run` だけ（持ち主が今回の作業に限って承認済み）。`main` には push しない
- 持ち主はパソコンの前にいないことがある。結果はスマホとクラウド側の Claude が見るので、区切りごと（長くても15分ごと）に `setup/first-run` へ push する
- この指示書に無い判断をしたら、報告書の「食い違った点」に、何をなぜそうしたかを書く

## 終わったら

`docs/first-run-report.md` に平易な日本語で短くまとめ、`setup/first-run` に push する。同じ内容をセッションの最後の返事にも書く。

何をしたか・確かめたこと（実行したコマンドと出力）・試運転の結果（投稿件数、使われたモデル）・時間と費用の実測と月の見込み・`BOT_ENABLED` の状態・食い違った点・まだ怪しい点・持ち主に決めてほしいこと。
