---
name: knowledge-base-maintainer
description: Maintain this repository's engineering knowledge base by classifying, adding, reorganizing, deduplicating, and cross-linking Markdown content. Use when Codex needs to create or revise knowledge notes, place material in the correct topic directory, update README indexes, repair internal links, consolidate overlapping documentation, or review knowledge-base structure and consistency.
---

# Knowledge Base Maintainer

Keep knowledge easy to find without creating duplicate sources of truth.

## Workflow

1. Read the repository `AGENTS.md`, `.agents/constraints.md`, the root `README.md`, and the nearest topic `README.md`.
2. Search filenames and contents for the target subject before creating a document.
3. Choose the existing canonical document when one exists. Create a new document only when the subject has a distinct long-term scope.
4. Place new content in the narrowest existing topic directory that owns it.
5. Make the smallest coherent edit. Preserve unrelated content and user changes.
6. Update affected indexes and relative links after adding, moving, or renaming content.
7. Verify changed links and inspect the final diff. Report any validation that could not be run.

## Organization Rules

- Treat directory `README.md` files as navigation, not as copies of detailed articles.
- Prefer links and short summaries over duplicated paragraphs.
- Preserve established terminology and naming unless the user requests a taxonomy change.
- Separate facts, procedures, checklists, templates, and personal plans when their maintenance cycles differ.
- Do not modify binary engineering project files as part of documentation cleanup.

## Editing Rules

- Preserve the source encoding and use UTF-8 for new Markdown files.
- Confirm encoding before treating terminal mojibake as file corruption.
- Keep headings descriptive and links relative within the repository.
- Do not perform bulk renames or broad rewrites without an explicit request.
