---
name: github-authored-review-feedback
description: 自分が GitHub 上で過去に行った PR レビュー内容を抽出し、実際に自分が書いた review body、inline review comment、actionable な conversation comment だけを Markdown に整理する skill。自分のレビューが妥当だったかを後から確認・評価したいときに使う。
argument-hint: '[owner/repo] [reviewer-login] [date: YYYY-MM-DD] [include-open(optional)]'
---

# GitHub Authored Review Feedback

この skill は、特定ユーザーが過去に GitHub 上で行ったレビュー内容を抽出し、振り返りや自己評価に使える Markdown を作るためのものです。

今回の方針は次だけに絞る。

- GitHub CLI の GraphQL API を使う
- 対象 PR は `reviewed-by` を起点に集める
- 自分 authored のレビュー本文とコメントだけを残す
- 出力は docs 配下の 1 ファイルにまとめる

## Use When

- 自分が過去に行ったレビューを一覧したい
- 自分の指摘が妥当だったかを後から見直したい
- 自分のレビュー傾向を分析したい
- レビュー品質のセルフレビュー用データを作りたい

## Inputs To Confirm

作業前に、最低限次の 3 点を確認する。

- GitHub repository: `owner/repo`
- 対象ユーザー: reviewer login
- 出力日付: `YYYY-MM-DD`

必要なら追加で確認する。

- open PR も含めるか
- conversation comment も含めるか
- 自分が author の PR を除外するか

## Successful Procedure

### 1. GitHub CLI の利用可否を確認する

まず GitHub CLI が使えることを確認する。

```bash
gh --version
gh auth status
```

この skill の本体は `gh api graphql` で行う。ブラウザ操作や HTML のスクレイピングは不要。

### 2. 対象 PR 一覧を `reviewed-by` ベースで取得する

まず、自分がレビューした PR をまとめて取る。

検索クエリの基本形:

```text
repo:<owner>/<repo> is:pr reviewed-by:<login> sort:updated-desc
```

closed / merged を優先したい場合:

```text
repo:<owner>/<repo> is:pr is:closed reviewed-by:<login> sort:updated-desc
```

open PR も必要な場合だけ、別途 `is:open` で追加確認する。

補足:

- `reviewed-by` は正式な review を起点に PR を見つけるために使う
- formal review を伴わない conversation comment まで拾いたい場合は、必要に応じて `commenter:<login>` の結果もマージする

取得対象の最小フィールドはこれで足りる。

```graphql
query ($searchQuery: String!) {
  search(query: $searchQuery, type: ISSUE, first: 50) {
    issueCount
    nodes {
      ... on PullRequest {
        number
        title
        state
        url
        closedAt
      }
    }
  }
}
```

### 3. 各 PR のレビュー情報を取得する

PR ごとに以下を取得する。

- `reviews.nodes[].body`
- `reviews.nodes[].comments.nodes[]`
- `comments.nodes[]`

成功パターンとして使う取得フィールド:

```graphql
query ($owner: String!, $name: String!, $number: Int!) {
  repository(owner: $owner, name: $name) {
    pullRequest(number: $number) {
      number
      title
      state
      url
      author {
        login
      }
      reviews(first: 100) {
        nodes {
          author {
            login
          }
          state
          body
          submittedAt
          url
          comments(first: 100) {
            nodes {
              author {
                login
              }
              path
              line
              originalLine
              body
              createdAt
              url
            }
          }
        }
      }
      comments(first: 100) {
        nodes {
          author {
            login
          }
          body
          createdAt
          url
          authorAssociation
        }
      }
    }
  }
}
```

### 4. 自分が書いたコメントだけを残す

この skill では、対象 reviewer login と一致する authored comment だけを残す。

分類ルール:

- authored review body: `reviews.nodes[]` のうち `review.author.login === reviewerLogin`
- authored inline review comment: `reviews.nodes[].comments.nodes[]` のうち `comment.author.login === reviewerLogin`
- authored conversation comment: `comments.nodes[]` のうち `comment.author.login === reviewerLogin`

デフォルトでは、自分が author の PR は除外してよい。目的が「他人の PR に対して自分がしたレビューの妥当性確認」だから。

### 5. ノイズを除外する

成功パターンでは、次を除外対象にする。

- bot コメント
- deploy 通知
- Codex の定型 review body
- `@codex` の起動コメント
- commit リンクだけの返信
- close 通知
- `LGTM`、`thanks` などの非 actionable コメント

bot 判定の実装例:

```js
function isBotLike(login) {
  return /bot/i.test(login) || login === 'cloudflare-workers-and-pages'
}
```

commit リンクだけの返信を除外する実装例:

```js
function isCommitLinkOnly(body) {
  const text = body.trim()
  return /^\[[^\]]+\]\(https:\/\/github\.com\/.+\/commits\/[a-f0-9]+\)(で対応)?$/i.test(text)
}
```

conversation comment 側で除外した代表例:

- deploy 通知
- `@codex review`
- close notice
- 純粋な感謝や LGTM

### 6. Markdown を生成する

出力ファイルは docs 配下に置く。

推奨ファイル名:

```text
docs/<reviewer>-authored-review-feedback-<YYYY-MM-DD>.md
```

推奨セクション構成:

```text
# Authored Review Feedback Investigation for <login>

- Repository: <owner>/<repo>
- Reviewer: <login>
- Generated at: <date>
- PR count: <count>

## PR #<number> <title>

- state: <state>
- pr_author: <login>
- url: <url>

### Authored review bodies

### Authored inline review comments

### Authored actionable conversation comments
```

各コメントには次を含める。

- reviewer
- comment_type
- review_state
- file_path
- line_number
- created_at
- url
- full original body

自分のレビュー内容を後で評価しやすくするため、PR author と PR state も残す。

### 7. コメント本文のフェンスは `~~~text` を使う

レビューコメント本文に `ts のようなコードフェンスが含まれることがある。そのため、外側の囲みは `text ではなく `~~~text` を使う。

安全な例:

```text
- body:
~~~text
ここに元コメントをそのまま入れる
~~~
```

元コメントの中に triple backticks が入っていても、外側の Markdown が壊れにくい。

### 8. 原文は改変しない

コメント本文は要約しない。誤字や記号も含めて、元の本文をそのまま保持する。

この skill の目的は「自己評価用の元データ作成」であり、良し悪しの評価やスコアリングは別タスクに分ける。

## Output Rules

- PR description は含めない
- 自分 authored のレビューだけを載せる
- comment type ごとに分ける
- 原文を改変しない
- ノイズ除外ルールは冒頭に明記する
- open PR を含めた場合は、その理由を冒頭に明記する
- 自分 authored の PR を除外した場合は、その方針も冒頭に明記する

## Example Invocation

```text
/github-authored-review-feedback botica-ai-studio/idea-hub-v2 nicole208 2026-03-17 include-open
```

## Expected Result

期待する成果物は 1 つの Markdown ファイル。

- 保存先: `docs/<reviewer>-authored-review-feedback-<YYYY-MM-DD>.md`
- 中身: PR ごとに整理された self-authored review feedback
- 用途: 自己評価、レビュー傾向分析、レビュー品質の振り返り
