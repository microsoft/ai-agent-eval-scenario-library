# Trajectory & Stepwise Evaluation

> Scenarios for evaluating the complete sequence of reasoning steps, tool calls, and decisions your agent takes to reach an outcome — not just the final answer. End-to-end evaluation tells you _whether_ the agent succeeded; trajectory evaluation tells you _how_ it got there and _where_ it went wrong.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Trajectory Evaluation Matters

Most evaluation approaches only check the final output: did the agent answer correctly? Did it complete the task? This misses a critical class of failures:

- **Correct answer, wrong path.** The agent arrives at the right answer through flawed reasoning or unnecessary steps. This is fragile — it will break on harder inputs.
- **Partial credit.** The agent completed 4 out of 5 steps correctly but failed at the last one. End-to-end scoring gives this a zero, the same as an agent that failed at step 1.
- **Efficiency problems.** The agent calls 8 tools when 3 would suffice. The final answer is correct, but the cost and latency are unacceptable.
- **Silent reasoning failures.** The agent's intermediate reasoning contains errors or hallucinations that happen to cancel out in the final result.

Trajectory evaluation addresses all of these by examining the complete path the agent took — every reasoning step, tool call, and decision point.

---

## 1. Verifying Action Sequence Correctness

Does the agent take the right steps in the right order to accomplish the task?

### When to Use

Your agent performs multi-step tasks involving tool calls, API requests, or sequential actions, and you need to verify it follows the correct workflow — not just that it produces the right final output. This is the most fundamental trajectory test.

Use this scenario when:
- Your agent orchestrates multiple tools or APIs to complete a request
- There is a defined "golden path" — a known-correct sequence of steps for a given task
- You have observed the agent taking unnecessary detours or calling tools in the wrong order
- Regulatory or business rules require specific steps to happen in a specific sequence

> **Related scenarios:** For testing whether the correct tool fires at all, see [Tool & Connector Invocations](tool-and-connector-invocations.md). For testing whether multi-turn context is preserved, see [Multi-Turn Conversation Quality](multi-turn-conversation-quality.md). This scenario goes deeper — it evaluates the full chain of actions across a task.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Trajectory Match (Strict) | Verify the agent's action sequence exactly matches the expected golden path — same tools, same order |
| Trajectory Match (In-Order) | Verify all required steps occur in the correct order, allowing extra steps in between |
| Trajectory Match (Any-Order) | Verify all required steps are present regardless of order — for tasks where sequence is flexible |
| Capability Use (All) | As a lighter check, confirm that all expected tools were invoked |

> **Tip:** Start with In-Order matching rather than Strict matching. Strict matching is brittle — it fails if the agent adds a reasonable but unexpected step (like a confirmation check). In-Order matching verifies the critical path without penalizing harmless additions.

### Setup Steps

1. For each task your agent handles, define the **golden path**: the ideal sequence of tool calls, reasoning steps, and actions. Document the tool name, key parameters, and expected order.
2. Create test cases with clear user requests that should trigger the golden path.
3. Configure your evaluation to capture the full trajectory — not just the final response. This typically means logging all tool calls, their inputs and outputs, and any intermediate reasoning.
4. Set the expected trajectory as your reference. Choose the matching mode (Strict, In-Order, or Any-Order) based on how rigid the sequence needs to be.
5. Run the evaluation. Compare the agent's actual trajectory against the reference trajectory.
6. Review mismatches: missing steps, extra steps, wrong order, or wrong tool selections.

### Anti-Pattern

> **Anti-Pattern: Only Checking the Final Output**
> If your agent processes an expense report by (1) validating the receipt, (2) checking the policy, (3) calculating the reimbursement, and (4) submitting the request — but you only check whether the final submission succeeded — you will miss cases where the agent skips the policy check. The submission might succeed today, but it violates your compliance requirements. Always check intermediate steps for tasks with mandatory workflows.

### Evaluation Patterns

**Pattern: Golden Path Comparison**
Define the ideal sequence of actions for a task and compare the agent's actual trajectory against it. This is the most direct form of trajectory evaluation. For each test case, annotate the expected tool calls in order: `[validate_receipt → check_policy → calculate_amount → submit_request]`. The agent passes if its trajectory matches.

**Pattern: Required Checkpoints**
Rather than matching the full trajectory, define a set of required checkpoints — steps that _must_ happen at some point during execution. The agent can take additional steps, but it must hit every checkpoint. This is more flexible than golden path matching and works well when there are multiple valid approaches.

**Pattern: Forbidden Actions**
Define actions the agent must _never_ take during a task. For example: the agent should never call `delete_record` during a read-only query, or should never access customer data before authentication. This is a negative trajectory test.

### Practical Examples

| # | Scenario | Task | Expected Trajectory | Method |
|---|----------|------|-------------------|--------|
| 1 | Expense submission follows policy | "Submit my $450 dinner expense from the NYC client meeting" | `validate_receipt → check_expense_policy → calculate_reimbursement → submit_expense` | Trajectory Match (Strict) |
| 2 | Order fulfillment hits all checkpoints | "Process order #12345 for shipping" | Must include: `verify_inventory`, `charge_payment`, `create_shipping_label` (any order) | Trajectory Match (Any-Order) |
| 3 | Password reset follows security protocol | "I need to reset my password" | `verify_identity → check_security_questions → generate_reset_link → send_notification` (in order) | Trajectory Match (In-Order) |
| 4 | Read-only query avoids mutations | "What's the status of my order?" | No calls to `update_order`, `cancel_order`, or `modify_order` | Forbidden Actions check |
| 5 | Authentication before data access | "Show me the customer's payment history" | `authenticate_user` must precede `query_payment_history` | Trajectory Match (In-Order) |

### Tips

- **Start with your most critical workflows.** You don't need golden paths for every task — start with workflows where the sequence matters for compliance, security, or correctness.
- **Version your golden paths.** When your workflows change, your expected trajectories must change too. Track them alongside your tool configurations.
- **Allow for valid variations.** Two agents might take different valid paths to the same outcome. If your task has multiple correct approaches, use Any-Order matching or define multiple acceptable trajectories.
- **Rerun after:** Changes to tool configurations, system prompts, orchestration logic, or when adding new tools that might interfere with existing workflows.

---

## 2. Measuring Step-Level Correctness

Is each individual step in the agent's trajectory correct and well-formed?

### When to Use

Beyond checking that the right steps happen in the right order, you need to verify that each individual step is executed correctly — with the right parameters, appropriate reasoning, and valid outputs. This catches a class of bugs where the agent calls the right tool but with wrong inputs.

Use this scenario when:
- Your agent passes parameters to tools and incorrect parameters could cause silent failures
- You need to verify the agent's reasoning at each decision point, not just the final decision
- Tool calls have side effects (creating records, sending emails, making payments) where wrong parameters are costly
- You want partial credit scoring — understanding how far the agent got before failing

> **Related scenarios:** For verifying tool parameter collection from users, see [Tool & Connector Invocations — Verifying Input Collection](tool-and-connector-invocations.md). This scenario evaluates the parameters the agent passes to tools programmatically, not what it asks the user.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Step-Level Grading | Grade each step independently: correct tool, correct parameters, correct reasoning |
| Keyword Match (All) | Verify specific parameter values appear in tool call arguments |
| LLM-as-Judge (per step) | Use an LLM to assess whether each reasoning step is logically sound |

> **Tip:** Step-level correctness grading enables **partial credit scoring**. An agent that completes 4 of 5 steps correctly scores 0.8, not 0.0. This is far more useful for diagnosing regressions and guiding improvement than binary pass/fail.

### Setup Steps

1. For each step in your golden path, define what "correct" looks like: the expected tool name, required parameters and their values, and any constraints on the output.
2. Create test cases with detailed expected values for each step — not just the final outcome.
3. Configure your evaluation framework to grade each step independently. This typically requires a custom grader or LLM-as-judge prompt that receives the step context.
4. For each step, assign a correctness score (binary or graded). Aggregate step scores into an overall trajectory score.
5. Track which steps fail most frequently — this reveals systematic weaknesses in your agent.

### Anti-Pattern

> **Anti-Pattern: Ignoring Parameter Correctness**
> Your agent calls the right tool (`book_flight`) but passes the wrong date format, an incorrect airport code, or swaps the origin and destination. The tool call looks right at a glance but produces the wrong result. Always validate tool call parameters, not just tool selection.

### Evaluation Patterns

**Pattern: Parameter Validation**
For each tool call in the trajectory, verify that all required parameters are present and have valid values. Define expected parameter schemas and acceptable value ranges. Flag missing parameters, type mismatches, and out-of-range values.

**Pattern: Reasoning Chain Validation**
Use an LLM-as-judge to evaluate the agent's reasoning at each step. The judge receives the conversation context up to that point and the agent's reasoning/action, and assesses whether the reasoning is logically sound and the action is appropriate. This catches cases where the agent makes the right choice for the wrong reason.

**Pattern: Partial Credit Scoring**
Score each step as correct (1), partially correct (0.5), or incorrect (0). The trajectory score is the average across all steps. This gives you a continuous metric that is far more sensitive to improvements than binary pass/fail. A score improvement from 0.6 to 0.8 is meaningful even if the overall task still fails.

### Practical Examples

| # | Scenario | Step Under Test | Expected Parameters | What to Check | Method |
|---|----------|----------------|--------------------|----|--------|
| 1 | Flight booking — date parsing | `search_flights` call | `origin: "SFO"`, `destination: "JFK"`, `date: "2026-04-15"` | Correct airports, correctly parsed date, not swapped | Parameter Validation |
| 2 | Expense policy check — amount threshold | `check_policy` call | `amount: 450`, `category: "meals"`, `currency: "USD"` | Amount matches user input, category correctly inferred | Parameter Validation |
| 3 | Reasoning about eligibility | Agent's reasoning step before decision | N/A — evaluate reasoning text | Does the agent correctly identify that the user is a contractor and apply contractor-specific rules? | LLM-as-Judge |
| 4 | Multi-step calculation | Each arithmetic step | Intermediate values | Each calculation step produces the correct intermediate result | Step-Level Grading |

### Tips

- **Prioritize steps with side effects.** A wrong parameter in a read-only query is annoying; a wrong parameter in a payment API call is costly. Focus your step-level validation on high-impact steps first.
- **Use partial credit for development, binary for production gates.** During development, partial credit helps you iterate. For go/no-go decisions, set a minimum step correctness threshold (e.g., all critical steps must be 100% correct).
- **Track step failure rates over time.** If step 3 of your 5-step workflow fails 40% of the time, that's your improvement target — much more actionable than "the overall success rate is 60%."
- **Rerun after:** Prompt changes, model updates, tool schema changes, or when adding new steps to existing workflows.

---

## 3. Evaluating Trajectory Efficiency

Does the agent complete the task without unnecessary steps, redundant tool calls, or wasted effort?

### When to Use

Your agent completes tasks correctly but you suspect it is taking longer, costing more, or making more API calls than necessary. Efficiency evaluation catches "correct but wasteful" behavior that end-to-end evaluation misses entirely.

Use this scenario when:
- You are tracking cost per task and need to reduce token usage or API call volume
- Users experience slow response times due to unnecessary intermediate steps
- Your agent calls the same tool multiple times with the same parameters
- The agent "thinks out loud" excessively, generating reasoning steps that don't contribute to the outcome
- You are comparing two agent configurations and need to determine which is more efficient

> **Related scenarios:** For evaluating response quality and conciseness, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md). This scenario focuses on the efficiency of the agent's internal process, not the response itself.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Step Count | Compare the number of steps taken versus the minimum required |
| Redundancy Detection | Identify repeated tool calls with identical or near-identical parameters |
| Cost Tracking | Measure tokens consumed and API calls made per task |
| LLM-as-Judge (efficiency) | Use an LLM to assess whether the trajectory is efficient or contains unnecessary steps |

> **Tip:** Efficiency must always be measured _alongside_ correctness. An agent that completes a task in 2 steps instead of 5 is only "efficient" if those 2 steps actually produce the correct result. Never optimize for fewer steps at the expense of correctness.

### Setup Steps

1. For each task, determine the **minimum viable trajectory** — the fewest steps needed to complete it correctly.
2. Run the agent on your test cases and capture the full trajectory with step counts and token usage.
3. Compare actual step counts against the minimum. Calculate the **efficiency ratio**: `minimum_steps / actual_steps`. A ratio of 1.0 is perfectly efficient; below 0.5 means the agent is taking more than twice the necessary steps.
4. Scan for redundant tool calls — same tool, same parameters, called more than once.
5. Track cost metrics: total tokens consumed, number of API calls, and wall-clock time.

### Anti-Pattern

> **Anti-Pattern: Optimizing for Efficiency Before Correctness**
> If your agent's task success rate is below 90%, focus on correctness first. Efficiency optimization on a broken agent is pointless — you'll just get to the wrong answer faster. Efficiency evaluation is most valuable once your agent is reliably correct and you want to reduce cost and latency.

### Evaluation Patterns

**Pattern: Step Count Ratio**
Calculate `minimum_steps / actual_steps` for each task. Track this ratio across your test suite. A mean ratio below 0.7 suggests systematic inefficiency. Investigate the most inefficient trajectories to find common patterns (e.g., the agent always makes a redundant verification call).

**Pattern: Redundant Call Detection**
Flag any trajectory where the same tool is called with identical parameters more than once. Some redundancy is acceptable (e.g., re-checking a value after an update), but most is waste. Categorize redundant calls as "justified" or "unjustified" and target unjustified ones for elimination.

**Pattern: Token Budget Compliance**
Set a token budget per task type and flag any trajectory that exceeds it. For example: simple FAQ queries should complete in under 500 tokens of reasoning; complex multi-step tasks should complete in under 5,000. This prevents token usage from creeping up unnoticed.

**Pattern: Comparative Efficiency**
Run the same task set against two agent configurations (e.g., different prompts, models, or orchestration strategies) and compare their efficiency metrics side by side. This is how you determine whether a change made the agent faster or slower.

### Practical Examples

| # | Scenario | Task | Minimum Steps | What to Measure | Threshold |
|---|----------|------|---------------|-----------------|-----------|
| 1 | FAQ answered without tool calls | "What are your office hours?" | 1 (retrieve from knowledge) | Agent should not call any tools — direct knowledge retrieval | 0 tool calls |
| 2 | Order lookup efficiency | "What's the status of order #12345?" | 2 (authenticate + query) | Agent should not call `query_order` multiple times | ≤ 3 total steps |
| 3 | Redundant re-verification | "Book a meeting room for tomorrow 2pm" | 3 (check availability + book + confirm) | Agent should not check availability twice | 0 redundant calls |
| 4 | Token budget compliance | "Help me troubleshoot my VPN connection" | Varies | Total reasoning tokens should stay under 3,000 | < 3,000 tokens |
| 5 | Comparative prompt efficiency | Same task set, two prompts | Same for both | Which prompt produces shorter trajectories at equal correctness? | Lower mean step count wins |

### Tips

- **Set efficiency baselines before optimizing.** Run your current agent on a representative task set and record step counts, token usage, and timing. This is your baseline — you need it to measure improvement.
- **Efficiency varies by task complexity.** Don't apply a single step-count threshold across all tasks. Simple queries should be 1–2 steps; complex workflows might legitimately require 8–10.
- **Watch for efficiency-correctness tradeoffs.** If reducing the system prompt makes the agent faster but less accurate, the efficiency gain is not worth it. Always report efficiency and correctness together.
- **Rerun after:** System prompt changes, model swaps, orchestration logic updates, or changes to tool response formats.

---

## 4. Error Recovery and Backtracking

When a step fails, does the agent recover gracefully and find an alternative path?

### When to Use

Your agent operates in environments where tools can fail, APIs return errors, or intermediate results are unexpected. This scenario tests whether the agent can detect failures, backtrack when needed, and find alternative paths to complete the task.

Use this scenario when:
- Your agent calls external APIs that may return errors or timeouts
- Tools sometimes return unexpected or empty results
- The agent must handle partial failures (some steps succeed, others fail)
- You want to verify the agent does not get stuck in retry loops

> **Related scenarios:** For testing how the agent communicates failures to the user, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). This scenario focuses on the agent's _internal_ recovery behavior — whether it adapts its plan when something goes wrong.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Fault Injection | Simulate tool failures and observe the agent's recovery behavior |
| Recovery Path Validation | Verify the agent takes a valid alternative path after a failure |
| Loop Detection | Check that the agent does not retry the same failed action more than N times |
| LLM-as-Judge (resilience) | Assess whether the agent's recovery strategy is reasonable |

### Setup Steps

1. Identify the tools and steps in your agent's workflows that can fail (API errors, timeouts, empty results, malformed responses).
2. For each failure point, define the expected recovery behavior: retry with different parameters, try an alternative tool, ask the user for clarification, or gracefully report the failure.
3. Create test cases that inject failures at specific points in the trajectory. This requires a testing framework that can mock or stub tool responses.
4. Run the evaluation and capture the full trajectory including the failure and the agent's response to it.
5. Grade the recovery: Did the agent detect the failure? Did it take an appropriate recovery action? Did it avoid infinite retry loops? Did it eventually complete the task or escalate appropriately?

### Anti-Pattern

> **Anti-Pattern: No Failure Testing**
> If all your test cases assume tools always succeed, you have zero coverage for real-world conditions. Production agents encounter tool failures regularly — API rate limits, network timeouts, stale data, permission errors. If you don't test for these, your first indication of a problem will be user complaints.

### Evaluation Patterns

**Pattern: Retry with Backoff**
Inject a transient failure (e.g., API returns 503) and verify the agent retries with appropriate backoff rather than immediately retrying in a tight loop or giving up after one attempt.

**Pattern: Alternative Path Discovery**
Inject a permanent failure in one tool and verify the agent finds an alternative way to complete the task. For example, if the primary shipping API fails, the agent should try the backup carrier API rather than reporting failure immediately.

**Pattern: Graceful Degradation**
Inject a failure that makes the ideal outcome impossible, and verify the agent achieves the best possible partial outcome. For example, if the user's preferred meeting time is unavailable, the agent should suggest alternative times rather than simply failing.

**Pattern: Loop Detection**
Verify the agent does not retry the same failed action more than a reasonable number of times (typically 2–3). Infinite retry loops waste resources and create poor user experiences.

### Practical Examples

| # | Scenario | Injected Failure | Expected Recovery | Method |
|---|----------|-----------------|-------------------|--------|
| 1 | API timeout recovery | `get_order_status` returns timeout | Agent retries once, then informs user of temporary unavailability | Fault Injection + Recovery Path |
| 2 | Alternative tool fallback | Primary `search_flights` API returns error | Agent tries `backup_flight_search` API | Recovery Path Validation |
| 3 | Graceful partial completion | 2 of 3 items in order are in stock, 1 is not | Agent processes available items and notifies user about the unavailable one | LLM-as-Judge |
| 4 | No infinite retry loop | `send_email` consistently fails | Agent stops after 2–3 retries and escalates or reports failure | Loop Detection |
| 5 | Data validation recovery | Tool returns malformed JSON | Agent handles the error and does not pass corrupted data to the next step | Fault Injection + Step-Level Grading |

### Tips

- **Test both transient and permanent failures.** Transient failures (timeouts, rate limits) should trigger retries. Permanent failures (404, permission denied) should trigger alternative paths or escalation — not retries.
- **Define maximum retry counts.** Set explicit limits on how many times the agent should retry before escalating. Three retries is a reasonable default for most scenarios.
- **Test cascading failures.** What happens when multiple tools fail in sequence? The agent should handle compound failures, not just single-point failures.
- **Rerun after:** Changes to error handling logic, tool configurations, system prompt instructions about failure handling, or when adding new tools.

---

## 5. Reasoning Quality Assessment

Is the agent's reasoning at each step logically sound, even when the final answer is correct?

### When to Use

Your agent produces visible reasoning traces (chain-of-thought, scratchpad, or explanation text) and you need to verify the reasoning is sound — not just that the conclusion is right. This catches "right answer, wrong reason" failures that are fragile and unreliable.

Use this scenario when:
- Your agent uses chain-of-thought or step-by-step reasoning
- Correct reasoning is important for trust and explainability (e.g., medical, legal, financial domains)
- You have observed the agent reaching correct conclusions through flawed logic
- You want to identify reasoning weaknesses before they cause visible failures

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| LLM-as-Judge (reasoning) | Use an LLM to evaluate whether each reasoning step logically follows from the previous one |
| Rubric-Based Scoring | Score reasoning against a predefined rubric (logical validity, relevance, completeness) |
| Faithfulness Check | Verify the agent's stated reasoning actually matches its actions — it does what it says it will do |

> **Tip:** Reasoning quality assessment requires an LLM-as-judge approach — you cannot evaluate reasoning soundness with keyword matching or deterministic checks. Use a capable model as the judge (at least as capable as the agent being evaluated) and provide a detailed rubric.

### Setup Steps

1. Define a reasoning quality rubric. Key dimensions: **logical validity** (each step follows from the previous), **relevance** (each step contributes to the goal), **completeness** (no critical reasoning steps are skipped), and **faithfulness** (stated reasoning matches actual actions).
2. Create test cases for tasks that require multi-step reasoning. Include the expected reasoning approach (not just the expected answer).
3. Configure an LLM-as-judge evaluator with your rubric. The judge receives the full trajectory and scores each reasoning step.
4. Run the evaluation and collect per-step reasoning scores.
5. Analyze patterns: which types of reasoning steps score lowest? Are there systematic logical errors?

### Evaluation Patterns

**Pattern: Logical Chain Validation**
For each reasoning step, the LLM judge assesses: "Does this step logically follow from the information available at this point in the conversation?" Steps that introduce unsupported assumptions, make logical leaps, or contradict earlier information are flagged.

**Pattern: Faithfulness Alignment**
Compare the agent's stated plan with its actual actions. If the agent says "I'll check the inventory first, then process the payment" but actually processes the payment first, that's a faithfulness violation — the reasoning and actions are misaligned.

**Pattern: Counterfactual Robustness**
Present the agent with slightly modified inputs and verify its reasoning adapts appropriately. If changing the user's budget from $500 to $50 doesn't change the agent's reasoning about product recommendations, the reasoning is not actually grounded in the input.

### Practical Examples

| # | Scenario | Reasoning to Evaluate | Quality Dimension | Method |
|---|----------|----------------------|------------------|--------|
| 1 | Policy eligibility determination | Agent reasons about whether a contractor qualifies for benefits | Logical validity — does the reasoning correctly apply policy rules? | LLM-as-Judge + Rubric |
| 2 | Troubleshooting diagnosis | Agent's diagnostic reasoning chain | Completeness — does it consider all relevant causes before concluding? | LLM-as-Judge |
| 3 | Plan-action alignment | Agent states a plan then executes it | Faithfulness — do the actions match the stated plan? | Faithfulness Check |
| 4 | Budget-aware recommendation | Agent recommends products within stated budget | Relevance — does each reasoning step reference the budget constraint? | LLM-as-Judge + Rubric |

### Tips

- **Reasoning quality is domain-specific.** A financial agent's reasoning should be precise and cite specific policy numbers. A creative writing agent's reasoning can be more exploratory. Tailor your rubric to your domain.
- **Use reasoning assessment for high-stakes domains.** In healthcare, legal, and financial applications, correct reasoning is as important as correct conclusions. A correct diagnosis for the wrong reason will fail on the next patient.
- **Start with faithfulness checks.** Faithfulness (does the agent do what it says?) is the easiest reasoning quality to evaluate and catches the most impactful bugs.
- **Rerun after:** Model changes (different models reason differently), system prompt updates that affect reasoning style, or when moving to a new domain.

---

## Aggregating Trajectory Metrics

When running trajectory evaluations across a test suite, aggregate your results into actionable metrics:

| Metric | Formula | What It Tells You |
|--------|---------|-------------------|
| **Task Success Rate** | `successful_tasks / total_tasks` | Baseline: does the agent complete tasks? |
| **Mean Step Correctness** | `correct_steps / total_steps` (averaged across tasks) | How accurate is each individual step? |
| **Efficiency Ratio** | `minimum_steps / actual_steps` (averaged) | How much waste is in the agent's process? |
| **Recovery Rate** | `successful_recoveries / injected_failures` | How well does the agent handle failures? |
| **Reasoning Faithfulness** | `faithful_steps / total_reasoning_steps` | Does the agent do what it says? |
| **Critical Step Pass Rate** | `correct_critical_steps / total_critical_steps` | Are the most important steps reliable? |

> **Key insight:** Track these metrics over time, not just at a single point. A trajectory metric dashboard that shows trends across deployments is far more valuable than a one-time evaluation. Regression in step correctness often predicts regression in task success before it becomes visible.

---

## Getting Started Checklist

If you are new to trajectory evaluation, follow this sequence:

1. **Instrument your agent to log full trajectories** — tool calls, parameters, reasoning traces, and intermediate results. You cannot evaluate what you cannot observe.
2. **Define golden paths for your top 5 critical workflows.** Start with the tasks that matter most to your business.
3. **Implement action sequence validation** (Scenario 1) using In-Order matching. This gives you the highest value for the lowest effort.
4. **Add step-level parameter validation** (Scenario 2) for steps with side effects — tool calls that create, update, or delete data.
5. **Measure efficiency** (Scenario 3) to establish baselines. You'll need these when you start optimizing.
6. **Add fault injection tests** (Scenario 4) for your most failure-prone integrations.
7. **Layer in reasoning quality assessment** (Scenario 5) for high-stakes domains where explainability matters.

---
