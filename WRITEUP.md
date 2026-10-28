# Write-up

**The design in one line:** the AI reads each proposal and pulls out facts with exact quotes, plain code applies the admin's criteria to those facts, and a person makes the award (see the [diagram](diagram/process-flow.png)).

## 1. AI vs. rules

**AI is used in two places:**
- **Step 4:** extracting facts from each proposal.
- **Step 6:** as a second opinion in the manipulation check.

**Plain code does everything else:** reading files, verifying quotes, the keyword scan, mandatory checks, scoring, ranking, tie-breaks and the reason text.

**Why the line is drawn there:** every bidder writes differently, and understanding free text is what AI is good at. For example, it can tell that a subcontractor's licence is not the bidder's own. Checking, scoring and ranking are policy and arithmetic. Code applies them the same way every time and can be tested and explained line by line. An AI that scores is harder to audit and can be talked into things.

## 2. Consistency and explainability

- **Repeatable results.**
  - Extraction runs with a pinned model version, temperature 0, a versioned prompt and an enforced JSON schema.
  - Each result is cached against the file's fingerprint and those versions.
  - Ranking the same proposals twice reuses the same facts, and scoring is deterministic code, so the result is identical.
- **Robust to formula choice.** Two different scoring formulas (proportional-to-best and capped) give the same order for A, E and C.
- **Explainable to the reviewer.** For each criterion, the reviewer sees the points, the formula, the extracted value and the quote it came from. The one-line reason is built from that breakdown by a template, not written by the AI, so it cannot claim something the scores do not support.

## 3. Human in the loop

**Checkpoints:**
1. **Admin:** approves each criteria version before it can be used.
2. **Reviewer:** every flagged or ineligible proposal needs a human action. The options are to confirm it, correct a field (pointing to the supporting text), request information from the bidder, or override the result. An override needs a written reason.
3. **Award:** a named person makes the decision. The ranking is labelled as advice only.

**What is logged, append-only:**
- criteria and prompt versions, and the model
- file fingerprint and extracted text
- raw AI output, quote checks, flags and scores
- every reviewer action: who, when, what changed and why
- the final decision

## 4. Changing criteria

Criteria are versioned, and each run is locked to one version. Editing creates a new version. Earlier results stay tied to their version and are marked "out of date", never silently changed.

- **A weight or formula change** re-runs only scoring and ranking, in code. That takes seconds and needs no AI call.
- **A new or changed field,** such as a new mandatory criterion, re-extracts only that field.
- **The reviewer sees the old and new rankings side by side** before the new one is used.
- **Fairness:** edits after scoring need a logged reason, since changing rules after seeing bids can look like steering. After an award, changes apply only to future projects.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| **Wrong extraction** | Every fact needs a quote, and code checks the quote really appears in the proposal. Types and ranges are checked. Anything doubtful becomes "unclear" for the reviewer, who sees each value next to its quote. |
| **Made-up facts** | The AI must answer "not stated" instead of guessing. A quote that is not in the text is rejected. A missing value is flagged, never filled in (D). |
| **Manipulation** | Proposal text is passed as data inside a marked block and is never treated as instructions. A keyword scan in code and the AI's integrity check work independently, and either one flags the proposal (C). |
| **Price lures** | The mandatory checks run first, so an ineligible bid's low price never affects anyone's score (B). The AI records facts, not marketing claims. |
| **Bias** | The AI never sees other proposals and never scores tone or writing quality. Every bid goes through the same formulas and the same yes/no checklist, so polish gains nothing. |
| **Cost** | One AI call per proposal, cached. Re-ranking after a weight change is free. File size and page limits cap the cost per proposal. |

**On manipulation,** neither detector is perfect on its own: code misses reworded instructions, and the AI can be fooled by clever ones. So safety does not depend on detection. The AI cannot score or rank, and every fact it returns must match a real quote. An injection that slips past both checks still cannot move a proposal up the list.

## The sample set

A 1st (98), E 2nd (85). C is flagged for the embedded note (77 on merit). D is flagged for the missing schedule, which could place it 2nd to 4th, so the reviewer asks the bidder. B is ineligible on the licence. Reasons are in [ranking.json](ranking.json).

## Assumptions

- **Experience window:** the 5-year window counts back from the scoring date. C's stated period starts earlier, so its count is marked for the reviewer to confirm. A lower count would not change C's position relative to A and E.
- **The embedded note:** it sits after C's text and before D's heading, so it is treated as part of C.
- **The licence:** the bidder itself must hold the licence.
- **Formulas:** formulas are admin settings. I used "best eligible value gets full marks, others in proportion". This depends on which bids are eligible, so it is recalculated and logged on every run.
