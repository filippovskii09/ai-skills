# Frontend Testing Rules

> Rules for the dedicated **testing chat**.
> Enforcement levels: **MUST** = mandatory · **SHOULD** = expected default · **MAY** = optional.

---

## 1. Mission

- **MUST** write and edit tests only.
- **MUST NOT** change production behavior unless a testability fix is explicitly approved.

---

## 2. Scope

- **MUST** focus on the requested testing task scope only.
- **MUST** explicitly avoid out-of-scope areas.

---

## 3. Coverage & Definition of Done

- **MUST** ensure tests are valid and deterministic.
- **MUST** target at least **90% coverage** for the requested target area.
- **MUST** cover positive, negative, and edge cases for target behavior.
- **MUST** include empty/non-empty states where relevant.

---

## 4. Testing Philosophy

- **MUST** test **behavior**, not implementation details.
- **MUST** validate user-visible outcomes.
- **SHOULD** prefer integration-style component tests over low-value unit internals when both are possible.
- **MUST** use isolated unit tests for complex business logic, pure functions, calculations, and specific edge cases where integration tests would be overly cumbersome or fail to provide sufficient coverage depth.
- **MUST** validate initial UI state before user action when the behavior depends on that state.

> **Example:** assert "button not shown" → user clicks → "button appears".

---

## 5. RTL Query Strategy

### 5.1 Always use `screen`

- **MUST** use `screen` for all queries and debugging — do **not** destructure query methods from the `render` return value.
- Exception: `rerender`, `unmount` may be destructured from `render`.

```ts
// ❌
const { getByRole } = render(<Example />)
getByRole('alert')

// ✅
render(<Example />)
screen.getByRole('alert')
```

### 5.2 Query priority order

Use queries in **strict priority order**:

| Priority | Query |
|----------|-------|
| 1 ✅ | `getByRole` / `findByRole` |
| 2 | `getByLabelText`, `getByPlaceholderText`, `getByText`, `getByDisplayValue` |
| 3 | `getByAltText`, `getByTitle` |
| 4 ⚠️ last resort | `getByTestId` |

- **MUST NOT** use `container.querySelector` for primary assertions.
- **SHOULD** prefer `within(container)` when multiple similar elements exist to ensure the test targets the correct instance.

### 5.3 Accessibility attributes

- **MUST NOT** add redundant `role` or `aria-*` attributes to elements that already have the correct implicit ARIA role from their HTML semantics.
- **SHOULD** follow [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) when building non-native UI that genuinely requires ARIA.

```tsx
// ❌ — redundant, <button> already has role="button" implicitly
<button role="button">Click me</button>

// ✅
<button>Click me</button>
```

> If you can't query an element by role without adding an explicit `role`, that's a signal the markup may not be semantic enough — fix the HTML, not the test.

---

## 6. Interaction & Async Rules

- **MUST** use `@testing-library/user-event` for user interactions.
- **MUST** use `findBy*` for async appearance.
- **MUST** use `waitFor` only with a concrete assertion inside.
- **MUST** keep side effects **outside** `waitFor`.
- **MUST** use `queryBy*` only for non-existence assertions.

---

## 7. Mocking & Test Data

- **MUST NOT** mock internal components unless technically required.
- **SHOULD** mock network on boundary level (MSW preferred where applicable).
- **MUST** globally mock generic browser APIs absent in JSDOM (e.g. `ResizeObserver`) in `setupTests` — not in individual test files.
- **MUST** keep business-logic mocks local and deterministic.
- **SHOULD** use centralized fixtures (`JSON`/`TS`) for complex response payloads instead of large inline object literals.
- **MUST** keep fixture structure aligned with the real API schema.
- **MUST** reuse existing project fixtures/constants/messages when equivalent values already exist.
- **SHOULD** build scenario-specific fixtures via small factory helpers instead of duplicating ad-hoc object literals.

---

## 8. Assertions & Readability

### 8.1 Use `jest-dom` semantic matchers

- **MUST** use `@testing-library/jest-dom` matchers instead of checking raw DOM properties — they provide better error messages and express intent more clearly.

```ts
// ❌ — raw DOM property, poor error output
expect(button.disabled).toBe(true)

// ✅ — semantic, clear error message
expect(button).toBeDisabled()
```

Prefer semantic matchers over equivalent raw checks:

| Instead of | Use |
|---|---|
| `.disabled === true` | `.toBeDisabled()` |
| `.textContent === 'foo'` | `.toHaveTextContent('foo')` |
| `getComputedStyle(el).display !== 'none'` | `.toBeVisible()` |
| `el !== null` | `.toBeInTheDocument()` |

### 8.2 General assertion rules

- **MUST** prefer readable extraction over positional chains like `mock.calls[0][1]`.
- **SHOULD** use named helpers for call argument extraction when needed.
- **SHOULD** prefer object-level assertions (`toEqual`, `toMatchObject`, `objectContaining`) for payload checks when this improves clarity.
- **MUST** keep explicit field assertions for business-critical or optional/omitted fields where object-level assertions can hide regressions.
- **MUST** align assertions with test intent.
  - For wrapper/factory tests: assert prop-forwarding contract first, not mock placeholder markup.
- **MUST NOT** use hardcoded mock-only labels/text in assertions when they do not represent real product behavior.
- **SHOULD** extract shared mock labels/selectors into constants if UI presence assertion is still needed.
- **MUST** prefer values from `fixtures` / `mocks` / `constants` / `messages` over ad-hoc literal strings when asserting mock-rendered content.

```ts
// ✅ prefer
expect(someEl).toHaveTextContent(__mocks__/someModule.label);

// ❌ avoid
expect(someEl).toHaveTextContent('Some hardcoded label');
```

- **SHOULD** scope assertions to the correct container using `within` when multiple instances exist, to avoid false positives.

---

## 9. `act` Usage

- **MUST NOT** wrap `render` or standard `userEvent` flows in manual `act`.
- **MUST** use `act` for explicit timer advancement:

```ts
// ✅ correct use of act
act(() => {
  jest.advanceTimersByTime(500);
});
```

---

## 10. Stability & Speed

- **MUST** avoid flaky assertions and race-prone timing assumptions.
- **SHOULD** keep tests small and parallelizable.
- **SHOULD** avoid heavy setup per test when shared setup is possible.

---

## 11. Domain Consistency

Before writing or refactoring assertions, **MUST** run a constants/messages discovery step:

1. Check feature-local files first: `constants.*`, `messages.*`, `__mocks__/**/*`.
2. Search within the target feature for exported constants/enums/messages.
3. Reuse them when equivalent.

**MUST** use project constants/enums instead of hardcoded domain strings:

```ts
// ✅
ASSET_CATEGORIES.IMAGE
ASSETS_STATUSES.ACTIVE

// ❌
'image'
'active'
```

**MUST** use project i18n messages when UI text is message-driven:

```ts
// ✅
messages['someKey'].defaultMessage

// ❌
'Some column name'
```

**MAY** use hardcoded strings only if no constant/message exists — this must be justified in the PR/summary.

---

## 12. Style Conventions

- **MUST** follow **AAA** structure (Arrange → Act → Assert).
- **SHOULD** keep `describe` blocks shallow.
- **MUST** use descriptive test names based on use case. Double-check that the test body actually matches its description.
- **MUST NOT** add boilerplate cleanup if the framework handles it automatically.
- **MUST NOT** assert or query by CSS utility classes:

```ts
// ❌ tests implementation detail, not behavior
container.querySelector('.my-3')
container.querySelector('.d-flex')
```

- **MUST** rely on React Testing Library semantics (roles, text, accessible names) rather than DOM structure traversal.
- **MUST NOT** use DOM traversal like `.closest('.utility-class')` or `.parentElement`. Tests should focus on **behavior**, not HTML tree shape.
