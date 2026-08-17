# Raven Public Library

AIとの実務で使う仕組みを、ほかのPCでも組み立てられる形で配布する場所。

## 30秒で使う

### AI日誌

CodexやClaude Codeとの1日を、判断と作業の流れまで読み返せるMarkdownへまとめる。

1. 使っているAIを選ぶ
2. リンク先にある1つの枠をコピーする
3. 新しいCodexまたはClaude Codeの作業へ貼り付ける

[Codex向けプロンプト](ai-daily-journal/prompts/codex.md) ｜ [Claude Code向けプロンプト](ai-daily-journal/prompts/claude-code.md)

AIがそのPCの実ログ、保存先、自動実行方法を調査し、実装、テスト、修正、再テストまで進める。

> 完成済みアプリのインストーラーではない。PCごとに違う環境へAIが実装するための構築プロンプト。確認できない機能を「完成」と報告しない合格条件まで含んでいる。

詳しい説明は [AI日誌のREADME](ai-daily-journal/README.md)、確認済み範囲は [検証記録](ai-daily-journal/VERIFICATION.md) に掲載。

## 配布物

| 配布物 | できること | 状態 |
| --- | --- | --- |
| [AI日誌](ai-daily-journal/README.md) | そのPCで取得できるAI作業ログから、1日1件の日誌を作る仕組みを構築 | v1.1 |

## 公開範囲

公開するのは、構築プロンプト、架空データの出力例、検証条件、使い方、変更履歴だけ。

個人の会話ログ、顧客情報、認証情報、本人用のファイルパス、運用中の内部スクリプトは含めない。

## ライセンス

[MIT License](LICENSE)。複製、変更、再配布、商用利用が可能。ライセンス表示は残すこと。
