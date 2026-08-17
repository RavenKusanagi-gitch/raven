# Raven Public Library

AIとの実務で使った仕組みを、ほかのPCでも再現できる形で配布する場所。

## 30秒で使う

### AI日誌

CodexやClaude Codeとの1日を、あとから判断と作業の流れまで復元できるMarkdownへまとめる。

1. 使っているAIを選ぶ
2. リンク先のプロンプトを右上のボタンでコピーする
3. CodexまたはClaude Codeへ貼り付ける

[Codex向けプロンプト](ai-daily-journal/prompts/codex.md) ｜ [Claude Code向けプロンプト](ai-daily-journal/prompts/claude-code.md)

AIがPC環境と取得可能なログを確認し、保存先の決定、実装、テスト、毎朝7時の自動実行設定まで進める。

> これは完成済みアプリのインストーラーではなく、各PCに合う仕組みをAIへ構築させるプロンプト。実行前に、AIが変更内容と利用可能なログを確認する。

詳しい説明と出力例は [AI日誌のREADME](ai-daily-journal/README.md) に掲載。

## 配布物

| 配布物 | できること | 状態 |
| --- | --- | --- |
| [AI日誌](ai-daily-journal/README.md) | 1日分のAI作業ログを抽出・統合し、Markdownで保存する仕組みを構築 | v1.0 |

## 公開範囲

公開するのは、再現用プロンプト、架空データの出力例、使い方、変更履歴だけ。

個人の会話ログ、顧客情報、認証情報、本人用のファイルパス、運用中の内部スクリプトは含めない。

## ライセンス

[MIT License](LICENSE)。複製、変更、再配布、商用利用が可能。ライセンス表示は残すこと。
