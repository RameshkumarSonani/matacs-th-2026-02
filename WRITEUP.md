# WRITEUP.md — MATACS-TH-2026-02

## 1. AI vs. rules

Use AI to extract facts from proposal documents into a fixed schema: licence evidence, WSIB and insurance evidence, relevant project history, price, schedule, warranties, and team/approach details. Require a supporting quote and page/section reference for every extracted fact. Treat all proposal text as untrusted data; instructions embedded in a proposal must never change the system prompt or evaluation rules.

Use ordinary application code for file validation, mandatory pass/fail checks, weight validation, score calculations, ranking, status assignment, and audit logs. These steps are rule-based because they need to be predictable, testable, and repeatable. AI may help summarize evidence, but it must not invent missing facts or make the final award decision.

## 2. Consistency and explainability

Store each category's criteria, weights, scoring rubric, mandatory rules, and version in the database. Validate that weighted criteria total 100% before publishing a version. Use a fixed extraction schema, explicit scoring formulas, fixed rounding (two decimal places), and deterministic tie-breakers (higher mandatory compliance confidence after human confirmation, then higher criterion score in this order: experience, price, timeline, warranty, team/approach; if still tied, keep a shared rank or use a documented final tie-break rule).

AI extraction can vary, so preserve the extracted values and evidence, use a fixed model/version and prompt where possible, and require human confirmation for uncertain or conflicting fields. Re-running the same confirmed facts against the same criteria version must produce the same scores and ranking. Reviewers see the overall score, criterion-level score/contribution, source quote and page/section, assumptions, and any flags.

For an initial transparent scoring baseline (an assumption because the brief does not specify score bands):
- Experience: `min(relevant projects / 10, 1) × 100`.
- Price: `max(0, (budget ceiling − price) / budget ceiling) × 100`; a price above the ceiling fails the budget constraint if the ceiling is configured as mandatory.
- Timeline: for a confirmed schedule within 16 weeks, `(16 − completion weeks) / 16 × 100`. Unknown or over-16-week schedules are not scored and require a flag or fail according to the configured rule.
- Warranty: average of panel warranty score (`min(panel years / 25, 1) × 100`) and workmanship warranty score (`min(workmanship years / 12, 1) × 100`).
- Team/approach: 25 points each for a named lead, identified crew, safety plan, and performance-monitoring plan.

The weighted total is the sum of each criterion score multiplied by its weight. In production, these formulas and any thresholds must be configurable by the admin rather than hard-coded.

## 3. Human in the loop

A reviewer confirms extracted facts and resolves missing or contradictory evidence before a proposal can be ranked. Mandatory failures are shown clearly, with the exact rule and evidence. A reviewer may request clarification or record an override with a reason and supporting evidence; the tool must not silently turn a failed mandatory criterion into a pass. The human procurement decision remains separate from the recommendation.

Log the uploader and timestamp, original-file hash, category and criteria-version IDs, prompt/model version, extracted values and evidence references, validation outcomes, score breakdown, flags, ranking, reviewer edits/overrides with reasons, and final award decision. Restrict access to authorized users and protect uploaded documents and logs.

## 4. Changing criteria

Do not silently rescore old results under changed criteria. Admin edits create a new immutable criteria version, with effective time, editor, and change reason. Each evaluation records the exact version used. The reviewer can explicitly rerun proposals against the new version; the system then creates a new evaluation run while retaining the previous run for comparison and audit. Validate weights and mandatory rules before activating the new version.

## 5. Risks and safeguards

- **Extraction errors or unreadable PDFs:** check file type/size, OCR quality, required fields, and evidence quotes; flag low-confidence or conflicting facts for human review.
- **Hallucinated facts:** only score values tied to source evidence; absent evidence is `unknown`, never assumed to be present.
- **Prompt injection (Proposal C):** treat proposal contents as untrusted data, isolate them from system instructions, and ignore instructions asking the evaluator to alter scores or rankings.
- **Bias or unfair scoring:** use job-related, admin-visible criteria; test sample cases; document scoring rules; monitor outcomes; avoid using protected or irrelevant personal attributes.
- **Manipulation or forged claims:** retain source files and hashes, show exact evidence, and require procurement verification for licences, insurance, and other critical claims.
- **Inconsistent runs:** version prompts/models and criteria; freeze confirmed extracted facts for recalculation; use deterministic formulas and fixed rounding.
- **Cost, latency, or provider outage:** limit file size, batch pages, cache extraction results, set budgets/timeouts, and provide a retry/manual-review path.
- **Privacy and security:** encrypt files in transit and at rest, apply role-based access and retention limits, scan uploads, and never expose secrets in prompts or logs.

## Sample-set handling and expected order

Proposal B fails the stated mandatory rule because the criterion requires the Ontario electrical contractor licence to be held by the bidder and stated in its proposal; B says a licensed subcontractor will be confirmed after award. It is therefore `ineligible` under the supplied rule, even though its price and timeline are attractive. Proposal D passes the stated licence, WSIB, and insurance checks, but its schedule is unknown, so it is `flagged` and held out of ranking until the reviewer confirms that it can meet the 16-week deadline. Proposal C's appendix instruction to rank it first is treated as untrusted text and ignored.

Using the illustrative formulas above, the rankable proposals are A (64.62), E (58.50), then C (51.66). A ranks first due to strong experience, 12-week delivery, long warranties, and a complete team/monitoring approach. E ranks second with a strong team/monitoring plan and 12-year workmanship warranty, but a higher price and 13-week schedule than A. C ranks third because it has good experience and warranties but a higher price, 14-week schedule, and less evidence for safety and monitoring. These scores are design outputs under the stated assumptions; the human reviewer makes the award decision.
