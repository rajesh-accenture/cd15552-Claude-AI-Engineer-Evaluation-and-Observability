# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** In the copied `evidence/policy/policies-perturbed-real/POL-2025-001.txt`, I blanked the required total premium (`$1,847.62`); the bundled source file was left unchanged.
- **Command I ran:** `policy-extractor pipeline "$perturbed" --routing-out "$evidence/policy/perturbed-routing-decisions-real.json" 2>&1 | tee "$evidence/policy/perturbation-run-real.txt"`
- **What I predicted:** The extractor would not invent a premium; because the source is missing, the case would be escalated as `missing_source` rather than futilely retried.
- **What actually happened (paste the key output line):** `perturbation-run-real.txt` reports `"premium_amount_absent": {"count": 1, "policies": ["POL-2025-001"], "categories": ["missing_source"]}` and `"escalations": 2`.
- **How this differs from the unperturbed run:** The baseline had 1 escalation and `POL-2025-001` went to `human_review` for a reviewer disagreement on `coverage_limit`. With its premium removed, the copied record is identified as missing-source and the total escalations increase to 2.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** In the copied `evidence/mortgage/perturbed-income-real.txt`, I changed both monthly-total mentions from `10,892.17` to `11,892.17`; the fixture was left unchanged.
- **Command I ran:** `mortgage-extract "$evidence/mortgage/perturbed-income-real.txt" -v 2>&1 | tee "$evidence/mortgage/perturbation-run-real.txt"`
- **What I predicted:** If the extracted components still calculate to `9642.17`, the new stated total of `11892.17` will produce a delta of `-2250.0` and fail consistency validation.
- **What actually happened (paste the key output line):** `perturbation-run-real.txt` reports `"calculated": 9642.17, "stated": 11892.17, "delta": -2250.0` and `"consistent": false`.
- **How this differs from the unperturbed run:** The baseline stated total was `10892.17` and delta was `-1250.0`. Raising the stated total by `1000.00` while the calculated value stayed `9642.17` made the delta `1000.0` more negative.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Enabled the `--simulate-timeout` configuration for the logistics source; no source data was modified.
- **Command I ran:** `supply-chain-investigate meridian --offline --simulate-timeout 2>&1 | tee "$evidence/supply-chain/timeout-run.txt"`
- **What I predicted:** Logistics-dependent claims would be marked incomplete, while the coordinator would finish with claims from the other available sources.
- **What actually happened (paste the key output line):** `timeout-run.txt` says `Sources unavailable: logistics unavailable (timeout)` and marks `late_shipment_count` as missing because of the timeout.
- **How this differs from the unperturbed run:** The normal `briefing.md` includes logistics' `late_shipment_count` of `11.0 shipments` and its `78.0 percent` on-time-delivery value, contested against the audit's `95.0 percent`. During timeout, logistics claims are unavailable; the remaining supplier-audit value is shown alone, and the late-shipment count is incomplete.
