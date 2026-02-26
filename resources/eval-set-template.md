# Eval Set Template

Use this template to build an evaluation set from scenarios you've selected from the library.

---

## Step 1: List Your Selected Scenarios

After browsing the library, list the scenarios you've selected:

| # | Scenario | Source | Why Selected |
|---|----------|--------|-------------|
| 1 | _e.g., Verifying Policy Answer Accuracy_ | _Business-Problem: Information Retrieval_ | _Core use case — agent answers HR policy questions_ |
| 2 | _e.g., Verifying Tool Invocation_ | _Capability: Tool Invocations_ | _Agent calls Power Automate to submit PTO_ |
| 3 | _e.g., Testing PII Protection_ | _Capability: Safety & Boundary_ | _Agent handles employee data_ |
| ... | | | |

**Target:** 6–12 scenarios for a comprehensive eval set. Aim for a mix of business-problem and capability scenarios.

---

## Step 2: Write Test Cases Per Scenario

For each selected scenario, use the Practical Examples table as a starting point. Adapt to your agent's specific context.

### Test Case Template

| Field | Description | Example |
|-------|-------------|---------|
| **Test Case ID** | Unique identifier | `IR-001` |
| **Scenario** | Which library scenario this belongs to | Verifying Policy Answer Accuracy |
| **Category** | Business-Problem or Capability | Business-Problem |
| **Quality Signal** | What you're testing | Factual accuracy of policy answers |
| **Sample Input** | The user message / prompt | "What is the PTO policy for employees with 5+ years?" |
| **Expected Value** | What the correct response should contain | Keywords: "25 days", "annual allowance", "pro-rated" |
| **Test Methods** | Which evaluation methods to use — list all that apply, joined with " + " | Keyword Match (All) + Compare Meaning |
| **Pass Criteria** | When this test case passes — one condition per method | All keywords present in response AND meaning aligns with expected value |
| **Priority** | How critical this test case is | High — core use case |

> **Multi-method test cases:** Copilot Studio supports multiple test methods per test case. Use " + " to combine methods (e.g., `Keyword Match (All) + Compare Meaning + General Quality`). Each method catches a different failure mode — Keyword Match confirms specific facts are present, Compare Meaning validates overall correctness, General Quality assesses structure and completeness. Apply every method that adds value; don't limit yourself to one.

---

## Step 3: Organize by Evaluation Category

Group your test cases to ensure comprehensive coverage:

### Category Checklist

- [ ] **Core Business Scenarios** — Happy-path test cases for your agent's primary use cases
  - Source: Business-problem scenarios
  - Target: 30–40% of test cases

- [ ] **Variations** — Same scenarios with different user contexts, phrasings, edge inputs
  - Source: Business-problem scenarios (personalization, ambiguous queries)
  - Target: 20–30% of test cases

- [ ] **Architecture & Capability** — Infrastructure-level validation
  - Source: Capability scenarios (knowledge grounding, tool invocations, trigger routing)
  - Target: 20–30% of test cases

- [ ] **Edge Cases & Safety** — Adversarial inputs, out-of-scope, failure handling
  - Source: Capability scenarios (safety, graceful failure, compliance)
  - Target: 10–20% of test cases

---

## Step 4: Coverage Check

Before finalizing, verify:

- [ ] At least one business-problem scenario covers each major use case of your agent
- [ ] At least one capability scenario covers each infrastructure component (knowledge sources, tools, topics)
- [ ] Edge cases and negative tests are included (not just happy paths)
- [ ] Compliance and safety scenarios are included if your agent handles sensitive data
- [ ] Test cases span multiple user contexts / phrasings (not all identical patterns)
- [ ] Each test case has a clear pass/fail criteria

---

## Step 5: Set Thresholds

Define what "passing" means for your eval set:

| Metric | Suggested Threshold | Your Threshold |
|--------|-------------------|---------------|
| Overall pass rate | ≥ 85% | ___________ |
| Core business scenarios | ≥ 90% | ___________ |
| Capability scenarios | ≥ 90% | ___________ |
| Safety & compliance | ≥ 95% | ___________ |
| Edge cases | ≥ 70% | ___________ |

> **Tip:** Start with lenient thresholds and tighten as you iterate. A failing eval set that's too strict teaches you nothing — you need signal, not noise.

---

[Back to library](../README.md)
