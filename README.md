# デイトレ ダッシュボード（PWA）

`data/` 配下の記録データ（資金推移・ウォッチリスト・銘柄別成績・バックテスト結果・
日次振り返り履歴）を、スマホ・iPadからいつでも見られるようにするための
個人用ダッシュボード。**発注機能は一切ない、読み取り専用の閲覧アプリ。**

## 仕組み

- `src/generate_webapp_data.py` が `data/*.json` / `*.jsonl` を読み込み、
  このフォルダの `data.json` に集約して書き出す（表示に必要な情報だけを抜粋。
  板・歩み値などの生データは含めない）。
- `index.html` が `data.json` を読み込んで描画する、素のHTML/CSS/JS（フレームワーク不使用）。
- `manifest.json` / `service-worker.js` によりPWA化。スマホ・iPadのブラウザで
  「ホーム画面に追加」するとアイコンから起動でき、オフラインでも直近のデータを閲覧できる。
- GitHub Pagesで公開する想定（このフォルダ単体を1つのGitリポジトリとして扱う）。

## データの更新〜再公開の流れ（自動化済み・2026-08-10〜）

`scripts/run_daily_record.ps1`（平日15:35自動実行）の最終ステップとして
`scripts/publish_webapp.ps1` が呼ばれ、以下を自動で行う。

1. `python src/generate_webapp_data.py` で `webapp/data.json` を最新化
2. 変更があれば `git add -A && git commit && git push`
3. 数十秒〜数分でGitHub Pages上のサイトに反映される（変更がなければpushはスキップ）

手動で今すぐ更新したい場合は `scripts/publish_webapp.ps1` を単独実行すればよい
（`powershell.exe -File scripts\publish_webapp.ps1`）。

認証はGitHub CLI（`gh auth login`済み）をgitのcredential helperとして使っており、
Windows資格情報マネージャーに保存されているため、タスクスケジューラ実行時も
追加のログイン操作は不要。

## アクセス保護について（重要・限界の説明）

`index.html` にはごく簡単なパスワードゲートを実装している
（入力された合言葉のSHA-256ハッシュを、埋め込んだハッシュ値と比較するだけの
クライアントサイドJS）。これは**「たまたまURLを踏んだ人・検索botに中身を
見せない」程度の目隠しであり、本当のセキュリティ境界ではない**。
ブラウザの開発者ツールでソースやネットワーク通信を見れば、ハッシュ元の合言葉を
知らなくても中身（`data.json`）を直接取得することは技術的に可能。

- 表示される情報は口座残高やAPIキーなどの機微情報ではなく、
  シミュレーション上の損益・勝率・ウォッチリストのみ（実口座の資産状況は含まない）。
- とはいえ公開してよい情報ではないため、`robots.txt`で検索エンジンのインデックスを
  拒否し、GitHubリポジトリ名やURLを他人に共有しないこと。
- より強固にしたい場合、GitHub Pro（有料）にすると非公開リポジトリのPagesを
  「リポジトリの共同編集者のみ閲覧可」に制限できる。必要になったら検討。

現在の合言葉は本人にのみ別途共有済み。変更したい場合は `index.html` 内の
`PASS_HASH` を新しい合言葉のSHA-256ハッシュに差し替える
（PowerShellなら `[System.Security.Cryptography.SHA256]::Create()` で計算可能）。

## ホーム画面への追加

- **iPhone/iPad（Safari）**: サイトを開く → 共有ボタン → 「ホーム画面に追加」
- **Android（Chrome）**: サイトを開く → メニュー → 「アプリをインストール」

## 今後の拡張候補（未着手）

- 朝の市場分析ログ（`morning_analysis_log/`）や書籍レビューログの表示
- ウォッチリスト・振り返り履歴のソート/フィルタUI
- データ更新〜pushの自動化
