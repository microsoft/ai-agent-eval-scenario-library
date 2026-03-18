# Multi-Agent Orchestration

> Scenarios for evaluating agents that coordinate work across multiple specialized sub-agents, including handoff, routing, sequential pipeline, concurrent fan-out, and group-chat patterns. Applies to any agent architecture where an orchestrator delegates to specialist agents.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## 1. Verifying Correct Agent Handoff

Does the orchestrator delegate to the right specialist agent based on the user's intent?

### When to Use

Your system uses a handoff or routing pattern where a front-door agent analyzes the request and transfers control to one of several specialist agents. You need to verify that the right specialist receives the task — especially when domains overlap (e.g., billing vs. account management, IT support vs. HR support).

> **Related scenarios:** If your agent routes within a single agent's topics rather than across separate agents, see [Trigger Routing](trigger-routing.md). If you need to verify the specialist agent's response quality, pair this with [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md) or the relevant business-problem scenario.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the expected specialist agent was invoked — the primary check for delegation accuracy |
| Custom (Classification) | Uses a grader to verify the routing decision was correct given the input |
| Keyword Match (Any) | Catches specialist-specific language in the response as a secondary signal |

> **Tip:** The handoff decision is the highest-leverage evaluation point in a multi-agent system. A wrong handoff means the entire downstream response will be from the wrong specialist — even if that specialist performs flawlessly within its own domain.

### Setup Steps

1. Map every specialist agent in your system and the domains each handles.
2. For each specialist, write 3–5 test inputs that clearly belong to that domain plus 2–3 inputs that are near the boundary with adjacent specialists.
3. Create test cases with **Capability Use (All)** set to the expected specialist agent.
4. Add boundary cases where two specialists have overlapping domains — these are the highest-value test cases.
5. Include at least 2 test cases where no specialist applies (the orchestrator should either handle it directly, ask for clarification, or escalate).

### Anti-Pattern

> **Anti-Pattern: Testing Only Clear-Cut Routing**
> If every test input is unambiguously in one specialist's domain ("I need to reset my password" for the IT agent), you're testing the easy cases. Real users say things like "I can't access the benefits portal" — which could be IT (access issue) or HR (benefits question). Test the overlaps.

### Evaluation Patterns

**Pattern A: One-to-One Specialist Verification**
For each specialist agent, verify that 3–5 different phrasings of domain-specific requests all route to that specialist. This confirms the orchestrator's intent classification covers natural variation.

**Pattern B: Boundary Disambiguation**
Test inputs that sit at the boundary between two specialists. Example: "My paycheck seems wrong" could route to Payroll (calculation issue) or HR (policy question). Define which specialist should receive each boundary case, or verify that the orchestrator asks a clarifying question.

**Pattern C: No-Match Handling**
Test inputs that don't match any specialist's domain. The orchestrator should not force-route to the closest specialist — it should acknowledge the limitation and escalate or redirect. Example: "What's the weather today?" sent to an internal HR/IT support system.

**Pattern D: Multi-Intent Routing**
Test inputs containing two intents spanning different specialists. Example: "I need to reset my password AND check my PTO balance." Verify the orchestrator either handles both (sequentially delegating to each specialist) or clearly addresses the primary intent and acknowledges the secondary one.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Clear routing to Billing specialist | "I was charged twice for my subscription last month" | Capability: `Billing Agent` invoked | Capability Use (All) |
| 2 | Clear routing to Technical Support | "My app crashes every time I try to upload a file" | Capability: `Tech Support Agent` invoked | Capability Use (All) |
| 3 | Boundary: Billing vs. Account Management | "I want to cancel my subscription" | Capability: `Account Management Agent` invoked (not Billing) | Capability Use (All) + Custom |
| 4 | Multi-intent across specialists | "Reset my password and also tell me about the new dental plan" | Both `IT Agent` and `HR Agent` invoked, or primary handled with acknowledgment | Capability Use (All) + Keyword Match |
| 5 | No-match escalation | "Can you recommend a good restaurant nearby?" | Orchestrator declines gracefully — no specialist invoked | Custom (Classification) |
| 6 | Ambiguous intent with clarification | "I have a problem with my account" | Orchestrator asks clarifying question OR routes to most likely specialist | Custom (Classification) |

### Tips

- Target: **95%+ of clear-domain test cases route to the correct specialist.** Boundary cases should achieve **85%+** correct routing or appropriate clarification.
- Map every specialist pair that shares vocabulary or overlapping domains — these boundaries generate the highest-value test cases.
- Rerun this scenario **every time you add or remove a specialist agent** or change the orchestrator's routing instructions.
- Track which specialist pairs generate the most routing errors — these indicate domain boundary ambiguity that may need prompt or architecture changes.

---

## 2. Testing Sequential Pipeline Integrity

When agents process work in a defined sequence, does each stage receive correct input and produce expected output?

### When to Use

Your system uses a sequential (pipeline) orchestration pattern where Agent A's output feeds into Agent B, then Agent B's output feeds into Agent C, and so on. You need to verify that the pipeline produces correct end-to-end results AND that each stage contributes appropriately — not just that the final output looks reasonable.

> **Related scenarios:** If your pipeline includes tool calls at individual stages, pair with [Tool & Connector Invocations](tool-and-connector-invocations.md). For verifying that the final response is grounded in source data, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom (Classification) | Grades whether the final output reflects contributions from all pipeline stages |
| Compare Meaning | Verifies the final response is semantically equivalent to the expected answer |
| Keyword Match (All) | Confirms that stage-specific artifacts appear in the output (e.g., compliance language from a review stage) |

### Setup Steps

1. Document your pipeline: list each agent in order and what it contributes (e.g., Stage 1: draft, Stage 2: compliance review, Stage 3: tone polish).
2. Create end-to-end test cases with inputs that exercise the full pipeline, using **Compare Meaning** or **Custom** to evaluate final output quality.
3. Create stage-isolation test cases: for each intermediate agent, verify that its specific contribution is present in the output (e.g., compliance disclaimers from the review stage).
4. Add error-propagation test cases: introduce a deliberately tricky input at Stage 1 and verify that downstream stages handle it gracefully rather than amplifying the problem.

### Anti-Pattern

> **Anti-Pattern: Only Testing Final Output**
> If you only evaluate the pipeline's final response, you can't tell which stage failed when something goes wrong. A compliance review agent that passes everything through unchanged would be invisible in end-to-end testing. Test for evidence of each stage's contribution.

### Evaluation Patterns

**Pattern A: End-to-End Quality**
Test that the full pipeline produces the expected final output for representative inputs. This is the baseline — if the final output is wrong, dig into which stage caused it.

**Pattern B: Stage Contribution Verification**
For each stage, verify that its specific contribution is present. Example: if Stage 2 is a regulatory compliance check, confirm that the output includes required disclaimers or that non-compliant content was flagged.

**Pattern C: Error Propagation Resilience**
Feed the pipeline an input that is ambiguous or partially malformed. Verify that downstream stages handle the uncertainty rather than confidently building on flawed upstream output. The pipeline should either correct the issue, flag uncertainty, or escalate.

**Pattern D: Stage Ordering Sensitivity**
If the pipeline order is configurable, test whether reordering stages changes the output. Some orderings may produce degraded quality (e.g., tone polishing before compliance review may cause compliance language to be softened).

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Full pipeline produces correct output | Contract request: "Standard NDA for Contoso partnership" | Output includes template selection, customized clauses, compliance review, and risk assessment | Custom (Classification) |
| 2 | Compliance stage contributes disclaimer | Contract request involving EU data processing | Output includes GDPR-specific clauses | Keyword Match (All): "data processing agreement", "GDPR" |
| 3 | Error propagation: ambiguous input | "Draft a contract" (no specifics) | Pipeline asks for clarification rather than generating generic contract | Custom (Classification) |
| 4 | All stages contribute to final output | "Employment agreement for senior engineer in California" | Output reflects CA-specific employment law (compliance), competitive salary ranges (customization), risk flags for non-compete limitations (risk assessment) | Custom (Classification) + Keyword Match |

### Tips

- Test at least **2 inputs per pipeline stage** that specifically exercise that stage's logic.
- When a test fails, use stage-level tracing to identify which agent introduced the error — don't just re-run the whole pipeline.
- Target: **90%+ of end-to-end test cases pass.** Stage-contribution tests should also pass at **90%+** — if a stage consistently contributes nothing, it may be redundant.

---

## 3. Evaluating Concurrent Agent Coordination

When multiple agents process the same input in parallel, are results correctly aggregated?

### When to Use

Your system uses a concurrent (fan-out/fan-in) orchestration pattern where the same input is sent to multiple specialist agents simultaneously, and their results are combined into a single response. You need to verify that the aggregation logic produces coherent, comprehensive output — not contradictory or redundant information.

> **Related scenarios:** If each parallel agent uses different tools, pair with [Tool & Connector Invocations](tool-and-connector-invocations.md). For evaluating the quality and coherence of the final merged response, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom (Classification) | Grades whether the aggregated response includes contributions from all expected agents |
| Compare Meaning | Verifies the aggregated response is semantically equivalent to the expected comprehensive answer |
| Custom (Evaluation Instructions) | Evaluates whether the response is coherent and non-contradictory |

### Setup Steps

1. List all parallel agents and the perspective each provides (e.g., financial analysis, technical analysis, risk assessment).
2. Create test cases where all agents should contribute. Use **Custom (Classification)** to verify that the output reflects each agent's perspective.
3. Add test cases where one agent's perspective should dominate (e.g., a safety-critical input where the safety agent's assessment should override optimistic signals from other agents).
4. Add test cases where agents might produce conflicting recommendations — verify the aggregation logic handles conflicts explicitly rather than ignoring them.

### Anti-Pattern

> **Anti-Pattern: Checking Only for Presence, Not Coherence**
> Verifying that each agent's perspective appears in the output is necessary but not sufficient. If Agent A says "proceed with investment" and Agent B says "high risk — avoid", the aggregated response should acknowledge the tension and explain the tradeoff, not simply list both conclusions side by side.

### Evaluation Patterns

**Pattern A: Comprehensive Coverage**
Verify that the aggregated output includes meaningful contributions from all parallel agents. If one agent's perspective is consistently missing, the fan-out or aggregation may be dropping results.

**Pattern B: Conflict Resolution**
Test inputs where parallel agents produce contradictory recommendations. Verify that the aggregation explicitly addresses the conflict with a reasoned synthesis, priority ordering, or clear caveat — not silent omission of the minority view.

**Pattern C: Dominance Appropriateness**
Test inputs where one perspective should dominate (e.g., safety concerns should override performance optimization). Verify that the aggregation weights safety-critical inputs appropriately.

**Pattern D: Graceful Degradation**
Test behavior when one parallel agent fails or times out. The system should still return results from the remaining agents with a note about incomplete analysis, not fail entirely.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | All perspectives included | "Evaluate ACME Corp stock" | Response includes financial, technical, sentiment, AND ESG analysis | Custom (Classification) |
| 2 | Conflicting recommendations handled | Stock with strong financials but poor ESG rating | Response acknowledges tension between financial opportunity and ESG risk | Custom (Evaluation Instructions) |
| 3 | Safety-critical dominance | Investment with regulatory investigation pending | Risk/compliance perspective prominently featured with clear warning | Custom (Classification) |
| 4 | Partial agent failure | One analysis agent times out | Response includes available analyses with note about incomplete coverage | Custom (Evaluation Instructions) |

### Tips

- Target: **95%+ of aggregated outputs include contributions from all expected parallel agents.**
- Conflict-resolution test cases are the highest-value tests — they reveal whether your aggregation logic is sophisticated or just concatenating outputs.
- Test with inputs that produce different levels of agreement across agents (all agree, split, all disagree) to stress-test aggregation.

---

## 4. Validating Context Preservation Across Handoffs

When control passes from one agent to another, is conversation context preserved correctly?

### When to Use

Your multi-agent system transfers control between agents during a conversation, and you need to verify that the receiving agent has access to relevant context from the previous agent's interaction. This applies to handoff patterns, sequential pipelines, and any architecture where agents share state.

> **Related scenarios:** For verifying that the agent's responses remain grounded in source data across handoffs, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For evaluating tone consistency, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom (Evaluation Instructions) | Grades whether the post-handoff response demonstrates awareness of pre-handoff context |
| Compare Meaning | Verifies the post-handoff response is consistent with what was established before the handoff |
| Keyword Match (Any) | Confirms that key details from the pre-handoff conversation appear in the post-handoff response |

### Setup Steps

1. Design multi-turn test conversations where the user provides key details in early turns, then the conversation is handed off to a different agent.
2. After the handoff, ask a question that can only be answered correctly if the receiving agent has the pre-handoff context.
3. Test with progressively more context — short exchanges (1–2 turns) and longer exchanges (5+ turns) — to verify context transfer at different scales.
4. Include test cases where the user explicitly references something from the pre-handoff conversation ("as I mentioned earlier...").

### Anti-Pattern

> **Anti-Pattern: Testing Handoffs Without Multi-Turn Context**
> If your test cases start the post-handoff conversation fresh (e.g., the user restates their entire problem), you're not testing context preservation at all. The receiving agent should already know what was discussed. Test with follow-up questions that assume shared context.

### Evaluation Patterns

**Pattern A: Key Detail Retention**
The user provides specific details (account number, product name, error code) before the handoff. After the handoff, verify the receiving agent uses these details without the user repeating them.

**Pattern B: Conversation Summary Accuracy**
If your system generates a summary during handoff, verify the summary is accurate and complete. Test with conversations that include multiple topics, corrections, and nuances — summaries often drop important details.

**Pattern C: Emotional Context Continuity**
If the user expressed frustration or urgency before the handoff, verify the receiving agent acknowledges the emotional context. A customer who said "I've been trying to fix this for three hours" should not receive a cheery "How can I help you today?"

**Pattern D: Long Conversation Transfer**
Test handoffs after extended conversations (5+ turns) with multiple details. Verify the receiving agent retains the most important context even when the full conversation is long.

### Practical Examples

| # | Scenario | Sample Input (post-handoff) | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Key detail retention | "So can you process that refund now?" (user gave order #12345 before handoff) | Response references order #12345 without asking user to repeat it | Keyword Match (Any): "12345" + Custom |
| 2 | Summary accuracy | "What did I tell the previous agent?" | Response accurately summarizes pre-handoff conversation | Compare Meaning |
| 3 | Emotional continuity | "Can we please just resolve this?" (user was frustrated before handoff) | Response acknowledges frustration, does not restart with generic greeting | Custom (Evaluation Instructions) |
| 4 | Multi-detail retention after long conversation | "And what about the second issue I mentioned?" | Response correctly identifies the second issue from the pre-handoff conversation | Custom (Evaluation Instructions) |

### Tips

- Context preservation failures are one of the top user-frustration drivers in multi-agent systems. Users strongly dislike repeating information.
- Target: **90%+ of key details from pre-handoff conversation are available post-handoff.**
- Test both structured context (order numbers, dates, names) and unstructured context (user preferences, emotional state, previously attempted solutions).
- If context preservation is poor, investigate whether the handoff mechanism passes full conversation history, a summary, or just the latest message.

---

## 5. Testing Orchestrator Failure Recovery

When a specialist agent fails, does the orchestrator handle the failure gracefully?

### When to Use

Your multi-agent system needs resilience — when a specialist agent errors out, times out, or produces unusable output, the orchestrator should recover gracefully rather than surfacing a raw error to the user or hanging indefinitely.

> **Related scenarios:** For single-agent failure handling, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). For verifying safety boundaries are maintained during failures, see [Safety & Boundary Enforcement](safety-and-boundary-enforcement.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom (Classification) | Classifies the orchestrator's failure response as graceful, partial, or ungraceful |
| Custom (Evaluation Instructions) | Evaluates whether the response explains the limitation and offers alternatives |
| Keyword Match (Any) | Checks for user-friendly error language rather than technical error messages |

### Setup Steps

1. Identify failure modes for each specialist agent: timeout, error response, empty response, malformed output, hallucinated output.
2. For each failure mode, create test scenarios (if your test framework supports fault injection) or design inputs that are known to trigger edge cases in specific specialists.
3. Evaluate the orchestrator's response: does it explain the issue in user-friendly terms, offer an alternative path, and avoid exposing technical details?
4. Test cascading failures: what happens when multiple specialists fail simultaneously?

### Anti-Pattern

> **Anti-Pattern: Only Testing Happy Paths in Multi-Agent Systems**
> Multi-agent systems have more failure modes than single-agent systems (N agents x M failure modes). If you only test cases where all agents succeed, you'll be surprised in production. Dedicate at least 20% of your multi-agent test cases to failure scenarios.

### Evaluation Patterns

**Pattern A: Single Specialist Failure**
One specialist agent fails. The orchestrator should either retry, fall back to an alternative agent, provide a partial response from the remaining agents, or explain the limitation and escalate.

**Pattern B: Timeout Handling**
A specialist agent takes too long to respond. Verify the orchestrator doesn't hang indefinitely — it should timeout and provide a response within an acceptable latency window.

**Pattern C: Cascading Failure Prevention**
Multiple specialist agents fail. Verify the orchestrator doesn't attempt increasingly desperate routing that produces worse results than simply acknowledging the failure.

**Pattern D: Malformed Output Handling**
A specialist agent returns unexpected output format. Verify the orchestrator detects the malformation and doesn't pass garbage through to the user.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Specialist timeout | Request to billing agent that times out | User receives helpful message within SLA, not a hang or raw error | Custom (Classification) + Keyword Match |
| 2 | Graceful degradation message | Technical support agent returns error | Response explains temporary limitation and offers alternative (e.g., "I can't access your account details right now. Would you like me to create a support ticket?") | Custom (Evaluation Instructions) |
| 3 | Cascading failure | Both billing and account agents unavailable | Orchestrator acknowledges system limitations and offers human escalation | Custom (Classification) |
| 4 | No technical jargon in errors | Any specialist failure | Response does NOT contain stack traces, error codes, or internal agent names | Keyword Match (Any) — negative match for technical terms |

### Tips

- Target: **100% of failure scenarios produce a user-friendly response** (no raw errors, no hangs, no technical jargon).
- Partial responses are often better than no response — if 3 of 4 parallel agents succeed, return those results with a note about incomplete analysis.
- Test failure recovery with real latency constraints — a graceful error message after 60 seconds is still a bad user experience.
- Track failure recovery patterns in production to identify which specialist agents fail most often and improve their resilience.

---

## 6. Evaluating Duplicate Work Prevention

In systems with multiple agents, does the architecture prevent redundant processing?

### When to Use

Your multi-agent system routes tasks to specialists, and you need to verify that the same work isn't being done by multiple agents unnecessarily. This is especially relevant in concurrent architectures where agents might overlap, or handoff architectures where re-routing could cause repeated processing.

> **Related scenarios:** For verifying correct routing (which prevents some duplicate work), see [Scenario 1: Verifying Correct Agent Handoff](#1-verifying-correct-agent-handoff). For cost implications, consider tracking token usage across agents.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom (Evaluation Instructions) | Evaluates whether the final response contains redundant or repeated information from different agents |
| Custom (Classification) | Classifies whether the response shows signs of duplicate processing (repeated recommendations, contradictory-then-aligned conclusions) |

### Setup Steps

1. Create test inputs that could plausibly be handled by multiple agents.
2. Evaluate the response for signs of duplication: repeated information, contradictory initial analysis that converges, or unnecessarily long responses that cover the same ground from multiple angles.
3. If your system exposes agent traces, check whether multiple agents processed the same sub-task.

### Anti-Pattern

> **Anti-Pattern: Ignoring Token Costs from Duplicate Processing**
> Even if the user sees a coherent final response, duplicate processing wastes compute and increases latency. Monitor token usage across agents to detect hidden duplication — if your 3-agent system consistently uses 3x the tokens of a single agent for simple tasks, agents are likely duplicating work.

### Evaluation Patterns

**Pattern A: Response Redundancy Check**
Evaluate whether the final response contains the same information stated multiple ways, suggesting multiple agents independently covered the same ground.

**Pattern B: Efficiency Comparison**
For tasks that should only require one specialist, verify that only one specialist was invoked — not multiple agents doing overlapping analysis.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Simple task uses single specialist | "What's my account balance?" | Only the Account Agent is invoked, not Account + Billing + General | Custom (Classification) |
| 2 | No repeated information in output | Complex query spanning two domains | Response covers each aspect once, not multiple overlapping summaries | Custom (Evaluation Instructions) |

### Tips

- Duplicate work is a cost and latency issue more than a correctness issue — but it still matters for production systems at scale.
- Track average token usage and latency per request type. Sudden increases may indicate routing changes that cause unnecessary parallel processing.
- Target: **Simple single-domain tasks should invoke only 1 specialist agent 95%+ of the time.**
