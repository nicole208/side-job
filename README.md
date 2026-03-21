# side-job

副業を始めたばかりの人向けに、事務処理、税務、日々の運用で迷いやすい論点を整理していく private repository です。

このリポジトリでは、Markdown を主役にしつつ、必要に応じて PDF や画像を補助資料として管理します。

詳細な構成方針は [docs/repository-structure.md](docs/repository-structure.md) にまとめています。

## 置くもの

- 副業開始時のチェックリスト
- 契約、請求、入金管理、経費管理などの事務処理メモ
- 確定申告、帳簿、税区分などの税務整理メモ
- 公的情報へのリンク集と要点整理
- 参考資料としての PDF や画像

## 運用の基本方針

- 1トピック1Markdownを基本にする
- PDF や画像だけを置かず、必ず要点をまとめた Markdown を添える
- 個人情報、取引先情報、申告書原本などのセンシティブ情報は保存しない
- 税務や制度に関する記述には、対象年度と参照元を明記する

## 現在の構成

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

## 最初に読むドキュメント

- [docs/10_getting-started/opening-checklist.md](docs/10_getting-started/opening-checklist.md)
- [docs/20_admin/invoice-and-payment-flow.md](docs/20_admin/invoice-and-payment-flow.md)
- [docs/20_admin/expense-tracking.md](docs/20_admin/expense-tracking.md)
- [docs/30_tax/final-tax-return-overview.md](docs/30_tax/final-tax-return-overview.md)
- [docs/40_reference/official-links.md](docs/40_reference/official-links.md)

## 追加した詳細ドキュメント

- [docs/20_admin/contract-checkpoints.md](docs/20_admin/contract-checkpoints.md)
- [docs/20_admin/monthly-closing-format.md](docs/20_admin/monthly-closing-format.md)
- [docs/10_getting-started/first-month-tasks.md](docs/10_getting-started/first-month-tasks.md)
- [docs/30_tax/bookkeeping-basics.md](docs/30_tax/bookkeeping-basics.md)
- [docs/30_tax/blue-return-vs-white-return.md](docs/30_tax/blue-return-vs-white-return.md)
- [docs/30_tax/deductible-expenses.md](docs/30_tax/deductible-expenses.md)

## 再利用用テンプレート

- [templates/monthly-closing-checklist.md](templates/monthly-closing-checklist.md)
- [templates/invoice-template-notes.md](templates/invoice-template-notes.md)
- [templates/yearly-tax-prep-checklist.md](templates/yearly-tax-prep-checklist.md)
