# Cost-Efficiency Evaluation

> Scenarios for evaluating whether AI agent systems deliver results at optimal cost — routing queries to the cheapest capable model, leveraging caching effectively, choosing the right evaluation judges, compressing prompts without quality loss, and mapping the cost-quality Pareto frontier across the full system. Cost evaluation is not about spending less — it is about spending _right_: maximizing quality per dollar and identifying the 20–30% of use cases that consume 60–80% of your budget.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

<details>
<summary><strong>Why Cost-Efficiency Evaluation Matters</strong></summary>

Enterprise LLM inference spending has grown rapidly, and a significant share of that spend is addressable through optimization. Yet most teams treat cost as a deployment concern, not an evaluation concern. Cost inefficiency compounds silently — by the time you notice the bill, you have already wasted months of budget on suboptimal configurations.

The core problem is that cost and quality are not independent dimensions. A cheaper model is not always worse. A more expensive evaluation judge is not always better. The relationship between cost and quality is nonlinear, task-dependent, and changes with every model update. Evaluation is the only systematic way to navigate this landscape.

Three forces make cost-efficiency evaluation urgent:

- **Model routing is now mainstream.** Systems like RouteLLM demonstrate 2× cost reduction while preserving 95% of frontier model quality. But routing only works if you can _evaluate_ whether the cheaper model actually handles each query class correctly. Without evaluation, you are guessing which queries are "easy enough" for the small model — and guessing wrong either wastes money (routing too conservatively) or degrades quality (routing too aggressively).
- **Semantic caching is high-leverage but fragile.** Research shows a significant fraction of production queries are near-duplicates, and semantic caching achieves 40–67% hit rates compared to 8–12% for exact-match caching. But cache quality degrades over time as ground truth shifts, and a stale cached response is worse than an expensive fresh one. Evaluation must verify that cached responses remain correct.
- **Evaluation itself is a cost center.** The cost difference between LLM-as-Judge and Agent-as-Judge approaches can be an order of magnitude or more (with agent-based judges also being significantly slower), making judge selection itself a cost-quality tradeoff that must be evaluated. Running expensive judges on every production query is wasteful; running cheap judges on safety-critical decisions is reckless.

The **Route-Cache-Track** framework structures cost-efficiency evaluation in three layers: _Route_ queries to the cheapest capable model, _Cache_ responses to eliminate redundant computation, and _Track_ cost-per-task alongside quality to map the Pareto frontier and identify optimization opportunities.

> **Key metric — Cost-of-Pass:** The primary metric for agent cost-effectiveness is `cost_of_pass = total_cost / success_rate`. A cheaper but less reliable agent may have a worse cost-of-pass than a more expensive reliable one. Always evaluate cost-effectiveness, not just cost.

> **Key references:** RouteLLM (COLM 2025), "AI Agents That Matter" (arXiv 2407.01502), "Efficient Agents" (arXiv 2508.02694), Anthropic's Swiss Cheese evaluation model, Tokenomics (arXiv 2601.14470), LLM-as-Judge vs Agent-as-Judge (arXiv 2512.12791), syftr multi-objective optimization (arXiv 2505.20266).

</details>

---

## Scenario 1: Model Routing Effectiveness

Evaluate whether a model routing strategy correctly classifies query complexity and routes to the cheapest capable model while maintaining quality thresholds.

### When to Use

Your system routes queries to different models based on complexity, cost, or latency requirements — and you need to verify that the routing logic is actually saving money without degrading quality. Model routing is one of the highest-leverage cost optimizations (real-world cases show 2–5× cost reduction), but incorrect routing silently degrades user experience: a query routed to a model that cannot handle it produces a plausible but wrong answer, and no one notices until a user complains.

Use this scenario when:
- You have deployed a multi-model routing system (e.g., routing between GPT-4o-mini and GPT-4o, or between a fine-tuned small model and a frontier model)
- You are considering whether to introduce model routing to reduce costs
- You have changed routing thresholds or the complexity classifier and need to verify no quality regressions
- You want to quantify the actual cost savings from routing vs. the theoretical savings
- Users have reported quality inconsistencies that may stem from incorrect routing decisions

> **Related scenarios:** For evaluating trigger routing logic (which _topic_ handles a query), see [Trigger Routing](trigger-routing.md). This scenario evaluates which _model_ handles a query — a different routing decision with different failure modes.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify that responses from the routed (cheaper) model are semantically equivalent to the frontier model baseline |
| General Quality | Assess whether response quality from the cheaper model meets your quality floor for each query class |
| Exact Match | Verify that factual queries routed to cheaper models still produce correct answers |
| Capability Use (All) | Confirm that tool calls and structured outputs are preserved when queries are routed to smaller models |
| Latency Measurement | Measure end-to-end response time to verify routing overhead does not exceed latency thresholds |

> **Tip:** The critical evaluation is not whether the cheaper model produces _good_ responses — it is whether the _routing classifier_ correctly identifies which queries the cheaper model can handle. Build your test set around the routing boundary: queries that are just barely complex enough to need the frontier model, and queries that are just barely simple enough for the cheaper model. This boundary is where routing errors concentrate.

### Setup Steps

1. **Establish a frontier-model baseline.** Run your full evaluation suite against the frontier model only (no routing). Record quality scores, costs, and latency for every test case. This is your quality ceiling and cost ceiling.
2. **Classify your test cases by complexity.** Label each test case as "simple" (expected to be handled by the cheaper model) or "complex" (expected to require the frontier model). If you have a complexity classifier, use its labels. If not, manually classify based on domain knowledge.
3. **Enable routing and re-run the full suite.** With routing enabled, run the same evaluation suite. Record which model handled each query, the quality score, the cost, and the latency.
4. **Calculate routing accuracy.** For each test case: did the router send it to the correct model? A "simple" query routed to the frontier model is a _cost waste_ (correct but overspent). A "complex" query routed to the cheaper model is a _quality risk_ (may produce a worse answer). Measure both error types separately.
5. **Compute the quality preservation rate.** For queries routed to the cheaper model, what percentage achieved quality scores within your acceptable threshold of the frontier model baseline? Target: ≥95% quality preservation. If below 95%, your routing threshold is too aggressive.
6. **Calculate actual cost savings.** Compare total cost with routing vs. total cost without routing. Express as a ratio. Compare this to the theoretical maximum savings (if all "simple" queries were routed perfectly). The gap between actual and theoretical savings reveals routing classifier inefficiency.

### Anti-Pattern

> **"Route and Forget"** — Deploying model routing with a static complexity threshold and never re-evaluating. Model capabilities change with every update — a query that was too complex for GPT-4o-mini six months ago may be easy for the current version. Without periodic re-evaluation of routing boundaries, you either waste money (routing too conservatively with updated models) or degrade quality (routing too aggressively with models that have regressed). Re-evaluate routing thresholds after every model version change.

### Evaluation Patterns

#### Pattern: Quality Preservation Gating
For each query class routed to a cheaper model, require that the quality score is within a defined threshold (e.g., ≥95%) of the frontier model score on the same queries. If any query class falls below the threshold, escalate that class back to the frontier model. This creates a per-class routing decision rather than a global threshold.

#### Pattern: Routing Boundary Stress Testing
Generate synthetic queries at the boundary of your routing classifier — queries that are slightly more complex than "simple" and slightly less complex than "complex." These boundary cases are where routing errors concentrate. Measure the classifier's accuracy specifically on boundary cases. If boundary accuracy is below 85%, the classifier needs retraining.

#### Pattern: Cost-of-Pass Comparison
Calculate `cost_of_pass = total_cost / success_rate` for each model separately and for the routed system. A routing system is cost-effective only if its cost-of-pass is lower than using the frontier model for everything. If the cheaper model's lower success rate drives up cost-of-pass above the frontier model, routing is counterproductive.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Simple factual query routed to cheaper model | "What are your business hours?" | Correct answer with quality score ≥95% of frontier baseline; cost ≤30% of frontier model cost | Exact Match + Compare Meaning |
| 2 | Complex reasoning query escalated to frontier model | "Compare the three enterprise plans, recommend one based on our 500-employee company with remote workers in 12 countries, and explain the tax implications" | Routed to frontier model; comprehensive multi-factor response | General Quality + Capability Use (All) |
| 3 | Boundary query tests routing accuracy | "Summarize our return policy for international orders" (moderate complexity) | Router makes consistent classification; if routed to cheaper model, quality score ≥95% of frontier baseline | Compare Meaning + General Quality |
| 4 | Tool-calling query preserves function calls after routing | "Look up order #12345 and tell me the shipping status" (requires API call) | Cheaper model correctly invokes order lookup tool with correct parameters; response matches frontier model | Capability Use (All) + Exact Match |
| 5 | Routing does not increase latency beyond threshold | Any query mix representative of production traffic | p50 latency delta ≤10% vs. frontier-only; p99 latency delta ≤25% (routing overhead is small) | Latency measurement |

### Tips

- Track routing decisions as metadata on every evaluation. This lets you retroactively analyze which routing decisions caused quality drops.
- The biggest cost savings come from identifying high-volume, low-complexity query classes. Profile your production traffic to find these — they are often 50–70% of total queries.
- Fine-tuned small models often outperform general-purpose small models on domain-specific routing. If you have enough data, fine-tuning a routing classifier is typically higher ROI than fine-tuning the generation model.
- Real-world reference: Checkr achieved 5× cost reduction by fine-tuning Llama-3-8B for their specific use case. A healthcare company achieved 94% cost reduction with Mistral-7B on domain-specific queries. These are not anomalies — they represent the typical range of savings when routing is done well.

---

## Scenario 2: Semantic Caching Quality Assurance

Verify that semantic caching correctly identifies duplicate queries, that served cached responses maintain quality, and that cache staleness does not introduce errors.

### When to Use

Your system uses semantic caching to reduce API calls and latency — and you need to verify that cached responses are still correct and that the similarity threshold correctly distinguishes genuinely similar queries from superficially similar but different ones. Semantic caching is one of the highest-ROI optimizations (40–67% hit rates, 40–70% API call reduction), but a misconfigured cache actively harms users by serving stale or mismatched responses with high confidence.

Use this scenario when:
- You have deployed semantic caching and need to verify it is working correctly
- You are tuning the similarity threshold and need to find the optimal balance between hit rate and accuracy
- Your knowledge base has been updated and you need to verify that stale cached responses are invalidated
- Users have reported receiving outdated or incorrect information that may be coming from cache
- You are measuring the actual cost savings from caching vs. the theoretical savings

> **Related scenarios:** For evaluating knowledge grounding accuracy, see [Knowledge Grounding and Accuracy](knowledge-grounding-and-accuracy.md). For regression testing after knowledge updates, see [Regression Testing](regression-testing.md). This scenario specifically evaluates the _caching layer_ between the user query and the generation model.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify that cached responses are semantically equivalent to fresh responses for the same query |
| Exact Match | Confirm that factual data in cached responses matches current ground truth (catches staleness) |
| Keyword Match (All) | Verify that critical facts, dates, and policy details in cached responses are current |
| General Quality | Assess whether cached response quality matches fresh response quality |
| Statistical Analysis | Aggregate cache hit rates, latency, and quality metrics across a representative query sample to assess overall cache performance |

> **Tip:** The most dangerous caching failure is not a cache _miss_ (that just costs more) — it is a false _hit_: serving a cached response for a query that is superficially similar but semantically different. Build adversarial test pairs: queries with high lexical overlap but different intent (e.g., "cancel my order" vs. "can I cancel my order and reorder?"). These are where false hits concentrate.

### Setup Steps

1. **Create a cache evaluation dataset.** Build three categories of query pairs: (a) true duplicates — same question in different words, should hit cache; (b) near-misses — similar questions with different answers, should NOT hit cache; (c) stale queries — questions whose correct answer has changed since the cache was populated.
2. **Populate the cache.** Run the first query in each pair to populate cache entries. Record the cached responses and timestamps.
3. **Test cache hit accuracy.** Run the second query in each pair. For true duplicates, verify a cache hit with correct response. For near-misses, verify a cache miss (or if a hit, verify the response is still correct for the _new_ query). For stale queries, verify the cache is invalidated or the stale response is flagged.
4. **Measure quality preservation.** For all cache hits, compare the cached response quality to what a fresh API call would produce. Calculate the quality preservation rate: what percentage of cache hits have quality within your acceptable threshold?
5. **Calculate cost savings.** Measure actual cache hit rate across a representative query set. Multiply by per-query API cost to get actual savings. Compare to theoretical savings (if all true duplicates hit cache and no near-misses hit cache).
6. **Test cache invalidation.** Update a knowledge source that affects cached responses. Verify that affected cache entries are invalidated within your SLA. Re-query to verify fresh responses are generated and cached.

### Anti-Pattern

> **"Set and Forget Similarity Threshold"** — Choosing a semantic similarity threshold (e.g., 0.95) at deployment and never re-evaluating. As your query distribution shifts and your knowledge base evolves, the optimal threshold changes. A threshold that worked well at launch may produce too many false hits six months later (if query diversity has increased) or too few hits (if queries have become more formulaic). Re-evaluate the threshold quarterly or after significant knowledge base changes.

### Evaluation Patterns

#### Pattern: False Hit Detection
Create adversarial query pairs with high lexical similarity but different intent. Measure the false hit rate: what percentage of these pairs incorrectly serve a cached response? A false hit rate above 2% indicates the similarity threshold is too permissive. Common adversarial patterns: negation ("can I" vs. "can't I"), scope change ("for me" vs. "for my team"), temporal shift ("this year" vs. "last year").

#### Pattern: Staleness Detection
After a knowledge base update, query for information that changed. If the cache serves the old (now incorrect) answer, cache invalidation is broken. Measure time-to-invalidation: how long after a knowledge update does the cache continue serving stale responses? Target: cache invalidation within the same deployment cycle as the knowledge update.

#### Pattern: Hit Rate vs. Quality Curve
Sweep the similarity threshold from 0.80 to 0.99 in 0.01 increments. At each threshold, measure cache hit rate and quality preservation rate. Plot the curve. The optimal threshold is the point where quality preservation rate is ≥98% and hit rate is maximized. This curve is specific to your query distribution — do not use someone else's threshold.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | True duplicate hits cache with correct response | First: "What is the return policy?" Then: "How do I return an item?" | Cache hit; response semantically equivalent to fresh response; cost = ~$0 (cache serving cost only) | Compare Meaning + Exact Match |
| 2 | Near-miss does not produce false cache hit | First: "Cancel my subscription" Then: "Can I pause my subscription instead of canceling?" | Cache miss (different intent); fresh response generated addressing pause, not cancellation | Compare Meaning + General Quality |
| 3 | Stale cache is invalidated after knowledge update | Cache populated before policy change. Query after policy change: "What is the refund window?" | If refund window changed from 30 to 60 days, response reflects new 60-day window, not cached 30-day answer | Exact Match + Keyword Match (All) |
| 4 | Similarity threshold catches subtle intent differences | First: "What are the fees for domestic transfers?" Then: "What are the fees for international transfers?" | Cache miss (different fee schedules); fresh response with correct international fee schedule | Exact Match + Compare Meaning |
| 5 | Cache performance under load | 1000 queries from production traffic sample | Cache hit rate 40–67%; quality preservation ≥98% on cache hits; average latency reduction ≥50% vs. uncached | Statistical analysis |

### Tips

- The near-duplicate rate in production traffic varies across industries. Profile your own traffic — customer support tends to be higher (40–60% duplicates), technical documentation queries tend to be lower (15–25%).
- Exact-match caching (8–12% hit rate) is worth implementing first as a baseline. It has zero false-hit risk and provides a comparison point for evaluating whether semantic caching's higher hit rate is worth the false-hit risk.
- Cache invalidation is harder than cache population. Design your cache with knowledge-source-aware TTLs: entries derived from frequently updated sources get shorter TTLs than entries derived from stable policy documents.
- Monitor cache hit rate trends over time. A steadily declining hit rate suggests your query distribution is diversifying (good for users, bad for cache ROI). A sudden spike in hit rate may indicate a bot or a service degradation funneling all users to the same error-handling query.

---

## Scenario 3: Judge Cost-Quality Tradeoff

Evaluate the cost vs. quality tradeoff between LLM-as-Judge and Agent-as-Judge approaches. Determine which evaluation method to use for continuous monitoring vs. pre-deployment audits.

### When to Use

You are choosing between evaluation approaches with dramatically different cost profiles — and you need to determine which approach provides sufficient quality for each evaluation context. The difference is not marginal: Agent-as-Judge approaches can cost an order of magnitude more than LLM-as-Judge and run significantly slower. Using the wrong judge for the wrong context either wastes evaluation budget or misses quality issues.

Use this scenario when:
- You are designing an evaluation pipeline and need to select the right judge for each evaluation stage
- Your evaluation costs are growing faster than your evaluation coverage and you need to optimize
- You are considering replacing human evaluators with automated judges and need to validate correlation
- You want to establish a tiered evaluation strategy: cheap judges for high-frequency monitoring, expensive judges for high-stakes audits
- You have noticed inconsistencies between different evaluation approaches and need to understand the accuracy-cost tradeoff

> **Related scenarios:** For evaluating grader model selection, see the triage playbook's grader model selection guide. This scenario focuses specifically on the cost-quality tradeoff between judge _architectures_ (single-LLM vs. agent-based), not between specific grader models.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Compare judge verdicts: does the cheap judge agree with the expensive judge on semantic equivalence assessments? |
| General Quality | Have both judges score the same responses; measure inter-judge agreement and correlation with human labels |
| Exact Match | For objective evaluations (factual accuracy), verify both judges produce the same pass/fail verdict |
| Statistical Analysis | Aggregate agreement rates, cost, and latency across the calibration dataset to quantify the cost-quality tradeoff |

> **Tip:** The key metric is not whether the cheap judge is as _good_ as the expensive judge — it is whether the cheap judge's errors are _tolerable_ for the evaluation context. A 5% disagreement rate may be acceptable for daily production monitoring (where you are detecting trends, not making per-query decisions) but unacceptable for pre-deployment certification (where every verdict matters).

### Setup Steps

1. **Create a calibration dataset.** Select 200–500 agent responses with human-labeled quality scores. These responses should span your full quality range: clearly good, clearly bad, and ambiguous borderline cases. The borderline cases are where judge disagreement concentrates.
2. **Run both judges on the calibration dataset.** Score all responses with both the LLM-as-Judge (single-prompt evaluation) and Agent-as-Judge (multi-step agent evaluation with tool use). Record scores, latency, and token usage for each.
3. **Calculate correlation with human labels.** For each judge, compute: Cohen's Kappa (agreement), Pearson correlation (for numerical scores), and confusion matrix (for pass/fail verdicts). The judge with higher human correlation is the _accuracy ceiling_ for that evaluation type.
4. **Calculate cost per evaluation.** For each judge, compute: average tokens consumed (input + output), average cost per evaluation, average latency per evaluation. Express the cost ratio (Agent ÷ LLM) and latency ratio.
5. **Analyze disagreement cases.** For responses where the two judges disagree, examine the patterns. Does the LLM judge miss nuanced quality issues? Does the Agent judge overcomplicate simple evaluations? Categorize disagreement types to understand where the cheap judge is unreliable.
6. **Define tiered judge assignment.** Based on the analysis, define which evaluation contexts use which judge. Typical assignment: LLM-as-Judge for daily production monitoring and regression testing; Agent-as-Judge for pre-deployment audits, safety evaluations, and cases flagged by the LLM judge as borderline.

### Anti-Pattern

> **"One Judge Fits All"** — Using the same evaluation approach for every context, whether it is a cheap LLM judge on safety-critical decisions or an expensive Agent judge on routine quality monitoring. Evaluation strategy should be tiered: match judge sophistication to decision stakes. The cost of a missed safety issue far exceeds the cost of an expensive judge. The cost of over-evaluating routine queries far exceeds the incremental quality gain.

### Evaluation Patterns

#### Pattern: Agreement Rate Stratification
Measure judge agreement rate stratified by difficulty: easy cases (clear pass/fail), medium cases (mostly correct with minor issues), and hard cases (borderline quality). LLM judges typically agree with Agent judges 90%+ on easy cases but only 60–70% on hard cases. Use this stratification to define where the cheap judge is reliable and where escalation to the expensive judge is necessary.

#### Pattern: Judge Cost-Effectiveness Ratio
Calculate `judge_cost_effectiveness = accuracy_gain / cost_increase` for upgrading from LLM to Agent judge. For example, if upgrading from an LLM judge (85% human correlation) to an Agent judge (92% human correlation) costs 10× more per evaluation, determine whether the 7-percentage-point accuracy gain justifies the cost increase for your use case. Compare this to other ways to spend that evaluation budget (e.g., more test cases with the cheaper judge).

#### Pattern: Escalation Pipeline
Use the LLM judge as a first-pass filter. Cases it scores as clearly passing or clearly failing go directly to verdict. Cases it scores as borderline (within a configurable uncertainty band) are escalated to the Agent judge. This captures most of the Agent judge's accuracy benefit at a fraction of the cost — only 10–20% of cases typically need escalation.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | LLM judge correctly identifies clear pass | Agent response that accurately answers a factual question with citations | Both judges score as pass; LLM judge cost is a fraction of Agent judge cost | General Quality + Compare Meaning |
| 2 | LLM judge correctly identifies clear fail | Agent response that contains a factual error | Both judges score as fail; verdicts agree; LLM judge sufficient for this case | Exact Match + General Quality |
| 3 | Borderline case reveals judge disagreement | Agent response that is partially correct but misses nuance | LLM judge may score as pass; Agent judge catches the nuance issue; human agrees with Agent judge | General Quality + Compare Meaning |
| 4 | Escalation pipeline reduces cost while maintaining accuracy | 500 evaluation cases run through escalation pipeline | ≤20% of cases escalated to Agent judge; overall accuracy within 2% of full Agent-judge pipeline; cost reduction ≥70% | Statistical analysis |
| 5 | Judge consistency on safety-critical evaluations | Agent responses involving safety-sensitive topics (medical, legal, financial) | Agent judge catches safety nuances that LLM judge misses; safety evaluations always use Agent judge | General Quality + Compare Meaning |

### Tips

- Run judge calibration quarterly. Model updates change judge behavior — an LLM judge that correlated well with humans six months ago may have drifted.
- For production monitoring, volume matters more than per-query precision. A cheaper LLM judge lets you evaluate many more queries than an Agent judge at the same budget. More coverage with slightly lower precision often catches more issues than less coverage with higher precision.
- Log judge confidence alongside verdicts. Low-confidence LLM judge verdicts are natural candidates for Agent judge escalation. Many LLM-as-Judge prompts can be configured to output a confidence score.
- The cost and latency ratios between LLM-as-Judge and Agent-as-Judge change with model pricing updates (see arXiv 2512.12791 for methodology). Re-measure after major pricing changes to keep your tiered strategy calibrated.

---

## Scenario 4: Token Efficiency and Prompt Compression

Evaluate prompt compression techniques and token usage efficiency. Measure whether compressed prompts maintain task quality while reducing costs.

### When to Use

Your agent's prompts consume a significant number of tokens — particularly system prompts, retrieved context, or few-shot examples — and you need to evaluate whether compression techniques can reduce cost without degrading quality. Token costs are the primary cost driver for most agent systems, and prompts with large context windows are especially expensive. Compression ratios of 4–20× are achievable, but the quality impact varies dramatically by task type.

Use this scenario when:
- Your system prompt exceeds 2,000 tokens and you want to evaluate whether a compressed version performs equivalently
- You are using RAG and retrieved context is consuming a large share of your token budget
- You are evaluating few-shot example selection and want to determine the minimum number of examples needed
- You want to profile where tokens are consumed across task stages (planning, tool calls, reasoning, output) to identify optimization targets
- Your per-query cost exceeds your budget and you need to identify compression opportunities

> **Related scenarios:** For evaluating knowledge grounding quality after changing retrieval strategies, see [Knowledge Grounding and Accuracy](knowledge-grounding-and-accuracy.md). This scenario focuses on evaluating whether _fewer tokens_ can achieve the same result.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify that responses from compressed prompts are semantically equivalent to full-prompt responses |
| General Quality | Assess whether response quality degrades with compression |
| Exact Match | Verify that factual accuracy is preserved after prompt compression |
| Keyword Match (All) | Verify that critical information from the full prompt still appears in responses |
| Capability Use (All) | Confirm that tool calls and structured outputs are preserved with compressed prompts |

> **Tip:** Not all tokens are equally compressible. System prompt instructions are often highly compressible (verbose instructions can be condensed). Retrieved context is moderately compressible (redundant passages can be deduplicated). Few-shot examples are minimally compressible (each example teaches a different pattern). Profile your token distribution first, then target the most compressible category.

### Setup Steps

1. **Profile token distribution.** For a representative sample of queries, measure token usage by stage: system prompt, user query, retrieved context, few-shot examples, model reasoning, tool calls, and output. Identify which stage consumes the most tokens — this is your optimization target.
2. **Establish a full-prompt baseline.** Run your evaluation suite with full (uncompressed) prompts. Record quality scores, token usage, and cost for every test case.
3. **Apply compression and re-evaluate.** For each compression technique you are testing (prompt condensation, context pruning, example reduction, LLMLingua-style compression), re-run the evaluation suite. Record the same metrics.
4. **Calculate compression-quality curves.** For each compression technique, plot quality score vs. compression ratio. Identify the "knee" — the compression ratio beyond which quality drops sharply. This is your maximum safe compression.
5. **Test task-type sensitivity.** Different task types have different compression tolerance. Factual QA may tolerate high compression (the key fact is a small part of the context). Nuanced reasoning may tolerate very little (removing context removes reasoning material). Measure compression sensitivity per task type.
6. **Calculate ROI.** For each compression technique, compute: tokens saved per query × query volume × cost per token = monthly savings. Compare to implementation complexity. Prioritize techniques with highest savings-to-effort ratio.

### Anti-Pattern

> **"Compress Everything Equally"** — Applying the same compression ratio to all prompt components regardless of their information density. System instructions and retrieved context have different compression ceilings. Compressing a safety instruction from "Under no circumstances should you provide medical diagnoses" to "No medical diagnoses" may lose the emphasis that prevents the agent from hedging. Evaluate compression impact per component, not globally.

### Evaluation Patterns

#### Pattern: Ablation Testing
Systematically remove prompt components one at a time and measure quality impact. Remove the system prompt's detailed instructions → measure quality. Remove few-shot examples → measure quality. Reduce retrieved context from 10 passages to 5 → measure quality. Each ablation reveals the marginal value of that component. Components with low marginal quality impact are safe compression targets.

#### Pattern: Token Budget Allocation
Set a total token budget per query (e.g., 4,000 tokens) and optimize the allocation across components. If system prompt takes 1,500 tokens and retrieved context takes 2,500, test whether reallocating to 800 system + 3,200 context (or vice versa) produces better results. The optimal allocation depends on your task distribution.

#### Pattern: Progressive Compression Ladder
Apply compression in stages: (1) remove duplicate context passages, (2) condense system instructions, (3) reduce few-shot examples to minimum effective set, (4) apply algorithmic compression (LLMLingua-style). Evaluate quality after each stage. Stop when quality drops below your threshold. This staged approach finds the maximum safe compression without overshooting.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Compressed system prompt preserves behavior | Full prompt (2,000 tokens) vs. condensed prompt (500 tokens) for the same user question | Responses semantically equivalent; quality score within 2% of baseline | Compare Meaning + General Quality |
| 2 | Reduced retrieved context maintains accuracy | Full RAG context (10 passages, 3,000 tokens) vs. pruned context (top 3 passages, 900 tokens) | Factual accuracy preserved; key facts present in response | Exact Match + Keyword Match (All) |
| 3 | Minimum few-shot examples identified | 8-shot, 4-shot, 2-shot, 1-shot, and 0-shot variations for the same task | Quality knee identified (e.g., 2-shot achieves 95% of 8-shot quality at 25% of the token cost) | General Quality + Compare Meaning |
| 4 | Algorithmic compression preserves quality | Original prompt vs. LLMLingua-compressed prompt (4–20× compression target) | Quality preservation ≥95% at 4× compression; measure degradation curve up to 20× | Compare Meaning + General Quality |
| 5 | Tool-calling survives prompt compression | Compressed prompt for a query requiring tool invocation | Tool is called with correct parameters; response incorporates tool output correctly | Capability Use (All) + Exact Match |

### Tips

- Input tokens are typically 3–10× cheaper than output tokens, but input tokens dominate total cost because prompts are much longer than responses. A 50% reduction in prompt length often saves more than a 50% reduction in output length.
- Research from Tokenomics (arXiv 2601.14470) shows that moderate planning is optimal — excessive reasoning steps cause "overthinking" that inflates token cost without improving quality. If your agent has a planning stage, evaluate whether shorter plans produce equivalent results.
- Simple memory configurations often beat complex summarization for multi-turn conversations. Before building an expensive summarization pipeline, test whether a sliding window of recent messages performs equivalently.
- LLMLingua-style compression achieves up to 20× compression with minimal quality loss on benchmarks, but real-world performance varies by domain. Always evaluate on your specific task distribution, not on published benchmarks.

---

## Scenario 5: End-to-End Cost Pareto Analysis

Holistic evaluation of the cost-quality Pareto frontier across an agent system. Identify tasks above the Pareto curve where optimization has the highest ROI, and enforce quality floors to prevent cost optimization from degrading critical capabilities.

### When to Use

You need to understand the full cost-quality landscape of your agent system — not just individual optimizations, but the system-wide Pareto frontier that shows the optimal tradeoff between spending and quality. This is the meta-evaluation that informs all other cost decisions: where to invest in optimization, where to accept higher costs, and where your current configuration sits relative to the theoretical optimum.

Use this scenario when:
- You are preparing a business case for agent optimization investment and need to quantify the opportunity
- You manage multiple agent configurations or model versions and need to select the Pareto-optimal one for production
- You want to identify the 20–30% of use cases that consume 60–80% of your budget and prioritize optimization there
- You have applied multiple optimizations (routing, caching, compression) and need to measure their combined impact
- Executive stakeholders want to understand the cost-quality tradeoff in terms they can act on

> **Related scenarios:** This scenario integrates the findings from the four preceding scenarios (routing, caching, judge selection, compression) into a system-level view. Run the individual scenarios first, then use this scenario to evaluate the combined impact.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Score overall response quality for each configuration on the Pareto frontier |
| Compare Meaning | Verify that cheaper configurations produce semantically equivalent responses to expensive ones |
| Exact Match | Confirm factual accuracy is preserved across the Pareto frontier |
| Statistical Analysis | Aggregate cost and quality metrics across configurations to map the frontier |

> **Tip:** The Pareto frontier is not static — it shifts with every model update, pricing change, or traffic pattern change. Treat Pareto analysis as a periodic evaluation (monthly or after major changes), not a one-time exercise. The goal is not to find _the_ optimal configuration but to maintain a _map_ of the frontier so you can navigate tradeoffs as conditions change.

### Setup Steps

1. **Define your configuration space.** List all controllable variables: model choice, routing thresholds, caching parameters, compression settings, prompt variants, retrieval parameters. Each combination is a configuration.
2. **Sample the configuration space.** You cannot evaluate every combination. Use multi-objective Bayesian optimization (syftr approach) or stratified random sampling to select 10–20 configurations that span the cost-quality spectrum. Include your current production configuration as a reference point.
3. **Run full evaluation on each configuration.** For each sampled configuration, run your complete evaluation suite. Record: total cost, per-query cost distribution, quality scores by task type, latency, and success rate. Compute cost-of-pass for each configuration.
4. **Map the Pareto frontier.** Plot configurations on a scatter plot with cost on the x-axis and quality on the y-axis. Identify the Pareto frontier: the set of configurations where no other configuration achieves higher quality at lower cost. Configurations below the frontier are suboptimal — they are dominated by frontier configurations.
5. **Identify the knee point.** The knee of the Pareto frontier is the point of maximum curvature — where small increases in cost yield diminishing quality returns. This is typically the best production configuration. Research shows the knee point often sacrifices only 2–5% quality while being 5–10× cheaper than the maximum-quality configuration.
6. **Analyze cost concentration.** For your current production configuration, profile cost by task type. Identify the 20–30% of task types that consume 60–80% of budget. These are your highest-ROI optimization targets. For each, determine whether routing, caching, or compression would be most effective.
7. **Set quality floors.** Define minimum quality thresholds per task type. Safety-critical tasks may require ≥99% quality (no cost optimization allowed). Routine informational queries may accept ≥90% quality (aggressive optimization allowed). Quality floors prevent cost optimization from degrading critical capabilities.

### Anti-Pattern

> **"Optimize for Average Cost"** — Reducing average cost per query without examining the distribution. Average cost can hide bimodal distributions where most queries are cheap but a small percentage are extremely expensive (long reasoning chains, multiple tool calls, retries). These outlier queries often represent the most valuable interactions. Optimize the distribution, not just the mean — and protect high-value queries from cost-cutting that degrades their quality.

### Evaluation Patterns

#### Pattern: Pareto Frontier Visualization
Plot all evaluated configurations on a cost-quality scatter plot. Draw the Pareto frontier connecting non-dominated points. Mark your current production configuration. The distance between your current configuration and the frontier is your optimization opportunity. Configurations far below the frontier are strong candidates for immediate improvement.

#### Pattern: Cost Concentration Analysis
Profile cost by task type and compute a concentration index (similar to a Gini coefficient). A high concentration index means a small number of task types dominate cost — and targeted optimization of those task types will have outsized impact. Typical finding: 60–80% of costs in 20–30% of use cases.

#### Pattern: Quality Floor Enforcement
For each task type, define a quality floor — the minimum acceptable quality score. When evaluating cost optimizations, verify that no task type drops below its quality floor. This prevents "race to the bottom" optimizations that save money on paper but degrade critical capabilities. Quality floors should be set by business stakeholders, not by the optimization algorithm.

#### Pattern: Bayesian Configuration Search
Use multi-objective Bayesian optimization to efficiently search the configuration space. This method (formalized in the syftr paper, arXiv 2505.20266) uses early stopping to prune clearly suboptimal configurations and focuses evaluation budget on promising regions. It can discover configurations that are 9× cheaper while preserving accuracy — configurations that manual exploration would miss.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Current config vs. Pareto-optimal config | Full evaluation suite on current production config and 3 frontier candidates | Frontier candidate identified that matches quality within 2% at ≤50% cost | General Quality + Compare Meaning |
| 2 | Cost concentration profiling | Production traffic sample (10,000 queries) profiled by task type | Top 3 task types by cost identified; combined, they represent ≥60% of total cost | Statistical Analysis |
| 3 | Quality floor enforcement | Apply aggressive cost optimization (cheapest model, max compression, max caching) to all task types | Safety-critical task types maintain quality ≥99%; routine tasks maintain ≥90% | General Quality + Exact Match |
| 4 | Knee point identification | 10 configurations spanning the cost-quality spectrum | Knee point identified: ≤5% quality reduction vs. best, ≥50% cost reduction vs. best | Statistical Analysis + General Quality |
| 5 | Combined optimization impact | Before: no routing, no caching, no compression. After: all three optimizations applied | Measure combined cost reduction (target: 40–70%) with quality preservation (target: ≥95%) | Compare Meaning + General Quality |

### Tips

- The Pareto frontier is task-dependent. A configuration that is Pareto-optimal for customer support queries may be far from optimal for technical documentation queries. If your agent serves diverse task types, map separate frontiers per task type.
- Simple baselines often beat complex agents on the Pareto frontier. Before investing in sophisticated optimization, test whether a simpler agent architecture achieves acceptable quality at much lower cost. "AI Agents That Matter" (arXiv 2407.01502) found that SOTA agents frequently fall below the Pareto frontier when cost is considered.
- Pareto analysis requires a meaningful quality metric. If your quality metric does not capture what users actually care about, optimizing along the Pareto frontier optimizes for the wrong thing. Validate your quality metric against user satisfaction before using it for cost-quality tradeoff decisions.
- Update your Pareto map monthly. Model pricing changes, new model releases, and shifting traffic patterns all move the frontier. A monthly cadence balances freshness with evaluation cost.
- For teams just starting with cost optimization: the highest-ROI first step is usually profiling cost by task type to find the cost concentration. This tells you _where_ to optimize before you decide _how_ to optimize.

---

## Quick Reference: Route-Cache-Track Framework

| Layer | What It Does | Key Metric | Target |
|-------|-------------|------------|--------|
| **Route** | Send each query to the cheapest capable model | Quality preservation rate | ≥95% of frontier model quality |
| **Cache** | Eliminate redundant computation via semantic caching | Cache hit rate × quality preservation | 40–67% hit rate, ≥98% quality |
| **Track** | Map cost-quality Pareto frontier per task type | Cost-of-pass = cost ÷ success rate | Below Pareto frontier knee point |

### Decision Tree

1. **Is this query a near-duplicate of a recent query?** → Serve from cache (verify cache freshness)
2. **Is this a high-stakes query (safety, legal, financial)?** → Route to frontier model (never optimize away quality on critical queries)
3. **Can a cheaper model handle this query class with ≥95% quality?** → Route to cheaper model
4. **Otherwise** → Route to frontier model and log as optimization candidate for future routing rules

### Cost-of-Pass Formula

```
cost_of_pass(model, task) = average_cost(model, task) / success_rate(model, task)
```

A model with $0.50 average cost and 80% success rate has cost-of-pass = $0.625.
A model with $0.30 average cost and 40% success rate has cost-of-pass = $0.75.
**The more expensive model is more cost-effective** because its higher reliability reduces waste from failed attempts.
