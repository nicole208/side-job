---
name: review-driven-refactor
description: 実装後からPR前までの仕上げとして、過去レビューで再発した観点を使って changed files を点検し、安全にリファクタリングする skill。Use when: push前の最終整頓, review観点ベースの見直し, ダッシュボード整理, Mastra整理, 仕様を崩さない改善。
argument-hint: "[scope(optional): changed-files|branch-diff] [base(optional): develop]"
---

# Review-Driven Refactor

このskillは、実装が一通り終わったあとに、過去レビューで繰り返し指摘された論点を使ってコードを仕上げるためのものです。

## Use When

- push前に、レビューで刺さりそうな点を先に潰したいとき
- ダッシュボード系やMastra系の変更で、仕様、UI、型、責務分割を見直したいとき
- 実装は終わっているが、長いファイルや曖昧な型、文言のズレを整えたいとき

## Reference Documents

- [PR前チェックリスト](../../../docs/nicole208-pr-preflight-checklist-2026-03-20.md)
- [ダッシュボード系 / Mastra系 実践チェックリスト](../../../docs/nicole208-dashboard-mastra-practical-checklists-2026-03-20.md)
- [短縮版 Top 10](../../../docs/nicole208-short-checklist-top10-2026-03-20.md)

## Goal

- 仕様を変えずにレビュー指摘の再発を減らす
- UIの誤解、状態判定の不安定さ、型の甘さ、責務の偏りを減らす
- PR説明時に変更理由を説明しやすい状態まで整える

## Hard Rules

- 機能追加を主目的にしない
- 仕様変更が必要な場合は、勝手に広げず、変更点として分離する
- セキュリティ、認証、RLS、秘密情報に触れない
- Feature-Driven の依存方向を壊さない
- src/app は薄い合成層を維持する
- push前の仕上げなので、最後に lint を必ず実行する

## Scope Detection

1. 対象を決める。

- 引数が changed-files の場合は現在の変更ファイルを対象にする
- 引数が branch-diff または未指定の場合は、ベースブランチとの分岐以降を対象にする

2. ベースブランチを決める。

- 引数があればそれを使う
- なければ develop を優先し、なければ main を試す

3. 対象ファイルを集める。

- ソースコードを優先する
- docs だけの変更なら、文言やリンク整合性の見直しに限定する

## Review-Driven Audit

対象ファイルごとに、次の順で見る。

### 1. 仕様固定条件

- 固定であるべきタイトル、デフォルト、ラベルが他条件で揺れていないか
- 補助トグルが別条件まで連動させていないか

### 2. UI意味づけ

- アイコン、色、活性状態、文言が操作意味と一致しているか
- 押せそうに見えるだけのUIがないか

### 3. 状態判定の文脈

- 表示用に絞った配列だけで status や stage を決めていないか
- role、search、stage のスコープが揃っているか

### 4. 型とスキーマ

- enum、配列、既存ライブラリ型で先に締められるものを後段ロジックに逃がしていないか
- 型アサーションや曖昧な文字列分岐を減らせるか

### 5. 責務分割

- page.tsx に判定ロジックを抱え込みすぎていないか
- feature 内で filter、type、component を責務ごとに切り出せるか

### 6. 不要物

- 未使用の変数、props、列、アイコン、補助表示を削れるか

## Project-Specific Refactor Guidance

### Dashboard changes

- status 判定と stage 判定の母集団を揃える
- 集計、カード、一覧の表示意味を統一する
- 見た目だけのフィルタアイコンや不要列を削る
- 巨大な画面ファイルは feature 側へ分解する

### Mastra changes

- schema を先に締めて tool の分岐を減らす
- API仕様を公式ドキュメントで確認してから prompt や tool 引数を整える
- 既存ライブラリ型を優先し、命名を類似ツールと揃える

## Safe Refactor Strategy

1. まず未使用コード削除、命名整理、定数抽出のような安全な変更から行う
2. 次に型強化、schema 強化、責務分割を行う
3. 挙動に影響し得る status 判定や filter 文脈修正は、意図を説明できる場合だけ行う
4. 仕様変更に見えるものは、このskill内で勝手に拡張しない

## Verification

1. 変更ファイルの差分を見て、仕様変更が混ざっていないか確認する
2. lint を実行する
3. UI変更がある場合は、主要導線を最低1回は操作して誤解を招くUIがないか確認する
4. フィルタやタブがある場合は、組み合わせで表示がぶれないかを見る

## Output Format

作業後は次を簡潔に報告する。

- Scope: どのファイル群を対象にしたか
- Main risks checked: 仕様固定条件、UI意味づけ、状態判定、型、責務分割のどれを確認したか
- Refactors applied: 何を安全に整理したか
- Verification: lint と主要確認結果
- Follow-up: もし仕様確認が必要で手を付けなかった点があれば列挙する

## Example Invocation

```text
/review-driven-refactor
/review-driven-refactor branch-diff develop
/review-driven-refactor changed-files
```
