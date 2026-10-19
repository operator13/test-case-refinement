# Ticket ↔ case traceability audit

Run before the first execution of a release and after every full run. Answers: "Is every ticket backed by at least one passing case on this build?"

## Inputs
- Ticket list (tracker filter for the release, including defects).
- Case library (all sections, with References and Status fields).
- Latest run results on the build under test (one verdict per case).

## Procedure
1. **Extract references.** For every case, collect every ticket key in its References field *and* anywhere in its body (notes and status lines often cite tickets). Also read the results' note column; runs often record the ticket a result was judged against.
2. **Join with results.** For each ticket, list cases by verdict.
3. **Classify each ticket:**

| Coverage class | Condition | Action |
|---|---|---|
| **Covered** | ≥1 PASS case cites it | None |
| **Covered, uncited** | No case cites it, but a PASS case checks its behaviour | Add the ticket to that case's References |
| **Backlog only** | Its only cases are BACKLOGGED | It is not passing; any "passed" claim for the ticket must name the deferred defect |
| **Retired coverage** | Its cases were moved out of scope (often after the fix landed) | Write a live regression case for the fixed behaviour |
| **Retest only** | Proven only by an ad-hoc retest comment, no case | Turn the retest's rows into a case |
| **Uncovered** | Nothing checks it | Write a case (SKILL §5), run it |
| **Failing** | ≥1 FAIL case cites it | Not ready; report |

4. **Verify "covered, uncited" claims by reading the case**, not by keyword: confirm the steps exercise the exact behaviour the ticket changed (the label, the link destination, the punctuation, the count).
5. **Write and run** the new cases; add references; record the first verdicts.
6. **Re-check any status claim already made** (tracker comments, reports) against the audit. Correct overstatements the same day, in place, with what is actually proven and what is still open.

## Keyword search tips
- Search case bodies for the ticket's distinctive nouns (control labels, error text, product names, endpoint names), not only the key.
- A ticket titled after a symptom ("link opens a 404") is covered by any case that clicks that link and asserts the destination.
- Defect tickets often list the cases that found them in their description; read it.

## Output
A table: Ticket | Coverage class | Passing cases | Other cases (verdict) | Action taken. Plus the list of references added and new cases with their first verdicts.
