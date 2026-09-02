/*
 *  Created On: 2026-08-26
 *  Author: Jonathon Hicke
 * 
 *  Last Modified: 2026-08-26
 *  Modified By: Jonathon Hicke
 * 
 *  Copyright 2026 © Cornell Pump Company, All Rights Reserved
 */
---
applyTo: "IntegrationTests/**/*.py"
---

# IntegrationTests review rules

QA tests that call the **live deployed API** with real Cognito credentials. Not unit tests.

- **Never suggest mocking or patching.** Hitting the real API is the point.
- **Every test needs two markers. Flag either one missing.**
  - `@mark.test_env(...)` — which environments it runs in: `"DEV,PROD"`, `"DEV"`, or `"PROD"`.
  - `@mark.parallelBlock1` **or** `@mark.sequential` — use `sequential` if the test mutates shared
    company data.
- **Order parametrized cases by authorization level**: RPM admin → company admin → basic user →
  unauthenticated.
- **Consistency check for status codes** — flag a `param(...)` whose `id=` ends in a status code
  that disagrees with the `STATUS_*` constant in the same block, e.g. `id="... - 201"` alongside
  `STATUS_OK`.
- **Naming here is resource- and verb-based**, not function-based. Classes are `Test_<Resource>`
  (`Test_AlertLog`, `Test_Assetgroup`); test functions are named for the HTTP verb
  (e.g., `test_getRequest` / `test_postRequest`) and should keep casing consistent within a file. The scenario belongs in the `id=` label, not the function name.
