---
name: generic-git-hooks
description: Universal git hooks recipe for JavaScript, TypeScript, and polyglot repositories. Use when initializing, auditing, or reusing pre-commit and pre-push quality gates across Husky/lint-staged or Python pre-commit framework setups.
---

# Generic Git Hooks Skill

## Goal
Provide one reusable automation pattern for two common git-hook stacks:
- `husky + lint-staged` (Node-first repos)
- `pre-commit` framework (polyglot repos with Python, JS, SCSS, YAML, etc.)

## Detection Strategy
Select hook stack using this order:
1. If `.pre-commit-config.yaml` exists -> use `pre-commit` framework profile.
2. Else if `package.json` has `prepare: husky` or `.husky/` exists -> use Husky profile.
3. Else bootstrap Husky profile for Node repos.

## Profile A: Husky + lint-staged
Recommended for Node/TS repos with staged-file optimization.

### Typical Hook Chain
- `.husky/pre-commit` -> `npx lint-staged`
- Optional `.husky/commit-msg` -> `commitlint --edit $1`

### Staged Rules (recommended baseline)
- `*.{ts,tsx,js,jsx}` -> `eslint --fix`
- `*.scss` -> `stylelint --fix` (if SCSS exists)
- `*.{json,md,yml,yaml,html}` -> `prettier --write`

## Profile B: pre-commit Framework
Recommended for polyglot repos where Python tooling already exists.

### Typical Hooks
- `pre-commit-hooks` (`check-yaml`, `end-of-file-fixer`, `trailing-whitespace`, `check-added-large-files`)
- language-native linters/formatters (e.g., `ruff`, `ruff-format`, `stylelint`)
- `pre-push` stage for slower checks (e.g., test suite, coverage report)

### Why
Centralizes hooks in one declarative YAML with per-stage control and deterministic versions.

## Dependencies Matrix
- Husky profile: `husky`, `lint-staged`, `eslint`, `prettier` (+ `stylelint` optional)
- pre-commit profile: `pre-commit` CLI + language tools referenced by hooks

## Runbook
### Husky Profile
1. Install dependencies:
```bash
npm install -D husky lint-staged eslint prettier
```
2. Initialize hooks:
```bash
npm run prepare
```
3. Ensure pre-commit hook calls lint-staged:
```bash
npx lint-staged
```
4. Validate by creating a test commit and verifying staged files are auto-linted/formatted.

### pre-commit Profile
1. Install pre-commit:
```bash
pip install pre-commit
```
2. Install hooks:
```bash
pre-commit install
pre-commit install --hook-type pre-push
```
3. Run all hooks:
```bash
pre-commit run --all-files
```

## Expected Outcome
Every commit (and optionally push) runs project-appropriate quality gates with predictable, reproducible tooling.
