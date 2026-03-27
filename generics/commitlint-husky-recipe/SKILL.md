---
name: commitlint-husky-recipe
description: Recipe for enforcing conventional commit messages with Husky commit-msg hook and commitlint. Use when standardizing commit semantics across frontend repositories.
---

# Commitlint Husky Recipe

## Goal
Enforce consistent commit messages at `commit-msg` stage.

## Extracted Pattern
From `shared-ui`:
- `.husky/commit-msg` runs:
```bash
npx --no -- commitlint --edit $1
```
- `commitlint.config.js` extends `@commitlint/config-conventional`
- custom `type-enum` defines allowed commit types

## Dependencies
- `@commitlint/cli`
- `@commitlint/config-conventional`
- `husky`

## Runbook
1. Install dependencies:
```bash
npm install -D @commitlint/cli @commitlint/config-conventional husky
```
2. Add config (`commitlint.config.js`) with type policy.
3. Add `.husky/commit-msg` hook to execute `commitlint --edit $1`.
4. Validate:
```bash
npx commitlint --from HEAD~1 --to HEAD --verbose
```

## Expected Outcome
Invalid commit messages are blocked before commit is finalized.
