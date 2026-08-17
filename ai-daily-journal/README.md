# AI日誌 v2.0

CodexやClaude Codeとの1日を、単なる会話要約ではなく「何を頼み、何が動き、何が残ったか」まで読み返せるMarkdownへ変える。日誌には検索用索引が入り、後から日付、AI、作業場所、結果状態、語句で探せる。

## 30秒で開始

1. 利用中のAIを選ぶ
2. リンク先の枠をすべてコピーする
3. 新しい作業へ貼り付ける

[Codex向けをコピー](prompts/codex.md) ｜ [Claude Code向けをコピー](prompts/claude-code.md)

貼り付け後は、AIが実際のPCを調査し、不明点だけを確認してから実装、試験、修正、公開後監査まで進める。

## v2.0で変わったこと

- 日次実行の担当を1つに固定し、古いスケジューラや入れ子実行との競合を検出する
- lockへ処理段階とheartbeatを持たせ、`RUN_BUSY`の原因を追えるようにする
- 追記中のログは、取得時点までのbyte prefixが不変なら継続し、変更・短縮なら安全停止する
- full digestをそのままAIへ渡さず、cwd別の限定されたgeneration contextだけで本文を作る
- 検索索引、全task、全sourceはseal済みdigestからpipelineがstageへ原子的に差し込む
- AIを使わなかった日も、0件を捏造せず有効な1日分として扱う
- 日付、AI、作業場所、結果状態、語句で探せる読み取り専用`search`を持つ

## 目指す流れ

```text
各AIの実ログ（読取専用）
        ↓ provider別抽出・伏字化
source manifest付きdigest + 検索索引
        ↓ sealしてhash固定
限定generation context ──→ AIがstage本文だけを生成
        ↓
pipelineが索引・全task・全sourceを差し込み
        ↓ 検証・既存非上書き・原子的publish
公開後audit ──→ 読み取り専用search
```

## 壊れにくくする9段階

1. `preflight`：設定、ログ、保存先、依存、実行主体の競合を確認
2. `begin`：対象日、既存成果物、lock、stageを確認しrun tokenを取得
3. `digest`：provider別に抽出し、source manifestと検索索引を作成
4. `seal`：schema、日付、必須配列、source prefix、digest hashを固定
5. `generation-context`：本文生成に必要な限定サンプルと集計だけを作成
6. `stage`：最終保存先ではなく一時ファイルへ本文を生成
7. `fill-generation-blocks`：索引、全task、全sourceをdigestから差し込み
8. `publish`：機械検証後、既存ファイルを上書きせず原子的に公開
9. `audit-published`：最終hash、digest、source、lock・stage残留を再確認

どこかが失敗したら、そのrunのtokenで一度だけabortし、同じrun内で自動再試行しない。失敗を隠して日誌だけ残すこともしない。

## 完成後に残るもの

- `YYYY-MM-DD_AI-Journal.md` 形式の日誌
- Obsidian Vault、または指定したローカルフォルダへの保存
- 利用者が指定した時刻に、設定タイムゾーンの前日分を1件だけ生成する自動実行
- 手動で対象日を指定できる実行入口
- 日付単位lock、stage、seal、非上書きpublish、公開後audit
- source manifest付きdigestと、全task・全sourceの追跡情報
- 既存の日誌とdigestだけを読む検索入口

## 後から探す

構築後の実行本体には、少なくとも次の条件を組み合わせられる読み取り専用`search`を持たせる。

- 開始日／終了日
- AI（Codex／Claude Code）
- 作業場所の部分一致
- 結果状態
- task本文の語句
- 最大件数

検索結果は、対象日、AI、task ID、作業場所、結果状態、依頼の短い説明、公開済み日誌のパスを返す。元ログを検索のたびに再読込せず、保存済みdigestを使う。

実際のコマンド名と保存場所はPCごとに異なる。構築したAIの完了報告にある`search`の絶対パスと実行例を使う。

## 日誌の構成

- この日の要約
- AI別・協業サマリー
- 個別案件の振り返り
- 検索用索引
- AI別作業履歴
- 元資料

[架空データの出力例](examples/2026-08-01_AI-Journal.md)で完成イメージを確認できる。

上部は人が読む要約、中部は案件別の事実と考察、下部はpipelineが生成する検索・証拠台帳という役割分担である。AIの最終応答は「AIによる報告」であり、外部状態の独立確認とは分けて記録する。

## 使う前の確認

- プロンプトはローカル環境へファイルや自動実行設定を追加する
- 実装前に、取得元、保存先、実行主体、外部送信の有無が報告される
- APIキー、トークン、個人情報、顧客情報を日誌へ入れない
- 元ログは読み取り専用で扱う
- rawログではなく、ローカルで伏字化したdigestだけを本文生成へ渡す
- 新しい外部サービスへログ由来情報を送る場合は、実行前に確認を求める
- 既存の自動実行を無断停止せず、競合時は原因と必要な選択を報告する
- 実行用フォルダと日誌保存先に、大文字小文字だけが違う名前を使わない

## 完成の判定

構築を担当したAIの自己申告ではなく、プロンプト内の受け入れ試験表で判定する。

最低でも、実ログ取得、複数ファイル探索、AI別抽出、meta-only入力除外、秘密情報除外、0件日、同日再実行、同時実行、競合scheduler、lock診断、追記型source、seal後改変、限定context、3ブロック差し込み、原子的公開、公開後audit、読み取り専用searchを確認する。

確認できない項目が一つでもあれば、`未確認`または`未対応`として残し、完成と表現しない。

詳しい検証条件と現在の確認範囲は [VERIFICATION.md](VERIFICATION.md) を参照。

## 配布内容

- [Codex向け実装プロンプト](prompts/codex.md)
- [Claude Code向け実装プロンプト](prompts/claude-code.md)
- [出力例](examples/2026-08-01_AI-Journal.md)
- [検証記録](VERIFICATION.md)
- [変更履歴](CHANGELOG.md)

## 実運用との関係

作者のローカル環境では、単一の実行主体、phase/heartbeat付きlock、append-only source検証、限定generation context、決定論的block差し込み、非上書きpublish、公開後audit、読み取り専用searchを含む仕組みが稼働している。

公開版は、作者の実装コードや個人環境を配るものではない。環境固有の処理と、環境をまたいで守る合格条件を分け、ほかのPCで同じ目的へ到達するための構築プロンプトを配布する。
