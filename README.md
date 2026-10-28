# AI Proposal Triage: Process Design

| | |
|---|---|
| **Name** | M. Abdullah Sarfraz |
| **Reference** | MATACS-TH-2026-02 |
| **Role** | Associate Software Developer |
| **Time actually spent** | About 1 hour building the deliverables (00:07–01:05, 29 Oct 2026), plus earlier time on 28 Oct reading the brief and planning the approach |

## Files

| File | What it contains |
|---|---|
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, plus the assumptions I made |
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow from upload to ranked output: AI steps, rule-based steps and human checkpoints |
| [PROMPTS.md](PROMPTS.md) | The extraction and evidence-repair prompts, exactly as they would be sent, with the call settings and output schema |
| [ranking.json](ranking.json) | Expected ranking of proposals A–E |

There is no optional prototype in this repository. I spent the time on the design and the edge cases instead.

## The design in short

- **The AI has one job:** reading each proposal and extracting facts, each backed by an exact quote. It never scores, ranks or recommends.
- **Plain code does the rest:** it verifies every quote against the proposal text, runs the mandatory pass/fail checks, applies the admin's scoring formulas and ranks the bids.
- **A person decides:** a reviewer resolves every flagged or ineligible proposal, and a named person makes the award. Overrides need a written reason.
- **Everything is logged:** results are versioned, so any ranking can be traced back to the criteria, prompt, model and people that produced it.

**Expected ranking of the sample set:**
- **A** 1st and **E** 2nd.
- **C** is flagged: its text tries to instruct automated reviewers.
- **D** is flagged: it gives no schedule.
- **B** is ineligible: the bidder does not hold the electrical licence.

## How I used AI tools

I used Claude (via Claude Code) to analyse the brief, calculate the scores, draft each file and render the diagram, while I reviewed each step and made the final calls, such as flagging C rather than ranking it. I disagreed with its plan for two separate manipulation checks, because I thought one strict check should be enough. After working through it, I accepted that no single check can catch every rewording, so I kept both but made the write-up rest safety on the AI having no power to score or rank.
