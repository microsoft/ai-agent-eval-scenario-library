# Multi-Agent System Evaluation

> Scenarios for evaluating systems where multiple AI agents collaborate, delegate, or compete to accomplish tasks. Single-agent evaluation tells you whether _one_ agent works; multi-agent evaluation tells you whether _the system_ works — including delegation accuracy, coordination overhead, conflict resolution, and fault tolerance.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Multi-Agent Evaluation Matters

Enterprise adoption of multi-agent architectures is accelerating rapidly — Gartner projects a 1,445% surge in multi-agent system deployments through 2028. Yet fewer than 10% of organizations successfully scale from single-agent pilots to multi-agent production systems. The gap is evaluation: single-agent test methods do not catch the failure modes that emerge when agents interact.

Multi-agent systems introduce failure categories that simply don't exist in single-agent settings:

- **Delegation errors.** The orchestrator routes a task to the wrong specialist agent. The specialist does its job correctly — on the wrong task. End-to-end metrics may still look acceptable because the response is coherent, but the answer comes from the wrong domain.
- **Coordination overhead.** Agents spend more tokens talking to each other than doing actual work. A system that works in demos collapses under real workloads because inter-agent communication consumes the token budget.
- **Conflicting outputs.** Two agents produce contradictory recommendations. Without conflict detection, the system either picks one arbitrarily or merges them into an incoherent response.
- **Cascade failures.** One agent times out or errors, and the failure propagates through the system. A single-agent failure becomes a system-wide outage because no degradation strategy exists.
- **Topology mismatches.** The coordination pattern (star, chain, tree, graph) doesn't fit the task structure. A linear chain topology forces sequential execution on tasks that could be parallelized, adding unnecessary latency.

The **Coordinate-Measure-Scale** framework structures evaluation across these dimensions: first verify agents _coordinate_ correctly, then _measure_ efficiency and quality, then test whether the system _scales_ under failure and load.

> **Key reference:** MultiAgentBench (ACL 2025) introduces milestone-based KPIs for multi-agent collaboration across four topology types. REALM-Bench (Feb 2025) tests progressive planning tasks across LangGraph, AutoGen, CrewAI, and Swarm frameworks.

---

## 1. Task Delegation Accuracy

Does the orchestrator route each task to the correct specialist agent?

### When to Use

Your system has an orchestrator (or router) agent that receives user requests and delegates them to specialist agents — for example, a customer service system with billing, technical support, and account management agents. You need to verify that the orchestrator selects the right specialist for each request, especially for ambiguous requests that could plausibly go to multiple agents.

Use this scenario when:
- Your system has 3 or more specialist agents with overlapping capabilities
- Users submit requests that don't cleanly map to a single specialist
- You have observed misrouting — requests going to the wrong agent
- You are adding new specialist agents and need to verify existing routing isn't disrupted
- Your orchestrator uses intent classification, LLM-based routing, or semantic similarity for delegation

> **Related scenarios:** For single-agent tool routing, see [Tool & Connector Invocations](tool-and-connector-invocations.md). For topic-level routing, see [Trigger Routing](trigger-routing.md). This scenario evaluates routing _across agents_, where each "tool" is itself an autonomous agent with its own capabilities.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirm the orchestrator invoked the expected specialist agent |
| Trajectory Match (In-Order) | Verify the delegation sequence: orchestrator receives request, routes to correct specialist, specialist processes, orchestrator returns result |
| Compare Meaning | Verify the final response is semantically correct for the domain (the right agent answered) |
| Keyword Match (All) | Check for domain-specific terminology that confirms the correct specialist handled the request |

> **Tip:** Delegation accuracy is the foundation of multi-agent evaluation. If the orchestrator routes incorrectly, all downstream metrics — quality, latency, cost — are meaningless because they measure the wrong agent's performance on the wrong task.

### Setup Steps

1. **Map your agent topology.** List every specialist agent, its domain, and its capabilities. Document the boundaries between agents — where does billing end and account management begin?
2. **Create unambiguous test cases.** For each specialist, write 3-5 requests that clearly belong to that agent's domain. These establish your baseline routing accuracy.
3. **Create ambiguous test cases.** Write requests that sit at the boundary between two or more specialists (e.g., "I was charged twice and I want to cancel" spans billing and account management). Document the expected routing for each.
4. **Create negative test cases.** Write requests that should _not_ be delegated to any specialist — they should be handled by the orchestrator directly or escalated to a human.
5. **Configure evaluation to capture the delegation decision.** Log which agent the orchestrator selected, the confidence score (if available), and any fallback behavior.
6. **Run the evaluation.** For each test case, verify that the expected specialist agent handled the request.

### Anti-Pattern

> **Anti-Pattern: Testing Only the Final Response**
> If you only check whether the final answer is correct, you will miss misrouting that happens to produce acceptable results. A billing question routed to the general knowledge agent might return a plausible-sounding answer from training data — but it won't reflect the customer's actual account state. Always verify _which agent_ handled the request, not just _what_ the response says.

### Evaluation Patterns

**Pattern A: One-to-One Routing Verification**
For each specialist agent, create a set of unambiguous requests that should route exclusively to that agent. Measure routing accuracy as the percentage of requests that reach the correct specialist. Target: 95%+ for unambiguous requests.

**Pattern B: Boundary Case Resolution**
Create requests that sit at the intersection of two or more specialist domains. Verify the orchestrator either (a) routes to the most appropriate specialist based on primary intent, or (b) asks a clarifying question to disambiguate. Both are acceptable — silent misrouting is not.

**Pattern C: Fallback and Escalation Behavior**
Create requests that no specialist can handle — out-of-scope questions, multi-domain requests requiring human judgment, or requests with insufficient information. Verify the orchestrator either handles them directly, asks for clarification, or escalates appropriately rather than forcing them to an ill-suited specialist.

**Pattern D: Routing Stability Under Agent Addition**
After adding a new specialist agent to the system, re-run the full routing test suite. Verify that existing routing decisions are not disrupted by the new agent's presence. This is regression testing for multi-agent routing.

### Practical Examples

| # | Scenario | Sample Input | Expected Delegation | Method |
|---|----------|-------------|-------------------|--------|
| 1 | Clear billing request routes to billing agent | "I was charged $49.99 but my plan is $29.99/month" | Orchestrator delegates to `billing_agent` | Capability Use (All) |
| 2 | Clear technical request routes to support agent | "My app crashes when I try to upload files larger than 10MB" | Orchestrator delegates to `technical_support_agent` | Capability Use (All) |
| 3 | Ambiguous request is disambiguated | "I need to change my plan and I think I was overcharged" | Orchestrator delegates to `billing_agent` (primary intent: overcharge) OR asks clarifying question | Capability Use (All) + Compare Meaning |
| 4 | Out-of-scope request triggers fallback | "What's the weather going to be like tomorrow?" | Orchestrator handles directly or responds with scope limitation — does NOT delegate to any specialist | Capability Use (None) + Compare Meaning |
| 5 | New agent doesn't steal existing routes | "I want to update my shipping address" (after adding a new `returns_agent`) | Still delegates to `account_management_agent`, not the new `returns_agent` | Capability Use (All) |
| 6 | Multi-domain request is decomposed | "Cancel my subscription and get a refund for this month" | Orchestrator delegates cancellation to `account_agent` AND refund to `billing_agent` (or sequences them) | Trajectory Match (In-Order) |

### Tips

- **Test with at least 5 requests per specialist agent**, including both clear-cut and boundary cases.
- **Measure routing confidence** if your orchestrator provides it. Low-confidence correct routing is fragile and may break with minor input variations.
- **Rerun after** adding, removing, or modifying specialist agents, changing the orchestrator's system prompt, or updating intent classification models.
- **Target: 95%+ routing accuracy** for unambiguous requests, 80%+ for boundary cases. Below these thresholds, the multi-agent system is actively degrading user experience.

---

## 2. Coordination Topology Comparison

Does your coordination pattern (star, chain, tree, graph) fit your task structure?

### When to Use

Your multi-agent system uses a specific coordination topology — star (hub-and-spoke), chain (sequential pipeline), tree (hierarchical), or graph (fully connected) — and you need to verify that this topology is appropriate for your workload. Different topologies excel at different task types, and the wrong choice can multiply latency, cost, or failure rates.

Use this scenario when:
- You are designing a new multi-agent system and need to select a topology
- Your system's performance doesn't match expectations despite individual agents performing well
- You want to benchmark multiple framework implementations (LangGraph, AutoGen, CrewAI, Swarm) on the same tasks
- You are considering migrating from one topology to another

> **Related scenarios:** For evaluating individual agent performance within the topology, see [Trajectory & Stepwise Evaluation](trajectory-and-stepwise-evaluation.md). For cost efficiency of the overall system, see [Cost-Efficiency Evaluation](cost-efficiency-evaluation.md). This scenario evaluates the coordination structure itself.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Trajectory Match (Any-Order) | Compare which agents are invoked and in what pattern across topologies |
| Token Count / Cost Tracking | Measure total tokens consumed, including inter-agent communication overhead |
| Latency Measurement | Measure end-to-end and per-step latency to identify serialization bottlenecks |
| Milestone Achievement Rate | Track completion of intermediate objectives (not just final success) |

> **Tip:** MultiAgentBench (ACL 2025) found that graph topologies outperform others for research and creative tasks, while star topologies are more efficient for structured workflows with a clear coordinator. There is no universally best topology — test yours against your actual workload.

### Setup Steps

1. **Define benchmark tasks** that represent your production workload. Include tasks of varying complexity: simple (single specialist sufficient), moderate (2-3 agents need to coordinate), and complex (requires information sharing across 4+ agents).
2. **Implement the same task in at least two topologies.** For example, implement a research task using both star topology (orchestrator delegates to specialists) and graph topology (agents communicate peer-to-peer).
3. **Instrument all inter-agent communication.** Log every message between agents, including sender, receiver, token count, and latency.
4. **Define milestone checkpoints.** For each task, identify intermediate objectives that indicate progress (e.g., "data gathered," "analysis complete," "draft written"). Track which milestones are achieved under each topology.
5. **Run identical inputs across all topology variants.** Use the same test cases, same specialist agent configurations, and same LLM models — only the coordination pattern should differ.
6. **Compare results** across task completion rate, milestone achievement, total tokens consumed, end-to-end latency, and coordination overhead ratio.

### Anti-Pattern

> **Anti-Pattern: Choosing Topology by Framework Default**
> Many teams adopt whatever topology their framework provides by default (e.g., star in LangGraph, graph in AutoGen) without testing whether it fits their tasks. A chain topology for a research task forces sequential execution, adding 3-5x latency for tasks that could be parallelized. Always benchmark your topology choice against your actual workload.

### Evaluation Patterns

**Pattern: Head-to-Head Topology Benchmark**
Run the same 20-50 test cases across two or more topologies. Score each on: task completion rate, milestone achievement rate, total tokens consumed, end-to-end latency, and coordination overhead (tokens spent on inter-agent communication as a percentage of total tokens). Present results as a comparison matrix.

**Pattern: Task-Topology Fitness Analysis**
Categorize your tasks by structure (sequential, parallel, iterative, hierarchical) and map each category to the topology that performs best. Use this mapping to implement dynamic topology selection — different tasks get different coordination patterns.

**Pattern: Scaling Stress Test**
Start with a small number of agents (3-4) and progressively add agents while measuring performance. Some topologies (star) degrade gracefully with more agents; others (fully connected graph) can experience communication explosion. Identify the scaling ceiling for your topology.

### Practical Examples

| # | Scenario | Task | Topologies Compared | Key Metric |
|---|----------|------|-------------------|------------|
| 1 | Research task across topologies | "Research the competitive landscape for our product and write a summary" | Star vs. Graph | Milestone achievement rate (data gathering, analysis, synthesis) |
| 2 | Sequential workflow comparison | "Process this insurance claim: validate, assess, approve, notify" | Star vs. Chain | End-to-end latency and step ordering correctness |
| 3 | Parallel subtask efficiency | "Translate this document into French, Spanish, and German simultaneously" | Star vs. Graph | Latency (sequential in chain vs. parallel in star/graph) |
| 4 | Agent scaling behavior | Add agents 4, 5, 6 to the system with the same task | Star and Graph | Tokens per task and coordination overhead ratio at each scale point |
| 5 | Framework comparison on identical topology | Same star topology in LangGraph, AutoGen, CrewAI, and Swarm | Same topology, different frameworks | Task score, token usage, latency |

### Tips

- **Run at least 20 diverse tasks per topology** to get statistically meaningful comparisons. Individual task results can be noisy.
- **Track coordination overhead ratio** (coordination tokens / total tokens). If more than 30% of tokens are spent on inter-agent communication rather than task work, your topology or communication protocol may be inefficient.
- **Don't over-optimize topology for one task type.** Production workloads are diverse — choose a topology that performs well across your task mix, not one that's optimal for a single case.
- **Document your topology decision** with benchmark data. When the team asks "why star instead of graph?", you should have data, not opinions.

---

## 3. Inter-Agent Conflict Resolution

When agents produce contradictory outputs, does the system detect and resolve the conflict?

### When to Use

Your multi-agent system has multiple specialist agents that may produce conflicting recommendations, contradictory facts, or incompatible action plans. You need to verify that the system detects these conflicts and resolves them — through voting, arbitration, confidence scoring, or escalation — rather than silently picking one or merging them into incoherence.

Use this scenario when:
- Multiple agents are consulted on the same question and may disagree
- Agents have access to different data sources that may contain conflicting information
- Your system uses ensemble or voting mechanisms to combine agent outputs
- Agents compete for shared resources (e.g., budget allocation, scheduling)
- You have observed incoherent or contradictory final responses

> **Related scenarios:** For evaluating individual agent accuracy, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For testing graceful degradation when an agent fails (as opposed to actively conflicting), see [Scenario 4: Fault Tolerance and Graceful Degradation](#4-fault-tolerance-and-graceful-degradation). This scenario specifically tests the conflict detection and resolution mechanism.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the final resolved output is semantically correct and coherent |
| LLM-as-Judge | Assess whether the conflict resolution produced a well-reasoned synthesis rather than arbitrary selection |
| Keyword Match (All) | Verify the final output acknowledges the conflict or presents the resolution rationale |
| Trajectory Match (In-Order) | Verify the system followed the expected conflict resolution workflow: detect → evaluate → resolve |

> **Tip:** The most dangerous conflict resolution failure is silent — the system picks one agent's output without detecting that a conflict existed. Always test with inputs that are _designed_ to produce agent disagreement, and verify the conflict detection mechanism fires.

### Setup Steps

1. **Identify conflict-prone scenarios.** Map where your specialist agents' domains overlap and where their knowledge sources might disagree. Common examples: legal vs. business agents on compliance questions, multiple data source agents with different update cadences.
2. **Create conflict-triggering test cases.** Design inputs where two or more agents will produce contradictory outputs. This may require configuring agents with different knowledge bases, different system prompts, or different data snapshots.
3. **Define expected resolution behavior.** For each conflict case, document how the system should resolve it: majority vote, confidence-weighted selection, orchestrator arbitration, user escalation, or synthesis of compatible elements.
4. **Instrument the conflict resolution mechanism.** Log conflict detection events, the conflicting outputs, the resolution strategy applied, and the final merged output.
5. **Run the evaluation.** For each test case, verify: (a) the conflict was detected, (b) the correct resolution strategy was applied, (c) the final output is coherent and correct.

### Anti-Pattern

> **Anti-Pattern: Last-Writer-Wins**
> The system resolves conflicts by simply using whichever agent responds last (or first). This produces non-deterministic behavior — the same input may produce different outputs depending on agent response times. It also means the fastest agent always wins, regardless of whether its answer is better. Conflict resolution should be based on quality signals, not timing.

### Evaluation Patterns

**Pattern: Deliberate Disagreement Injection**
Configure two agents with contradictory knowledge (e.g., Agent A's knowledge says the refund policy is 30 days, Agent B's says 60 days). Send a query that triggers both. Verify the system detects the disagreement and applies the correct resolution strategy (e.g., prefer the more authoritative source).

**Pattern: Confidence-Based Resolution Verification**
When agents provide confidence scores with their outputs, verify the system correctly selects the higher-confidence output. Create test cases where the correct answer comes from the lower-confidence agent to test whether the system blindly follows confidence or applies additional reasoning.

**Pattern: Escalation Threshold Testing**
Create conflicts of varying severity — from minor phrasing differences to fundamental factual contradictions. Verify the system resolves minor conflicts autonomously but escalates major conflicts to human review. Test that the escalation threshold is calibrated correctly.

### Practical Examples

| # | Scenario | Sample Input | Expected Behavior | Method |
|---|----------|-------------|-------------------|--------|
| 1 | Factual disagreement between agents | "What is our refund policy?" (billing agent says 30 days, policy agent says 60 days) | System detects conflict, defers to policy agent (authoritative source), notes the discrepancy | Compare Meaning + LLM-as-Judge |
| 2 | Contradictory recommendations | "Should we invest in Project A or B?" (finance agent recommends A, strategy agent recommends B) | System presents both perspectives with reasoning, does not arbitrarily pick one | LLM-as-Judge |
| 3 | Resource competition | "Schedule meetings with both clients on Thursday" (scheduling agent finds two conflicting slots) | System detects conflict, proposes alternatives, escalates to user for prioritization | Compare Meaning + Keyword Match |
| 4 | Silent conflict detection | "What's the current inventory for SKU-1234?" (warehouse agent and ERP agent report different counts) | System flags the inventory discrepancy rather than silently returning one number | Trajectory Match (detect → flag → resolve) |
| 5 | Minor vs. major conflict handling | "Summarize the Q3 report" (agents agree on substance but differ in emphasis) | System synthesizes without escalation (minor conflict) | LLM-as-Judge |
| 6 | Escalation to human review | "Is this transaction compliant with our AML policy?" (compliance agents disagree) | System escalates to human compliance officer rather than making autonomous decision | Trajectory Match + Capability Use |

### Tips

- **Create a conflict taxonomy** for your system: factual conflicts, recommendation conflicts, resource conflicts, confidence conflicts. Each type may require a different resolution strategy.
- **Test resolution determinism.** Run the same conflict scenario 10+ times. If the resolution changes between runs, your mechanism is non-deterministic and unreliable.
- **Track conflict frequency** in production. If conflicts are rare (< 1% of requests), a simple escalation strategy may suffice. If frequent (> 10%), you need a robust automated resolution mechanism.
- **Target: 100% conflict detection rate** for major conflicts (factual contradictions, incompatible actions). For minor conflicts (phrasing differences), tolerance is acceptable.

---

## 4. Fault Tolerance and Graceful Degradation

When an agent fails, does the system recover or degrade gracefully?

### When to Use

Your multi-agent system depends on multiple agents working together, and you need to verify that a single agent's failure (timeout, error, unavailability) does not cause a system-wide failure. The system should recover automatically where possible, degrade gracefully when full recovery isn't possible, and prevent cascade failures.

Use this scenario when:
- Your system runs in production and agent failures have business impact
- You use external services or APIs through agents that may become unavailable
- You have observed cascade failures where one agent's timeout causes the entire system to hang
- You need to meet availability SLAs despite individual agent unreliability
- You are scaling to more agents and want to understand failure behavior at scale

> **Related scenarios:** For testing single-agent error handling, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). For testing recovery across conversation turns, see [Multi-Turn Conversation Quality](multi-turn-conversation-quality.md). This scenario tests system-level resilience when agents within a multi-agent system fail.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Task Completion Rate (Under Failure) | Measure what percentage of tasks still complete successfully when one or more agents fail |
| Latency Measurement | Track recovery time — how long it takes the system to detect failure and activate fallback |
| Quality Degradation Score | Compare output quality with all agents healthy vs. with one or more agents failed |
| Cascade Failure Detection | Verify a single agent failure does not propagate to other agents |

> **Tip:** REALM-Bench found that most multi-agent frameworks handle "easy" failures (immediate errors) reasonably well, but struggle with "slow" failures — agents that respond with incorrect or partial results rather than explicit errors. Test both failure modes.

### Setup Steps

1. **Map your failure modes.** For each agent in the system, document: what happens if it times out? Returns an error? Returns incorrect/partial data? Is completely unavailable?
2. **Create failure injection mechanisms.** Build the ability to simulate agent failures: add configurable timeouts, error responses, and agent unavailability. Ideally, use a chaos engineering approach where failures are injected randomly.
3. **Define expected degradation behavior.** For each agent failure, document: what is the fallback? Should another agent take over? Should the system return a partial result? Should it escalate to a human? What is the maximum acceptable quality loss?
4. **Create test cases for each failure mode.** For each agent, create test cases where that agent fails during task execution. Include cases where: (a) only one agent fails, (b) multiple agents fail simultaneously, (c) the orchestrator itself fails.
5. **Run evaluations.** For each test case, measure: recovery time, task completion rate, output quality vs. baseline, and whether cascade failures occurred.

### Anti-Pattern

> **Anti-Pattern: Infinite Retry Loops**
> The system detects an agent failure and retries indefinitely, consuming resources and increasing latency until the request times out at the system level. Retries should have a maximum count (typically 2-3), with a fallback strategy after retries are exhausted. Implement exponential backoff with a circuit breaker.

### Evaluation Patterns

**Pattern: Single Agent Failure Sweep**
For each agent in the system, inject a failure and measure the impact on task completion. Create a failure impact matrix showing: which agent failed, recovery strategy used, task completion rate, quality degradation, and recovery time. This identifies which agents are single points of failure.

**Pattern: Cascading Failure Test**
Inject a failure in one agent and monitor all other agents. Verify that the failure does not propagate — other agents should continue operating normally. Specifically test: (a) agents that depend on the failed agent's output, (b) agents that share resources with the failed agent, (c) the orchestrator's ability to continue coordinating remaining agents.

**Pattern: Progressive Degradation**
Start with all agents healthy and progressively fail agents one at a time. Measure quality at each degradation level. The system should degrade gracefully — quality decreases proportionally to the number of failed agents, not catastrophically. Plot a degradation curve.

**Pattern: Recovery Time Measurement**
Measure the time from failure injection to: (a) failure detection, (b) fallback activation, and (c) task completion via fallback. Each should have a target SLA. For production systems, total recovery time should typically be under 5 seconds for user-facing flows.

### Practical Examples

| # | Scenario | Failure Injected | Expected Behavior | Key Metrics |
|---|----------|-----------------|-------------------|-------------|
| 1 | Specialist agent timeout | `technical_support_agent` times out after 30s | Orchestrator detects timeout, routes to general knowledge agent as fallback, response is less detailed but still helpful | Recovery time < 5s, quality degradation < 30% |
| 2 | Specialist returns error | `billing_agent` returns HTTP 500 | Orchestrator retries once, then apologizes and offers to escalate to human support | Task completion via escalation, no cascade |
| 3 | Slow/incorrect response | `data_agent` returns stale data from cache | System detects data staleness (if instrumented), flags uncertainty in response | Quality degradation measurable, staleness detected |
| 4 | Multiple simultaneous failures | `billing_agent` AND `account_agent` both down | System handles available requests with remaining agents, queues or escalates the rest | Partial completion, no system-wide outage |
| 5 | Orchestrator failure | The orchestrator agent itself times out | System-level fallback: direct routing to default agent, or graceful error message | Recovery time, availability SLA met |
| 6 | Cascade prevention | `data_agent` hangs (never responds) | Other agents continue processing; orchestrator doesn't block waiting indefinitely | Zero cascade failures, timeout mechanism works |

### Tips

- **Identify your single points of failure.** If one agent's failure causes the entire system to fail, that agent needs redundancy, a fallback strategy, or both.
- **Test slow failures, not just hard failures.** An agent that returns incorrect data is harder to detect than one that returns an error. Build validation checks for agent outputs.
- **Measure quality degradation quantitatively.** "The system still works" is not enough — measure exactly how much quality degrades when each agent fails. If billing agent failure causes 50% quality loss, that agent needs a stronger fallback.
- **Implement circuit breakers.** After N consecutive failures from an agent, stop sending requests to it for a cooldown period rather than continuing to fail on every request.
- **Target: Zero cascade failures.** A single agent failure should never take down the system. Task completion may decrease, quality may degrade, but the system must remain available.

---

## 5. Communication Efficiency and Token Budget

How much of the token budget goes to coordination vs. actual task work?

### When to Use

Your multi-agent system consumes significantly more tokens than equivalent single-agent approaches, and you need to understand where the tokens are going. Inter-agent communication — instructions, context passing, status updates, results sharing — can consume 30-70% of total tokens in poorly optimized systems. This scenario measures communication overhead and tests optimization strategies.

Use this scenario when:
- Your multi-agent system's token costs are 3x+ higher than single-agent baselines
- You want to compare verbose vs. compressed communication protocols between agents
- You are optimizing system prompts for inter-agent communication
- You need to set and enforce token budgets per agent or per coordination step
- You are evaluating whether a multi-agent approach is justified vs. a single-agent approach

> **Related scenarios:** For general cost-efficiency evaluation, see [Cost-Efficiency Evaluation](cost-efficiency-evaluation.md). For evaluating response quality, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md). This scenario focuses specifically on _inter-agent_ communication overhead in multi-agent systems.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Token Count (Per Agent) | Measure tokens consumed by each agent, separated into task work vs. coordination |
| Coordination Overhead Ratio | Calculate: coordination tokens / total tokens. Track this metric across tasks |
| Task Score Per Token | Measure task quality divided by total tokens — the efficiency metric |
| Information Loss Rate | When using compressed communication, measure whether important information is lost |

> **Tip:** Research shows that coordination overhead ratios above 30% indicate an optimization opportunity. If agents spend more tokens coordinating than working, consider: (a) reducing context passing, (b) using structured/compressed inter-agent messages, or (c) questioning whether the task needs multiple agents at all.

### Setup Steps

1. **Instrument token tracking.** For every message in the system, log: sender agent, receiver agent, token count, and message type (task instruction, context sharing, status update, result, coordination). If your framework doesn't support this natively, add logging middleware.
2. **Classify tokens.** Categorize every token as either "task work" (directly contributing to the user's request) or "coordination" (inter-agent communication, context passing, status reporting). The ratio is your coordination overhead.
3. **Establish a single-agent baseline.** Run the same tasks through a single agent and measure token consumption. This is your baseline — the multi-agent system should provide meaningful quality or capability gains to justify its additional token cost.
4. **Test communication protocol variants.** If possible, run the same tasks with different inter-agent communication strategies: verbose (full context in every message), structured (JSON schemas), compressed (summarized context), and minimal (only essential parameters).
5. **Set token budgets and test enforcement.** Define maximum token budgets per agent and per coordination step. Verify the system respects these budgets and degrades gracefully when approaching limits.

### Anti-Pattern

> **Anti-Pattern: Full Context Forwarding**
> Every agent receives the complete conversation history and every other agent's full output, regardless of relevance. This is the "email reply-all" of multi-agent systems. Token consumption scales quadratically with the number of agents. Use targeted context passing — each agent receives only what it needs.

### Evaluation Patterns

**Pattern: Communication Protocol Comparison**
Run the same 20+ tasks with different communication protocols between agents (verbose, structured, compressed, minimal). For each protocol, measure: task completion rate, output quality, total tokens, and coordination overhead ratio. Find the protocol that maximizes quality per token.

**Pattern: Token Budget Enforcement**
Set a total token budget for the system (e.g., 10,000 tokens per request) and measure: how often does the system exceed the budget? When it stays within budget, is quality acceptable? When it exceeds, by how much? This tests whether token budgets are enforceable without quality collapse.

**Pattern: Multi-Agent vs. Single-Agent Justification**
Run identical tasks through both your multi-agent system and a single-agent baseline. Compare: token cost, latency, and output quality. Calculate the "multi-agent premium" — the additional cost divided by the quality improvement. If the premium is too high (e.g., 5x tokens for 10% quality gain), the multi-agent approach may not be justified for that task type.

**Pattern: Redundant Message Detection**
Analyze inter-agent communication logs for redundancy: repeated information, unnecessary acknowledgments, context that was already available to the receiving agent. Calculate the redundant message rate and identify optimization targets.

### Practical Examples

| # | Scenario | Test Setup | Key Metrics | Target |
|---|----------|-----------|-------------|--------|
| 1 | Measure coordination overhead | Run 50 diverse tasks, log all inter-agent messages | Coordination overhead ratio | < 30% of total tokens on coordination |
| 2 | Compare verbose vs. compressed protocols | Same 20 tasks with full-context vs. summarized inter-agent messages | Quality delta, token savings | Compressed saves 40%+ tokens with < 5% quality loss |
| 3 | Multi-agent vs. single-agent justification | Same 30 tasks through multi-agent and single-agent systems | Task score per token, quality delta | Multi-agent quality gain > 15% to justify 2x+ token cost |
| 4 | Token budget enforcement | Set 10K token budget, run 50 tasks | Budget compliance rate, quality under budget | 90%+ tasks within budget, quality > 80% of uncapped |
| 5 | Redundant message detection | Analyze communication logs from 100 task runs | Redundant message rate | < 10% of inter-agent messages are redundant |
| 6 | Per-agent token profiling | Track token usage by agent across 50 tasks | Token distribution heatmap, highest consumers | No single agent consuming > 40% of total tokens |

### Tips

- **Measure coordination overhead from day one.** It's much harder to optimize communication after the system is built than during design. Set overhead targets early.
- **Structured inter-agent messages** (JSON schemas, typed interfaces) typically reduce token consumption by 30-50% compared to free-form natural language, with minimal quality loss.
- **Question the multi-agent assumption.** Not every task benefits from multiple agents. If your single-agent baseline achieves 90%+ of the multi-agent quality at 40% of the token cost, the task may not warrant multi-agent coordination.
- **Track token efficiency trends over time.** As you optimize, coordination overhead should decrease. If it increases after changes, investigate.
- **Target: Coordination overhead ratio < 30%.** Task score per token should be within 50% of single-agent efficiency (i.e., if you're using 2x the tokens, you should get at least 1.5x the quality).

---

## Getting Started

If you're new to multi-agent evaluation, start with this sequence:

1. **Start with Scenario 1 (Delegation Accuracy).** This is the foundation — if routing is broken, nothing else matters. Achieve 95%+ accuracy on unambiguous requests before moving on.

2. **Add Scenario 4 (Fault Tolerance).** Once routing works, verify the system is resilient. Inject failures and ensure no cascade failures occur. This is the most common production incident source.

3. **Add Scenario 5 (Communication Efficiency).** Measure your coordination overhead ratio. If it's above 30%, optimize before scaling further.

4. **Add Scenario 2 (Topology Comparison) and Scenario 3 (Conflict Resolution)** as your system matures and you need to optimize coordination patterns and handle edge cases.

> **Framework recap — Coordinate-Measure-Scale:** First verify agents _coordinate_ correctly (Scenarios 1, 3). Then _measure_ efficiency and quality (Scenarios 2, 5). Then test whether the system _scales_ under failure and load (Scenario 4). Each phase builds on the previous one.
