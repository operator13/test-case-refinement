# Case template and field rules

Field names can differ per project; the **roles** must exist. Markdown shown; the same fields map to TestRail, Xray, Zephyr, qTest, Azure Test Plans, a spreadsheet or a Gherkin feature file (mapping table at the end).

```markdown
### <ID> · <Surface> - Verify <observable behaviour>

| Field | Value |
|---|---|
| Status | Draft | Ready | Backlogged: <ticket>, <who>, <date> | Out of scope: <reason>, <date> |
| Section | <feature area> |
| Class | Happy path | Positive | Negative | Edge | E2E | Performance | Accessibility | Visual |
| Type | Functional | Visual | Performance | Accessibility | Analytics | Regression |
| Priority | High | Medium | Low |
| References | <TICKET-1> <criterion or short reason> · <design node> · <requirement page> |
| State | <starting-state code, if the project uses a state model> |
| Run matrix | <viewports / locales / browsers this case runs on> |
| Automation | Needs automation built: <what the automation must capture and assert> |

**Preconditions**
- Fresh browser: quit and reopen the browser so no cookies or site data remain (automation: a new browser context).
- <each setup action spelled out inline, with the exact path and controls>
- <test data stated as criteria; "Record it." where the tester chooses>
- Start page loaded; <a visible landmark that proves it loaded>.
- Page URL – https://[env]example.com/<path>

**Steps**

| # | Step | Expected result |
|---|---|---|
| 1 | <one imperative action on a named, visible control> | <one observable result: exact text in quotes, URL, value, count> |
| 2 | ... | ... |

**Notes**
- <examples of valid data, variants, known environment quirks, history>
```

## Field rules

| Field | Rule |
|---|---|
| Title | Surface first, then "Verify" + the observable behaviour. No ticket IDs in titles. |
| Status | Only the owner sets Backlogged / Out of scope; include who and when. |
| Class | Negative = the system must NOT do something (bounded window required). Performance = counts/timings with a stated window. |
| References | Every expected result traces to something listed here. Add a ticket here whenever a case proves that ticket's behaviour, even if the case was written for another ticket. |
| Preconditions | Self-contained. No "as in case X". Include environment caveats ("run on a deployed environment; the call is blocked locally"). |
| Steps | One action. Visible label in **bold**. Paths in backticks. Tools named. |
| Expected | Observable and literal. Absence claims bounded ("in 6 minutes", "anywhere in the header"). Recording steps say "<value> recorded". |
| Notes | Examples, caveats, history. Never required for the verdict. |
| Automation | Phrase as what needs automation built, never as manual vs automated. |

## Placeholders

- `[env]` — environment prefix, so the same case runs on dev, QA, staging and production.
- `[SELECTED TEST <THING>]`, `[SELECTED TEST <THING> B]` — tester-chosen data, defined by criteria in Preconditions.

## Mapping to other formats

| Role | TestRail | Xray / Zephyr | Gherkin | Spreadsheet |
|---|---|---|---|---|
| ID, Title | Case ID, Title | Key, Summary | Scenario name (ID as tag) | ID, Title columns |
| References | References field | Links "tests" | `@TICKET-123` tags | References column |
| Preconditions | Preconditions | Precondition issue | `Background` / `Given` | Before you start |
| Steps / Expected | Steps (separated) | Test steps (Action / Expected) | `When` / `Then` pairs | Step + Expected columns |
| Notes | Custom field | Description | Comment lines | Notes column |
| Status | Custom status | Test status / labels | `@backlogged` tag | Can I run it? column |
