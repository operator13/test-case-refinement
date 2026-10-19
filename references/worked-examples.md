# Worked examples

Each example shows the observation, the class, and what changed. They come from a real run and are rewritten with neutral names (a web shop with a product catalogue, saved accounts and a store locator) so they apply to any project.

## 1. Test data drift (T)
- **Case said:** "Account B: a customer whose saved address is in region code 12." Expected: product card shows "Available for delivery".
- **Seen:** the card showed "Available for delivery" with region 14, because the card displays the region that serves the saved address, and the address-to-region mapping had changed.
- **Class:** T (the case stated a specific data value where the requirement only needs a property).
- **Refinement:** "Account B: a customer whose saved address the product can be delivered to. Record the account and the region the card shows." The expected result gained "Region recorded."
- **Not done:** changing the expected message.

## 2. Script locator mistaken for a product failure (S)
- **Seen:** the run reported "Continue button disabled, click timed out" on a four-step picker.
- **Investigation:** the screenshot showed the first list re-selected; the script's "pick the first visible option in the last list" heuristic had clicked an option in the wrong list.
- **Class:** S. Fix: select the option by exact name. Rerun passed. Nothing about it went into the defect list.

## 3. A step that did not navigate (S), caught by an assertion on the next step
- **Seen:** a banner on the store listing named the account instead of the product chosen in the previous step.
- **Investigation:** the recorded URL after "Click View Product" was still the results page: the locator targeted a link role while the control was a button-like element. The product was never asked to do the step.
- **Class:** S. Fix: text locator filtered to visible elements; rerun of that half of the case passed (the banner named the product).
- **Lesson:** record the URL or a landmark after every navigation step so a silent no-op cannot masquerade as a later product failure.

## 4. Coverage that only looked missing (covered, uncited)
- **Audit:** six tickets had no passing case citing them.
- **Reading the cases:** four were exercised exactly by passing cases (a header label, a menu label shown only when a preference is saved, a colon in a banner label, a link destination). Action: references added.
- **Two real gaps:** a repeated-network-call defect (no case at all) and a data defect whose cases were retired once fixed. Action: two new cases written and run.

## 5. A new timing case that would have failed before the fix
- **Ticket:** stock checks were repeated after the cached responses expired (cache expiry about 2 to 3 minutes; before the fix, a new round of checks about once a minute).
- **Case design:** network capture filtered to the endpoint; count at load; idle 6 minutes (twice the trigger, with margin); assert zero new requests; repeat on the second page named in the ticket; check that the dependent UI (the "Buy in store" buttons) is unchanged after the wait.
- **First run result:** 2 calls at load (status 200), 0 in 6 minutes, on both pages; the same buttons before and after. PASS.

## 6. An overstated status claim corrected (traceability step 6)
- **Situation:** a ticket was reported as passed in a bulk comment, but its only cases were backlogged with last result FAIL (an icon that used to be too small is now too large: 30 × 30 px against 18 × 18 and 14 × 18).
- **Action:** the comment was edited the same day: PASS for what the ticket changed (no longer too small), plus a BACKLOGGED row for the still-open size match, naming the deferred ticket.

## 7. Wording vs requirement (W)
- **Case said:** "Click the Account button." The visible label is "My Account" when signed in and "Sign In" otherwise.
- **Class:** W. Refinement: name the visible label for each state. The verdict is judged on the requirement and is unaffected.

## 8. Unexpected failure in a broad run (D, reported immediately)
- **Seen:** reloading a filtered results page reset the list to the unfiltered view.
- **Handling:** rerun in a fresh state, same result; screenshot viewed; reported to the owner at once with the cause investigation; verdict FAIL; case unchanged. Whether to file a ticket is the owner's call.
