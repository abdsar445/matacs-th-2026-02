# AI Proposal Triage: Process Design

| | |
|---|---|
| **Name** | M. Abdullah Sarfraz |
| **Reference** | MATACS-TH-2026-02 |
| **Role** | Associate Software Developer |
| **Time actually spent** | TIME_SPENT |

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

I used Claude (via Claude Code) to analyse the brief, calculate the scores under two formulas, draft each file and render the diagram. I reviewed each step and made the final calls myself, for example choosing to flag C rather than rank it. I disagreed when it proposed two separate layers for catching manipulation, because I thought one strict check should be enough. Working through it, I accepted that no single check can catch every rewording. I kept both layers, but made the write-up rest safety on the AI having no power to score or rank, with detection only sending attempts to a human.
