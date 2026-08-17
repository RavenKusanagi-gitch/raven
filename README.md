# Raven Public Library

AIとの実務で使う仕組みを、ほかのPCでも安全に組み立てられる形で配布する場所。

## 30秒で使う

### AI日誌 v2.0

CodexやClaude Codeとの1日を、読み返せる日誌と、後から探せる作業索引へ変える。

1. 使っているAIを選ぶ
2. リンク先の枠をすべてコピーする
3. 新しいCodexまたはClaude Codeの作業へ貼り付ける

[Codex向けプロンプト](ai-daily-journal/prompts/codex.md) ｜ [Claude Code向けプロンプト](ai-daily-journal/prompts/claude-code.md)

AIがそのPCの実ログ、保存先、実行主体を調査し、実装、試験、修正、公開後監査まで進める。

> 完成済みアプリのインストーラーではない。PCごとに異なるログ形式や自動実行へ適応する構築プロンプトである。確認できない機能を「完成」と報告しない合格条件まで含む。

詳しい説明は [AI日誌のREADME](ai-daily-journal/README.md)、確認済み範囲は [検証記録](ai-daily-journal/VERIFICATION.md) に掲載。

## 配布物

| 配布物 | できること | 状態 |
| --- | --- | --- |
| [AI日誌](ai-daily-journal/README.md) | AIログから、壊さず生成でき、日付・AI・作業場所・結果で後から探せる日誌の仕組みを構築 | v2.0 |

## 公開範囲

公開するのは、構築プロンプト、架空データの出力例、検証条件、使い方、変更履歴だけ。

個人の会話ログ、顧客情報、認証情報、本人用のファイルパス、運用中の内部スクリプトは含めない。

## ライセンス

[MIT License](LICENSE)。複製、変更、再配布、商用利用が可能。ライセンス表示は残すこと。
