---
name: simplify
description: PR前の最終整頓として、ブランチ分岐点（merge-base）以降の変更ファイルのみを対象に、挙動を変えずにリファクタリングします。重複削減・可読性向上・効率改善を行い、最後に lint で検証します。Use when: /simplify, リファクタリング, PR前の整理, diff from branch point.
argument-hint: "[base-branch(optional): develop|main]"
---

# /simplify (branch-point based)

このスキルは、現在ブランチでの実装を壊さずに「仕上げる」ためのものです。
対象は必ず「分岐点からHEADまでの差分」に限定してください。

## Goal

- 機能を変えずにコード品質を上げる
- 分岐以降の変更だけを対象にする
- PRレビュー負荷を下げる

## Guardrails

- 外部仕様・戻り値・副作用・エラーハンドリングの意味を変えない
- 振る舞い変更が必要な提案は実施せず、提案として分離する
- 機密情報、認証、RLS/policy、秘密鍵に触れない
- 既存アーキテクチャ（Feature-Driven）を崩さない
- `src/app` は薄い合成層を維持し、業務ロジックは `src/features/*` / `src/lib/*` に置く

## Scope Detection (must)

1. ベースブランチを決定する。

- ユーザー引数があればそれを優先（例: `/simplify main`）。
- 引数がなければ `develop` を優先し、存在しなければ `main` を試す。

2. 分岐点コミットを取得する。

```bash
git merge-base <base-branch> HEAD
```

3. 分岐点からの差分ファイルを取得する。

```bash
git diff --name-only --diff-filter=ACMR <base-branch>...HEAD
```

4. 上記ファイルのうち、ソースコード中心に絞る（例: `src/**`、必要なら設定ファイルも含む）。

## Refactor Strategy

差分対象ファイルに対して、次を優先して適用する。

1. Code Reuse

- 重複ロジックを同一責務の関数に集約
- 同一変換・同一定数・同一バリデーションの重複排除

2. Code Quality

- 命名の明確化
- 深いネストをガード節で平坦化
- 型安全性の強化（緩い型を具体化）
- 長すぎる関数を責務で分割

3. Efficiency

- 不要な再計算の削減（例: メモ化、共通前計算）
- 計算量の悪い実装を同等挙動のより良い実装に置換
- 毎回同じI/Oや探索を行う処理の削減

## Project-specific Rules

- Feature依存は公開API経由（`@/features/<feature>`）を優先
- feature間の直接依存は避ける
- fetchを変更する場合はキャッシュ方針を明示する
- 変更影響範囲を明示する

## Verification (must)

1. 差分確認

```bash
git diff --stat <base-branch>...HEAD
```

2. lint実行（必須）

```bash
npm run lint
```

3. 想定外の挙動変更が疑われる場合は、変更を分離して提案扱いにする。

## Output Format

実行後は以下を簡潔に報告する。

- Base branch: `<base-branch>`
- Merge base: `<commit-hash>`
- Target files: `<count>`
- Applied refactors:
  - Reuse: `...`
  - Quality: `...`
  - Efficiency: `...`
- Verification:
  - `npm run lint`: `passed|failed`
- Notes:
  - 挙動変更はしていないこと
  - もし提案のみで残した項目があれば列挙

## Example Commands

```bash
# 例: develop を親にして分岐点以降を対象化
git merge-base develop HEAD
git diff --name-only --diff-filter=ACMR develop...HEAD

# 例: 差分全体の確認
git diff develop...HEAD
```
