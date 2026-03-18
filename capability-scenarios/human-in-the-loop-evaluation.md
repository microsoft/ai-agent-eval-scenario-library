# Human-in-the-Loop Evaluation

> Scenarios for evaluating the quality of human oversight in your agent pipeline — escalation accuracy, reviewer reliability, approval workflows, and the transition to AI-assisted evaluation with human governance. The question isn't just "does the agent work?" but "does the human oversight system work?"

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Human-in-the-Loop Evaluation Matters

Most agent evaluations focus on the agent's outputs. But many production agents operate within a human oversight system — humans review high-risk actions, approve escalations, score quality samples, and make final calls when the agent is uncertain. If the oversight system itself is unreliable, it doesn't matter how good the agent is.

Human-in-the-loop (HITL) evaluation addresses three problems that pure agent evaluation misses:

- **Escalation calibration.** Your agent decides when to involve a human. If it escalates too often, human reviewers are overwhelmed with low-risk cases. If it escalates too rarely, high-risk errors slip through. Evaluating escalation accuracy is as important as evaluating answer quality.
- **Reviewer reliability.** When human reviewers disagree with each other 30% of the time, your "ground truth" is noise. Without monitoring inter-rater reliability, you can't trust the quality labels your evaluation system produces.
- **Scaling limits.** Modern agents produce action traces too dense and numerous for comprehensive human review. The emerging pattern is AI-as-evaluator with human governance — but this architecture needs its own evaluation to confirm the AI evaluator is trustworthy and the governance layer is effective.

### The Escalate-Review-Govern Framework

The scenarios in this guide follow three levels of human involvement, from direct to supervisory:

1. **Escalate** — The agent identifies situations requiring human input and routes them correctly, with appropriate context.
2. **Review** — Human evaluators assess agent outputs consistently and reliably, producing ground truth you can trust.
3. **Govern** — Humans set rules, thresholds, and oversight policies while AI handles real-time monitoring and scoring at scale.

---

## 1. Confidence-Based Escalation Calibration

Does your agent correctly identify when to involve a human — and when to proceed autonomously?

### When to Use

Your agent makes autonomous decisions but is configured to escalate to human review in certain situations — high-risk actions, low-confidence answers, sensitive topics, or ambiguous requests. You need to evaluate whether the escalation triggers fire at the right times.

Use this scenario when:
- Your agent has configurable confidence thresholds that determine when to escalate
- Users report that the agent either escalates too often (frustrating delays) or too rarely (errors that should have been caught)
- You are tuning the boundary between autonomous and human-assisted operation
- Your agent handles a mix of routine and high-stakes tasks where the cost of errors varies significantly

> **Related scenarios:** For evaluating how the agent handles failure and handoff, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). That scenario tests whether the agent escalates _gracefully_; this scenario tests whether it escalates _accurately_ — at the right threshold, for the right reasons.

### Escalation Accuracy: The Core Metric

Escalation decisions are a binary classification problem. The agent must decide: "Can I handle this autonomously, or does a human need to be involved?" This creates four outcomes:

| | Agent Handles Autonomously | Agent Escalates to Human |
|---|---|---|
| **Should Handle Autonomously** | ✅ True Autonomous (correct) | ❌ False Escalation (unnecessary human burden) |
| **Should Escalate** | ❌ Missed Escalation (risk exposure) | ✅ True Escalation (correct) |

The key insight: **false escalations and missed escalations have very different costs.** A false escalation wastes a reviewer's time (minutes). A missed escalation on a high-stakes action can cause real harm (regulatory violation, financial error, safety incident). Your evaluation must weight these asymmetrically.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Threshold Sweep Analysis | Test escalation behavior across a range of confidence thresholds to find the optimal operating point |
| Labeled Escalation Test Set | Curate test cases with known correct escalation decisions; measure precision and recall |
| Expected Calibration Error (ECE) | Measure whether the agent's stated confidence actually predicts its accuracy |
| Cost-Weighted Escalation Score | Score escalation decisions weighted by the business cost of each error type |

> **Tip:** Start by measuring the false escalation rate and missed escalation rate separately. A single "escalation accuracy" number hides the tradeoff. In most deployments, you should tolerate a higher false escalation rate (5–15%) to keep the missed escalation rate very low (< 2%).

### Setup Steps

1. **Define your escalation taxonomy.** Categorize the reasons your agent should escalate: low confidence, high-stakes action, sensitive topic, ambiguous intent, explicit user request, policy requirement. Each category may need a different threshold.
2. **Build a labeled test set.** Create 100–200 test cases spanning all escalation categories. For each case, label the correct escalation decision (escalate or handle autonomously) with a brief rationale. Include borderline cases — these are where calibration matters most.
3. **Measure current escalation accuracy.** Run the test set through your agent and compute: escalation precision (what fraction of escalations were correct?), escalation recall (what fraction of cases that should escalate did escalate?), and the false escalation and missed escalation rates.
4. **Compute Expected Calibration Error.** Bin the agent's confidence scores into ranges (0–0.2, 0.2–0.4, etc.). Within each bin, compare the stated confidence to the actual accuracy. Perfect calibration means an agent that says "80% confident" is correct 80% of the time.
5. **Run threshold sweep analysis.** Vary the escalation threshold and plot precision vs. recall. Find the operating point that minimizes cost-weighted errors for your use case.
6. **Set up ongoing monitoring.** Track escalation rates in production. Alert on sudden changes — a spike in escalation rate may indicate model degradation; a drop may indicate missed escalations.

### Anti-Pattern

> **Anti-Pattern: Fixed Threshold Across All Categories**
>
> Setting a single confidence threshold (e.g., "escalate when confidence < 0.7") for all types of queries treats a low-confidence greeting the same as a low-confidence financial transaction. Use category-specific thresholds: routine queries can tolerate lower escalation sensitivity, while high-stakes actions need aggressive escalation triggers. A tiered threshold system — informational (0.5), standard (0.7), high-stakes (0.9) — catches critical errors without drowning reviewers in false escalations.

### Evaluation Patterns

- **Precision-Recall Tradeoff Mapping.** Plot escalation precision against recall across multiple thresholds. Identify the "knee" where further threshold tightening produces diminishing safety gains at increasing human cost. Document your chosen operating point and the rationale.
- **Category-Specific Calibration.** Measure ECE separately for each escalation category (financial, safety, routine, etc.). An agent may be well-calibrated for factual questions but poorly calibrated for safety-sensitive topics. Category-specific analysis reveals hidden weaknesses.
- **Escalation Latency Measurement.** Measure how quickly the agent escalates once it encounters an escalation trigger. Some agents "try and fail" before escalating, adding latency and potentially generating a partial (wrong) answer. Evaluate whether escalation happens early in the processing pipeline.
- **Context Sufficiency for Reviewers.** When the agent escalates, does it provide enough context for the human reviewer to make a good decision? Evaluate: does the escalation include the user's question, the agent's attempted answer, the confidence score, and the reason for escalation?

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 50 routine questions the agent should handle autonomously | Agent doesn't over-escalate on easy cases | False escalation rate < 10% |
| 50 high-stakes scenarios that require human review | Agent correctly identifies cases needing human involvement | Missed escalation rate < 2% |
| 100 cases spanning confidence levels 0.1–0.9 | Agent's confidence scores predict actual accuracy | ECE < 0.10 |
| 30 escalated cases reviewed by a human for context sufficiency | Escalation handoff includes enough context for the reviewer | ≥ 90% of escalations rated "sufficient context" by reviewer |

---

## 2. Human Evaluator Quality & Inter-Rater Reliability

When humans score your agent's outputs, do they agree with each other — and can you trust their labels?

### When to Use

Your evaluation process includes human reviewers who score agent conversations — for quality sampling, ground truth labeling, or eval set creation. You need to verify that human judgments are consistent, calibrated, and trustworthy. Without this, your entire evaluation pipeline may be built on unreliable ground truth.

Use this scenario when:
- Multiple human reviewers score agent conversations (quality audits, labeling, calibration)
- You are building or maintaining a human-labeled evaluation dataset
- You use human judgments to calibrate automated LLM judges
- Stakeholders question whether quality scores are consistent and fair

> **Related scenarios:** For automated quality sampling, see [Continuous Production Evaluation](continuous-production-evaluation.md) (online quality sampling). That scenario uses human-in-the-loop calibration to validate automated judges — this scenario evaluates the human calibration process itself.

### Why Inter-Rater Reliability Matters

Consider this: if two reviewers score the same conversation and disagree 40% of the time, your "ground truth" has a 40% noise floor. Any automated judge calibrated against this ground truth inherits that noise. And any quality metric derived from these scores has a wide, unacknowledged confidence interval.

Research shows that trained reviewers achieve 15–20% higher inter-rater reliability than untrained reviewers. The difference is not innate judgment — it's rubric clarity, calibration practice, and ongoing quality monitoring.

### Key Metrics

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Cohen's Kappa (κ)** | Agreement between two raters, corrected for chance | ≥ 0.80 (substantial agreement) |
| **Krippendorff's Alpha** | Agreement among multiple raters, handles missing data | ≥ 0.80 |
| **Percentage Agreement** | Simple agreement rate (use as a supplement, not primary metric) | ≥ 85% |
| **Annotator Drift** | Whether individual raters' scoring patterns change over time | < 5% shift per month |

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Overlapping Assignments | Assign the same conversations to multiple reviewers to measure agreement |
| Calibration Sessions | Regular sessions where reviewers score the same cases and discuss disagreements |
| Gold Standard Monitoring | Embed pre-scored "gold" conversations in review queues to track individual reviewer accuracy |
| Rubric Stress Testing | Test the rubric on ambiguous edge cases to identify dimensions that produce disagreement |

> **Tip:** The most common cause of low inter-rater reliability is not reviewer carelessness — it's rubric ambiguity. When reviewers disagree, the first question should always be "is the rubric clear enough?" not "which reviewer is wrong?"

### Setup Steps

1. **Design a clear rubric.** For each scoring dimension (accuracy, helpfulness, tone, etc.), provide: a definition, anchor examples for each score level (1–5), and explicit guidance for borderline cases. Ambiguous rubrics guarantee disagreement.
2. **Run an initial calibration session.** Have all reviewers independently score the same 20–30 conversations. Then meet to compare scores, discuss disagreements, and refine the rubric. Repeat until Cohen's κ ≥ 0.75 on the calibration set.
3. **Set up overlapping assignments.** In regular review operations, ensure at least 15–20% of conversations are scored by two or more reviewers. Use these overlaps to compute ongoing inter-rater reliability metrics.
4. **Embed gold standards.** Create 30–50 "gold" conversations with expert-consensus scores. Randomly insert them into reviewer queues (without marking them as gold). Track each reviewer's accuracy against gold standards.
5. **Monitor for annotator drift.** Track each reviewer's score distribution over time. Alert when a reviewer's mean score shifts by more than 0.5 points or when their agreement with gold standards drops below 80%.
6. **Schedule quarterly recalibration.** Every quarter, run a full calibration session with the current rubric. Update anchor examples based on new edge cases encountered. Document rubric changes with version numbers.

### Anti-Pattern

> **Anti-Pattern: Treating Majority Vote as Ground Truth**
>
> When three reviewers score a conversation and two agree, it's tempting to take the majority vote as truth. But if the rubric is ambiguous, the majority may be consistently wrong in the same way — they've converged on a shared misinterpretation, not the correct interpretation. Majority vote only works when you've already verified that individual reviewers are well-calibrated. Always check inter-rater reliability _before_ trusting aggregated labels.

### Evaluation Patterns

- **Calibration Session Cadence.** Run a 30-conversation calibration session monthly. Track κ over time. If κ drops below 0.75, halt production scoring and recalibrate before continuing. Document every rubric clarification.
- **Gold Standard Accuracy Tracking.** Plot each reviewer's gold standard accuracy weekly. Reviewers who fall below 80% accuracy receive additional calibration. This catches both consistent bias and gradual drift.
- **Dimension-Level Agreement.** Compute κ separately for each scoring dimension. Accuracy may show κ = 0.85 while helpfulness shows κ = 0.60 — indicating the helpfulness rubric needs refinement. Fix the weakest dimension first.
- **Disagreement Root Cause Analysis.** When reviewers disagree, categorize the cause: rubric ambiguity (fix the rubric), reviewer error (additional training), or genuinely borderline case (document as edge case). Track the distribution of root causes.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 30 conversations scored by all 3 reviewers | Reviewers agree on quality assessments | Cohen's κ ≥ 0.80 across all pairs |
| 50 gold standard conversations in review queue | Individual reviewers are accurate | Each reviewer ≥ 80% agreement with gold standard |
| Reviewer score distributions over 3 months | No individual reviewer is drifting | Score means within ±0.5 of group mean, no monthly trend > 0.3 |
| κ computed per dimension (accuracy, helpfulness, tone, safety) | All rubric dimensions produce reliable scores | κ ≥ 0.75 for every dimension |

---

## 3. Approval Workflow Evaluation

When your agent pauses for human approval before executing high-stakes actions, does the workflow function correctly and efficiently?

### When to Use

Your agent implements interrupt-and-resume patterns — it pauses execution at critical decision points and waits for human approval before proceeding with high-stakes actions (financial transactions, data modifications, external communications, irreversible operations). You need to evaluate the approval workflow itself: checkpoint accuracy, context packaging, latency, and error handling.

Use this scenario when:
- Your agent executes actions that require human authorization (transfers, deletions, escalations to external systems)
- You use frameworks with built-in approval patterns (LangGraph `interrupt()`, CrewAI `human_input`, HumanLayer `@require_approval()`, or custom implementations)
- Approval delays are causing user frustration or operational bottlenecks
- You need to verify that the approval workflow handles edge cases (timeout, rejection, partial approval)

> **Related scenarios:** For evaluating tool invocation correctness, see [Tool & Connector Invocations](tool-and-connector-invocations.md). For evaluating the agent's step-by-step execution, see [Trajectory & Stepwise Evaluation](trajectory-and-stepwise-evaluation.md). This scenario specifically evaluates the human approval gate between the agent's decision and its execution.

### Approval Workflow Components

A well-functioning approval workflow requires four things to work correctly:

| Component | What It Does | Failure Mode |
|-----------|-------------|-------------|
| **Checkpoint Trigger** | Agent correctly identifies when an action needs approval | Missed checkpoint: high-risk action executes without approval |
| **Context Package** | Agent provides the reviewer with enough information to make a good decision | Insufficient context: reviewer approves blindly or rejects conservatively |
| **State Preservation** | Agent's execution state is saved so it can resume exactly where it left off | State loss: agent restarts from scratch or loses user context after approval |
| **Outcome Handling** | Agent correctly handles approval, rejection, modification, and timeout | Poor handling: agent crashes on rejection, ignores modifications, or hangs on timeout |

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Checkpoint Coverage Testing | Verify that all high-risk actions trigger an approval checkpoint |
| Context Sufficiency Review | Have reviewers rate whether the approval request contains enough information to decide |
| State Persistence Testing | Approve after delays (1 min, 1 hour, 1 day) to verify state is correctly preserved |
| Rejection and Modification Paths | Test that the agent handles non-happy-path outcomes gracefully |

> **Tip:** The most overlooked failure mode is the _slow approval_. Test what happens when a reviewer takes 30 minutes, 2 hours, or 24 hours to respond. Does the agent maintain state? Does the user get stuck? Does the system time out gracefully? Many approval implementations only work when the reviewer responds within seconds.

### Setup Steps

1. **Inventory your approval points.** List every action your agent can take that requires human approval. For each, document: what triggers the checkpoint, who approves, what information they need, and the expected response time.
2. **Build checkpoint coverage tests.** For each approval point, create test cases that should trigger the checkpoint. Run them and verify the checkpoint fires. Also create test cases for similar-but-lower-risk actions that should _not_ trigger the checkpoint — verify these proceed autonomously.
3. **Evaluate context packages.** For each triggered checkpoint, have a reviewer (not the regular approver) rate the context package on a 1–5 scale: "Given only this information, could you make a confident approve/reject decision?" Target ≥ 4.0/5.0.
4. **Test state preservation.** Trigger an approval checkpoint, then approve after increasing delays (immediately, 5 minutes, 1 hour, simulated overnight). After approval, verify the agent resumes correctly — same user context, same action parameters, same conversation state.
5. **Test non-happy paths.** For each checkpoint: (a) reject the action — does the agent offer alternatives? (b) modify the action parameters — does the agent execute with the modifications? (c) let the request time out — does the system handle it gracefully?
6. **Measure approval latency impact.** Track the end-to-end time from user request to action completion, broken down by: agent processing time, time waiting for approval, and post-approval execution time. Identify bottlenecks.

### Anti-Pattern

> **Anti-Pattern: Approval Without Context**
>
> Some implementations send approval requests that say "Agent wants to execute action X. Approve?" without explaining why the agent chose this action, what the user asked for, or what the consequences are. Reviewers facing context-free approval requests either rubber-stamp everything (defeating the purpose) or reject conservatively (frustrating users). Every approval request must include: what the user asked, what the agent proposes, why, and what the impact will be.

### Evaluation Patterns

- **Checkpoint Completeness Audit.** Inventory all high-risk actions. For each, verify a checkpoint exists and fires correctly. Any high-risk action without an approval gate is a critical gap. Review quarterly as new capabilities are added.
- **False Pause Rate Analysis.** Track how often the agent pauses for approval on actions that don't actually need it. A high false pause rate (> 10% of approvals are for low-risk actions) indicates the checkpoint triggers are too broad and need refinement.
- **Approval Decision Quality.** Track what fraction of approval requests are approved, rejected, or modified. If > 95% are approved, the checkpoint may be too sensitive (too many false pauses). If > 20% are rejected, the agent may be proposing incorrect actions.
- **Resume Fidelity Testing.** After an approval delay, compare the agent's resumed behavior against a baseline where the approval was instant. Any difference in behavior, context, or output indicates a state preservation issue.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 20 high-risk actions (financial, data deletion, external communication) | Every high-risk action triggers an approval checkpoint | 100% checkpoint coverage for high-risk actions |
| 20 low-risk actions similar to high-risk ones | Low-risk variants don't unnecessarily trigger checkpoints | False pause rate < 10% |
| 15 approval requests rated by independent reviewer | Context packages are sufficient for decision-making | Mean sufficiency rating ≥ 4.0/5.0 |
| 10 checkpoint-triggered actions approved after 1-hour delay | Agent resumes correctly after delay | 100% state preservation; resumed output matches instant-approval baseline |
| 10 rejected actions | Agent handles rejection gracefully | Agent explains the rejection to the user and offers alternatives |

---

## 4. Hybrid AI-Human Evaluation at Scale

As your evaluation needs grow beyond what human reviewers can handle, does your AI-assisted evaluation system produce trustworthy results under human governance?

### When to Use

Your agent's production volume or action trace complexity exceeds what human reviewers can comprehensively evaluate. You've implemented (or are considering) AI-as-evaluator — using LLM judges to score agent outputs at scale, with humans setting the rules, monitoring the AI evaluator's quality, and handling cases the AI evaluator flags as uncertain.

Use this scenario when:
- Your agent handles thousands of conversations per day and you cannot sample enough for human review alone
- Agent action traces are complex (multi-step, multi-tool) and too dense for efficient human review
- You are using LLM judges for automated scoring and need to validate their reliability
- You need to define the governance layer: who sets the rules, how are they enforced, and what's the audit trail?

> **Related scenarios:** For setting up automated quality sampling, see [Continuous Production Evaluation](continuous-production-evaluation.md). For ensuring human evaluator quality, see scenario 2 above. This scenario evaluates the architecture that combines AI evaluation with human governance at scale.

### The Scaling Challenge

Pure human evaluation has hit a scaling wall. A 2026 analysis found that modern agent architectures produce action traces too dense for comprehensive human review — a single multi-step agent task can generate dozens of tool calls, intermediate reasoning steps, and branching decisions. Reviewing 5% of production traffic at this density would require a dedicated evaluation team larger than the development team.

The emerging pattern is **layered evaluation**: AI handles volume (scoring every conversation), humans handle governance (setting rules, calibrating judges, reviewing edge cases, auditing quality). But this architecture introduces its own evaluation requirements — you need to evaluate the evaluator.

### Three-Layer Governance Architecture

| Layer | Role | Evaluated By |
|-------|------|-------------|
| **AI Evaluator** | Scores every production conversation in real-time against rubrics defined by the governance layer | Human spot-checks, gold standard tests, agreement metrics |
| **Human Governance** | Sets evaluation rubrics, defines escalation thresholds, reviews AI-flagged edge cases, approves rubric changes | Audit trails, governance coverage metrics, response time SLAs |
| **Audit & Oversight** | Periodic review of the entire evaluation system for blind spots, bias, and drift | Scheduled audits, external review, red-teaming of the evaluator itself |

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| AI-Human Agreement Testing | Compare AI evaluator scores against expert human scores on the same conversations |
| Evaluator Red-Teaming | Deliberately feed the AI evaluator adversarial or tricky cases to test its robustness |
| Governance Coverage Audit | Verify that every evaluation rule, threshold, and escalation path is documented and owned |
| Escalation Path Testing | Confirm that cases the AI evaluator flags as uncertain actually reach a human reviewer in time |

> **Tip:** The most dangerous failure mode is not when the AI evaluator gets a score wrong — it's when it gets a score wrong _confidently_. An AI judge that says "this response is clearly good" when it's subtly harmful is worse than one that says "I'm uncertain" and escalates. Evaluate your AI evaluator's calibration, not just its accuracy.

### Setup Steps

1. **Define the governance scope.** Document every evaluation dimension the AI evaluator is responsible for. For each dimension, specify: the rubric, the score scale, the escalation threshold (below which the AI should flag for human review), and the governance owner (who can change these parameters).
2. **Build an agreement test set.** Create 200+ conversations scored by expert humans. Run the AI evaluator on the same set. Compute agreement metrics: Cohen's κ for categorical decisions, Pearson r for continuous scores, and the rate of "high-confidence errors" (AI evaluator is very confident but wrong).
3. **Red-team the AI evaluator.** Create adversarial test cases: subtly harmful responses that sound helpful, correct answers with wrong reasoning, responses that violate policy in non-obvious ways. Measure how many the AI evaluator catches vs. misses.
4. **Implement escalation paths.** Configure the AI evaluator to flag uncertain cases (below a confidence threshold) for human review. Test: do flagged cases actually reach a human? How quickly? Is there a queue backlog?
5. **Set up governance audit trails.** Log every rubric change, threshold adjustment, and escalation path modification with: who changed it, when, why, and what the old value was. Schedule monthly reviews of the audit trail.
6. **Run quarterly blind audits.** Have an independent reviewer (not the regular governance team) evaluate a random sample of 100 conversations. Compare their scores against the AI evaluator's scores. This catches systematic biases that the regular governance team may share with the AI evaluator.

### Anti-Pattern

> **Anti-Pattern: "Set It and Forget It" AI Evaluation**
>
> Deploying an AI evaluator, validating it once, and then trusting it indefinitely is a recipe for silent evaluation failure. AI evaluators drift just like the agents they evaluate — the scoring model may be updated, the conversation distribution may shift, or the rubric may become outdated. An AI evaluator that was 90% accurate six months ago may be 70% accurate today. Continuous monitoring of AI-human agreement is not optional — it's the core governance requirement.

### Evaluation Patterns

- **Agreement Trend Monitoring.** Track AI-human agreement metrics weekly. Plot κ and Pearson r over time. Alert when agreement drops below thresholds (κ < 0.75). Investigate root causes: rubric drift, evaluator model change, or conversation distribution shift.
- **High-Confidence Error Rate.** Separately track cases where the AI evaluator was highly confident (> 0.9 confidence) but wrong. These are the most dangerous errors because they won't be flagged for human review. Target: < 2% high-confidence error rate.
- **Governance Response Time.** Track how quickly the governance team responds to AI-flagged edge cases. Set SLAs (e.g., urgent flags reviewed within 4 hours, standard flags within 24 hours). Unreviewed flags are evaluation blind spots.
- **Evaluator Bias Detection.** Compare AI evaluator scores across demographic groups, topic categories, and conversation lengths. Flag any systematic scoring differences that aren't explained by actual quality differences. An evaluator that consistently scores shorter responses higher, for example, may be encoding a length bias rather than measuring quality.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 200 expert-scored conversations vs. AI evaluator scores | AI evaluator agrees with human experts | Cohen's κ ≥ 0.80, Pearson r ≥ 0.85 |
| 50 adversarial test cases (subtle policy violations, harmful-but-helpful) | AI evaluator catches non-obvious failures | Detection rate ≥ 80% on adversarial set |
| AI-flagged uncertain cases from the past month | Flagged cases actually reach human reviewers | 100% routing accuracy, < 24hr review time for standard flags |
| AI evaluator scores segmented by topic, length, and user group | No systematic scoring bias | No segment differs from the mean by > 0.3 points (on 1–5 scale) without explanation |

---

## Aggregation & Reporting

When using multiple HITL evaluation scenarios together, here's how to roll them into a single oversight health picture:

### HITL Evaluation Dashboard

| Dashboard Section | Key Metrics | Update Frequency |
|------------------|-------------|-----------------|
| **Escalation Health** | False escalation rate, missed escalation rate, ECE, escalation volume trend | Daily |
| **Reviewer Quality** | Cohen's κ (overall and per-dimension), gold standard accuracy per reviewer, annotator drift | Weekly |
| **Approval Workflows** | Checkpoint coverage, false pause rate, approval latency, resume fidelity | Per-deployment |
| **AI Evaluator Trust** | AI-human κ, high-confidence error rate, governance response time, bias metrics | Weekly |

### Coverage Targets

- **Escalation calibration**: Test with ≥ 100 labeled cases per escalation category, quarterly
- **Inter-rater reliability**: Maintain ≥ 15% overlapping assignments; κ ≥ 0.80
- **Approval workflows**: 100% checkpoint coverage for all designated high-risk actions
- **AI-human agreement**: Monthly agreement testing on ≥ 200 conversations; κ ≥ 0.80

---

## Getting Started Checklist

If you're implementing human-in-the-loop evaluation for the first time, follow this prioritized sequence:

1. ☐ **Audit your escalation points** — list every situation where your agent should involve a human; verify each has a working trigger
2. ☐ **Measure escalation accuracy** — build a 100-case labeled test set and compute false escalation and missed escalation rates
3. ☐ **Write evaluation rubrics with anchor examples** — clear rubrics are the single highest-leverage improvement for reviewer quality
4. ☐ **Run a calibration session** — have all reviewers score 20–30 of the same conversations; compute κ and discuss disagreements
5. ☐ **Set up overlapping assignments** — ensure 15–20% of reviews are dual-scored for ongoing reliability monitoring
6. ☐ **Evaluate approval context sufficiency** — review 20 recent approval requests for completeness
7. ☐ **Implement gold standard monitoring** — embed pre-scored conversations in review queues to track individual reviewer accuracy
8. ☐ **If using AI evaluators, build an agreement test set** — score 200+ conversations with both AI and expert humans
9. ☐ **Set up governance audit trails** — log every rubric change, threshold adjustment, and escalation path modification
10. ☐ **Schedule quarterly oversight audits** — review the entire HITL evaluation system for blind spots and drift

> **Start with steps 1–5.** These provide the highest value with the lowest effort and apply to any agent with human oversight. Steps 6–7 are for teams with approval workflows and dedicated review teams. Steps 8–10 are for mature deployments using AI-assisted evaluation at scale.

---

[Back to library](../README.md) | [All capability scenarios](README.md)
