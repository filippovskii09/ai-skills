---
name: eslint-config-recipe
description: Universal ESLint recipe for JavaScript and TypeScript repositories. Use when setting up or validating lint architecture, automatically selecting JS-only, TS, or hybrid profiles.
---

# ESLint Config Recipe

## Goal
Provide one reusable linting recipe that adapts to project language mode:
- JavaScript-only
- TypeScript-enabled
- Hybrid (both JS and TS source files)

## Detection Strategy (Mandatory)
Select profile using this order:
1. **TypeScript profile** if at least one condition is true:
   - `tsconfig.json` (or `tsconfig.*.json`) exists;
   - `typescript` is present in `dependencies` or `devDependencies`;
   - source contains `*.ts` or `*.tsx`.
2. **JavaScript profile** if no TS signals are found.
3. **Hybrid profile** if both JS (`*.js`, `*.jsx`) and TS (`*.ts`, `*.tsx`) are present in source.

## Profiles

## 1) JavaScript Profile
Use when project has only JS/JSX.

### Base
- `@eslint/js` recommended
- `eslint-plugin-react-hooks` (if React)
- `eslint-plugin-react-refresh` (if Vite React)
- `eslint-plugin-prettier` + `eslint-config-prettier`

### Core JS Rules
- `no-unused-vars` with `_` ignore pattern
- `no-undef`
- `no-console` as warning (optional policy)
- `react-refresh/only-export-components`

## 2) TypeScript Profile
Use when TS signals are detected.

### Base
- `@eslint/js` recommended
- `typescript-eslint` strict + stylistic type-checked
- parser project references from `tsconfig*`
- `eslint-plugin-react-hooks`
- `eslint-plugin-react-refresh`
- `eslint-plugin-prettier` + `eslint-config-prettier`

### Core TS Rules
- `@typescript-eslint/no-unused-vars` with `_` ignore pattern
- `@typescript-eslint/no-explicit-any`
- `@typescript-eslint/no-floating-promises`
- `@typescript-eslint/require-await`
- `react-refresh/only-export-components`

## 3) Hybrid Profile
Use when JS and TS coexist.

### Approach
- Apply shared base config to all source files.
- Apply TS type-aware overrides only to `*.ts`/`*.tsx`.
- Apply JS rules to `*.js`/`*.jsx`.

### Why
This avoids over-constraining JS files with TS-only rules and keeps strictness for typed code.

## Dependency Matrix
- Always: `eslint`, `@eslint/js`, `eslint-plugin-prettier`, `eslint-config-prettier`
- React repos: `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`
- TS repos only: `typescript-eslint`, `typescript`

## Runbook
1. Detect profile (JS/TS/Hybrid).
2. Apply matching config blocks.
3. Execute lint:
```bash
npm run lint
```
4. Auto-fix safe issues:
```bash
npm run lint:fix
```
5. Validate no TS parser project drift (TS/Hybrid only).

## Detection Command Example
Use this shell logic to classify project type before generating config:

```bash
has_tsconfig=$(ls tsconfig*.json >/dev/null 2>&1; echo $?)
has_ts_dep=$(node -e "const p=require('./package.json'); const d={...p.dependencies,...p.devDependencies}; process.exit(d.typescript?0:1)")
has_ts_files=$(rg --files src -g '*.ts' -g '*.tsx' | head -n 1)
has_js_files=$(rg --files src -g '*.js' -g '*.jsx' | head -n 1)

if [ -n \"$has_ts_files\" ] && [ -n \"$has_js_files\" ]; then
  echo hybrid
elif [ \"$has_tsconfig\" -eq 0 ] || [ \"$has_ts_dep\" -eq 0 ] || [ -n \"$has_ts_files\" ]; then
  echo typescript
else
  echo javascript
fi
```

## Validation Checklist
- Profile selection matches actual source/dependencies.
- TS rules are not applied to JS-only files.
- JS rules remain active in Hybrid mode.
- `lint` and `lint:fix` are stable in CI/local runs.
