---
name: stylelint-config-recipe
description: Reusable Stylelint recipe for SCSS/CSS codebases with Prettier integration. Use when setting up or validating style lint architecture in frontend projects.
---

# Stylelint Config Recipe

## Goal
Standardize style linting for SCSS-first repositories with one reusable baseline and profile-level overrides.

## Source Basis
This recipe is extracted from:
- `/Users/arturfilippovskij/Desktop/code/rg/ksk-ki/.stylelintrc.json`
- `/Users/arturfilippovskij/Desktop/code/rg/shared-ui/.stylelintrc.json`

## Shared Baseline (Both Repositories)
- `extends: stylelint-config-standard-scss`
- `plugins: stylelint-prettier`
- `prettier/prettier: true`
- `color-named: never`
- `color-no-invalid-hex: true`
- `selector-class-pattern: null`
- `declaration-empty-line-before: null`
- `at-rule-empty-line-before: null`
- `max-nesting-depth: 4` with:
  - `ignore: blockless-at-rules, pseudo-classes`
  - `ignoreAtRules: include`

## Profiles

## 1) `ui-library-strict`
Recommended when CSS architecture quality is critical (component libraries).
- `no-descending-specificity`: warning mode
- plus shared baseline
- optional compatibility relaxations:
  - `scss/no-global-function-names: null`
  - `media-feature-range-notation: null`
  - `value-keyword-case: null`

## 2) `scss-relaxed`
Recommended when legacy SCSS needs gradual adoption.
- `no-descending-specificity: null`
- plus shared baseline

## Dependencies
- `stylelint`
- `stylelint-config-standard-scss`
- `stylelint-prettier`
- `prettier`

## Runbook
1. Install dependencies:
```bash
npm install -D stylelint stylelint-config-standard-scss stylelint-prettier prettier
```
2. Add `.stylelintrc.json` using baseline + selected profile.
3. Add scripts:
```json
{
  "lint:scss": "stylelint 'src/**/*.scss'",
  "lint:scss:fix": "stylelint 'src/**/*.scss' --fix"
}
```
4. Optional staged integration:
```json
{
  "*.scss": ["stylelint --fix"]
}
```

## Validation Checklist
- Stylelint runs on intended glob only.
- Prettier plugin is active and deterministic.
- Selected profile matches codebase maturity (`ui-library-strict` vs `scss-relaxed`).
