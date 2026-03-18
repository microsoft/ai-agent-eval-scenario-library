# Continuous Production Evaluation

> Scenarios for monitoring your agent's quality, detecting degradation, and maintaining reliability after deployment. Pre-launch evaluation tells you whether your agent _works_; continuous production evaluation tells you whether it _keeps working_ — and alerts you before users notice a problem.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Continuous Production Evaluation Matters

An agent that passes all pre-launch tests can still degrade in production. Real-world conditions introduce failure modes that offline evaluation cannot anticipate:

- **Knowledge drift.** The knowledge sources your agent relies on change — policies update, documents are revised, links break — but your agent's behavior doesn't adapt. Yesterday's correct answer becomes today's wrong answer.
- **Input distribution shift.** Users start asking questions outside the patterns your test set covers. New topics, new phrasing, new edge cases appear gradually.
- **Prompt and model drift.** System prompt edits, model version updates, or configuration changes introduce subtle behavioral shifts that pre-launch tests don't catch.
- **Silent quality erosion.** Quality doesn't fail all at once — it degrades by fractions of a percent per week. Without monitoring, you discover the problem months later through user complaints.

Research consistently shows that the vast majority of ML systems degrade without proactive monitoring. Continuous evaluation transforms "hope it still works" into "know it still works."

### The Map-Measure-Manage Framework

The scenarios in this guide follow a three-phase approach that has emerged as best practice for production agent monitoring:

1. **Map** — Define what "good" looks like. Establish canonical behaviors, quality baselines, and expected distributions.
2. **Measure** — Quantify ongoing quality through sampling, automated judges, and statistical tests.
3. **Manage** — Operationalize with alerts, dashboards, and response playbooks so degradation triggers action.

---

## 1. Drift Detection & Alerting

Is your agent's behavior shifting away from its established baseline?

### When to Use

Your agent has been running in production for at least 2–4 weeks (enough to establish a baseline), and you need to detect when its behavior changes in ways that may indicate quality degradation. This is the foundation of continuous production evaluation — without drift detection, you are flying blind.

Use this scenario when:
- Your agent relies on knowledge sources that change over time (documents, policies, databases)
- You have updated the agent's system prompt, model version, or configuration
- User traffic patterns shift seasonally or due to business events
- You need early warning before quality degradation becomes user-visible

> **Related scenarios:** For pre-launch regression testing, see [Regression Testing](regression-testing.md). For evaluating whether knowledge sources are correctly grounded, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). This scenario extends both into continuous production monitoring.

### Four Types of Agent Drift

| Drift Type | What Changes | Detection Method | Example |
|-----------|-------------|-----------------|---------|
| **Concept drift** | The agent's output quality or behavior changes | Score distribution monitoring | Agent starts giving shorter, less helpful answers |
| **Data drift** | The distribution of user inputs shifts | Input feature monitoring | Sudden spike in questions about a new product not in the knowledge base |
| **Prompt drift** | System prompts or instructions are modified | Version-controlled prompt diffing | A prompt edit inadvertently removes a safety instruction |
| **Context drift** | Knowledge sources become stale or change | Source freshness tracking | A linked FAQ page is deleted, causing retrieval failures |

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Statistical Distribution Tests | Compare current score distributions against baseline using KS test, PSI, or KL divergence |
| Threshold Alerting | Fire alerts when rolling averages cross predefined quality floors |
| Cohort Comparison | Compare quality metrics across time windows (this week vs. last week vs. baseline) |
| Automated LLM Judge (Periodic) | Run a grading model on sampled production conversations to compute quality scores |

> **Tip:** Start with simple threshold alerting on a single quality metric (e.g., LLM-judge helpfulness score). Add statistical tests as you accumulate enough data for meaningful comparisons — typically after 2–4 weeks of baseline data.

### Setup Steps

1. **Establish your baseline.** Run your standard evaluation suite on the current production agent. Record the distribution of scores — not just the mean, but percentiles (p25, p50, p75, p95). This is your reference.
2. **Define drift thresholds.** Set alert levels for each metric. Example: a warning when the 7-day rolling mean drops more than 5% below baseline; a critical alert at 10% below.
3. **Choose your statistical test.** For continuous scores (like helpfulness ratings), use the Kolmogorov-Smirnov test or Population Stability Index. For categorical outcomes (pass/fail), use a chi-square test.
4. **Instrument your pipeline.** Log every production conversation with metadata: timestamp, user query, agent response, any tool calls, and configuration version (prompt hash, model version).
5. **Schedule drift checks.** Run statistical comparisons daily or weekly depending on traffic volume. Low-traffic agents (< 100 conversations/day) should use weekly windows; high-traffic agents can use daily.
6. **Build an alert response playbook.** When a drift alert fires, who investigates? What do they check first? Define escalation steps before you need them.

### Anti-Pattern

> **Anti-Pattern: Setting Thresholds Too Tight**
>
> A common mistake is alerting on any score change, no matter how small. Natural variance means your metrics will fluctuate — a 1–2% swing is normal. If you alert on every fluctuation, the team will start ignoring alerts (alert fatigue), and the real degradation signals get lost. Set thresholds based on statistical significance and business impact, not on noise.

### Evaluation Patterns

- **Baseline Cohort Comparison.** Compare this week's score distribution against the initial baseline cohort. Use PSI > 0.1 as a warning threshold and PSI > 0.25 as critical.
- **Rolling Window Trend.** Track a 7-day rolling average of quality scores. Alert on sustained downward trends (3+ consecutive declining windows) rather than single-day dips.
- **Prompt Version Diffing.** Hash every version of the system prompt. When the hash changes, automatically run a targeted eval suite to detect behavioral changes before they affect users.
- **Knowledge Freshness Monitor.** Track the last-modified dates of knowledge sources. Alert when sources haven't been updated in longer than their expected refresh cycle, or when referenced URLs return errors.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 500 sampled production conversations from this week | Quality score distribution hasn't shifted from baseline | PSI < 0.1, KS test p-value > 0.05 |
| Same 50 "golden" test queries run weekly | Fixed benchmark scores remain stable across weeks | No metric drops > 5% from baseline |
| Prompt change deployed Tuesday; compare Mon vs. Wed scores | Prompt edit didn't introduce regression | Score distributions are statistically indistinguishable |
| Knowledge source URL check (automated crawl) | All referenced documents are still accessible and current | Zero 404 errors, no documents older than refresh SLA |

---

## 2. Online Quality Sampling

Are you continuously measuring the quality of real production conversations?

### When to Use

Your agent handles enough production traffic that you cannot manually review every conversation, and you need an automated system to estimate ongoing quality from a representative sample. This is the "measure" phase of Map-Measure-Manage.

Use this scenario when:
- Your agent handles more than 50 conversations per day (manual review doesn't scale)
- You need quality metrics that reflect real user interactions, not just synthetic test cases
- You want to detect emerging failure patterns that your pre-built test set doesn't cover
- Stakeholders require regular quality reporting (weekly, monthly)

> **Related scenarios:** For using LLM judges to evaluate responses, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md). For drift detection using the scores from sampling, see scenario 1 above. This scenario focuses on the sampling and scoring infrastructure itself.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Stratified Random Sampling | Select a representative subset of conversations across topics, user segments, and time periods |
| LLM-as-Judge Scoring | Use a grading model to evaluate sampled conversations on predefined rubrics |
| Human-in-the-Loop Calibration | Periodically have human reviewers score the same samples to validate the automated judge |
| Funnel Analysis | Track conversation completion rates, escalation rates, and user satisfaction signals |

> **Tip:** Sample at least 5–10% of production traffic for automated scoring, and have humans review a calibration set of 50–100 conversations monthly to catch judge drift. If your LLM judge disagrees with human reviewers more than 15% of the time, recalibrate the rubric.

### Setup Steps

1. **Define your quality rubric.** Decide which dimensions matter for your agent: accuracy, helpfulness, safety, tone, completeness. Weight them based on business priority.
2. **Design your sampling strategy.** Random sampling is a start, but stratified sampling is better — ensure you sample proportionally across topics, times of day, and user segments. Over-sample rare but high-risk categories (e.g., escalations, safety-flagged conversations).
3. **Configure automated scoring.** Set up an LLM judge (or custom evaluator) to score each sampled conversation against your rubric. Log scores alongside the conversation metadata.
4. **Establish human calibration.** Each month, have 2–3 human reviewers independently score 50–100 of the same conversations the automated judge scored. Compute inter-rater agreement (Cohen's kappa) and judge-vs-human agreement.
5. **Build dashboards.** Create a dashboard showing: overall quality score trend, per-dimension scores, score distribution (not just averages), and lowest-scoring conversations for review.
6. **Set review triggers.** Flag any conversation scoring below a threshold (e.g., bottom 5%) for manual review. These are your canaries — emerging failure patterns often appear first in the tail.

### Anti-Pattern

> **Anti-Pattern: Scoring Only on Accuracy**
>
> An agent can be technically accurate and still deliver a poor experience — terse answers, condescending tone, unnecessary jargon. Production quality sampling must cover multiple dimensions. If you only score accuracy, you'll miss the quality problems that drive user dissatisfaction and escalation.

### Evaluation Patterns

- **Multi-Dimensional Rubric Scoring.** Score every sampled conversation on 3–5 dimensions (accuracy, helpfulness, tone, completeness, safety) with a 1–5 scale per dimension. Track each dimension independently to identify which aspects are degrading.
- **Tail Analysis.** Analyze the bottom 5% of scored conversations in detail. Cluster them by failure type (wrong answer, refusal, hallucination, tool error, tone issue). Track how the failure distribution changes over time.
- **Confidence-Weighted Sampling.** Sample more heavily from conversations where the agent expressed low confidence, requested clarification, or escalated. These are higher-risk interactions that deserve more evaluation attention.
- **Topic-Stratified Reporting.** Break quality metrics down by topic or intent category. A global score of 4.2/5 may hide the fact that "refund requests" score 2.8 while "product info" scores 4.7.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| 200 randomly sampled conversations from this week | Overall quality remains above threshold | Mean helpfulness ≥ 4.0/5.0, accuracy ≥ 90% |
| 50 conversations scored by both LLM judge and human reviewer | Automated judge is calibrated to human judgment | Cohen's kappa ≥ 0.7, disagreement rate < 15% |
| Bottom 5% of scored conversations | Failure patterns are identified and actionable | Each conversation has a failure category and root cause |
| Per-topic quality breakdown | No topic is significantly underperforming | All topics within 1 standard deviation of global mean |

---

## 3. Regression Testing & CI/CD Integration

Does every change to your agent go through automated quality gates before reaching users?

### When to Use

Your team makes regular changes to the agent — prompt edits, knowledge source updates, model upgrades, configuration changes — and you need automated evaluation to catch regressions before they reach production. This is the guardrail that prevents "it worked on my machine" failures.

Use this scenario when:
- Multiple team members edit the agent's prompts or configuration
- Knowledge sources are updated on a regular schedule
- You are migrating between model versions
- You have experienced production incidents caused by changes that weren't tested

> **Related scenarios:** For foundational regression testing concepts, see [Regression Testing](regression-testing.md). This scenario extends regression testing into a continuous, automated CI/CD pipeline with deployment gating.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Golden Test Suite | A curated set of test cases that must all pass before any deployment |
| Before/After Comparison | Run the same eval suite on the current and proposed versions; compare scores |
| Statistical Significance Testing | Ensure observed differences are real, not random noise |
| Canary Deployment | Route a small percentage of traffic to the new version and compare metrics |

> **Tip:** Organize your golden test suite into tiers: Tier 1 (P0 — must pass, blocks deployment), Tier 2 (P1 — should pass, triggers review), Tier 3 (P2 — informational, tracked but non-blocking). This prevents a single flaky test from blocking all deployments.

### Setup Steps

1. **Build a golden test suite.** Curate 50–200 test cases covering your critical scenarios. Include happy-path cases, known edge cases, and cases derived from past production incidents. Tag each with a tier (P0/P1/P2).
2. **Automate the eval pipeline.** Set up CI/CD integration so that every change (prompt edit, knowledge update, model swap) automatically triggers the golden test suite before deployment.
3. **Define deployment gates.** Set clear rules: all P0 tests must pass, P1 pass rate must be ≥ 95%, no individual P1 score can drop by more than 10%.
4. **Run before/after comparisons.** For each proposed change, run the test suite on both the current production version and the proposed version. Report the delta for every metric.
5. **Apply statistical rigor.** For changes that affect P1/P2 tests, use paired statistical tests (e.g., McNemar's test for pass/fail, paired t-test for scores) to distinguish real regressions from noise.
6. **Maintain the test suite.** Review and update the golden test suite quarterly. Add new test cases from production incidents. Remove tests that are no longer relevant. Track test suite coverage metrics.

### Anti-Pattern

> **Anti-Pattern: "We'll Test It in Production"**
>
> Skipping pre-deployment evaluation because "we can always roll back" is a false economy. Users who receive bad answers during the rollback window may not come back. And rollbacks are rarely instant — detection, decision, and execution all take time. Every change should pass automated gates before reaching any user.

### Evaluation Patterns

- **Deployment Blocking Gate.** Configure your CI/CD pipeline to block deployment when any P0 test fails. No manual override except with VP-level approval and an incident ticket.
- **Delta Reporting.** For every proposed change, generate a report showing: tests that improved, tests that regressed, tests unchanged, and the statistical significance of each change.
- **Incident-Driven Test Expansion.** After every production incident, create a new test case that would have caught it. Add it to the golden suite with appropriate tier. This ensures the same failure never recurs.
- **Shadow Evaluation.** Run proposed changes against live production traffic in shadow mode (compute the response but don't show it to users). Compare shadow responses against production responses to detect regressions at scale.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| Golden suite run after prompt edit | Edit didn't break existing behavior | All P0 pass, P1 pass rate ≥ 95% |
| Same 100 test cases on current vs. proposed model | Model upgrade doesn't regress quality | No statistically significant score decrease (p < 0.05) |
| Shadow evaluation on 1,000 production queries | New version matches or exceeds current quality | New version score ≥ current version score - 2% |
| Test suite coverage audit | Test suite covers all critical scenarios | Every active topic/intent has at least 3 test cases |

---

## 4. Cost & Efficiency Monitoring

Are you tracking what your agent costs to operate alongside how well it performs?

### When to Use

Your agent runs in production and you need to understand its operational cost profile — token consumption, API costs, and latency — alongside its quality metrics. Quality without cost awareness leads to unsustainable deployments; cost cutting without quality measurement leads to degraded experiences.

Use this scenario when:
- Your agent makes multiple LLM calls per conversation (planner + executor + judge)
- Token costs are a significant line item in your operating budget
- You are comparing model configurations and need cost-quality tradeoffs
- Users complain about response latency

> **Related scenarios:** For evaluating whether the agent takes an efficient path, see [Trajectory & Stepwise Evaluation](trajectory-and-stepwise-evaluation.md) (efficiency scenarios). This scenario focuses on the operational cost and latency monitoring in production.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Token Accounting | Track input/output tokens per conversation, per tool call, and per session |
| Cost-per-Task Analysis | Compute the dollar cost to complete each type of user task |
| Latency Percentile Tracking | Monitor p50, p90, and p99 response latency — averages hide tail latency |
| Cost-Quality Pareto Analysis | Plot quality score vs. cost for different configurations to find the efficient frontier |

> **Tip:** The most useful metric is **cost per successful task completion**, not cost per conversation. An agent that takes 2 conversations to solve a problem costs more than the per-conversation metric suggests. Track end-to-end task resolution cost.

### Setup Steps

1. **Instrument token counting.** Log input tokens, output tokens, and total tokens for every LLM call. Include all calls — planner, executor, judge, retry attempts.
2. **Map tasks to conversations.** Define what constitutes a "task" for your agent. Track whether each task was successfully completed. Compute cost per successful completion.
3. **Set cost budgets.** Define per-conversation and per-task cost ceilings. Alert when individual conversations or rolling averages exceed the budget.
4. **Track latency at percentiles.** Log end-to-end response latency for every turn. Report p50, p90, and p99. Set SLA targets for each percentile.
5. **Build cost-quality dashboards.** Show cost and quality metrics side by side. Include: cost per task over time, cost by topic/intent, quality-cost scatter plot, and anomaly flags.
6. **Run periodic Pareto analysis.** When evaluating configuration changes (model swaps, prompt modifications, tool changes), plot the cost-quality frontier. Choose configurations that are on or near the Pareto frontier.

### Anti-Pattern

> **Anti-Pattern: Optimizing Cost in Isolation**
>
> Cutting costs without monitoring quality is a recipe for false savings. Switching to a cheaper model that degrades accuracy by 15% doesn't save money — it shifts costs to human escalation, user churn, and trust erosion. Always measure cost changes alongside quality changes. If a configuration is cheaper but lower quality, quantify the quality tradeoff before committing.

### Evaluation Patterns

- **Token Budget Enforcement.** Set maximum token budgets per conversation type. Log and alert on conversations that exceed the budget. Analyze over-budget conversations to identify inefficient patterns (e.g., excessive retries, verbose prompts).
- **Cost Anomaly Detection.** Monitor for sudden cost spikes — a single conversation that costs 10x the average is a signal. Common causes: infinite loops, excessive tool retries, or malicious prompt injection that triggers expensive operations.
- **Model Configuration Benchmarking.** Periodically benchmark 2–3 model configurations on the same test suite. Plot cost vs. quality for each. Switch configurations only when the new config is Pareto-dominant or the quality tradeoff is explicitly accepted.
- **Latency-Quality Correlation.** Track whether slower responses are also higher quality. If not, the extra latency is waste. If yes, consider async patterns for complex queries where users can tolerate a longer wait.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| All conversations from the past 7 days | Average cost per task is within budget | Cost per successful task ≤ $0.15 (or your budget) |
| Per-conversation token log | No conversations exceed 10x the median token count | < 1% of conversations exceed the cost ceiling |
| Three model configurations on 200 test cases | Identify the most cost-efficient configuration | Clear Pareto frontier; chosen config is on or near it |
| Hourly latency percentiles for the past 30 days | Latency SLAs are met consistently | p50 < 3s, p90 < 8s, p99 < 15s |

---

## 5. End-to-End Observability Architecture

Can you trace any production conversation from user input through every processing step to final output — and correlate it with business outcomes?

### When to Use

Your agent involves multiple components (retrieval, orchestration, tool execution, response generation) and you need visibility into every step to diagnose issues, optimize performance, and correlate agent behavior with business metrics. This is the infrastructure that makes all other production evaluation scenarios possible.

Use this scenario when:
- Your agent has 3+ components in its processing pipeline
- Diagnosing production issues currently requires manual log searching across multiple systems
- You need to correlate agent quality metrics with business KPIs (resolution rate, user satisfaction, escalation rate)
- Your team has grown beyond 2–3 people and needs shared visibility into agent behavior

> **Related scenarios:** For evaluating the correctness of each step, see [Trajectory & Stepwise Evaluation](trajectory-and-stepwise-evaluation.md). For tool invocation correctness, see [Tool & Connector Invocations](tool-and-connector-invocations.md). This scenario provides the observability infrastructure that feeds data to all other evaluation scenarios.

### Three-Level Tracing Architecture

| Level | What It Captures | Example |
|-------|-----------------|---------|
| **Session** | The full user interaction across multiple turns | User's complete conversation about a refund request |
| **Trace** | A single request-response cycle within a session | One turn: user asks "what's my refund status?" |
| **Span** | An individual operation within a trace | Knowledge retrieval call, LLM inference, tool execution |

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Trace Completeness Validation | Verify that every production conversation has a complete trace with no missing spans |
| Root-Cause Analysis Drill-Down | Given a low-quality conversation, trace back through spans to identify which component failed |
| Business KPI Correlation | Correlate agent quality scores with downstream business metrics (CSAT, resolution rate) |
| Dashboard Coverage Audit | Verify that dashboards expose the metrics needed for every evaluation scenario in this guide |

> **Tip:** Adopt OpenTelemetry-compatible tracing from the start. It's the emerging standard for agent observability, and most monitoring tools (Datadog, Grafana, Azure Monitor) support it natively. Retrofitting tracing into an untraced pipeline is significantly harder than building it in from day one.

### Setup Steps

1. **Implement three-level tracing.** Instrument your agent pipeline to create spans for every significant operation: retrieval calls, LLM inferences, tool executions, and response generation. Group spans into traces (per turn) and sessions (per conversation).
2. **Attach evaluation metadata.** For each trace, log: the user query, retrieved context (or a hash/summary), the full agent response, any tool calls and results, model version, prompt version hash, and latency.
3. **Validate trace completeness.** Run a daily check: what percentage of production conversations have complete traces with no missing spans? Target > 99% completeness.
4. **Build root-cause analysis workflows.** Create a playbook: when a conversation is flagged as low quality, the analyst can pull up the full trace, see which span contained the failure (bad retrieval? wrong tool? hallucinated response?), and take action.
5. **Correlate with business metrics.** Join agent quality scores with business KPIs in your data warehouse. Analyze: do conversations with higher quality scores correlate with higher CSAT? Lower escalation rates? Higher task completion?
6. **Audit dashboard coverage.** For each evaluation scenario in this guide (drift detection, quality sampling, regression testing, cost monitoring), verify that the relevant metrics are visible on a dashboard and alerting is configured.

### Anti-Pattern

> **Anti-Pattern: Logging Everything Without Structure**
>
> Dumping every log line into a single stream creates the illusion of observability without the utility. If you can't pull up a specific conversation and see every processing step in order — with timing, input/output, and outcome for each step — you don't have observability. You have a log pile. Structure first (session → trace → span), verbose logging second.

### Evaluation Patterns

- **Trace Completeness Score.** Compute daily: (conversations with complete traces / total conversations) × 100. Target ≥ 99%. Missing spans indicate instrumentation gaps.
- **Mean Time to Root Cause (MTRRC).** Track how long it takes an analyst to identify the root cause of a flagged conversation using the tracing tools. Target: < 15 minutes. If it takes longer, the tooling needs improvement.
- **Business Outcome Correlation.** Compute the Pearson correlation between agent quality scores and business KPIs (CSAT, resolution rate, escalation rate). If correlation is weak (r < 0.3), your quality metrics may not be measuring what users actually care about.
- **Alerting Coverage Matrix.** For each evaluation scenario in this guide, verify: (1) the relevant metric is computed, (2) a threshold is set, (3) an alert is configured, (4) a response playbook exists. Any gap is a monitoring blind spot.

### Practical Examples

| Test Input | What You're Checking | Expected Behavior |
|-----------|---------------------|-------------------|
| All production conversations from yesterday | Trace completeness is above threshold | ≥ 99% of conversations have complete session → trace → span chains |
| 10 randomly selected low-scoring conversations | Root cause can be identified within 15 minutes | Analyst can trace failure to a specific span and component |
| 30-day quality scores + CSAT data | Quality scores predict user satisfaction | Pearson r ≥ 0.5 between quality score and CSAT |
| Alerting coverage audit across all 5 scenarios in this guide | No monitoring blind spots | Every scenario has metrics, thresholds, alerts, and playbooks |

---

## Aggregation & Reporting

When using multiple production evaluation scenarios together, here's how to aggregate and report the results:

### Recommended Dashboard Structure

| Dashboard Section | Key Metrics | Update Frequency |
|------------------|-------------|-----------------|
| **Health Overview** | Overall quality score (7-day rolling), drift status (green/yellow/red), cost trend | Real-time |
| **Quality Deep-Dive** | Per-dimension scores, per-topic breakdown, bottom-5% failure analysis | Daily |
| **Deployment Status** | Last deployment date, golden suite pass rate, pending changes | Per-deployment |
| **Cost & Efficiency** | Cost per task, token consumption trend, latency percentiles | Daily |
| **Observability Health** | Trace completeness, alert response times, business KPI correlation | Weekly |

### Coverage Targets

- **Drift detection**: Monitor 100% of quality dimensions with automated statistical tests
- **Quality sampling**: Score ≥ 5% of production traffic; human-calibrate monthly
- **Regression testing**: Gate 100% of deployments with automated golden suite
- **Cost monitoring**: Track 100% of production conversations for token/cost data
- **Observability**: Achieve ≥ 99% trace completeness across all conversations

---

## Getting Started Checklist

If you're implementing continuous production evaluation for the first time, follow this prioritized sequence:

1. ☐ **Instrument logging** — ensure every conversation is logged with basic metadata (timestamp, query, response, model version)
2. ☐ **Set up a quality sampling pipeline** — start with random 10% sampling and a single LLM judge scoring helpfulness
3. ☐ **Establish a baseline** — run for 2 weeks to collect enough data for meaningful statistical comparisons
4. ☐ **Add drift detection** — implement threshold-based alerting on your quality scores
5. ☐ **Build a golden test suite** — curate 50+ test cases from production data and known edge cases
6. ☐ **Automate regression testing** — connect the golden suite to your deployment pipeline
7. ☐ **Add cost monitoring** — instrument token counting and set cost budgets
8. ☐ **Implement structured tracing** — add session/trace/span hierarchy for root-cause analysis
9. ☐ **Correlate with business KPIs** — join quality data with business metrics for executive reporting
10. ☐ **Conduct a quarterly review** — audit the entire monitoring stack for gaps and update thresholds

> **Start with steps 1–4.** These provide the highest value with the lowest effort. Steps 5–7 are the next tier. Steps 8–10 are for mature deployments. Don't try to implement everything at once.

---

[Back to library](../README.md) | [All capability scenarios](README.md)
