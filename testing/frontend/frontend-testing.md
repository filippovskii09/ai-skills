Test Chat Rules
This file defines strict rules for the dedicated testing chat.

1. **Mission**
MUST write and edit tests only.
MUST not change production behavior unless a testability fix is explicitly approved.

2. **Scope**
MUST focus on requested testing task scope only.
MUST explicitly avoid out-of-scope areas

3. **Coverage and DoD**
MUST ensure tests are valid and deterministic.
MUST target at least 90% coverage for the requested target area.
MUST cover positive, negative, and edge cases for target behavior. 
MUST include empty/non-empty states where relevant.

4. **Testing philosophy**
MUST test behavior, not implementation details.
MUST validate user-visible outcomes.
**SHOULD** prefer integration-style component tests over low-value unit internals when both are possible.
**MUST** use isolated unit tests for complex business logic, pure functions, calculations, and specific edge cases where integration tests would be overly cumbersome or fail to provide sufficient coverage depth.
MUST validate initial UI state before user action when the behavior depends on that state (e.g., “button not shown” -> user clicks -> “button appears”).

5. **RTL query strategy (priority)**
MUST use queries in this order:
getByRole / findByRole
getByLabelText, getByPlaceholderText, getByText, getByDisplayValue
getByAltText, getByTitle
getByTestId only as last resort.
MUST NOT use container.querySelector for primary assertions.
SHOULD prefer `within(container)` for assertions when multiple similar elements exist to ensure test targets the correct instance.

6. **Interaction and async rules**
MUST use @testing-library/user-event for user interactions.
MUST use findBy* for async appearance.
MUST use waitFor only with a concrete assertion.
MUST keep side effects outside waitFor.
MUST use queryBy* only for non-existence assertions.

7. **Mocking and test data rules**
MUST avoid mocking internal components unless technically required.
SHOULD mock network on boundary level (MSW preferred where applicable).
MUST globally mock generic browser APIs absent in JSDOM (like `ResizeObserver`) in `setupTests` rather than duplicating setup/teardown in individual test files.
MUST keep business-logic mocks local and deterministic.
SHOULD use centralized fixtures (JSON/TS) for complex response payloads instead of large inline object literals.
MUST keep fixture structure aligned with real API schema.
MUST reuse existing project fixtures/constants/messages when equivalent values already exist.
SHOULD build scenario-specific fixtures via small factory helpers instead of duplicating ad-hoc object literals.

8. **Assertions and readability rules**
MUST prefer readable extraction over positional chains like mock.calls[0][1].
SHOULD use named helpers for call argument extraction when needed.
SHOULD prefer object-level assertions (toEqual, toMatchObject, objectContaining) for payload checks when this improves clarity.
MUST keep explicit field assertions for business-critical or optional/omitted fields where object-level assertions can hide regressions.
MUST align assertions with test intent (for wrapper/factory tests: assert prop forwarding contract first, not mock placeholder markup).
MUST avoid hardcoded mock-only labels/text in assertions when they do not represent real product behavior.
SHOULD extract shared mock labels/selectors into constants if UI presence assertion is still needed.
MUST prefer values from fixtures/mocks/constants/messages over ad-hoc literal strings when asserting mock-rendered content.
Example: use values from `__mocks__/...` unless it is the actual UI text.
SHOULD scope assertions to the correct container using `within` when multiple instances exist, to avoid false positives.

9. **Act usage**
MUST NOT wrap render or standard userEvent flows in manual act.
MUST use act for explicit timer advancement (jest.advanceTimersByTime) and equivalent scheduler control.

10. **Stability and speed**
MUST avoid flaky assertions and race-prone timing assumptions.
SHOULD keep tests small and parallelizable.
SHOULD avoid heavy setup per test when shared setup is possible.

11. **Domain consistency rules**
MUST run a constants/messages discovery step before writing or refactoring assertions.
Check feature-local files first (for example constants.*, messages.*, **mocks**/*).
Search within the target feature for exported constants/enums/messages and reuse them when equivalent.
MUST use project constants/enums when they exist, instead of hardcoded domain strings.
Example: ASSET_CATEGORIES.IMAGE over 'image'.
Example: ASSETS_STATUSES.ACTIVE over 'active'.
MUST use project i18n messages in assertions when UI text is message-driven.
Example: assert by messages['...'].defaultMessage rather than hardcoded column names.
MAY use hardcoded strings only if no constant/message exists; this must be justified in summary.

12. **Style conventions for tests**
MUST follow AAA structure (Arrange, Act, Assert).
SHOULD keep describes shallow.
MUST use descriptive test names based on use case. дабл чек для того аби сам тест відповідав опису, додам приклад
MUST avoid boilerplate cleanup if framework handles it automatically.
MUST NOT assert or query by CSS utility classes (like `.my-3`, `.d-flex`, etc.) as this tests implementation details, not standard behavior.
MUST rely on React Testing Library semantics (roles, text, accessible names) rather than DOM structure traversal.
MUST NOT use DOM travesal elements like `.closest('.utility-class')` or `.parentElement`. Tests should focus on behavior and what the code does, not how the HTML tree is shaped.
