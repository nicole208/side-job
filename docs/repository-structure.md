# 副業ナレッジリポジトリの構成案

## このリポジトリの目的

このリポジトリは、副業を始めたばかりの人がつまずきやすい「何をやるべきか」「いつやるべきか」「どう整理すると後で困らないか」を継続的に蓄積するための知識ベースとして使います。

特に以下のような内容を主対象にします。

- 開業初期に必要な準備
- 契約、請求、入金確認、経費精算などの事務処理
- 帳簿、領収書整理、確定申告などの税務対応
- 公的機関や制度の情報を参照する際の要点整理

## 推奨するトップレベル構成

新しい情報は、まず Markdown で整理し、PDF や画像は補助資料として扱う構成にするのが運用しやすいです。

```text
.
|- README.md
|- AGENTS.md
|- docs/
|  |- 00_overview/
|  |- 10_getting-started/
|  |- 20_admin/
|  |- 30_tax/
|  |- 40_reference/
|  \- repository-structure.md
|- assets/
|  |- images/
|  \- pdf/
|- templates/
\- archive/
```

現時点では全ディレクトリを先に作り切る必要はありません。必要なトピックが出たタイミングで増やしていけば十分です。

## ディレクトリごとの役割

### docs/00_overview

このリポジトリ全体の使い方、更新ルール、年間の見直し方針など、全体説明を置きます。

想定ファイル例:

- repo-policy.md
- yearly-review-checklist.md

### docs/10_getting-started

副業を始めた直後に必要な確認事項を置きます。最初に読む導線として使う想定です。

想定ファイル例:

- opening-checklist.md
- accounts-and-services.md
- first-month-tasks.md

### docs/20_admin

日常の事務処理をまとめる場所です。契約、請求、入金、経費、証憑整理など、運用の実務をここに寄せます。

想定ファイル例:

- contract-checkpoints.md
- invoice-and-payment-flow.md
- expense-tracking.md
- document-retention.md

### docs/30_tax

確定申告や帳簿まわりなど、税務対応に関する情報をまとめます。制度変更が起きやすい領域なので、対象年度と参照元を明記する前提で運用します。

想定ファイル例:

- final-tax-return-overview.md
- bookkeeping-basics.md
- deductible-expenses.md
- blue-return-vs-white-return.md

### docs/40_reference

用語集、公的リンク集、FAQ のような横断的な参照情報を置きます。

想定ファイル例:

- glossary.md
- official-links.md
- faq.md

### assets/images

画面キャプチャ、図解、フロー図などの画像を置きます。Markdown から参照する前提で保存します。

ルール:

- 単体で意味が伝わらない画像だけを置かない
- 関連する Markdown と同じトピック名でまとめる
- 個人情報や取引先情報が映る画像は保存しない

### assets/pdf

国税庁や自治体の資料、参考にした PDF、整理して残したい配布資料を置きます。

ルール:

- PDF を置く場合は、必ず要点をまとめた Markdown も作る
- 原本の丸写しではなく、何を参照したい資料かが README や本文から分かるようにする
- サイズが大きいファイルが増える場合は Git LFS を検討する

### templates

再利用するチェックリストやひな形を置きます。毎回ゼロから書かなくて済む状態を作るための置き場です。

想定ファイル例:

- monthly-closing-checklist.md
- invoice-template-notes.md
- yearly-tax-prep-checklist.md

### archive

古くなった情報や、制度改定前の内容を退避させる場所です。現行運用のファイルと混ざらないようにします。

## ファイル命名ルール

ファイル名は ASCII の kebab-case を基本にすると、OS や GitHub 上で扱いやすくなります。

- 良い例: opening-checklist.md
- 良い例: final-tax-return-overview.md
- 避けたい例: 確定申告まとめ最新版.md
- 避けたい例: memo1.md

並び順を固定したい場合は、ディレクトリ名に数値プレフィックスを付ける方が管理しやすいです。

## push するときの単位

継続的に育てるリポジトリなので、1回の push で複数テーマを混ぜすぎない方が後から追いやすくなります。

おすすめの単位:

- 1テーマ追加
- 1カテゴリ整理
- 1つの制度変更への追従
- 1つの PDF や画像と、それを説明する Markdown の追加

避けたい push:

- 契約、経費、確定申告を一度に全部追加する
- 画像や PDF だけを大量に入れて説明を付けない
- 年度や参照元が分からないまま税務情報を増やす

## 記述ルール

このリポジトリは「後で自分が読み返してすぐ動けること」を重視して、以下の書き方に寄せると使いやすくなります。

- 冒頭で「何のためのメモか」を1段落で書く
- 手順ものはチェックリスト形式にする
- 税務や制度は「対象年度」「最終確認日」「参照元」を書く
- 迷いやすい論点は「判断ポイント」として分ける
- PDF や画像は本文からリンクし、単独保管にしない

## 保存しない方がよい情報

private repository であっても、以下のような情報は原則保存しない方が安全です。

- マイナンバー
- 本名、住所、電話番号などの個人情報
- 実際の請求書、契約書、申告書の原本
- 取引先名や金額が特定できるスクリーンショット
- API キーやログイン情報

必要な場合は、匿名化や黒塗りを前提にしたサンプルだけを置く運用に寄せます。

## 最初に作ると良いファイル

最初の数本は、汎用性が高くて後続ドキュメントの土台になりやすいものから始めるのがよいです。

- docs/10_getting-started/opening-checklist.md
- docs/20_admin/invoice-and-payment-flow.md
- docs/20_admin/expense-tracking.md
- docs/30_tax/final-tax-return-overview.md
- docs/40_reference/official-links.md

## 運用の要点

このリポジトリは「資料置き場」ではなく「あとで再利用できる知識ベース」として運用するのが重要です。

そのため、push のたびに以下を満たす状態を目指します。

- 新しい資料には説明用の Markdown がある
- どのカテゴリに属する情報かが明確
- 税務や制度の情報に年度と参照元がある
- センシティブ情報が含まれていない