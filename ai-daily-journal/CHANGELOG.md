# Changelog

このファイルには、公開版AI日誌の変更だけを記録する。

## [2.0.0] - 2026-08-18

### 改善

- 日次生成の実行主体を1つに固定し、入れ子AI、二重fill、競合schedulerを禁止
- lockへphase、heartbeat、owner、主要pathを追加し、`RUN_BUSY`を診断可能にする契約へ更新
- 追記型sourceを、取得済みbyte prefixのSHA-256が不変な場合だけ許可
- digestをschema v4とし、meta-only入力除外、0件日の有効digest、決定論的retrieval indexを追加
- full digestからcwd別の限定generation contextを作り、大規模入力で25%以下に抑える条件を追加
- 検索索引、全task、全sourceを3 markerへpipelineが原子的に差し込む方式へ変更
- `preflight → begin → digest → seal → generation-context → stage → fill → publish → audit` の9段階へ更新
- 保存済みdigestだけを読むread-only `search`を追加
- 日次自動実行は前日1日だけ、過去日は明示日付の手動入口だけで扱う契約へ変更
- 日誌を、要約、AI別・協業、案件別振り返り、検索索引、全task、全sourceの読順へ更新
- 必須受け入れ試験を34項目へ拡張
- 架空例、README、検証記録をv2.0へ更新

## [1.1.0] - 2026-08-17

### 改善

- 標準実装を固定せず、環境調査と受け入れ条件を固定する構成へ変更
- 終了状態を `PASS / PARTIAL / BLOCKED` に分離
- 実ログのルート、複数ファイル、実スキーマ、対象日件数の確認を必須化
- provider別の独立抽出、秘密情報を除外したdigest、意味統合の実行主体を明確化
- schema調査では本文値を表示せず、ローカルで伏せたdigestだけをAIへ渡す境界を追加
- `preflight → begin → extract → seal → generate → publish → audit` と同等の処理境界を追加
- 実行本体と日誌保存先の大文字小文字衝突、symlink、同一場所を実行前に拒否する条件を追加
- lock所有権、残留stage、空の既存日誌、seal後のsource変更、未置換placeholderの失敗条件を追加
- 毎朝7時の設定再読込、同じ実行入口の手動試験、補完試験を追加
- 実ログとfixtureを区別した24項目の必須受け入れ試験を追加
- 日誌へ `AI別作業履歴` と `元資料` を追加
- v1.0／v1.1の隔離試験結果、修正再試験、未確認範囲を `VERIFICATION.md` へ追加

### v1.0隔離試験で確認した問題

- macOSで `AI-Journal` と `ai-journal` が同一フォルダになった
- 単一の固定ログファイルだけを読む実装へ収束した
- 7時の設定は確認できたが、実時刻発火は未確認だった

## [1.0.0] - 2026-08-17

### 追加

- Codex向け実装プロンプト
- Claude Code向け実装プロンプト
- 1日1ファイルのMarkdown出力契約
- AI別抽出、重複除去、日単位統合の要件
- 毎朝7時に前日分を生成する自動実行要件
- 重複防止、失敗時補完、原子的な保存、公開後監査の要件
- 架空データによる出力例
- 公開範囲と安全要件

### 未確認

- 第三者のPCでの構築結果
- OS、Codex、Claude Codeの版ごとのログ形式への対応
- 各環境で利用できるスケジューラの差
