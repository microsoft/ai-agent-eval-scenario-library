# Evaluation Metrics Aggregation Guide

How to combine individual test case scores into meaningful, actionable metrics that track agent quality over time.

---

## The Problem

You have test cases across multiple scenarios — knowledge grounding, tool invocations, safety, tone — each producing scores on different scales (binary pass/fail, 1–5 Likert, percentage match). How do you combine them into a single quality picture without losing the signal that tells you *where* to fix things?

This guide covers: normalizing scores across evaluator types, aggregating within and across scenarios, setting thresholds, and monitoring trends in production.

---

## Step 1: Normalize Scores to a Common Scale

Different test methods produce different score types. Before aggregating, normalize everything to a **0–1 float range**.

| Score Type | Example Source | Normalization |
|-----------|---------------|---------------|
| Binary (pass/fail) | Exact match, contains check | 0 or 1 — no change needed |
| Binary (LLM judge) | Classification grader (Correct/Incorrect) | 0 or 1 — no change needed |
| Likert (1–5) | Tone quality, helpfulness rating | `(score - 1) / 4` → maps 1→0, 3→0.5, 5→1 |
| Likert (1–3) | Compliance completeness | `(score - 1) / 2` → maps 1→0, 2→0.5, 3→1 |
| Percentage | Keyword coverage, verbatim match % | `score / 100` → already a ratio |
| Multi-label classification | 3 of 4 labels correct | `correct_labels / total_labels` |

> **Rule: Pick one normalization convention and document it.** Mixing unnormalized scores from different evaluator types is the most common aggregation mistake.

---

## Step 2: Aggregate Within a Scenario

Each scenario (e.g., Knowledge Grounding) typically has multiple test cases. Aggregate them into a **scenario-level score**.

### Recommended: Weighted Pass Rate

```
scenario_score = (passed_test_cases / total_test_cases)
```

Where a test case "passes" if its normalized score meets the scenario threshold (see Step 4).

### When to Use Mean Instead

Use the arithmetic mean of normalized scores when you need to distinguish between "barely passing" and "strongly passing":

```
scenario_score = mean(normalized_scores)
```

This is useful for Likert-scale evaluations (tone, helpfulness) where a score of 3/5 vs 5/5 matters — both might "pass" a threshold of 0.5, but the mean reveals quality trends.

### When to Use Minimum Instead

Use `min(normalized_scores)` for **safety-critical scenarios** where any single failure is unacceptable:

```
scenario_score = min(normalized_scores)  # safety, compliance
```

This ensures a single compliance failure or safety breach dominates the aggregate.

---

## Step 3: Aggregate Across Scenarios

Combine scenario-level scores into an **overall agent quality score**. This is where weighting matters.

### Tiered Weighting by Criticality

Not all scenarios are equally important. Group them by business criticality:

| Tier | Weight | Scenarios (typical) |
|------|--------|-------------------|
| **Critical** — failures cause harm or liability | 3x | Safety & Boundary, Compliance & Verbatim, Graceful Failure |
| **Core** — failures break the primary use case | 2x | Knowledge Grounding, Tool Invocations, Trigger Routing |
| **Quality** — failures degrade experience | 1x | Tone & Helpfulness, Response Quality |

```
overall_score = sum(scenario_score * tier_weight) / sum(tier_weights)
```

### Example Calculation

| Scenario | Score | Tier | Weight | Weighted |
|----------|-------|------|--------|----------|
| Safety & Boundary | 0.95 | Critical | 3 | 2.85 |
| Compliance | 1.00 | Critical | 3 | 3.00 |
| Knowledge Grounding | 0.85 | Core | 2 | 1.70 |
| Tool Invocations | 0.90 | Core | 2 | 1.80 |
| Tone & Helpfulness | 0.80 | Quality | 1 | 0.80 |
| **Totals** | | | **11** | **10.15** |

**Overall score: 10.15 / 11 = 0.923**

> **Important:** The overall score is useful for trend tracking, but always drill into the scenario-level breakdown for diagnosis. A high overall score can mask a critical-tier failure. Consider implementing a **hard gate**: if any critical-tier scenario falls below its threshold, the overall score is capped or flagged regardless of the weighted average.

---

## Step 4: Set Thresholds

Thresholds define what score constitutes "passing" at each level.

### Recommended Starting Thresholds

| Tier | Threshold | Rationale |
|------|-----------|-----------|
| Critical scenarios | >= 0.95 | Near-zero tolerance for safety/compliance failures |
| Core scenarios | >= 0.80 | Strong correctness expected for primary functionality |
| Quality scenarios | >= 0.70 | Acceptable tone/quality — room for stylistic variation |
| Overall agent score | >= 0.85 | Weighted floor for production readiness |

### Threshold Calibration Process

1. **Baseline**: Run your eval set against the current agent version. Record scores.
2. **Validate with humans**: Have domain experts review a sample of test cases near the threshold boundary. Do they agree with the pass/fail classification?
3. **Adjust**: If humans say a score of 0.75 on knowledge grounding "feels wrong" as a pass, raise the threshold.
4. **Document**: Record the threshold, when it was set, and the rationale. Update when the agent's scope or audience changes.

---

## Step 5: Track Trends Over Time

A single eval run tells you the current state. Tracking runs over time reveals **quality drift** — gradual degradation that individual runs don't catch.

### Key Trend Metrics

| Metric | What It Reveals |
|--------|----------------|
| **Score delta (run-over-run)** | Immediate regression detection. Alert if any scenario drops > 5% between runs. |
| **Rolling average (last 5 runs)** | Smooths noise from non-deterministic LLM evaluations. |
| **Percentile distribution** | Are most test cases clustering near 1.0, or spreading out? Spreading indicates inconsistency. |
| **Pass rate by scenario** | Which scenarios are improving vs. degrading over time? |
| **Failure concentration** | Are failures spread across test cases, or concentrated in a few? Concentration = specific bug. Spread = systemic issue. |

### Alerting Rules

| Condition | Action |
|-----------|--------|
| Any critical-tier scenario < threshold | Block deployment, trigger immediate investigation |
| Overall score drops > 5% from rolling average | Flag for review before next deployment |
| Any single test case fails 3+ consecutive runs | Create a targeted fix ticket — this is a persistent bug, not flakiness |
| New test cases added with no baseline | Run 3 times to establish baseline before including in gates |

---

## Step 6: Handle Non-Determinism

LLM-based evaluations are inherently non-deterministic. The same test case can score differently across runs. Account for this:

### Run Multiple Trials

For LLM-judged test cases, run each test case **3–5 times** and use the **majority result** (for binary) or **median score** (for Likert scales).

```
# Binary: pass if >= 3 of 5 trials pass
final_result = "pass" if sum(trials) >= 3 else "fail"

# Likert: take median to reduce outlier influence
final_score = median(trial_scores)
```

### Report Confidence

When reporting scores to stakeholders, include the **consistency rate** — what percentage of test cases produced the same result across all trials:

```
consistency_rate = test_cases_with_unanimous_trials / total_test_cases
```

A consistency rate below 80% suggests your evaluation instructions need tightening (see the [Triage Playbook](https://github.com/microsoft/triage-and-improvement-playbook) for grader calibration guidance).

### Pass@k for Agent Tasks

For agents that can retry or take multiple approaches, consider **pass@k** — the probability that at least one of k attempts succeeds:

```
pass@k = 1 - C(n-c, k) / C(n, k)
```

Where n = total attempts sampled, c = successful attempts, k = number of attempts allowed.

**When to use pass@k:**
- Evaluating coding agents or tool-use agents that iterate
- Measuring whether an agent *can* solve a problem, not whether it solves it *every time*
- Comparing agent capability ceilings

**When NOT to use pass@k:**
- Production reliability measurement — users get one attempt, so pass@1 is what matters
- Safety evaluation — a safety failure on *any* attempt is unacceptable

> **Tip:** Report pass@1, pass@5, and pass@10 together. The gap between them reveals how much the agent benefits from retries — a large gap suggests inconsistent reasoning that may be improvable.

---

## Putting It All Together: Eval Results Dashboard

Structure your eval results in this hierarchy:

```
Agent Quality Report — v2.4.1 — 2026-03-15
│
├── Overall Score: 0.914 (threshold: 0.85) ✅
│   Rolling avg (5 runs): 0.910 | Delta: +0.004
│
├── Critical Tier (threshold: 0.95)
│   ├── Safety & Boundary:     0.95 ✅  (12/12 pass, consistency: 92%)
│   ├── Compliance & Verbatim: 1.00 ✅  (8/8 pass, consistency: 100%)
│   └── Graceful Failure:      0.96 ✅  (24/25 pass, consistency: 88%)
│
├── Core Tier (threshold: 0.80)
│   ├── Knowledge Grounding:   0.85 ✅  (17/20 pass, consistency: 85%)
│   ├── Tool Invocations:      0.90 ✅  (9/10 pass, consistency: 90%)
│   └── Trigger Routing:       0.88 ✅  (7/8 pass, consistency: 95%)
│
├── Quality Tier (threshold: 0.70)
│   ├── Tone & Helpfulness:    0.80 ✅  (mean: 0.80, consistency: 78%)
│   └── Response Quality:      0.75 ✅  (mean: 0.75, consistency: 82%)
│
└── Alerts
    ├── ⚠ Tone consistency below 80% — review grader instructions
    └── ⚠ Knowledge Grounding test case #14 failed 3 consecutive runs
```

---

## Common Mistakes

| Mistake | Why It's a Problem | Fix |
|---------|-------------------|-----|
| Averaging binary and Likert scores without normalizing | A binary 1 and a Likert 5 aren't equivalent | Normalize to 0–1 first (Step 1) |
| Weighting all scenarios equally | A safety failure is not equivalent to a tone miss | Use tiered weighting (Step 3) |
| Single-run eval for LLM-judged cases | Non-determinism produces false positives/negatives | Run 3–5 trials per test case (Step 6) |
| Setting thresholds without human calibration | Thresholds may not match what "good enough" means to users | Validate with domain experts (Step 4) |
| Reporting only the overall score | Hides which scenarios are degrading | Always show scenario-level breakdown |
| Never updating thresholds | Agent scope and user expectations evolve | Re-calibrate thresholds quarterly or after major scope changes |

---

## Related Resources

- [Eval Set Template](eval-set-template.md) — Structure for organizing your test cases
- [Worked Example: HR Benefits Agent](worked-example-hr-benefits-agent.md) — Complete eval set with threshold recommendations
- [Triage Playbook](https://github.com/microsoft/triage-and-improvement-playbook) — What to do when scores are low
- [Eval Cost Management](https://github.com/microsoft/triage-and-improvement-playbook/blob/main/guides/eval-cost-management.md) — Strategies for managing evaluation costs at scale
