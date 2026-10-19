---
name: test-case-refinement
description: "**Test Case Refinement During Execution**: A framework-agnostic method for running written test cases exactly as written, classifying every mismatch (product defect, case wording, test data, script/locator, environment, known backlog), refining case text without weakening it, closing ticket-to-case traceability gaps, writing and immediately running new cases for uncovered tickets, and reporting verdicts with verified evidence. Works with any stack (Playwright, Cypress, Selenium/WebDriver, Appium, API clients, manual). TRIGGERS: refine test cases, run test cases, execute test case, case wording, not executable as written, traceability, coverage gap, ticket coverage, missing test cases, new test case, retest, regression run, flaky test, verdict, PASS FAIL BACKLOGGED, case library audit. Use whenever cases are executed against a build and the case library must stay accurate, or when asked whether tickets are covered by passing cases."
---

# Test Case Refinement During Execution

A written test case is a claim: "if you do these steps on this build, you will see these results." Every execution is a chance to check that claim in two directions at once:

1. **Is the product right?** (the verdict)
2. **Is the case right?** (the refinement)

This skill is the loop that does both without letting one contaminate the other. It is deliberately independent of any project, tool or framework. The project supplies the cases, the environment and the runner; this skill supplies the method.

> Companion files:
> - `references/case-template.md`: the case format this method assumes, with field rules.
> - `references/triage.md`: the mismatch decision tree and the evidence each class needs.
> - `references/runner-contract.md`: what any automation runner must provide, with adapter notes for Playwright, Cypress, Selenium/WebDriver, Appium, API and manual runs.
> - `references/result-schema.json`: the per-case result record every runner writes.
> - `references/traceability.md`: the ticket-to-case audit, coverage classes and how to close gaps.
> - `references/worked-examples.md`: real refinements from a live run, generalized.

---

## 0. Ground rules (never break these)

1. **Run the case as written.** Do not "fix" a step in your head while running it. If the step cannot be done literally, that is a finding about the case (class W, below), not permission to improvise silently.
2. **A bug stays a bug.** Never narrow, split, reword or drop an expected result so that a failing case passes. Refinement may make a case *more precise*; it may never make it *easier to pass* than the requirement.
3. **Separate the run from the edit.** Record what happened first (per-step actuals), decide the class, then edit the case. Never edit the case and the verdict in the same thought.
4. **One rerun before a FAIL.** A failing step is repeated once in a fresh state (new browser context / new session / reset data) before it is called a product failure. Record both attempts.
5. **Look at every screenshot you cite.** Evidence that has not been viewed is not evidence. Captions must describe what the image actually shows.
6. **Only the build under test counts.** Results from another environment, an older build or a previous day are history, not verdicts. Date-stamp everything.
7. **Do not invent.** Expected results come from a source (ticket description, acceptance criteria, product-owner ruling, approved design, requirements page). If a case asserts something with no source, flag it; do not defend it.
8. **New cases get run immediately.** A case that has never been executed is a draft hypothesis. Write it, run it, refine it, then count it.
9. **Secrets and personal data never enter cases or evidence.** No passwords, tokens, public IPs, real customer data. Authentication comes from the runner's configuration, never typed from the case.
10. **Never answer browser permission prompts for the user** (location, notifications, camera). Stub or deny the provider in the runner and state that in the evidence.

---

## 1. The loop

```
        ┌────────────────────────────────────────────────────────┐
        │ 1. LOAD     case + its sources (ticket, AC, rulings)   │
        │ 2. PREPARE  fresh state, preconditions, test data      │
        │ 3. EXECUTE  step by step, literal, record actuals      │
        │ 4. COMPARE  expected vs seen, per step                 │
        │ 5. TRIAGE   classify every mismatch (D/W/T/S/E/B)      │
        │ 6. RERUN    once, fresh, for any D or unclear result   │
        │ 7. VERDICT  PASS / FAIL / BACKLOGGED / NOT EXECUTABLE  │
        │ 8. REFINE   edit the case for W and T findings only    │
        │ 9. TRACE    update ticket <-> case references          │
        │10. RECORD   result record + verified evidence          │
        └────────────────────────────────────────────────────────┘
                 (gaps found in 9 → write new case → back to 1)
```

### 1. Load
- Read the case in full: title, preconditions, start page, steps, expected results, notes, references, status.
- Read every source it cites. For a ticket that means the description, the **separate acceptance-criteria field if the tracker has one**, and every comment (rulings in comments supersede the description; say so when you follow one).
- Note the case's status. Out of scope or retired cases are not run; say why. Backlogged cases are run (see verdicts).

### 2. Prepare
- Fresh state for every case unless the case is explicitly about persistence (then the case says how state is carried).
- Close or decline consent banners first (privacy-preserving choice) so they cannot intercept clicks.
- Stub browser permission providers (deny) and any non-deterministic inputs the case does not control (IP-based location, time, A/B buckets) where the case requires a defined starting state.
- Pick test data **by the criteria the case states** ("an account with no saved payment method"), record exactly what you chose. A case that names one specific data item should be refined to state criteria plus an example (see §4).

### 3. Execute
- One step at a time, in order. Perform exactly the action named, on the control named, using the label the user sees.
- After each step capture: the observable result (text, URL, state store value, request log), a screenshot when the step's expected result is visual, timing when the step is about waiting or performance.
- Never skip a step because a later step "would prove it anyway". Every expected result is judged on its own.

### 4. Compare
Judge each expected result **literally**:
- Exact text means exact text (case, punctuation, colon). "Reads Sign In" fails on "SIGN IN" if the requirement says no all caps.
- Absence claims ("no selector appears", "no new request") need a positive observation window: say how long and where you looked.
- "Matches design" needs a measurement (px, colour value, order), not an impression.
- If the expected result is ambiguous enough that two honest testers could disagree, that is class W even if the step "looked fine".

### 5. Triage
Every mismatch gets exactly one primary class. Full decision tree in `references/triage.md`.

| Class | Meaning | What changes |
|---|---|---|
| **D** Product defect | Build does not meet a sourced expected result; reproduced in a fresh state | Verdict FAIL; case unchanged; defect recorded or linked |
| **W** Case wording | Step cannot be done literally (wrong control name, wrong URL, location of a control, ambiguous expected result, outdated label) but the product meets the requirement | Verdict for the requirement; case text refined |
| **T** Test data | Chosen or prescribed data no longer has the needed property (sold out, renamed, size changed, no longer fits) | Case refined to criteria; rerun with valid data |
| **S** Script / locator | Automation could not perform a step the product supports (locator, timing, viewport, coordinate, iframe/shadow DOM) | Fix the script, rerun; case unchanged; **never** a product FAIL |
| **E** Environment | Outage, auth refusal, missing deploy, rate limit, unreachable dependency | Verdict BLOCKED / NOT RUNNABLE with reason; rerun later |
| **B** Known backlog | Mismatch is the documented symptom of a defect the owner explicitly deferred | Verdict BACKLOGGED naming the deferred ticket |

Rules that keep triage honest:
- **S before D.** When automation fails, first prove the product can do the step (manual repeat, a different locator, a screenshot you look at). Only then consider D.
- **W never hides D.** If the product does not meet the requirement *and* the wording is off, it is D; fix the wording too, but the verdict is FAIL.
- **B only by explicit decision.** A failure becomes BACKLOGGED only when the owner deferred that specific defect. "Probably known" is D.
- **Unexpected behaviour is reported immediately**, with a cause investigation, before it goes into any report.

### 6. Rerun
- Fresh state, same data, same steps. If the second run passes, record both and classify the first as S or E with the reason (load timing, network blip). If the second run fails identically, it is D (or B).
- Intermittent product behaviour (fails 1 of N fresh runs) is reported as D with the rate, not hidden as flaky.

### 7. Verdict
| Verdict | When |
|---|---|
| **PASS** | Every expected result matched (after any S fixes and W refinements that did not change what the requirement asks). |
| **FAIL** | At least one sourced expected result is not met, reproduced in a fresh state. One failing step fails the case. |
| **BACKLOGGED** | The only mismatches are the symptoms of explicitly deferred defects (name them). If the deferred defect is gone, say PASS and say it is gone. |
| **NOT EXECUTABLE AS WRITTEN** | The case cannot be performed literally and the intended check cannot be inferred with confidence. Report with suggested wording; do not guess a verdict. |
| **BLOCKED / NOT RUNNABLE** | Environment or data the tester cannot create prevents the run. Give the specific reason and what would unblock it. |
| **OUT OF SCOPE** | Case is retired or outside the release scope; not run. |

Percent complete = PASS ÷ (total − out of scope). Report backlogged separately; never fold them into PASS.

### 8. Refine
Only W and T findings change the case text. Refinement standards in §3 and §4. Every refinement is:
- **minimal** (change only what was wrong),
- **sourced** (the new wording matches what the product does *and* what the requirement asks),
- **recorded** (a short note on what changed and why, in the run record; the case itself is a document, not a diary).

### 9. Trace
After each run, check the ticket ↔ case map (`references/traceability.md`). Typical findings:
- a ticket is covered by a passing case that does not cite it → add the reference,
- a ticket's only cases are out of scope or backlogged → coverage gap or open defect, say which,
- a ticket has no case at all → write one (§5) and run it.

### 10. Record
Write one result record per case (`references/result-schema.json`) and keep evidence beside it: per-step actuals, screenshots named `<ticket-or-case>-<n>-<what>-<date>.png`, request logs for timing or network claims. A short run note explains reruns and refinements in plain language.

---

## 2. What "refining during execution" catches (checklist)

Run this checklist mentally on every step, every time:

- [ ] **Control name**: does the label in the step match the label on screen exactly (including case and the visible text vs the accessible name)?
- [ ] **Control location**: is the control where the step says (header vs drawer, which dropdown, which tab)?
- [ ] **Navigation**: does the URL or menu path still exist? Does it redirect?
- [ ] **Preconditions complete**: could a new tester reach the start state from the text alone, with no reference to another case?
- [ ] **One action per step**: if a step needs two actions, split it.
- [ ] **Observable expected result**: is each expected result something you can see, read, measure or query? (Not "works correctly".)
- [ ] **Literal match**: exact strings in quotes; numbers with units; colours as values.
- [ ] **Data still valid**: does the prescribed data still have the property the case needs?
- [ ] **Timing claims bounded**: "does not happen" has a window ("in 6 minutes idle").
- [ ] **State location named**: where a hidden value lives (storage key, request, data layer) and how to read it.
- [ ] **Viewport / platform stated** where layout matters.
- [ ] **Variants and caveats in Notes**, not in steps.
- [ ] **Source cited** for every expected result (ticket + criterion, ruling, design node).
- [ ] **No internal shorthand** that a reader outside the team cannot decode in a client-facing case.

---

## 3. Refinement standards (case wording)

1. **Use what the user sees.** Name controls by visible label and physical description ("the red Checkout button at the bottom of the menu drawer"), not by internal component names.
2. **Self-contained.** Spell out setup inline. Never "see case X for setup".
3. **Short imperative steps**, one action each; expected result one line each.
4. **Exact strings in quotes**, including punctuation that a requirement cares about (a missing colon can be the whole defect).
5. **Name the tool** when a step needs one ("DevTools > Network, filter `availability`, Preserve log on").
6. **Explicit start page** with full path, and the environment as a placeholder (`https://[env]example.com/path`) so the case runs on every environment.
7. **Record steps** for values later steps depend on ("Order number recorded").
8. **Bounded negatives**: "No new request appears in 6 minutes" not "no repeated requests".
9. **Caveats, examples and history go in Notes.** Steps stay clean.
10. **When the product legitimately changed** (a ruling, an approved design), update the expected result *and* cite the ruling. When it changed without a source, that is D, not W.

What refinement is **not**:
- replacing a failing expected result with what the build does,
- removing a step that fails,
- splitting a case so the failing half can be deferred while the passing half is counted, unless the owner explicitly split the requirement.

---

## 4. Test data refinement

- **Criteria over named items.** Write the property the data must have ("an item that is out of stock in the selected region", "a postcode with at least 10 stores within 10 mi"), then give a known-good example in Notes. Testers record what they used.
- **Record the observed value** whenever data drives an assertion (size shown, price shown, count shown), because data drifts between builds.
- **When prescribed data stops working**, check whether the product or the data changed. Data change → T (refine). Product change without a source → D.
- **Data a tester cannot create** (needs back-office authoring, a special account) is the only valid reason to block on data. Say exactly what is missing.

---

## 5. Writing a new case for an uncovered ticket

1. **Source the behaviour**: ticket description, acceptance-criteria field, reproduction steps in the ticket, and the rows of any earlier QA verification comment (those rows are proven checks; convert each into steps).
2. **Pick the section** the behaviour belongs to; take the next free ID; follow the library's template exactly (`references/case-template.md`).
3. **Classify it** (happy path / positive / negative / edge / performance / accessibility), set priority, write the automation note as "needs automation built: <what>".
4. **Write steps that reproduce the original defect's conditions** so the case would have failed before the fix. For timing defects include the wait long enough to cross the trigger (e.g. a cache expiry) with margin.
5. **Run it immediately** on the current build. Expect at least one refinement: the first run of a new case usually reveals a data or wording assumption.
6. **Refine, then record** the run (verdict, data used, what changed in the case and why).
7. **Add the reference** on the ticket side of the trace and, if the project keeps one, in the run workbook.

---

## 6. Running at scale

- **Plan the fastest safe run first**: list all cases, group by shared setup and by environment, decide what can run in parallel.
- **Parallelize by independent lanes** (each lane = its own fresh browser contexts / sessions), at most the number the environment tolerates. State does not leak across lanes.
- **Shared session for the tracker is not shared with test lanes.** Keep any signed-in tool session (issue tracker) in its own browser/tab; clear only the application's cookies and storage, never the whole browser profile.
- **Long waits run in the background** (idle and timeout checks), while other cases continue.
- **Quick-check known outcomes**: a case whose outcome was just verified on this build can be spot-checked rather than fully rerun, *only* if nothing relevant changed; say so.
- **Every lane writes the same result schema**, so results merge mechanically.

---

## 7. Reporting

- **Lead with the verdict**, then the counts (n of m pass, how many cannot be tested and why), then failures with expected vs seen.
- **Per-step table** for anything non-PASS: step, expected, seen, class, evidence.
- **Refinements made**: case ID, old wording → new wording, reason (one line each).
- **Coverage changes**: references added, new cases written and their first verdicts.
- **Do not narrate tooling problems as product problems.** Script/locator fixes appear in the run note, not in the defect list.
- Client-facing outputs state the content itself; internal IDs, open-question numbers and tooling chatter stay internal.

---

## 8. Adapting to a new project

1. Map the project's case format to `references/case-template.md` (field names may differ; the roles must exist).
2. Implement the runner contract (`references/runner-contract.md`) once for the project's stack: fresh state, auth from config, permission stubs, banner dismissal, screenshot + network capture, result record writer.
3. Build the ticket ↔ case map (`references/traceability.md`) before the first run, so gaps are known up front.
4. Agree the verdict vocabulary and who may declare BACKLOGGED.
5. Run, triage, refine, trace, record. Repeat every build.
