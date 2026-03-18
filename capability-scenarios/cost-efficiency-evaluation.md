# Cost-Efficiency Evaluation

> Scenarios for measuring and optimizing the cost-effectiveness of your AI agent — profiling token usage, building accuracy-vs-cost Pareto frontiers, automating configuration search, detecting cost anomalies in production, and catching cost regressions in CI/CD. Goes beyond "does it work?" to answer "is it worth what it costs?"

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Cost-Efficiency Evaluation Matters

An agent that scores 95% on accuracy benchmarks sounds great — until you discover it costs $4.50 per task while a simpler configuration scores 92% at $0.35. Without cost-efficiency evaluation, you are optimizing for quality alone and potentially overspending by 10x or more.

Cost-efficiency evaluation addresses three problems that quality-only evaluation misses:

- **Hidden cost variance.** The same agent can cost 50x more on one task than another depending on how many reasoning steps, tool calls, and retries it needs. Average cost is misleading — you need to understand the distribution and identify the expensive tail.
- **Accuracy-cost tradeoffs are non-obvious.** Research consistently shows that simple baselines often beat complex SOTA agents when cost is factored in. The most expensive model is rarely the most cost-effective. You need Pareto analysis to find the real optimal configurations.
- **Cost drifts silently in production.** A model update, a new prompt template, or a subtle change in user behavior can increase token consumption without anyone noticing until the invoice arrives. Continuous cost monitoring is as important as quality monitoring.

### The Measure-Optimize-Monitor Framework

The scenarios in this guide follow three phases of cost-efficiency evaluation:

1. **Measure** — Profile where tokens go, calculate cost-of-pass (cost per successful completion), and establish baselines per task type.
2. **Optimize** — Build Pareto frontiers to find optimal quality-cost tradeoffs, then use automated search to discover configurations you would never find manually.
3. **Monitor** — Detect cost anomalies in production and catch cost regressions in CI/CD before they reach users.

> **Relationship to other scenarios:** Cost-efficiency evaluation complements every other evaluation type. [Regression Testing](regression-testing.md) covers functional regressions — this guide adds cost regression. [Tool & Connector Invocations](tool-and-connector-invocations.md) covers whether tools work — this guide asks whether they work *efficiently*. Apply cost-efficiency evaluation as a cross-cutting concern across all your eval scenarios.

---

## Scenarios in This File

| # | Scenario | What It Tests |
|---|----------|--------------|
| 1 | [Cost-of-Pass Measurement & Token Profiling](#scenario-1-cost-of-pass-measurement--token-profiling) | What does a successful task completion actually cost, and where do tokens go? |
| 2 | [Pareto Frontier Analysis & Model Selection](#scenario-2-pareto-frontier-analysis--model-selection) | Which agent configurations offer the best accuracy-vs-cost tradeoffs? |
| 3 | [Automated Configuration Search for Cost Optimization](#scenario-3-automated-configuration-search-for-cost-optimization) | Can you automatically discover Pareto-optimal agent configurations? |
| 4 | [Cost Anomaly Detection & Budget Enforcement](#scenario-4-cost-anomaly-detection--budget-enforcement) | Are you catching unexpected cost spikes before they become expensive surprises? |
| 5 | [Cost-Quality Regression Testing in CI/CD](#scenario-5-cost-quality-regression-testing-in-cicd) | Do agent changes increase cost without improving quality? |

---

## Scenario 1: Cost-of-Pass Measurement & Token Profiling

### When to Use

You need to understand what your agent actually costs per successful task — not just average token counts, but the real cost-effectiveness across task types and difficulty levels. This applies when:

- You are comparing models at different price points and need an apples-to-apples cost-effectiveness metric
- Stakeholders ask "what does each successful interaction cost?" and you need a precise answer
- You suspect certain task types or difficulty levels are disproportionately expensive
- You want to identify which stages of agent execution (planning, tool calls, reasoning, output) consume the most tokens so you can optimize them
- You are setting pricing for an agent-powered product and need accurate cost-per-completion data

> **Related scenarios:** This is the foundation metric for all other cost-efficiency scenarios. [Scenario 2](#scenario-2-pareto-frontier-analysis--model-selection) uses cost-of-pass as one axis of the Pareto frontier. [Scenario 4](#scenario-4-cost-anomaly-detection--budget-enforcement) monitors cost-of-pass trends in production.

### The Core Metric: Cost-of-Pass

Cost-of-pass is the primary metric for agent cost-effectiveness. Unlike raw cost-per-call, it captures both expense and reliability:

```
cost-of-pass = total_cost / success_rate
```

A cheaper but less reliable agent may have a *worse* cost-of-pass than a more expensive reliable one. For example:

| Agent | Cost per Call | Success Rate | Cost-of-Pass |
|-------|-------------|-------------|-------------|
| Agent A (GPT-4.1) | $0.50 | 80% | $0.625 |
| Agent B (GPT-4.1-mini) | $0.30 | 40% | $0.750 |
| Agent C (GPT-4.1-nano) | $0.08 | 25% | $0.320 |

Agent B looks cheaper per call, but Agent A is more cost-effective per *success*. Agent C is cheapest per call but might be cost-effective only if the 75% failure rate is acceptable for your use case (e.g., speculative attempts with fallback).

**Important:** Track input and output tokens separately. They are priced differently (often 3-4x ratio), and the optimization strategies differ — input tokens are reduced by better prompting, output tokens by better stopping criteria.

### Token Profiling by Stage

Break down token consumption by execution stage to identify optimization targets:

| Stage | What It Includes | Typical Share | Optimization Lever |
|-------|-----------------|--------------|-------------------|
| System/planning | System prompt, task decomposition, chain-of-thought planning | 10-20% | Prompt compression, fewer planning steps |
| Tool invocations | Tool call formatting, tool result parsing, retrieval context | 30-50% | Better retrieval, fewer tool calls, result summarization |
| Reasoning | Intermediate reasoning, self-correction, backtracking | 20-35% | Model selection (some models reason more verbosely), fewer retries |
| Output generation | Final response to user | 5-15% | Output length constraints, structured output |

Research from the Efficient Agents benchmark (2025) found that moderate planning is optimal — agents that plan too extensively show "overthinking" that inflates cost without improving accuracy. Similarly, simple memory configurations (full conversation history with truncation) often beat complex summarization strategies that add processing tokens.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| **Per-task-type cost profiling** | Run your eval set with token counting instrumented, grouped by task type and difficulty. Report mean, median, p90, and p99 cost per task type. |
| **Cost-of-pass calculation** | For each model/config, calculate cost-of-pass = total_cost / success_rate. Compare across configurations. |
| **Token stage attribution** | Instrument your agent to tag tokens by stage (planning, tools, reasoning, output). Aggregate to find where tokens are consumed. |
| **Difficulty stratification** | Split tasks by difficulty (easy/medium/hard). Cost variance across difficulty levels reveals whether your agent handles complexity efficiently or degenerates into expensive retry loops. |

### Example Evaluation Setup

```yaml
scenario: cost-of-pass-measurement
task_set: your_standard_eval_set  # minimum 50 tasks across difficulty levels

configurations_to_compare:
  - name: "baseline-gpt4.1"
    model: gpt-4.1
    max_turns: 10
  - name: "mini-model"
    model: gpt-4.1-mini
    max_turns: 10
  - name: "nano-with-fallback"
    model: gpt-4.1-nano
    max_turns: 5
    fallback_model: gpt-4.1-mini  # escalate on failure

metrics_to_collect:
  per_task:
    - total_input_tokens
    - total_output_tokens
    - total_cost_usd
    - task_success (boolean)
    - token_breakdown_by_stage
    - number_of_tool_calls
    - number_of_retries

  aggregate:
    - cost_of_pass_by_config
    - mean_cost_by_task_type
    - p90_cost_by_task_type
    - token_stage_distribution
    - cost_variance_ratio  # p90/p50 — high ratio means unpredictable costs

pass_criteria:
  - cost_of_pass < $1.00 per task  # adjust to your budget
  - cost_variance_ratio < 5.0  # costs should be reasonably predictable
  - tool_call_tokens < 50% of total  # tool overhead should not dominate
```

### What to Watch For

- **Bimodal cost distributions.** If your cost histogram has two peaks, your agent likely has two execution paths — one efficient, one expensive. Investigate what triggers the expensive path.
- **Retry cascades.** An agent that retries failed tool calls can generate 5-10x normal token consumption on hard tasks. Set retry limits and monitor retry rates.
- **Context window stuffing.** If tool results or retrieval context fills the context window, every subsequent generation becomes expensive. Implement result summarization for long tool outputs.
- **Input token dominance.** For most agents, input tokens (from system prompts, context, and tool results) far exceed output tokens. Optimizing input is usually higher-leverage than optimizing output.

---

## Scenario 2: Pareto Frontier Analysis & Model Selection

### When to Use

You are choosing between multiple agent configurations (different models, prompt strategies, tool setups, or architectural approaches) and need to find the ones that offer the best quality-vs-cost tradeoffs. This applies when:

- You have multiple candidate models and need to select one for production based on both quality and cost
- Stakeholders want to understand what quality you would sacrifice for a given cost reduction
- You are evaluating whether a complex multi-agent architecture is justified versus a simpler (and cheaper) single-agent design
- A new model release prompts re-evaluation of your current configuration choices
- You need to present tradeoff analysis to decision-makers who control the budget

> **Related scenarios:** [Scenario 1](#scenario-1-cost-of-pass-measurement--token-profiling) provides the cost and quality metrics that feed into this analysis. [Scenario 3](#scenario-3-automated-configuration-search-for-cost-optimization) automates the search for configurations to place on the frontier.

### Understanding Pareto Frontiers

A Pareto frontier (or Pareto front) is the set of configurations where you cannot improve quality without increasing cost, or reduce cost without sacrificing quality. Configurations ON the frontier are "Pareto-optimal" — none is strictly dominated by another.

```
Quality (%)
  |        * Config E (Pareto-optimal)
  |      *
  |    * Config C (Pareto-optimal)
  |        x Config D (dominated — C is cheaper AND better)
  |  * Config B (Pareto-optimal)
  | x Config F (dominated)
  |* Config A (Pareto-optimal — cheapest)
  +--------------------------------→ Cost ($)
```

**Key concepts:**
- **Dominated configurations** fall below the frontier — there exists another config that is both cheaper and better. These should be eliminated from consideration.
- **The knee point** is where the frontier curve bends most sharply. At the knee, you get the best marginal quality per dollar. Research from "AI Agents That Matter" (2024) found that knee points often sacrifice only 2-5% accuracy while costing 5-10x less than the highest-quality configuration.
- **Simple baselines belong on every frontier.** A recurring finding in agent evaluation research is that simple baselines (single model, basic prompting, no complex orchestration) frequently appear on or near the Pareto frontier, outperforming complex multi-agent systems when cost is considered.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| **Multi-configuration eval run** | Run the same eval set across all candidate configurations. Collect quality score and cost-of-pass for each. |
| **Frontier construction** | Plot all configurations on quality-vs-cost axes. Identify the Pareto frontier by removing dominated points. |
| **Knee point identification** | Calculate the "marginal quality per dollar" at each frontier point. The knee is where this ratio drops sharply. |
| **Baseline anchoring** | Always include at least one simple baseline (e.g., single model call, no tools) on the frontier. This prevents the trap of only comparing complex systems to each other. |
| **Statistical significance testing** | For configurations near each other on the frontier, run enough samples (n ≥ 100) to confirm the quality difference is statistically significant, not just noise. |

### Example Evaluation Setup

```yaml
scenario: pareto-frontier-analysis
task_set: your_standard_eval_set  # 100+ tasks for statistical power

configurations:
  # Simple baselines (MUST include these)
  - name: "single-call-gpt4.1"
    type: baseline
    model: gpt-4.1
    tools: none

  - name: "single-call-mini"
    type: baseline
    model: gpt-4.1-mini
    tools: none

  # Standard agent configs
  - name: "agent-gpt4.1-full-tools"
    model: gpt-4.1
    tools: [search, calculator, code_exec]
    max_turns: 15

  - name: "agent-mini-full-tools"
    model: gpt-4.1-mini
    tools: [search, calculator, code_exec]
    max_turns: 15

  # Hybrid configs
  - name: "router-mini-to-full"
    router_model: gpt-4.1-mini
    execution_model: gpt-4.1
    strategy: "route simple tasks to mini, complex to full"

  # Add your actual candidate configurations here

analysis:
  frontier_axes:
    x: cost_of_pass_usd
    y: task_success_rate_percent

  report:
    - pareto_frontier_plot
    - dominated_configurations_list
    - knee_point_identification
    - marginal_quality_per_dollar_at_each_point
    - recommendation_at_budget_levels: [$0.25, $0.50, $1.00, $2.00]

pass_criteria:
  - selected_config_is_on_pareto_frontier
  - selected_config_quality >= minimum_acceptable_quality
  - at_least_one_simple_baseline_evaluated
```

### What to Watch For

- **The "complex is better" trap.** Multi-agent architectures, RAG pipelines with reranking, and chain-of-thought prompting all add cost. They often add quality too — but not always enough to justify the cost. The Pareto frontier reveals when complexity is not paying for itself.
- **Frontier instability.** If your frontier changes significantly when you add or remove 10% of eval tasks, your eval set is too small or not representative. The frontier should be stable across reasonable task samples.
- **Budget-dependent optima.** The best configuration at a $0.50 budget is different from the best at $2.00. Present frontier analysis at multiple budget levels to support different deployment contexts.
- **Diminishing returns near the top.** The last 2-3% of quality improvement often costs as much as the first 90%. Make this tradeoff visible to stakeholders.

---

## Scenario 3: Automated Configuration Search for Cost Optimization

### When to Use

Your agent system has many configurable parameters (model choice, prompt templates, retrieval strategy, memory configuration, tool selection, max turns) and you want to automatically discover which combinations are Pareto-optimal. This applies when:

- Manual testing of configurations is impractical because the search space is too large (e.g., 5 models × 3 prompt templates × 4 retrieval strategies = 60 combinations)
- You are building a RAG pipeline with many tunable components and want to find cost-optimal settings
- A new model release means the entire configuration landscape needs re-evaluation
- You want to systematically verify that your current production configuration is still Pareto-optimal

> **Related scenarios:** [Scenario 2](#scenario-2-pareto-frontier-analysis--model-selection) explains how to analyze the Pareto frontier once you have it. This scenario automates the process of *finding* configurations to populate that frontier. [Scenario 1](#scenario-1-cost-of-pass-measurement--token-profiling) defines the metrics each configuration is evaluated on.

### Why Automated Search Matters

The key insight from research on workflow optimization (syftr, 2025) is that the configuration space for modern agent systems is vast, and human intuition about which configurations will be cost-effective is often wrong. Automated multi-objective search routinely discovers configurations that are 5-9x cheaper than defaults while maintaining quality.

The two main approaches:

**Multi-objective Bayesian optimization** uses a surrogate model to predict which configurations are likely to be Pareto-optimal, focusing evaluation budget on the most promising candidates rather than exhaustive grid search. It works well when evaluations are expensive (each config takes minutes to test).

**Early stopping** prunes clearly suboptimal configurations after a small sample of tasks, saving evaluation budget for more promising candidates. If a configuration fails 8 out of 10 initial tasks, there is no need to run the remaining 90.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| **Configuration space definition** | Define all tunable parameters and their ranges. Include model choice, prompt variations, tool configurations, retrieval settings, and agent architecture parameters. |
| **Multi-objective Bayesian search** | Use a framework like Optuna, Ax, or a custom Bayesian optimizer targeting both quality and cost. Each trial evaluates one configuration on a subset of tasks. |
| **Early stopping with bandit methods** | After a minimum sample (10-20 tasks), stop configurations whose quality is statistically below the current Pareto frontier. Redistribute budget to frontier candidates. |
| **Validation on held-out tasks** | Once the search identifies Pareto-optimal configs, validate them on a held-out task set to ensure the results generalize. |

### Example Evaluation Setup

```yaml
scenario: automated-config-search
task_set: your_standard_eval_set
search_budget: 500 configuration evaluations  # adjust to your compute budget

configuration_space:
  model:
    type: categorical
    values: [gpt-4.1, gpt-4.1-mini, gpt-4.1-nano]

  prompt_strategy:
    type: categorical
    values: [zero-shot, few-shot-3, chain-of-thought, react]

  max_turns:
    type: integer
    range: [1, 20]

  retrieval_top_k:
    type: integer
    range: [3, 20]
    condition: tools_include_retrieval

  memory_strategy:
    type: categorical
    values: [full-history, sliding-window-10, summarize-every-5]

  tool_set:
    type: categorical
    values: [none, search-only, full-tools]

optimization:
  algorithm: multi_objective_bayesian  # or grid_search for small spaces
  objectives:
    - maximize: task_success_rate
    - minimize: cost_of_pass_usd

  early_stopping:
    enabled: true
    min_tasks_before_stop: 15
    stop_if_below_frontier_by: 10%  # prune if 10%+ worse than current best

  n_initial_random: 20  # random exploration before Bayesian kicks in

output:
  pareto_frontier_configs: top_10_pareto_optimal
  comparison_to_current_production: true
  estimated_savings_at_equal_quality: true

pass_criteria:
  - found_config_cheaper_than_current_at_equal_quality
  - search_converged  # frontier stable in last 20% of budget
  - validation_quality >= 95% of search_quality  # no overfitting to search set
```

### What to Watch For

- **Overfitting to the eval set.** A configuration optimized on 50 tasks may not generalize. Always validate on held-out data and check that search-time quality matches validation quality within 5%.
- **Interaction effects.** Some parameter combinations interact non-linearly — e.g., chain-of-thought prompting might help a small model but add unnecessary cost to a large one. Bayesian optimization captures these interactions better than independent parameter tuning.
- **Search budget allocation.** Spend roughly 70% of budget on exploration (finding new frontier candidates) and 30% on exploitation (confirming promising candidates with more tasks). Pure exploitation converges prematurely.
- **Memory strategy impact.** Research shows that memory configuration has outsized impact on cost. Complex summarization strategies often add more token overhead than they save. Profile memory-related tokens separately.

---

## Scenario 4: Cost Anomaly Detection & Budget Enforcement

### When to Use

Your agent is running in production and you need to detect unexpected cost increases before they become expensive surprises. This applies when:

- You are operating an agent at scale and need to control costs across thousands of daily interactions
- A model update or prompt change might have unintended cost implications that you need to catch quickly
- You need per-user, per-tenant, or per-task-type budget limits to prevent runaway costs
- Seasonal patterns or user behavior changes might shift cost distributions
- You have contractual or organizational budget constraints that require automated enforcement

> **Related scenarios:** [Continuous production evaluation](regression-testing.md) covers quality monitoring in production. This scenario adds cost monitoring as a parallel concern. [Scenario 5](#scenario-5-cost-quality-regression-testing-in-cicd) catches cost issues before deployment; this scenario catches them in production.

### Cost Anomaly Detection Approaches

Production cost monitoring borrows from statistical process control — the same techniques used to detect manufacturing defects apply to detecting cost anomalies:

| Approach | How It Works | Best For |
|----------|-------------|----------|
| **Z-score detection** | Flag individual interactions where cost > μ + 3σ (or a configurable threshold). Simple, fast, works well for normally distributed costs. | Per-interaction alerting, catching individual runaway tasks |
| **CUSUM (Cumulative Sum)** | Track the cumulative deviation of costs from the expected mean. Detects gradual drift that z-scores miss because individual points may be within bounds even as the mean shifts. | Detecting slow cost increases over days/weeks |
| **Sliding window comparison** | Compare the cost distribution of the last N hours to a baseline period. Use Kolmogorov-Smirnov or similar tests to detect distribution shifts. | Detecting changes in cost distribution shape, not just mean |
| **Per-cohort monitoring** | Track cost separately by task type, user segment, or tenant. A global average can mask localized spikes. | Multi-tenant systems, diverse task portfolios |

### Budget Enforcement Strategies

When costs exceed budgets, you need a graduated response — not just "shut everything down":

| Level | Trigger | Response |
|-------|---------|----------|
| **Warning** | Cost trending toward 80% of budget | Alert ops team, increase monitoring frequency |
| **Soft limit** | Cost reaches 90% of budget | Downgrade to cheaper model for non-critical tasks, reduce max turns, compress prompts |
| **Hard limit** | Cost reaches 100% of budget | Queue non-urgent requests, route only critical tasks, apply strict token limits |
| **Emergency** | Cost exceeds 120% of budget | Halt non-essential agent operations, alert leadership |

### Example Evaluation Setup

```yaml
scenario: cost-anomaly-detection
monitoring_period: continuous  # this is an ongoing production concern

baseline_establishment:
  method: rolling_30_day_window
  metrics:
    - mean_cost_per_task_type
    - stddev_cost_per_task_type
    - p50_p90_p99_cost_per_task_type
    - daily_total_cost
  update_frequency: daily  # recalculate baseline daily

anomaly_detection:
  per_interaction:
    method: z_score
    threshold: 3.0  # flag if cost > mean + 3 stddev
    action: log_and_tag  # don't block, but tag for review

  trend_detection:
    method: cusum
    target_mean: baseline_mean_cost
    slack: 0.5  # standard deviations of acceptable drift
    decision_interval: 4.0  # CUSUM threshold for alert
    check_frequency: hourly

  distribution_shift:
    method: kolmogorov_smirnov
    comparison_window: 24_hours
    baseline_window: 7_days
    p_value_threshold: 0.01
    check_frequency: every_6_hours

budget_enforcement:
  per_interaction:
    max_tokens: 50000  # hard limit per interaction
    max_tool_calls: 20
    max_retries: 3

  per_user_daily:
    soft_limit_usd: 5.00
    hard_limit_usd: 10.00

  per_tenant_monthly:
    configured_per_tenant: true  # different tenants may have different budgets

  global_daily:
    warning_threshold: 80%
    soft_limit: 90%
    hard_limit: 100%

alerting:
  channels: [pagerduty, slack, email]
  severity_mapping:
    per_interaction_anomaly: low  # individual outliers
    trend_shift: medium  # gradual drift
    distribution_shift: high  # fundamental change
    budget_breach: critical

pass_criteria:
  - anomaly_detection_latency < 1_hour  # detect shifts within 1 hour
  - false_positive_rate < 5%  # alerts should be actionable
  - budget_enforcement_prevents_overspend  # hard limits actually work
  - graduated_response_preserves_critical_tasks  # essential tasks still work at soft limit
```

### What to Watch For

- **Alert fatigue.** If your anomaly detector fires on 20% of interactions, the team will ignore it. Tune thresholds so alerts are rare and actionable. Start with high thresholds and lower them gradually.
- **Baseline staleness.** A baseline from two months ago may not reflect current usage patterns. Use rolling windows that adapt to legitimate changes (new features, seasonal patterns) while still catching anomalies.
- **Per-tenant variance.** In multi-tenant systems, one tenant's normal is another's anomaly. Monitor per-tenant baselines, not just global averages.
- **Cost-quality coupling.** When budget enforcement triggers model downgrades, monitor quality metrics simultaneously. A cost optimization that causes a quality regression is not a real optimization — it is a tradeoff that should be made deliberately, not automatically.

---

## Scenario 5: Cost-Quality Regression Testing in CI/CD

### When to Use

You are making changes to your agent (prompt updates, model swaps, code changes, tool additions) and need to verify that these changes do not increase costs without proportional quality improvement. This applies when:

- You have a CI/CD pipeline for your agent and want to add cost checks alongside quality checks
- Prompt engineers are iterating on prompts and you want to catch cost-inflating changes early
- A model provider releases an update and you need to verify cost characteristics have not changed
- You are adding new tools or capabilities and want to quantify their cost impact
- You need to enforce a policy that cost increases require explicit approval

> **Related scenarios:** [Regression Testing](regression-testing.md) covers functional regression detection. This scenario extends regression testing to include cost as a first-class metric. [Scenario 1](#scenario-1-cost-of-pass-measurement--token-profiling) defines the cost metrics to track; this scenario monitors them for regressions.

### Defining Cost Regressions

A cost regression occurs when a change increases cost without a corresponding quality improvement. The key is defining "corresponding" — some cost increases are justified:

| Change Type | Cost Increase | Quality Change | Regression? |
|-------------|--------------|---------------|-------------|
| Prompt optimization | -20% | No change | No (improvement!) |
| New tool added | +15% | +10% quality | No (justified) |
| Prompt rewrite | +40% | +2% quality | **Yes** (disproportionate) |
| Model upgrade | +100% | +5% quality | **Likely yes** (unless 5% is critical) |
| Bug fix | +5% | No change | **Investigate** (why did cost increase?) |

The decision framework: **a change is a cost regression if it moves the agent away from the Pareto frontier** — i.e., there existed a configuration before the change that had equal or better quality at lower cost.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| **Baseline cost profile** | Maintain a cost baseline (mean, p50, p90, p99) per task type from the current production configuration. Update the baseline when changes are deliberately accepted. |
| **Cost delta analysis** | For each PR/change, run the eval set on both old and new configurations. Calculate cost deltas per task type. Flag if any delta exceeds threshold. |
| **Cost-of-pass comparison** | Compare cost-of-pass before and after. If cost-of-pass increased, the change made the agent less cost-effective regardless of whether raw quality improved. |
| **Pareto position check** | Plot the new configuration on the existing Pareto frontier. If it falls below the frontier (dominated), it is a regression. |

### Example Evaluation Setup

```yaml
scenario: cost-quality-regression-testing
trigger: on_pull_request  # run on every PR that changes agent behavior
eval_set: cost_regression_subset  # can be smaller than full eval (30-50 tasks)

baseline:
  source: last_accepted_production_config
  metrics:
    - cost_of_pass_by_task_type
    - mean_cost_by_task_type
    - p90_cost_by_task_type
    - total_quality_score

comparison:
  run_both: true  # run baseline and candidate on same tasks
  paired_analysis: true  # compare per-task, not just aggregates

thresholds:
  cost_of_pass_increase_max: 10%  # flag if cost-of-pass rises > 10%
  mean_cost_increase_max: 15%  # flag if mean cost rises > 15%
  p90_cost_increase_max: 25%  # allow more variance in tail
  quality_decrease_max: 2%  # flag if quality drops > 2%

  # Combined threshold: cost-quality ratio
  max_cost_per_quality_point: $0.50  # each 1% quality improvement
                                      # should cost no more than $0.50 increase

ci_cd_integration:
  block_merge_on: cost_regression_detected
  require_approval_for: justified_cost_increase  # when cost rises but quality rises more
  auto_approve: cost_decrease_no_quality_loss  # improvements pass automatically

reporting:
  pr_comment:
    - cost_delta_summary_table
    - cost_of_pass_before_after
    - pareto_position_plot
    - per_task_type_breakdown
    - recommendation: approve | flag_for_review | block

  dashboard:
    - cost_trend_over_last_20_prs
    - pareto_frontier_evolution
    - cost_regression_rate

pass_criteria:
  - cost_of_pass_delta <= threshold
  - no_task_type_with_cost_increase > 50%  # even if aggregate is fine
  - quality_not_degraded
  - pipeline_completes_in < 15_minutes  # cost checks should not slow CI/CD significantly
```

### What to Watch For

- **Eval set size for cost testing.** Cost measurements have high variance. With 30 tasks, you may not detect a 10% cost increase with statistical significance. For critical cost thresholds, use larger eval sets (100+ tasks) or accept higher false-negative rates.
- **Prompt-induced cost changes.** A seemingly minor prompt change ("think step by step" → "think carefully step by step") can change token consumption significantly because it changes the model's generation pattern. Always measure cost impact of prompt changes.
- **Cascading tool costs.** Adding a new tool might not just add the cost of calling that tool — it might cause the agent to use existing tools differently (more or fewer calls), changing the overall cost profile in non-obvious ways.
- **CI/CD performance budget.** Running cost regression tests adds time to your pipeline. Use a smaller, representative task subset for CI/CD (30-50 tasks) and run the full eval set less frequently (nightly or weekly). Balance detection sensitivity against developer velocity.

---

## Further Reading

- **"AI Agents That Matter"** (2024) — Establishes the case for mandatory cost reporting in agent benchmarks and introduces Pareto frontier analysis for agent evaluation. Key finding: simple baselines frequently outperform complex agents on the Pareto frontier.
- **"Efficient Agents"** (arXiv 2508.02694, 2025) — Introduces the cost-of-pass metric and token profiling methodology. Finds that moderate planning and simple memory configurations are more cost-effective than complex alternatives.
- **"syftr: Multi-Objective Workflow Optimization"** (arXiv 2505.20266, 2025) — Demonstrates automated multi-objective Bayesian search over agent configuration spaces, finding configurations 5-9x cheaper at equal quality.
- **"Tokenomics"** (arXiv 2601.14470, 2026) — Detailed analysis of token distribution patterns across agent execution stages, with practical optimization strategies.
