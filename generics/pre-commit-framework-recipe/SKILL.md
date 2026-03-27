---
name: pre-commit-framework-recipe
description: Declarative pre-commit framework recipe for polyglot repositories with stage-aware hooks (pre-commit and pre-push). Use when repositories need non-Node quality gates or mixed toolchains.
---

# Pre-commit Framework Recipe

## Goal
Define reproducible multi-language git hooks via `.pre-commit-config.yaml`.

## Extracted Pattern
From `ksk-ki`:
- baseline hygiene hooks (`check-yaml`, `trailing-whitespace`, `end-of-file-fixer`, `check-added-large-files`)
- language hooks (e.g., `ruff`, `ruff-format`, local `stylelint`)
- dedicated `pre-push` stage for heavy tests (coverage/test command)

## Dependencies
- `pre-commit`
- language toolchain binaries used by configured hooks

## Runbook
1. Install framework:
```bash
pip install pre-commit
```
2. Install git hooks:
```bash
pre-commit install
pre-commit install --hook-type pre-push
```
3. Validate all configured hooks:
```bash
pre-commit run --all-files
```

## Design Rules
- Keep fast checks on `pre-commit`.
- Keep expensive checks on `pre-push`.
- Pin hook versions (`rev`) for reproducibility.
- Prefer local hooks for project-specific lint/test commands.

## Expected Outcome
Polyglot projects get deterministic, version-pinned hook execution with stage-level performance control.
