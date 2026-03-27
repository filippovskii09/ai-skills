---
name: prettier-config-recipe
description: Reusable Prettier formatting recipe for TypeScript and frontend repositories. Use when enforcing consistent code style and integrating format checks in hooks.
---

# Prettier Config Recipe

## Goal
Provide deterministic formatting rules for project-wide consistency.

## Extracted Source (Current Project)
- Config file: `.prettierrc`
- Rules:
  - `semi: true`
  - `trailingComma: es5`
  - `singleQuote: true`
  - `printWidth: 120`
  - `tabWidth: 2`

## Dependencies
- `prettier`

## Runbook
1. Format files manually:
```bash
npx prettier --write .
```
2. Format staged files via lint-staged:
```bash
npx lint-staged
```
3. Keep Prettier plugin order aligned with ESLint integration when both are used.

## Expected Outcome
The codebase keeps stable, predictable formatting across TS, JS, JSON, CSS, and Markdown files.
