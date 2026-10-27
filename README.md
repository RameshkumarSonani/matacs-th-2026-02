# MATACS-TH-2026-02 — AI-Assisted Proposal Triage Design

**Candidate:** Ramesh Kumar Sonani  
**Assessment reference:** MATACS-TH-2026-02  
**Time actually spent:** [Enter your actual time before submission; do not estimate or copy a suggested value.]

## Deliverables

- [Process-flow diagram](diagram/process-flow.png)
- [Design write-up](WRITEUP.md)
- [AI prompts](PROMPTS.md)
- [Expected ranking](ranking.json)

## AI tool usage

I used ChatGPT to help structure the design, draft an extraction prompt, and check the scoring logic against the stated criteria. I reviewed and corrected the approach where needed: the proposal text that asks an automated reviewer to rank Proposal C first is treated as untrusted document content, not as an instruction; Proposal B does not pass the bidder-held licence rule merely because it plans to use a subcontractor. Before submitting, I will verify every assumption, calculation, and file against the assessment brief and make sure I can explain the design myself.

## Design summary

The design uses AI for structured extraction from untrusted proposal documents and uses deterministic application rules for mandatory checks, scoring, ranking, and audit logging. Missing or conflicting evidence is surfaced for human review rather than invented. The human reviewer retains the final award decision.

## Assumptions to review before submission

- The brief does not provide detailed score bands for each weighted criterion. `WRITEUP.md` therefore defines transparent, repeatable normalization formulas as explicit design assumptions. In a production system, admins would configure the scoring rubrics in the category screens, and the application would validate that weights total 100%.
- A proposal that does not meet a mandatory requirement is marked `ineligible`. A proposal that cannot yet be assessed because a required fact is missing is marked `flagged` and is not ranked until reviewed.
- Scores in `ranking.json` are illustrative results from the documented formulas, not a claim that the brief itself specified those exact formulas.

## Before you submit

1. Replace the time-spent placeholder with the time you actually spent.
2. Read and understand each file; adjust anything that does not reflect your own reasoning.
3. Create a public GitHub repository named exactly `matacs-th-2026-02`.
4. Commit the required files and push them to GitHub. Commit as you go, as requested.
5. Submit only the repository URL through the Google Form. Do not submit this folder as a ZIP.
