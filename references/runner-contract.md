# Runner contract

Any automation stack (or a manual tester) can execute cases under this method if it provides the capabilities below. Implement them once per project; cases stay framework-neutral.

## Required capabilities

| # | Capability | Why |
|---|---|---|
| R1 | **Fresh isolated state per case** (new browser context / session / app install / API token) | Cases must not inherit cookies, storage, caches or saved entities. |
| R2 | **Persistent state on demand** (a named profile that survives close and relaunch) | Persistence cases ("survives a browser restart") need it explicitly. |
| R3 | **Auth from configuration** (HTTP basic auth, SSO test user, API key) loaded by the runner, never printed, never in case text | Secrets stay out of cases and logs. |
| R4 | **Permission and input stubs**: deny geolocation, notifications, camera; optionally block IP-location endpoints; fix the clock where needed | Deterministic starting state; never answer a real prompt for the user. |
| R5 | **Consent dismissal** helper run after every navigation (privacy-preserving choice) | Banners intercept clicks. |
| R6 | **Viewport / device selection** per case | Layout cases run at the stated widths. |
| R7 | **Locators by role and visible text**, with a shadow-DOM-piercing fallback and frame switching | Steps name visible labels; the runner must find them the same way a user does. |
| R8 | **Screenshot capture** (viewport, element, clip) with the naming convention `<ticket-or-case>-<n>-<what>-<YYYY-MM-DD>.png` | Evidence. |
| R9 | **Network capture** with timestamp, URL path, status (and a filter) | Timing, repeated-call and API claims. Strip query strings that carry personal data or IPs. |
| R10 | **State store read** (cookies, local/session storage, IndexedDB, app database, data layer) | Hidden values the case names. |
| R11 | **Long-wait support** without blocking other lanes (background execution) | Idle and timeout checks. |
| R12 | **Result record writer** using `result-schema.json` | Mechanical merging across lanes and runs. |
| R13 | **Retry-once helper** that reruns a case in fresh state and keeps both attempts | Rule 4 of the method. |

## Adapter notes

### Playwright (Node or Python)
- R1: `browser.newContext({ httpCredentials, viewport })` per case; R2: `chromium.launchPersistentContext(dir)`.
- R4: `context.addInitScript` overriding `navigator.geolocation` to call the error callback; `context.route(/location-by-ip/, r => r.abort())`; `context.clearCookies({ domain: /app-domain/ })` to clear only the app.
- R7: `getByRole(role, { name, exact: true })`, `getByText(text, { exact: true }).filter({ visible: true })`; role/text locators pierce open shadow roots; `frameLocator()` for iframes.
- R9: `page.on('response', r => …)` with `Date.now()` deltas; R10: `page.evaluate(() => indexedDB.open(...))`.
- Disabled-button waits: poll `isEnabled()` before clicking; a click timeout on a disabled control usually means a previous selection did not register (S).

### Cypress
- R1: `testIsolation: true` (default) or `cy.session` for deliberate reuse; R3: `cy.visit(url, { auth })` with values from `Cypress.env`, never committed.
- R4: `cy.visit(url, { onBeforeLoad(win) { cy.stub(win.navigator.geolocation, 'getCurrentPosition').callsFake((s, e) => e({ code: 1 })) } })`; `cy.intercept('GET', /location-by-ip/, { forceNetworkError: true })`.
- R7: `cy.findByRole` (Testing Library); `includeShadowDom: true` in config.
- R9: `cy.intercept(/availability/).as('avail')` + `cy.get('@avail.all')` with timestamps recorded in an interceptor callback.
- R11: long idle checks are slow in Cypress; split into their own spec and run in a parallel job.

### Selenium / WebDriver (Java, Python, C#, JS)
- R1: new driver session per case (or delete cookies + clear storage via JS for speed, but prefer a new session).
- R3: basic auth via a proxy, CDP `Network.setExtraHTTPHeaders`, or `https://user:pass@host` only in local config (never logged).
- R4: Chrome prefs `profile.default_content_setting_values.geolocation = 2`; CDP `Emulation.setGeolocationOverride` with an error, or JS override before load via CDP `Page.addScriptToEvaluateOnNewDocument`.
- R7: `By.xpath` on visible text as last resort; `element.getShadowRoot()` (Selenium 4) for shadow DOM; explicit `switchTo().frame`.
- R9: Selenium 4 BiDi / CDP network events, or a proxy (BrowserMob, mitmproxy) recording timestamps.

### Appium (mobile)
- R1: `noReset: false`, `fullReset` when the case needs a clean install.
- R4: deny permissions via capabilities (`autoGrantPermissions: false`) and dismiss dialogs with the deny option; never grant on the user's behalf.
- R6: device/emulator per run matrix; R7: accessibility id = visible label where possible.
- R9: proxy (mitmproxy/Charles) for network claims.

### API / contract tests
- R1: fresh token and fresh test entities per case; clean up after.
- Steps become requests; expected results become status, schema and field assertions; "seen" is the response body excerpt (no personal data).
- R9 is the request log itself; timing claims use server timestamps when available.

### Manual execution
- The contract still applies: fresh profile or private window (R1), DevTools device toolbar for viewports (R6), DevTools Network with Preserve log for R9, Application tab for R10, a stopwatch for R11, the same result record (R12).

## Minimal runner shape (pseudo-code)

```
for case in plan:
    attempt = run(case, fresh_state())
    if attempt.has_mismatch and attempt.mismatch_class in {D, unclear}:
        attempt2 = run(case, fresh_state())
    verdict, classes = triage(attempt, attempt2)
    write_result(case, verdict, attempts, evidence, refinements=[])
    if classes includes W or T:
        propose_refinement(case)   # human/agent edits case text, then rerun
```

`run` executes steps literally, captures per-step `seen`, screenshots and network, and never edits the case.
