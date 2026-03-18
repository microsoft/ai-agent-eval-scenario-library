# Evaluation Method Selection Guide

How to choose the right evaluation methods for your test cases. This guide helps you match what you're testing to how you should test it — so every test case uses the method that actually catches the failure mode you care about.

---

## Why Method Selection Matters

The wrong evaluation method can pass a broken agent or fail a working one:

- **Too lenient:** General Quality alone won't catch a factually wrong answer that's well-written
- **Too strict:** Keyword Match on a conversational response fails when the agent rephrases correctly
- **Wrong target:** Checking keywords when you should be checking which tool was invoked

Choosing the right method is the difference between an eval set that finds real problems and one that generates noise.

---

## The Four Evaluation Methods

| Method | What It Checks | How It Works | Best For |
|--------|---------------|-------------|----------|
| **Keyword Match** | Specific words or phrases appear in the response | Checks if all (or any) specified keywords are present | Factual accuracy, policy compliance, required disclaimers |
| **Compare Meaning** | The response conveys the same meaning as the expected answer | Uses semantic similarity to compare response meaning | Correct answers with flexible phrasing, hallucination detection |
| **Capability Use** | The agent invoked the right tool, source, or topic | Verifies which capabilities were triggered during the response | Tool invocations, knowledge source selection, topic routing |
| **General Quality** | Overall response quality on a rubric | Scores the response on dimensions like helpfulness, clarity, and tone | Tone, empathy, response structure, conversational quality |

---

## Quick Decision Tree

Use this flow to select your primary method, then add a secondary method for depth.

```
What are you testing?
│
├─ "Did the response contain specific facts, numbers, or required phrases?"
│   └─ Primary: Keyword Match
│       ├─ Use (All) when every keyword must appear
│       └─ Use (Any) when at least one keyword suffices
│
├─ "Did the response convey the correct meaning, even if phrased differently?"
│   └─ Primary: Compare Meaning
│       └─ Write the ideal answer as your expected value
│
├─ "Did the agent use the right tool, knowledge source, or topic?"
│   └─ Primary: Capability Use
│       ├─ Use (All) when multiple capabilities must fire
│       └─ Use (Any) when one of several capabilities is acceptable
│
└─ "Is the response well-written, helpful, and appropriate in tone?"
    └─ Primary: General Quality
        └─ No expected value needed — scored on a rubric
```

---

## Method Selection by Quality Signal

Use this table to select methods based on what quality signal your test case targets. Each row recommends a primary method (the one that directly tests the signal) and a secondary method (which catches additional failure modes).

| Quality Signal | Primary Method | Secondary Method | Expected Value Format |
|---------------|---------------|-----------------|----------------------|
| **Factual accuracy** — specific facts, numbers, dates must be present | Keyword Match (All) | Compare Meaning | List of required factual keywords |
| **Factual accuracy** — correct meaning, flexible phrasing acceptable | Compare Meaning | Keyword Match (Any) | Ideal answer in natural language |
| **Policy compliance** — mandatory phrases must appear verbatim | Keyword Match (All) | Capability Use (All) | Exact required phrases |
| **Tool invocation** — correct tool or flow must execute | Capability Use (All) | Keyword Match (Any) | Exact tool/flow name from agent config |
| **Knowledge source selection** — correct source must be retrieved | Capability Use (All) | Compare Meaning | Expected knowledge source name |
| **Topic routing** — correct topic must trigger | Capability Use (All) | — | Expected topic name |
| **Personalization** — response must reflect user context | Keyword Match (All) | Compare Meaning | User-specific keywords (role, location, etc.) |
| **Response quality** — helpfulness, clarity, structure | General Quality | Compare Meaning | No expected value needed for General Quality |
| **Tone and empathy** — appropriate emotional register | General Quality | Keyword Match (Any) | No expected value; optionally check for empathetic phrases |
| **Edge case handling** — agent should decline or redirect | Keyword Match (Any) | General Quality | Acceptable deflection/redirect phrases |
| **Hallucination prevention** — agent should not fabricate | Compare Meaning | General Quality | What the agent *should* say (including "I don't know") |
| **Negative test** — agent should NOT do something | Keyword Match (All) — negative | Capability Use — negative | Keywords or capabilities that must be *absent* |

> **Rule of thumb:** Use two methods per test case wherever possible. The primary method tests the core signal; the secondary method catches a different failure mode on the same response.

---

## Combining Methods Effectively

Copilot Studio supports multiple test methods per test case, joined with " + ". Here's how combinations work and when to use them.

### Common Method Combinations

| Combination | When to Use | Example |
|-------------|------------|---------|
| **Keyword Match (All) + Compare Meaning** | Response must contain specific facts AND be semantically correct overall | PTO policy answer: keywords "25 days" + "annual" must appear, and overall meaning must match |
| **Keyword Match (All) + Capability Use (All)** | Response must contain required content AND the right source/tool must be used | Compliance answer: disclaimer text present AND sourced from the compliance knowledge base |
| **Capability Use (All) + Compare Meaning** | Right tool/source must be used AND the output must be semantically correct | Tool execution: PTO flow fires AND confirmation message matches expected outcome |
| **Capability Use (All) + Keyword Match (Any)** | Right capability fires AND response includes at least one expected indicator | Topic routing: correct topic triggers AND response contains a relevant greeting or context phrase |
| **Compare Meaning + General Quality** | Response must be correct AND well-structured/helpful | Troubleshooting guidance: advice is accurate AND clearly organized with numbered steps |
| **General Quality + Keyword Match (Any)** | Response quality is high AND includes at least one expected element | Empathetic response: tone is appropriate AND includes phrases like "I understand" or "I'm sorry" |

### Anti-Patterns: Combinations to Avoid

| Don't Do This | Why Not | Do This Instead |
|---------------|---------|----------------|
| **General Quality alone** for factual claims | General Quality assesses style, not facts — a well-written wrong answer passes | Keyword Match or Compare Meaning + General Quality |
| **Keyword Match (All)** with too many keywords | Brittle — fails on valid rephrasings that omit one keyword | Reduce to essential keywords + add Compare Meaning |
| **Compare Meaning alone** for compliance content | Semantic similarity may pass a paraphrase when verbatim text is legally required | Keyword Match (All) for required phrases + Compare Meaning |
| **Capability Use alone** without checking the response | The right tool can fire but return the wrong output | Capability Use + Compare Meaning or Keyword Match |

---

## Writing Effective Expected Values

The expected value is what you compare the agent's response against. How you write it depends on the method:

### For Keyword Match

- List only the **essential** keywords — facts, numbers, required phrases
- Don't include common words the agent will naturally use
- For **(All)**: every keyword must appear → use for non-negotiable facts
- For **(Any)**: at least one keyword must appear → use for flexible indicators

**Good:** `"25 days", "annual allowance", "pro-rated"`
**Bad:** `"The PTO policy states that employees with 5 or more years receive 25 days"` (too many words, will break on any rephrasing)

### For Compare Meaning

- Write the **ideal answer** in natural language
- Include the key facts and reasoning, but don't worry about exact phrasing
- Be specific enough that a wrong answer would clearly differ semantically

**Good:** `"Employees with 5 or more years of tenure receive 25 days of annual PTO, pro-rated for partial years."`
**Bad:** `"PTO information"` (too vague — almost any response about PTO would match)

### For Capability Use

- Use the **exact name** of the tool, knowledge source, or topic from your agent configuration
- Check your agent profile to confirm the exact string

**Good:** `"Submit-PTO-Request"` (matches the flow name in Copilot Studio)
**Bad:** `"PTO flow"` (informal name that won't match the configured capability)

### For General Quality

- No expected value needed — the method scores against a built-in quality rubric
- If you want to anchor scoring, add a brief description of what a good response looks like in the test case notes

---

## Handling Special Cases

### Negative Tests

When the agent should **not** do something (e.g., not reveal PII, not call an unauthorized tool):

- Append **"— negative"** to the method
- The expected value lists what should be **absent**
- Example: `Keyword Match (All) — negative` with expected value `"SSN", "social security"` means the test fails if those keywords appear

### Multi-Turn Conversations

For test cases that span multiple conversation turns:

- Apply methods to the **final response** (or the specific turn being evaluated)
- Use Capability Use to verify tools fired at the correct step in the conversation
- Consider separate test cases for each critical turn rather than one test covering the entire conversation

### Non-Deterministic Responses

When the agent may give different valid responses on different runs:

- Prefer **Compare Meaning** over **Keyword Match** — it's more tolerant of variation
- Use **Keyword Match (Any)** instead of **(All)** for non-essential elements
- Run the test case 3 times and check consistency — if results vary, the test case may be too strict

---

## Method Selection Checklist

Before finalizing your eval set, verify:

- [ ] Every test case has at least one method assigned
- [ ] Factual accuracy test cases use Keyword Match or Compare Meaning (not General Quality alone)
- [ ] Compliance test cases use Keyword Match (All) for verbatim requirements
- [ ] Tool and source tests use Capability Use with exact names from the agent config
- [ ] At least 50% of test cases use two or more methods
- [ ] Negative tests use the "— negative" suffix
- [ ] Expected values follow the format guidance for each method type

---

## From Scenario to Method: Worked Example

Here's how method selection works in practice for an HR Q&A agent:

**Scenario:** Verifying Policy Answer Accuracy (from Information Retrieval & Policy Q&A)

| Test Case | Quality Signal | Methods | Expected Value | Why These Methods |
|-----------|---------------|---------|---------------|-------------------|
| "What's the PTO policy for 5+ year employees?" | Factual accuracy (specific) | Keyword Match (All) + Compare Meaning | Keywords: "25 days", "annual", "pro-rated" / Meaning: Full PTO policy description | Keywords catch missing facts; Compare Meaning catches wrong context |
| "Can I carry over unused PTO?" | Factual accuracy (flexible) | Compare Meaning + General Quality | "Unused PTO can be carried over up to 5 days, must be used by March 31" | Meaning catches incorrect policy; Quality checks if answer is clear |
| "Tell me about PTO" (vague) | Edge case handling | Compare Meaning + Keyword Match (Any) | "The PTO policy covers annual allowance, carryover, and requesting time off" / Keywords: "annual", "carryover", "request" | Vague input should get a helpful overview, not a random fragment |
| "What's my coworker's PTO balance?" | Negative — PII protection | Keyword Match (All) — negative + General Quality | Absent: "balance", "days remaining", any specific number / Quality: polite decline | Must not reveal another employee's data; should decline helpfully |

---

[Back to library](../README.md)
