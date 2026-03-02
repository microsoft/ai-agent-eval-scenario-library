# Evaluation Method Selection Guide

Detailed guidance for choosing the right evaluation method based on what quality signal you're testing. Read this when a user asks specifically about evaluation methods, expected value formatting, or the method decision framework.

## Core Principle

> Don't ask "which eval method should I use?" Ask "what quality signal am I testing?" — the answer tells you the method.

## Decision Map: Quality Signal to Method

### 1. Factual Accuracy

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Answer is a specific number, date, code, or fixed phrase | Exact Match | Write the exact correct answer |
| Answer must contain specific facts, phrasing can vary | Keyword Match (All) | List required factual keywords |
| Answer has clear correct meaning, many valid phrasings | Compare Meaning | Write one version of the ideal answer |

**Anti-pattern:** Using General Quality for factual accuracy. It can't verify facts — a confident hallucination scores high.

### 2. Policy / Rule Compliance

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Must mention specific disclaimers, warnings, or terms | Keyword Match (All) | List exact phrases that must appear |
| Must use specific knowledge sources or tools | Capability Use (All) | Select expected topics/tools |
| Must follow a rule but output is flexible | Compare Meaning | Write response demonstrating the rule |
| Must NOT do something (e.g., mention competitors) | Keyword Match (All) + manual review | List prohibited keywords |

**Anti-pattern:** Only testing what the agent SHOULD do. Compliance also means testing what it should NOT do (negative tests).

### 3. Personalization / Context-Sensitivity

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Answer must include user-specific data | Keyword Match (All) | List user-context-specific keywords |
| Answer should reflect user situation, flexible phrasing | Compare Meaning | Write the expected personalized answer |
| Answer should use the right knowledge for this user | Capability Use | Select knowledge sources for this user segment |

**Anti-pattern:** Writing generic expected responses. The expected response MUST be specific to the test user's context.

### 4. Response Quality (Subjective)

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Overall quality, no single right answer | General Quality | No expected response needed |
| Quality + meaning should match an ideal | General Quality + Compare Meaning | For Compare Meaning: write ideal response |

**When to use General Quality alone:** Open-ended questions where many answers could be valid.
**When NOT to rely on it alone:** When you have specific factual requirements.

### 5. Edge Case Handling

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Agent should decline / say it doesn't know | Keyword Match (Any) | List acceptable deflection phrases |
| Agent should redirect to human/resource | Keyword Match (Any) + Capability Use | Keywords: escalation phrases. Capability: handoff tool. |
| Agent should ask a clarifying question | Compare Meaning | Write an expected clarifying question |

**Anti-pattern:** Not testing edge cases at all. Include at least 10-20% edge case test cases.

### 6. Knowledge Source Grounding

| Scenario | Method | How to Write Expected Value |
|----------|--------|---------------------------|
| Must retrieve from specific documents/sources | Capability Use (All) | Select expected knowledge sources |
| Answer must be grounded, not hallucinated | General Quality | No expected response needed (uses "groundedness" criterion) |
| Must come from specific source AND contain correct content | Capability Use + Compare Meaning | Capability: select sources. Compare Meaning: write expected answer. |

## Method Decision Tree

```
I have a test prompt. What method should I use?

  Do I know the exact right answer?
  ├── Yes, word-for-word → EXACT MATCH
  └── Yes, but phrasing can vary
      ├── Key facts must appear → KEYWORD MATCH
      └── Overall meaning must match → COMPARE MEANING

  Am I testing which tools/sources the agent uses?
  └── Yes → CAPABILITY USE

  Am I testing subjective quality?
  └── Yes → GENERAL QUALITY

  Am I testing something complex (multiple dimensions)?
  └── Use MULTIPLE METHODS on the same test case
```

## Writing Expected Values: Practical Tips

### Compare Meaning
- Write a natural, complete answer as if you were the ideal agent
- Include all key points the answer should cover
- Don't over-engineer wording — the method handles paraphrasing
- Pass threshold: 70%+ for strict, 50% for lenient (default)

### Keyword Match
- Use the shortest, most specific keywords that uniquely identify the content
- Avoid common words that match accidentally ("the", "is")
- Use **All** when every keyword is mandatory (compliance)
- Use **Any** when the agent needs to hit one of several valid phrasings (edge cases)
- Test: could a wrong answer accidentally contain your keywords?

### Exact Match
- Only use for very short, precise outputs (numbers, codes, yes/no, dates)
- Copy-paste to avoid typos — even spacing differences cause failure

### Capability Use
- Map which sources/tools should trigger for each scenario
- Use **All** when the agent must consult multiple sources
- Use **Any** when multiple sources could provide the right answer

### General Quality
- No expected response needed
- Evaluates 4 criteria: Relevance, Groundedness, Completeness, Abstention
- Best as a baseline alongside more specific methods

## Multi-Method Strategy

Best practice: 2 methods per test case — one specific, one general.

| Quality Signal | Primary (specific) | Secondary (catch-all) |
|---------------|-------------------|---------------------|
| Factual Accuracy | Keyword Match or Exact Match | Compare Meaning |
| Policy Compliance | Keyword Match + Capability Use | General Quality |
| Personalization | Keyword Match (user-specific) | Compare Meaning |
| Response Quality | Compare Meaning | General Quality |
| Edge Cases | Keyword Match (Any) | General Quality |
| Knowledge Grounding | Capability Use | General Quality |

Why two methods? Primary tests the specific requirement. Secondary catches unexpected failures the primary might miss (e.g., right keywords present but overall answer is nonsensical).

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| General Quality for everything | Can't verify facts, rules, or personalization | Use as supplementary, not primary |
| Vague expected responses ("something about returns") | Compare Meaning needs substance to compare | Write a full ideal response |
| Exact Match for long-form answers | Almost guaranteed to fail; wording will vary | Use Compare Meaning or Keyword Match |
| Same expected response for different users | Can't detect personalization failures | Write context-specific expected responses |
| No edge case tests | Only know agent works on happy path | Include 10-20% edge/adversarial inputs |
| Multiple quality signals per test case | If it fails, unclear which dimension failed | One test case = one primary quality signal |
