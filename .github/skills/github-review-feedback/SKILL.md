---
name: github-review-feedback
description: GitHub 上の PR レビューコメントを抽出し、実際のフィードバックだけを Markdown に整理する skill。assignee ベースで対象 PR を集め、review body、inline review comment、actionable な conversation comment を docs 配下へまとめたいときに使う。
argument-hint: '[owner/repo] [assignee-login] [date: YYYY-MM-DD] [include-open(optional)]'
---

# GitHub Review Feedback

この skill は、特定ユーザーが受けた GitHub のレビューコメントを抽出し、再利用しやすい Markdown にまとめるためのものです。

今回の成功パターンでは、次の方針だけを使います。

- GitHub の PR 一覧ページを人手で読むのではなく、GitHub CLI の GraphQL API を使う
- 対象 PR は assignee で絞る
- PR 本文は除外し、実際のレビューコメントだけを集める
- 出力は docs 配下の 1 ファイルにまとめる

## Use When

- 自分が受けたレビューコメントをまとめて棚卸ししたい
- 同じ reviewer からの指摘傾向を見たい
- 後続で recurring themes や PR 前チェックリストを作る素材が欲しい

## Inputs To Confirm

作業前に、最低限次の 3 点を確認する。

- GitHub repository: `owner/repo`
- 対象ユーザー: assignee login
- 出力日付: `YYYY-MM-DD`

必要なら追加で確認する。

- open PR も含めるか
- 出力先ファイル名を固定するか

## Successful Procedure

### 1. GitHub CLI の利用可否を確認する

まず GitHub CLI が使えることを確認する。

```bash
gh --version
gh auth status
```

この skill では、レビュー抽出の本体は `gh api graphql` を使う。HTML のスクレイピングやブラウザ操作は不要。

### 2. 対象 PR 一覧を assignee ベースで取得する

まず対象 PR をまとめて取る。成功した手順では GraphQL の `search` を使った。

検索クエリの基本形:

```text
repo:<owner>/<repo> is:pr assignee:<login> sort:updated-desc
```

closed / merged を優先したい場合:

```text
repo:<owner>/<repo> is:pr is:closed assignee:<login> sort:updated-desc
```

open PR も必要な場合だけ、別途 `is:open` で追加確認する。

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

### 3. 各 PR の actual review feedback を取得する

PR ごとに以下を取得する。

- `reviews.nodes[].body`
- `reviews.nodes[].comments.nodes[]`
- `comments.nodes[]`

成功した手順で使った取得フィールド:

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

### 4. コメントを 3 種類に分ける

出力では、必ず次の 3 区分に分ける。

- Review bodies
- Inline review comments
- Actionable conversation comments

分類ルール:

- review body: `reviews.nodes[].body` のうち、空でないもの
- inline review comment: `reviews.nodes[].comments.nodes[]`
- conversation comment: `comments.nodes[]` のうち、実務上の指摘として意味があるものだけ

### 5. ノイズを除外する

成功した手順では、次を除外対象にした。

- PR author 自身の review body や comment
- bot のコメント
- Cloudflare などの deploy 通知
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
docs/<assignee>-review-feedback-<YYYY-MM-DD>.md
```

推奨セクション構成:

```text
# Review Feedback Investigation for <login>

- Repository: <owner>/<repo>
- Assignee: <login>
- Generated at: <date>
- PR count: <count>

## PR #<number> <title>

- state: <state>
- author: <login>
- url: <url>

### Review bodies

### Inline review comments

### Actionable conversation comments
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

この skill の目的は「分析用の元データ作成」であり、要約は別タスクに分ける。

## Output Rules

- PR description は含めない
- 実際のレビューコメントだけを載せる
- comment type ごとに分ける
- 原文を改変しない
- ノイズ除外ルールは冒頭に明記する
- open PR を含めた場合は、その理由を冒頭に明記する

## Example Invocation

```text
/github-review-feedback botica-ai-studio/idea-hub-v2 nicole208 2026-03-17 include-open
```

## Expected Result

期待する成果物は 1 つの Markdown ファイル。

- 保存先: `docs/<assignee>-review-feedback-<YYYY-MM-DD>.md`
- 中身: PR ごとに整理された actual review feedback
- 用途: recurring themes 抽出、未修正指摘の確認、PR 前チェックリスト化
