# Mismatch triage

Apply to every step whose seen result differs from the expected result, and to every step that could not be performed.

## Decision tree

```
Could the step be performed at all?
├─ NO
│  ├─ Is the environment reachable, authenticated, deployed? ── NO → E (Environment)
│  ├─ Does the control / URL / menu exist under another name or place?
│  │     YES → W (Case wording). Perform the intended action, judge the requirement.
│  ├─ Did the automation fail where a human could do it?        YES → S (Script/locator). Fix, rerun.
│  └─ Is the intended action impossible to infer?               YES → NOT EXECUTABLE AS WRITTEN
└─ YES, but the result differs
   ├─ Is the expected result sourced (ticket, AC, ruling, design)?
   │     NO  → flag the case (unsourced assertion); ask the owner; do not count either way
   ├─ Does the difference come from data that changed (renamed, resized, discontinued, no longer eligible)?
   │     YES → T (Test data). Choose valid data by criteria, rerun.
   ├─ Could the automation have misread it (wrong element, timing, hidden duplicate, shadow DOM)?
   │     YES/UNSURE → look at the screenshot; read the value another way; rerun fresh.
   │                  Still differs → continue. Was a misread → S.
   ├─ Rerun once in a fresh state. Passes now? → S or E (say which and why), verdict from rerun.
   ├─ Is it the documented symptom of a defect the owner explicitly deferred?
   │     YES → B (Backlogged), name the ticket. If the symptom is gone, PASS and say so.
   └─ Otherwise → D (Product defect). Verdict FAIL. Report now, with cause investigation.
```

## Evidence each class needs

| Class | Minimum evidence |
|---|---|
| D | Both attempts (fresh state each), expected vs seen with the source, screenshot(s) viewed, any value read from state or network, environment + build + time, steps to reproduce. Check whether production/previous build does the same (existing bug vs regression) but report that separately. |
| W | The exact step text, what is actually on screen (label/location/URL), suggested wording, confirmation the requirement itself was met. |
| T | Data originally used, the property it lost, the replacement data and why it qualifies, the observed value now recorded. |
| S | What failed in the script (locator, wait, viewport), proof the product supports the step (manual or alternative read), the fix, the rerun result. Never listed as a defect. |
| E | Error/status observed (HTTP code, timeout), time, what would unblock it. |
| B | Deferred ticket, who deferred it and when, the symptom observed today. |

## Common traps

- **Hidden duplicates**: the same label exists in a closed drawer, a footer or a carousel clone. Filter to visible elements; confirm with a screenshot.
- **Shadow DOM / iframes**: `querySelector` does not pierce them; use role/text locators or a recursive shadow-root walk; switch frames explicitly.
- **Carousels and lazy lists**: scroll or page through before counting; count only rendered, visible cards; state the total.
- **Timing**: assert on an event (URL change, request finished, element visible), not a fixed sleep, except for deliberate idle windows.
- **Redirect after action**: a navigation started by a click may abort a programmatic navigation issued right after; wait for the first to settle, then retry.
- **IP / geolocation defaults**: a location derived from the tester's IP appears when nothing is saved; that is expected unless the case says otherwise. Block or stub it when the case needs a clean default.
- **Consent banners** intercept the first click; dismiss them as part of preparation, both layers if there are two.
- **Analytics events** are pushed on the page being left; hold the page on the click before reading the data layer.
- **Global state reset** between lanes: clearing the whole browser also signs you out of other tools; clear by domain.
- **A passing first run of a new case** may only mean the case is too weak; check that the steps would have failed before the fix.
