---
applyTo: "**/*.{ts,tsx,scss}"
---

# Frontend (React + TypeScript + SCSS) Style Guide

Review ONLY the changed lines in this PR. Flag issues in the categories below and
nothing else. Do NOT comment on architecture, performance, naming of business concepts,
or personal preference. The one place you may go beyond style is **Category 14 (Bugs and
Risks)**, and only for obvious, high-confidence problems.

Older files in this repo predate some of these rules. Only flag lines that were added or
modified in this PR. Never ask the author to churn untouched code (for example old `../../`
imports, short callback parameter names, or existing ternaries on lines the PR did not touch).

Files under `cypress/` and generated files are out of scope.

---

## 1. File Header

Every `.ts`, `.tsx`, and `.scss` file must start with this header:

```
// --------------------------------------------------------------
// Created On: YYYY-MM-DD
// Author: <name>
//
// Last Modified: YYYY-MM-DD
// Modified By: <name>
//
// Copyright YYYY © Cornell Pump Company, All Rights Reserved
// --------------------------------------------------------------
```

- Flag a new file with a missing or malformed header.
- Flag an edited file where `Last Modified` and `Modified By` were not updated in this PR.
- Flag an edited file whose copyright year range does not extend to the current year
  (e.g. `2024 - 2026`) when the file was created in an earlier year.
- Do not apply the comment rules in Category 4 to this header.

---

## 2. Formatting

- Indentation is 2 spaces. Tabs are never permitted.
- Lines must not exceed 120 characters.
- Follow Prettier output (double quotes, trailing commas, semicolons).

### ❌ Flag these
```ts
function getThing(): void {
    doWork();               // 4 spaces — flag
}
```

---

## 3. Naming

| Identifier | Convention | Example |
|---|---|---|
| Variables and functions | camelCase | `lastValidTime` |
| Acronyms | Treated as ordinary words | `apiUrl`, `isoToDate`, `nasaRocket` |
| Constants exported from `src/constants` | UPPER_SNAKE_CASE | `ASSET_POLLING_DELAY_SECONDS` |
| Components and interfaces | PascalCase | `AssetCard`, `Props` |

### ❌ Flag these
```ts
const APIUrl = "..."; // acronym capitalised — flag, should be apiUrl
const last_valid_time = 0; // snake_case — flag
export const maxItems = 10; // constant in src/constants not UPPER_SNAKE_CASE — flag
```

### Callback parameter names
The parameter of a callback passed to an array method (`.find`, `.map`, `.filter`, `.forEach`,
`.some`, `.every`, `.reduce`, ...) must be the full, singular name of the items in the array.
Never a single letter or an abbreviation. Callbacks that compare two items, such as
`.sort((a, b) => ...)`, may use `a` and `b`.

```ts
// ✅ Correct
attributes.find((attribute) => attribute.attributeId === id);

// ❌ Flag
attributes.find((a) => a.attributeId === id);
attributes.find((attr) => attr.attributeId === id);
```

If the full name collides with a variable in the enclosing scope, the suggestion is to rename
the outer variable (e.g. `editedAttribute`), not to shorten the callback parameter.

---

## 4. Comments

- Comments are full sentences in sentence case with punctuation: `// This is a comment.`
- Flag comments that are lowercase, missing a period, or fragments.
- Flag `//word` with no space after `//`.
- Every component and utility must have a one-line `//` comment directly above its definition
  describing what it does. Flag any new component or utility without one.
- Flag `TODO` comments and commented-out code (see Category 13).
- Do not apply this rule to the file header, ESLint directives, or `/* Do nothing. */`.

### ✅ Correct
```tsx
// Renders a card showing the current status of an asset.
export default function AssetStatusCard(props: Props): Component {
```

### ❌ Flag
```tsx
export default function AssetStatusCard(props: Props): Component { // no comment above — flag
```
```ts
// get the asset    <- no capital, no period — flag
```

---

## 5. Functions

- Standard functions use declarations: `function myFunction(): void {`, not
  `const myFunction = () => {`.
- Arrow functions are only for passing callbacks, typically with arguments, as props or
  array-method arguments: `<MyComponent onClick={(value) => myHandler(value)} />`.
- Every named function declares its return type. Components return `Component`.
  `void` is required when nothing is returned.
- Functions inside a component must not return JSX. Only component functions return JSX.
  If a helper wants to return JSX, it should be extracted into its own component.
- Do not create HTML imperatively (`document.createElement`, `innerHTML`, HTML strings).
- Intentionally empty callbacks must contain `/* Do nothing. */` (ESLint `no-empty-function`).
- `eqeqeq` always (`===` / `!==`), and `const` over `let` when never reassigned.

### ❌ Flag these
```tsx
const handleSave = () => { ... };                     // arrow used as a declaration — flag
function getLabel(value: number) { ... }              // missing return type — flag
function renderRow(): Component { return <tr />; }    // helper inside a component returning JSX — flag
const onClose = () => {};                             // empty function without comment — flag
```

---

## 6. Components

- Function component with a default export:
  `export default function Name(props: Props): Component { ... }`
- Access props as `props.x`. Do not destructure props in the signature.
- `import React from "react";` at the top, hooks imported by name.
- Ordering inside a component: all `useState`, `useParams`, `useReduxSelector`, and other
  state at the top; then `useEffect` / `useApi` / `useMemo` and other hooks; then plain
  functions; then the `return`.
- All interfaces and types for a file go at the bottom of the file, below the component.
  The `Props` interface is always last. Optional props use `?`.
- Use `useReduxSelector` / `useReduxDispatch`. Never raw `useSelector` / `useDispatch`.
- Use `deepCopy` from `src/utilities/deepCopy` when cloning editable state.
- Prefer existing shared components (`Card`, `Spinner`, `Toast`, `ConfirmModal`,
  `DoubleConfirmModal`, `SaveChangesModal`, `ErrorBanner`, `AddResourceButton`, `Breadcrumbs`,
  `PaginationControl`, ...) over new hand-written markup that duplicates them.
- Anything fetched must show loading and error states.
- Explicit types everywhere. Flag `any`.
- Utilities in `src/utilities/` are pure, one per file, default-exported, and named after
  the function.
- A page must not import subcomponents from another page's folder. Shared pieces belong in
  `src/components`.

### ❌ Flag these
```tsx
export default function Card({ title }: Props): Component { ... }   // destructured props — flag
import { useSelector } from "react-redux";                          // raw hook — flag
const copy = JSON.parse(JSON.stringify(state));                     // should use deepCopy — flag
import Thing from "src/pages/OtherPage/Thing";                      // cross-page import — flag
```

---

## 7. Conditionals

### Explicit checks
Prefer `if (myVariable !== null)` over `if (myVariable)`. The implicit form treats `null`,
`0`, and `""` the same. The only exception is a variable that is certainly a boolean
(`if (isCar)` / `if (!isCar)` is fine). Flag implicit truthiness checks on non-booleans,
including `&&` guards in JSX on non-boolean values (e.g. `{items.length && ...}`).

### Ternaries (strict)
Ternaries are allowed in exactly two places:
1. Choosing what to render inside JSX.
2. Adding simple logic while constructing an object or JSON string.

Flag every other ternary, including assigning a variable, passing a function argument or a
non-render prop value, returning a value, and picking a value for a setter. Also flag nested
ternaries and ternaries mixed with `if` statements for control flow. Suggest an `if` / `else`
or a small helper function that returns the value.

```ts
// ❌ Flag
const label = isActive ? "On" : "Off";
setStatus(isActive ? "On" : "Off");
return isActive ? "On" : "Off";

// ✅ Allowed
<span>{isActive ? <OnBadge /> : <OffBadge />}</span>
const body = { status: isActive ? "On" : "Off" };
```

### Two-return functions
When a function has exactly two possible return values and one is returned from inside an
`if`, the other must be in an `else`. Early-return guard clauses are fine when there are
more than two outcomes.

```ts
// ❌ Flag
function getPercent(value: number): number {
  if (value < 0) {
    return 0;
  }
  return value;
}

// ✅ Preferred
function getPercent(value: number): number {
  if (value < 0) {
    return 0;
  } else {
    return value;
  }
}
```

### Nested returns in JSX
Do not use a `.map` callback with an `if` and an early `return (...)`. Wrap the item in a
`<Fragment key={...}>` and use `condition && (...)`.

---

## 8. Imports

- Anything outside the current folder is imported with the `src/...` alias
  (`import deepCopy from "src/utilities/deepCopy"`). `./` is only for sibling files.
- Flag NEW `../` relative imports. Do not flag existing ones.
- `import React from "react";` is present in `.tsx` files.

---

## 9. Data Fetching and Storage

- Never call `fetch` directly. Use `useApi` (effect-based) or `apiRequest` (event handlers).
- Use `getApiError(response, defaultMessage)` for 4xx error banners instead of hand-rolled messages.
- URLs are built from the `API` constant: `` `${API}/company/${companyId}/feature` ``.
- Every `localStorage` key is a `LOCAL_STORAGE_<NAME>_KEY` constant in
  `src/constants/localStorageVariables.ts`. Flag any string literal passed to
  `localStorage.getItem`, `setItem`, or `removeItem`, and flag a new key that has no constant.
- Prefer adding a constant in `src/constants/` over inlining magic strings or numbers
  (polling intervals, permissions, feature flags, limits).
- Permission checks go through `userHasPermission` and the constants in
  `src/constants/permissions.ts`, not inline strings.

---

## 10. API Types (`src/types/api/`)

- Every file in this folder is named `<Method><Resource>ResponseBody` or
  `<Method><Resource>RequestBody` and ends in `RequestBody` or `ResponseBody`.
- Each file starts with a leading comment giving the AWS Lambda name that backs the endpoint,
  then the HTTP method, path, and parameters. If the Lambda name is unknown, a `?` takes its
  place.
- Nested interfaces live in the same file. Only the body itself is exported (default export).
  Flag any `export` of a nested type.
- Never import a nested type from another file. A component that needs a piece of a response
  declares its own local interface for it. Flag imports of types from `src/types/api/` other
  than the default body import.
- Flag code that calls an endpoint with no file in `src/types/api/` and defines the request or
  response interface locally in a page or hook. Suggest creating the file instead.

---

## 11. SCSS and Styling

- Never write colours explicitly: no hex codes, `rgb()` / `rgba()` / `hsl()`, or named colours
  (`white`, `black`, `red`, ...). Use a variable from `src/App.scss` (`var(--white)`,
  `var(--border)`, ...) or add a new one there first. This includes the colour inside a
  `box-shadow`. Bootstrap theme variables (`$primary`, `$danger`, ...) are fine.
- Use the App-wide CSS custom properties (`--border`, `--background-close`, `--background-far`,
  `--content-text`, `--muted-text`, `--link`, `--overlay-z-index`, ...) instead of hard-coded
  colours or z-index numbers.
- Use `@include media-breakpoint-up(...)`. Flag `media-breakpoint-down` (the ApexCharts override
  block in `src/App.scss` is the only exception).
- Layouts must work on phones. Flag clearly fixed pixel widths on layout containers.
- Checkboxes use `className="form-check-input form-check-big"`. Flag a dropped
  `form-check-big` unless the space is obviously constrained.
- Keep existing `data-test="..."` attributes on interactive elements. Flag removals. New
  interactive elements should have one.
- SCSS modules are imported as `import styles from "./Name.module.scss"`.

### ❌ Flag these
```scss
.card { color: #fff; }                          // hex colour — flag
.card { box-shadow: 0 0 4px rgba(0, 0, 0, .2); } // colour in shadow — flag
.card { z-index: 1050; }                        // z-index magic number — flag
.card { width: 480px; }                         // fixed width on a layout container — flag
@include media-breakpoint-down(md) { ... }      // down breakpoint — flag
```

---

## 12. Tests

- Every new component folder has a `<Name>.test.tsx` with at least a render-without-crashing
  test using `@testing-library/react`. Flag a new component without one.
- Vitest only picks up `**/*.test.tsx`. Flag a new `*.test.ts` file.
- Vitest globals (`describe`, `test`, `vi`) need no import. Flag imports of them.
- Components that render through a portal (anything built on `Modal`) must mock
  `ReactDOM.createPortal` in `beforeAll` and `mockClear()` it in `afterEach`, following
  `src/components/ConfirmModal/ConfirmModal.test.tsx`.
- Behaviour worth covering gets interaction tests with `@testing-library/user-event`. Mention
  this as a suggestion only when a new component has clear interactive behaviour and no
  interaction test.

---

## 13. Cleanup

Flag any of the following in added lines:
- `TODO` comments
- Commented-out code
- `console.log` calls (and other stray `console.*` debugging)
- Unused imports or variables left behind by the change

---

## 14. Bugs and Risks

This is the only category outside style. Flag only obvious, high-confidence problems in
changed lines. When unsure, stay silent. Do not nitpick. Each finding should say what is
wrong and why in one sentence.

- A `useEffect` / `useApi` dependency array that is clearly missing a value the callback
  uses (including `pollCounter` in polling code), or that causes an obvious infinite loop.
- Timers, listeners, or subscriptions created in an effect without cleanup.
- Direct mutation of React state or Redux state (editable copies should use `deepCopy`).
- A missing `key` prop on mapped elements, or a `key` that is an array index on a reorderable list.
- A new route or page in `App.tsx` that skips the `userHasPermission` or feature-flag gate
  that comparable pages use.
- Tokens, passwords, or other secrets logged, rendered, or stored outside the redux slices.
- `dangerouslySetInnerHTML` with non-constant input.
- A missing `await` on `apiRequest`, or ignoring its `[response, responseBody]` result so an
  error path is never handled.
- Unhandled null from `.find()` results or optional props accessed without a guard.
- Polling or `useApi` requests that fire without an `openingFn` guard on missing ids.

---

## Output Format

For each issue found, respond with:

- **File**: `path/to/file.tsx`
- **Line**: 42
- **Category**: File Header | Formatting | Naming | Comments | Functions | Components | Conditionals | Imports | Data Fetching and Storage | API Types | SCSS and Styling | Tests | Cleanup | Bugs and Risks
- **Issue**: one-sentence description
- **Suggestion**: minimal fix

If no issues are found in a category, output: `✅ No [Category] issues found.`
