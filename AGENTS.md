# AGENTS.md

## Purpose

This repository is documentation-first. Agents working here should optimize for a maintainable knowledge base, not for application code.

## Primary workflow

1. Read README.md and docs/repository-structure.md before making structural changes.
2. Prefer editing or adding Markdown files.
3. Add binary files only when they support a Markdown document.
4. Keep the repository easy to browse on GitHub.

## File placement

- Put explanatory documents under docs/.
- Put images under assets/images/.
- Put PDFs under assets/pdf/.
- Put reusable checklists and templates under templates/.
- Move outdated material to archive/ instead of mixing it with current guidance.

## Writing conventions

- Write end-user facing documents in Japanese by default.
- Use short sections, explicit headings, and checklists where useful.
- Use ASCII kebab-case for filenames.
- Keep one topic per file.
- When a topic changes over time, note the applicable year and last review date.

## Guardrails

- Do not commit personal data, confidential client data, credentials, or original filing documents.
- Do not add PDFs or images without linking them from a Markdown document.
- Do not rewrite the repository into a code-centric structure.
- Do not remove existing context from README.md or structural guidance without replacing it.

## Expected updates when adding content

- Add the new Markdown file in the correct category.
- Update related index or overview files if navigation changes.
- Keep links working after moves or renames.
- Preserve the repository's role as a reusable reference for side-job administration and tax preparation.