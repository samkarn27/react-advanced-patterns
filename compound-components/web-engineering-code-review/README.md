# Web Engineering Code Review Standard

This package contains:
- `code-review-instructions.md` — complete, human-readable review standard.
- `.github/copilot-instructions.md` — concise repository-level Copilot guidance.
- `.ai/web-code-review.prompt.md` — explicit prompt for reviewing a diff.

## Install
Copy the files into the root of your repository, preserving their paths. Merge the Copilot instructions with any existing instructions rather than overwriting project-specific guidance.

## Important
Markdown instructions guide AI review but do not enforce merge or commit policy on their own. Enforce deterministic checks such as formatting, linting, type checking, tests, dependency/security scanning, and contract checks through CI and Git hooks. Use AI review as an additional advisory layer.
