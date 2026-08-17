# AI日誌 BUILD PROMPT v2.0｜Claude Code向け

下の枠を右上のボタンでコピーし、新しいClaude Codeの作業へ貼り付ける。

```text
【AI日誌 BUILD PROMPT v2.0｜Claude Code向け】

あなたの役割は、このPCの実環境に合わせて「AI日誌 v2」を実装し、確認できる範囲を実際に稼働させることです。

完成済みの共通実装や固定ログパスがあるとは仮定しないでください。実装方式は環境に合わせて選びますが、以下の処理境界、安全要件、検索契約、受け入れ試験、完了報告は省略しないでください。

## 0. 完了の定義

終了状態は次の3つだけです。

- `PASS`：必須受け入れ試験を実物で確認した
- `PARTIAL`：安全に実装できた範囲はあるが、必須項目に未確認または未対応が残る
- `BLOCKED`：安全、権限、ログ、保存先、日次実行主体などが不足し、主要経路を作れない

設定ファイルを作っただけ、fixtureだけで動いた状態、AI自身が「成功」と報告しただけでは `PASS` にしないでください。

## 1. 日次実行主体を1つにする

対象に最も近い `AGENTS.md`、`CLAUDE.md`、既存の自動化規則、現在の権限と利用可能な機能を最初に確認してください。

日次生成を所有する実行主体は1つだけにしてください。候補は、現在利用中のAIのネイティブ自動化、OSスケジューラ、既存runnerです。

- ネイティブ自動化を選んだ場合、その自動化自身が本文生成まで担当する
- 自動化の中から別の同種AIプロセス、同じ自動化、同じrunnerを再帰起動しない
- Codex自動化から別の `codex exec` や `run_daily.sh` を起動せず、Claude側でも同等の入れ子CLI起動をしない
- `fill-generation-blocks`の実行者も1つに固定する
- 同じ対象日を起動し得る既存scheduler、runner、常駐serviceを列挙する
- 競合があれば新しい自動実行を登録せず、どれを残すか確認する
- 既存schedulerの停止、無効化、削除、権限変更はユーザー承認なしに行わない
- OSスケジューラからCLIを起動する場合は、認証、非対話実行、PATH、権限、cwd、終了コードを同じ入口で実機確認する

## 2. ゴール

設定タイムゾーンの前日分について、その日にAIと行った会話・作業ログから、次を1日1つのMarkdownへ統合してください。

- 何を依頼したか
- 何が行われたか
- AIが何を報告したか
- 実物で何を確認できたか
- 何が決まったか
- 何が未確認、失敗、保留として残ったか
- 次に何をするのか

単なる会話の短縮やキーワードの羅列ではなく、後からその日の案件と判断を復元できるように、その日の意味を圧縮して残します。

日次自動実行では、前日分を1件だけ生成してください。複数日の自動補完は行いません。過去日の補完は、手動入口から対象日を明示した時だけ1日ずつ行ってください。日付をコードや自動実行指示へ固定しないでください。

## 3. 変更前の環境調査

書き込み前に、実物から次を確認してください。

1. OS、OS版、CPUアーキテクチャ、ローカルタイムゾーン
2. Codex、Claude Code、その他AIの導入状況、利用形態、版
3. 各AIが実際に保存するログのルート、構成、拡張子、ファイル数
4. 値を伏せた最小構造から、timestamp、role、本文、tool呼出し、tool結果、最終応答を識別できるschema
5. 対象日に存在するsession数、source数、event数
6. Obsidianの有無、候補Vault、既存の日誌フォルダ
7. 実行本体、状態、digest、stage、日誌を保存できる場所
8. 既存scheduler、runner、同名設定、常駐process
9. 日次の意味統合を担当できるAI実行主体
10. 現在利用中のAIへ渡す情報と、新しい外部送信の有無

構造調査でraw本文、`content`、`text`、`message`、tool引数、tool結果をterminalや会話へ表示しないでください。出力はキー名、型、配列長、件数、timestamp形式、`type`、`role`など許可した構造値だけに限定してください。

意味確認が必要な本文は、ローカル抽出処理で秘密情報と不要な個人情報を先に除外し、保護された一時領域へredacted digestとして保存してください。AIはrawログではなく、後述の限定generation contextだけを読んで本文を生成します。

実装前に、取得対象、対象日の計算、保存先、日次実行主体、競合の有無、外部送信、未確認点を短く報告してください。結果を大きく変える不明点だけユーザーへ確認し、それ以外は安全で可逆的な仮定を明示して進めてください。

## 4. パスと既存環境の契約

実行本体、状態、digest、stage、日誌保存先、元ログを分離してください。

- 実行本体と日誌保存先に、大文字小文字だけが違う名前を使わない
- 絶対パス、symlink解決後、可能ならinodeとfilesystemの大文字小文字規則を確認する
- 同一場所、危険な親子関係、別名で同一場所なら書き込み前に停止する
- 設定した全パスを実行時にも再検証する
- 元ログ、既存日誌、過去digest、他の自動化を移動、削除、整形、上書きしない
- stageと最終保存先は、原子的publishが可能か事前に確認する
- テストは専用の一時領域で行い、本人の実ログへ書き込まない

## 5. provider別の取得契約

Codex、Claude Code、その他取得可能なAIをprovider別adapterとして独立に扱ってください。

- 実在を確認したログルートから、日次・session単位の複数ファイルを探索する
- ファイル名だけで日付を決めず、各eventのtimestampを設定タイムゾーンへ変換する
- 読めたschemaだけを処理し、未知schemaや壊れた行を成功として捨てない
- 元ログは読み取り専用で開く
- provider別に探索source数、読込source数、event数、task数、解析失敗数を記録する
- 安定したsource ID、session ID、event ID、時刻、内容hashで重複を除く
- adapterの対応条件と確認済みschemaを検証記録へ残す

taskは、実際の依頼を含むuser入力を起点にしてください。環境情報、browser context、plugin一覧、推奨plugin一覧、skill説明、task notificationだけのmeta-only入力はtask化しません。同じ入力に実依頼も含まれる場合は、meta blockだけを除き、実依頼をtaskとして残してください。

各taskには最低限、provider、task ID、session ID、時刻、cwd、依頼、代表操作、tool結果件数、明示exit code、最終応答の短い抜粋、outcome状態、書込み結果、source IDを持たせてください。ログにない作業や結果を推測で補わないでください。

## 6. digest schema v4と検索索引

1日分のredacted digestをJSONで作り、`schema_version: 4` としてください。最低限、次を持たせます。

- `date`、`generated_at`
- provider別の全task配列
- provider別の`source_files`
- provider別の`source_manifest`
- `retrieval_index`

各source manifest entryには、安定したsource識別子、取得時のbyte size、取得したbyte prefix全体のSHA-256を持たせてください。

seal時のsource検証は次の通りです。

- 現在sizeが取得時size未満なら拒否
- 取得時sizeまでのbyte prefix SHA-256が不一致なら拒否
- 取得時sizeより後ろへの追記だけなら許可
- source消失、置換、短縮、prefix変更は拒否
- digest内のdate、schema、generated_at、必須配列、task ID、source_files、source_manifest、retrieval_indexを機械検証する
- seal済みdigestのSHA-256をlockへ固定する

`retrieval_index`は全taskと全sourceから決定論的に作り、task総数、source総数、AI別task数、cwd別task数、outcome別task数、tool/API/MCP/nonzero-exitのerror signalsを持たせてください。

対象日にAI利用がない場合も、provider配列、source_files、source_manifestを空配列、task数とsource数を0として有効なdigestを作ってください。0件を失敗や未取得へ変換しないでください。

## 7. 日付単位lock

lockには最低限、run token、owner、PIDまたは実行識別子、開始時刻、`heartbeat_at`、現在の`phase`、stage、digest、finalのパスを持たせてください。各主要工程の開始または完了でphaseとheartbeatを更新します。

別runがlockを所有している場合は終了コードを分けて `RUN_BUSY` とし、lock path、owner、PID、phase、開始時刻、heartbeat age、stage、digest、finalを報告してください。別runのlockを解除しないでください。

stale回収を実装する場合は、process不存在、heartbeat経過、stageの年齢をすべて確認し、由来を記録した退避先へ移してから始めてください。判定できないlockや新しいstageは回収せず安全停止します。

## 8. 9段階pipeline

実装名や言語は問いませんが、次と同等の境界と、各工程を単独で検証できる入口を持たせてください。

1. `preflight`：設定、path分離、依存、保存権限、source、scheduler競合を読取中心で確認
2. `begin`：対象日を確定し、既存成果物、空ファイル、lock、stageを確認してtoken取得
3. `digest`：provider別に一度だけ抽出し、schema v4 digestを作成
4. `seal-digest`：digestとsource prefixを検証し、digest hashをlockへ固定
5. `generation-context`：seal済みdigest不変を確認し、限定contextを作成
6. `stage`：限定contextだけから一時Markdownを生成
7. `fill-generation-blocks`：seal済みdigestから3ブロックをstageへ原子的に差し込み
8. `publish`：stageを機械検証し、既存非上書きで原子的に公開
9. `audit-published`：公開後状態を読み取り専用で再検証

token取得後に失敗した場合は、そのtokenで`abort`を一度だけ実行してください。同じrun内でdigest、seal、context、fill、publishを自動再試行しないでください。

## 9. generation contextと決定論的差し込み

本文生成用contextは、seal済みdigestから機械生成してください。providerとcwd単位で、task数、依頼の限定サンプル、tool集計、outcome集計、最終応答の限定サンプル、書込み先の限定サンプル、error signalsを持たせます。

- 全task配列と全source_filesをcontextへ入れない
- 最終応答はAIによる報告であり、外部状態の独立確認へ昇格させない
- 書込み結果は記録された操作であり、後続の再読や外部反映まで証明しない
- 大規模fixtureでは、generation contextをfull digestの25%以下へ抑える
- contextの集計・限定サンプルにない案件や成果を本文へ補わない

stageには次のmarkerを、それぞれ対応見出しの直下へ独立した1行として一度だけ置いてください。

- `{{JOURNAL_RETRIEVAL_INDEX_BLOCK}}`
- `{{AI_TASK_HISTORY_BLOCK}}`
- `{{SOURCE_LINES_BLOCK}}`

AI、jq、shell loop、別scriptでこの3ブロックを再構成しないでください。`fill-generation-blocks`だけがseal済みdigestから、検索索引、全task、全sourceを順序固定で差し込みます。stdoutへ巨大配列を返さず、status、件数、stage SHA-256だけを返してください。

## 10. 日誌フォーマット

1日1ファイル、UTF-8のMarkdownで保存してください。

ファイル名：

`YYYY-MM-DD_AI-Journal.md`

frontmatterは順序と値を決定論的にし、最低限、対象日、設定タイムゾーン、種別、状態を入れてください。秘密情報や不要な絶対パスは入れません。

本文には次を、この順序で一度ずつ含めます。

```markdown
# YYYY-MM-DD AI利用日誌

## この日の要約

## AI別・協業サマリー
### Codex
### Claude Code
### cross-ai-loop

## 個別案件の振り返り
## 案件: <短い案件名>
### やったこと（ログから確認できる事実）
### 意図
### AIの対応
### 生まれた成果

> 以下はAIによる考察であり、ログから直接確認できる事実ではない。

### どう機能するか・ベネフィット
### 残った課題・できなかったこと
### もっと良くできた点

## 検索用索引
{{JOURNAL_RETRIEVAL_INDEX_BLOCK}}

## AI別作業履歴
{{AI_TASK_HISTORY_BLOCK}}

## 元資料
{{SOURCE_LINES_BLOCK}}
```

案件はcwdと依頼内容を軸に統合し、1 taskを1案件として水増ししません。片方のAIだけを使った案件は、AI対応表のもう片方を `—` とします。cross-ai-loopは同じ案件で複数AIの分析、実装、検証が実際に接続した時だけ書き、確認できなければ「ログから確認できる協業なし」と書いてください。

`### 生まれた成果`まではdigest由来の事実またはAIによる報告だけを書きます。引用境界より下だけをAIの考察とします。未取得を0、成功、完了で補完しません。

task数が0の場合は「AI利用記録なし」の1案件を作り、task・sourceが0件であることだけを書いてください。架空の成果や未取得providerを補いません。

## 11. publishとaudit

publish前に、seal後digest不変、source prefix不変、stage非空、frontmatter、対象日、必須見出し、案件の必須節、考察境界、AI対応表、検索索引、全task ID、全source、未置換placeholder、秘密値候補を機械検証してください。

- 最終保存先へ直接生成しない
- 完成済みの非空日誌があれば、hash不変で「既存のためスキップ」
- 空の既存日誌は上書きせず失敗
- 最終保存先の不存在をpublish直前にも再確認
- 既存ファイルを上書きしない原子的publishを使う
- publish後に別のread-only auditを一度実行
- auditは最終hash、digest hash、source manifest、全task、全source、lock不存在、stage不存在を確認
- audit失敗を成功扱いしない

## 12. 読み取り専用search

保存済みdigestだけを読む `search` subcommandまたは同等の入口を実装してください。元JSONLを再読込せず、ファイルを変更しません。

最低限、次を任意に組み合わせられるようにします。

- `--date-from`
- `--date-to`
- `--ai`
- `--cwd-contains`
- `--outcome`
- `--text`
- `--limit`

結果はJSONで、scanned digest数、match数、truncated、対象日、AI、task ID、cwd、outcome、依頼の短い説明、公開済み日誌の絶対パスを返してください。壊れたdigestや未対応schemaは成功として混ぜず、skip件数またはerrorとして区別してください。

## 13. 自動実行

ユーザーが指定したローカル時刻に毎日1回動かしてください。時刻が未指定なら、候補を提示して確定してから登録します。

- 実行時に設定タイムゾーンの前日を動的計算する
- 対象日は1日だけ
- 自動実行と手動試験は同じpipeline入口を使う
- 日次のAI実行主体は1つだけ
- 固定日付、入れ子AI起動、二重fill、複数schedulerを禁止
- 設定作成後に時刻、timezone、引数、cwd、環境変数、実行ファイル、状態を再読込
- 実時刻発火を待っていない場合は「設定確認PASS／実時刻発火は未確認」と分ける
- 失敗時は対象日、生成されなかった最終パス、実行済み工程、終了コード、エラー全文を報告する

## 14. 安全要件

- APIキー、token、Cookie、password、認証header、秘密鍵を日誌、digest、terminal、通常／error log、テスト出力へ書かない
- 元ログは読み取り専用で扱う
- rawログをAIへ直接読み込ませない
- 現在利用中のAIへ渡すgeneration contextの範囲を実装前に説明する
- 新しい外部送信が必要なら、送信先、送信内容、保持条件を説明し、実行前に確認する
- 既存ファイル、scheduler、service、設定を勝手に削除、上書き、停止しない
- 新しい依存を増やす前に、標準機能で実現できないか確認する
- recovery artifactを作る場合は由来と復元方法を明示する

## 15. 必須受け入れ試験

各項目を `PASS / FAIL / 未確認 / 対象外` で記録し、証拠を示してください。

1. 実ログから対象日1日分を取得
2. provider別の実在path、探索source数、読込source数、event数、task数を確認
3. 複数providerを別々に抽出後に統合
4. 複数sourceを対象eventのtimestampで横断
5. UTC境界を設定timezoneへ正しく変換
6. meta-only入力をtask化せず、実依頼との混在時は実依頼を保持
7. AI報告と実物確認を区別
8. 明示されていない思考、決定、成功を捏造しない
9. 疑似秘密値が日誌、digest、terminal、logs、test outputへ出ない
10. task 0／source 0の日を有効な日誌として生成
11. 同日再実行がskip、1 file、hash不変
12. 同時実行で一方だけがtoken取得
13. `RUN_BUSY`がowner、phase、heartbeat age、主要pathを報告
14. 競合schedulerを検出し、無断停止や二重登録をしない
15. 空の既存日誌を上書きせず失敗
16. 片方のprovider不在を未取得として処理
17. 未知schema、壊れたJSONLを成功として捨てない
18. 取得後のsource追記はprefix不変時だけ許可
19. source短縮、置換、prefix変更を拒否
20. seal後digest変更を拒否
21. full digestを本文生成AIへ渡さない
22. 大規模fixtureでgeneration contextがfull digestの25%以下
23. marker欠落、重複、順序違反、事前task記入をfillが拒否
24. fill後の検索索引、全task ID、全sourceがdigestと完全一致
25. 生成中断後に最終pathへ半端なfileがない
26. 大文字小文字だけ違う実行本体／保存先を事前拒否
27. 元ログの取得済みbyte prefixが処理前後で不変
28. publishが既存非上書きで原子的
29. auditが最終hash、digest、source、lock・stage不存在を確認
30. searchが日付、AI、cwd、outcome、text、limitで絞り込み
31. searchが元JSONLと公開日誌を変更しない
32. 自動実行設定の実行主体が1つ、日付固定なし、同じpipeline入口
33. 自動実行と同じ入口を手動起動し、未生成日を1件生成
34. abortが所有tokenだけを処理し、別runのlockを解除しない

実ログ試験とfixture試験を区別し、fixtureだけで全体を `PASS` にしないでください。不具合があれば原因を特定し、修正して関連試験と回帰試験を再実行してください。

## 16. 完了報告

最後に次を報告してください。

1. 終了状態：`PASS / PARTIAL / BLOCKED`
2. 構築した9段階pipeline
3. 設定timezone、対象日の計算、日次実行時刻
4. provider別のログルート、探索方式、対応schema、取得数
5. 日誌、実行本体、状態、digest、stage、通常／error logの保存場所
6. 日次実行主体と、競合schedulerがない証拠
7. generation contextの内容、full digest比、外部送信
8. searchの絶対pathと実行例
9. 変更または作成したfile
10. 34項目の受け入れ試験表と証拠
11. 未確認、未対応、残るrisk
12. ユーザー操作が必要な場合、その理由と最小手順

成功時も長いlog全文は貼らず、対象日、最終日誌の絶対path、publish結果、audit結果、search確認を先に示してください。

確認できない項目を推測で埋めず、`PARTIAL` または `BLOCKED` として正確に終了してください。READMEだけ作って実装を止めないでください。

以上。
```
