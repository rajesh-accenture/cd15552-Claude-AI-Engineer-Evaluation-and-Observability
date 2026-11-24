# Reflection Brief — Evaluation and Observability Capstone

**Name:** labsuser
**Date:** 2026-11-24

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste
> it from your artifacts — a reviewer should be able to find it. Answers that are correct in the
> abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Ubuntu 20.04.6 LTS |
| Python version | 3.11.18 |
| Date run | 2026-11-24 |
| Ran any system live? (which) | Policy pipeline (live Vocareum run); mortgage perturbed-document run (live API); supply-chain investigation (offline) |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 48 |
| Routing output file | `evidence/policy/routing_decisions.json` |
| auto_approve / human_review / spot_check counts | 0 / 8 / 1 |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation
record. How many API calls did the system make, and why is retrying a futile case worse than
escalating it?

> In `evidence/policy/perturbation-run-real.txt` and `evidence/policy/perturbed-routing-decisions-real.json`, the copied `POL-2025-001` document with its premium removed produced `premium_amount_absent` with category `missing_source`. The missing-source case used one extraction API call and was escalated rather than retried; a retry cannot recover information absent from the source and would add cost and latency without resolving the failure. The perturbed run reported 2 escalations, compared with 1 in the baseline.

**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the
three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's
confidence alone, what would have happened?

> In `evidence/policy/routing_decisions.json`, `POL-2025-001` was routed to `human_review` with `reviewer_disagreements` containing `coverage_limit`. The confidence signals were at or above the threshold, but the independent reviewer disagreed. A confidence-only router could therefore have auto-approved a record with an unresolved coverage-limit issue.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags
its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a
single number hides?

> `evidence/policy/calibration-report.txt` reports `umbrella × exclusions: n=2, conf=0.93, acc=0.00`; `OVERALL brier=0.291`. The slice reveals a high-confidence, consistently incorrect subgroup that an aggregate score alone can hide.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 25 |
| Document run | `evidence/mortgage/extract-run.txt` |
| Classified type | `appraisal` |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the
validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot
catch.

> `evidence/mortgage/discrepancy-run.txt` reports calculated income `9642.17` versus stated income `10892.17`, with delta `-1250.0`. Tool/schema constraints enforce the structured JSON shape and field types; they do not prove the values are arithmetically consistent. The validator catches that semantic mismatch. Schema validation cannot catch a well-formed but incorrect source value, while the arithmetic validator cannot establish that every extracted value was actually stated in the document.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null
instead of an invented value? Point to the schema choice that allows it.

> In `evidence/mortgage/extract-run.txt`, `income.stated_monthly_total` is `null` (as are the income components) for the appraisal. The document does not provide income data, and `mortgage_extractor/schema.py` permits `null` through its nullable field definition; the extractor therefore represents absence instead of inventing an amount.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format
("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> `Build a Resilient Mortgage Document Extraction System/04-validate-mathematical-consistency/solution/fixtures/documents/appraisal_informal_sqft.txt` describes the area as about `2,400 sq ft`; `evidence/mortgage/extract-run.txt` records `gross_living_area_sqft: 2400`. Normalizing at extraction time gives downstream validation and consumers a consistent numeric value rather than making each consumer parse free-form wording differently.

---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 34 |
| Briefing file | `evidence/supply-chain/briefing.md` |
| Section the conflict landed in | `Contested` |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values,
sources, dates. Give one way a reader is better served by the preserved conflict than by a single
reconciled number.

> In `evidence/supply-chain/briefing.md`, `on_time_delivery_rate` is contested: `95.0 percent` from `supplier_audit` as of `2026-04-10`, versus `78.0 percent` from `logistics` as of `2026-04-05`. Preserving both values and their provenance lets a reviewer investigate the difference; a reconciled average would hide the disagreement and imply unsupported precision.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the
failed source. How is "unreachable" handled differently from "nothing to report," and why does the run
still finish?

> `evidence/supply-chain/timeout-run.txt` says `Sources unavailable: logistics unavailable (timeout)` and marks `late_shipment_count` as missing because of the timeout. That differs from `production_capacity_utilization`, for which no source reported a value. The coordinator can still finish using the remaining sources while explicitly marking the logistics-dependent result incomplete.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does
requiring a date stop a time difference from reading as a contradiction?

> `evidence/supply-chain/briefing.md` reports on-time delivery of `95.0 percent` from `supplier_audit` as of `2026-04-10` and `78.0 percent` from `logistics` as of `2026-04-05`. Keeping each observation date visible shows these are measurements from different dates rather than silently treating them as simultaneous. The system still labels them contested instead of assuming the date difference resolves the conflict.

---

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the
output, don't trust the model's word* most clearly caught something a trusting design would have
shipped.

> The policy router in `evidence/policy/routing_decisions.json` is the clearest example: `POL-2025-001` had a reviewer disagreement on `coverage_limit` and went to `human_review` despite confidence being above the threshold. A design that trusted the model's confidence alone could have approved it without the independent review signal.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using
something you observed.

> Confidence mattered most in the policy pipeline. The `coverage_limit` reviewer disagreement for `POL-2025-001` in `routing_decisions.json` shows that a confident extraction can still be wrong or incomplete. The calibration slice in `calibration-report.txt` reinforces this: `umbrella × exclusions` had confidence `0.93` but accuracy `0.00` across two samples.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input.
Which pattern — validated retry with escalation, independent review with deterministic routing, or
provenance-preserving conflict annotation — would you reach for first, and what would you instrument
to know when it broke?

> For an invoice-processing workflow, I would extract line items and totals, enforce a schema, then independently validate arithmetic and route unresolved discrepancies to a human. I would instrument schema/format failures, retry counts by error category, computed-versus-stated deltas, escalation and reviewer-disagreement rates, and calibration by document type and field. Those signals would distinguish a malformed response, a plausible but inconsistent total, and a model that is confidently wrong.
