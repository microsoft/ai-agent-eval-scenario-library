# Multi-Modal Agent Evaluation

> Scenarios for evaluating agents that process and reason across multiple modalities — vision, text, audio, and GUI interaction. Single-modality evaluation measures whether an agent can read _or_ see; multi-modal evaluation measures whether it can _reason_ across inputs, detect conflicts between what it reads and what it sees, and sustain coherent performance over extended sessions.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Multi-Modal Evaluation Matters

Multi-modal agents are deployed in high-stakes settings — document processing that combines text and images, GUI automation that requires screenshot understanding, and customer interactions that span voice, text, and visual media. Standard text-only evaluation misses the failure modes unique to multi-modal reasoning:

- **Fake visual grounding.** The agent appears to understand an image but is actually answering from text cues or parametric knowledge. Swap the image and the answer doesn't change — the agent never looked at it. Current VLMs can frequently use text shortcuts rather than genuine image understanding, as suggested by ablation studies in the literature.
- **Cross-modal contradictions.** Text says one thing, the image shows another. The agent picks one modality and ignores the conflict instead of flagging it. In document processing, this produces silently wrong extractions.
- **GUI task fragility.** The agent can describe a screenshot but cannot reliably execute multi-step GUI workflows from visual observation alone. OSWorld (ICML 2024) benchmarks suggest a significant gap between human performance and current AI agents on screenshot-based tasks.
- **Reliability illusions.** An agent that passes 80% of the time on any single attempt (pass@1) may fail catastrophically when you need it to succeed 8 times in a row (pass^8). Enterprise deployment needs consistency, not peak performance. The distinction between pass@k and pass^k (introduced by tau-bench) can help reveal whether an agent is production-ready.
- **Session meltdowns.** Multi-modal agents lose coherence over long sessions — not because they hit context limits, but because accumulated state degrades reasoning. These meltdowns may be sudden rather than gradual and may not correlate with context window utilization, as observed in Vending-Bench experiments.

The **Ground-Verify-Sustain** framework structures multi-modal evaluation in three phases: first verify the agent genuinely _grounds_ its reasoning in each modality, then _verify_ cross-modal consistency and task accuracy, then test whether performance _sustains_ across repeated runs and extended sessions.

> **Key references:** OSWorld (ICML 2024) benchmarks GUI agents on real operating systems. VisualWebArena tests multi-modal web agents. tau-bench (simmering.dev) introduces pass@k vs pass^k reliability metrics. Vending-Bench studies agent coherence degradation over long sessions.

---

## 1. Visual Grounding Verification

Does the agent genuinely understand visual content, or is it relying on text shortcuts?

### When to Use

Your agent processes images, screenshots, charts, or documents that contain visual information — and you need to verify it actually uses the visual content rather than answering from text context, parametric knowledge, or layout heuristics. This is the most fundamental multi-modal evaluation: if the agent isn't truly grounded in the visual input, every downstream multi-modal capability is built on sand.

Use this scenario when:
- Your agent extracts information from documents that contain images, charts, or diagrams
- Your agent describes or answers questions about user-uploaded images
- You suspect the agent gives correct answers even when the image is irrelevant or swapped
- Your agent processes medical images, architectural diagrams, or other domain-specific visuals
- You are comparing VLM providers and need to know which one actually looks at the image

> **Related scenarios:** For text-only knowledge grounding, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For tool use verification, see [Tool & Connector Invocations](tool-and-connector-invocations.md). This scenario specifically evaluates whether the agent's visual perception is genuine.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's description or extraction matches the actual visual content |
| Keyword Match (All) | Confirm the response includes visual-only details that cannot be inferred from text |
| Trajectory Match (In-Order) | Verify the agent's reasoning chain references specific visual elements before producing its answer |

> **Tip:** The single most revealing test for visual grounding is the **image swap ablation** — give the agent the same text prompt with a different (but plausible) image. If the answer doesn't change, the agent isn't using the image. This simple test catches the majority of fake grounding.

### Setup Steps

1. **Create image-dependent test cases.** Select 10-20 images where the correct answer _requires_ looking at the image and cannot be inferred from the text prompt alone. For example: "What color is the highlighted section in this chart?" where the color is not mentioned in any surrounding text.
2. **Create ablation variants.** For each test case, prepare a swapped version: same text prompt, different image that would produce a different correct answer. For example, swap a bar chart showing Q1 revenue for one showing Q3 revenue.
3. **Create text-only baselines.** Run the same text prompts without any image. If the agent answers correctly without the image, your test case doesn't require visual grounding — redesign it.
4. **Define visual-only ground truth.** For each image, list 3-5 specific visual details that can only be known from the image (colors, spatial relationships, specific values from charts, objects in specific positions).
5. **Run the evaluation.** For each test case: (a) run with correct image, (b) run with swapped image, (c) run without image. Compare answers across all three conditions.
6. **Calculate grounding score.** True grounding = correct answer with correct image AND different answer with swapped image AND failed/refused answer without image.

### Anti-Pattern

> **Anti-Pattern: Testing Visual Grounding with Captioned Images**
> If your test images contain embedded text captions that describe their content (e.g., a chart with a title "Q1 Revenue: $4.2M"), the agent can answer correctly by reading the caption text rather than understanding the visual content. Always use images where the answer requires interpreting visual elements (colors, spatial layout, trends, relative sizes) rather than reading embedded text.

### Evaluation Patterns

**Pattern A: Image Swap Ablation**
For each test case, swap the image while keeping the text prompt identical. Measure the change in answer. True visual grounding produces different answers for different images. Score: percentage of test cases where the answer correctly changes when the image changes. Target: 90%+.

**Pattern B: Visual Detail Extraction**
Ask the agent to list specific visual details from an image — colors, counts, spatial relationships, sizes. Compare against ground truth annotations. This tests perception accuracy independent of reasoning. Score: precision and recall of visual details extracted.

**Pattern C: Visual-Textual Integration**
Provide an image and related text, then ask questions that require combining information from both. For example: a product image showing a red widget, plus text saying "the blue widgets cost $10 and the red ones cost $15" — ask "how much does the widget in the image cost?" The agent must ground in the image (red) and integrate with text ($15).

**Pattern D: Domain-Specific Visual Reasoning**
For specialized domains (medical imaging, architectural plans, financial charts), test whether the agent can perform domain-appropriate visual reasoning. Use expert-annotated ground truth. Measure not just whether the agent sees the right things, but whether it interprets them correctly within the domain context.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Chart reading requires actual chart | "What was the revenue in Q3?" + bar chart image | Extracts correct Q3 value from the chart; different chart gives different value | Compare Meaning + image swap ablation |
| 2 | Object identification in scene | "How many red chairs are in this room?" + room photo | Counts chairs visible in the image, not from text context | Keyword Match (count) |
| 3 | Visual detail not in text | "What color is the warning indicator?" + dashboard screenshot | Names the correct color visible in the screenshot | Compare Meaning |
| 4 | Image-text integration | Product photo (red widget) + pricing text mentioning colors | "The widget in the image costs $15" (integrates visual red + text price for red) | Compare Meaning |
| 5 | Ablation: no image provided | Same prompts as above, but without images | Agent declines to answer or states it needs the image | Compare Meaning |
| 6 | Domain visual reasoning | Chest X-ray + clinical notes | Identifies visual findings consistent with the image, not just restating clinical notes | Keyword Match (All) + domain expert review |

### Tips

- **Always include image swap ablations** in your test suite. Without them, you cannot distinguish genuine grounding from text-shortcut answers.
- **Use at least 3 ablation conditions**: correct image, swapped image, no image. The three-way comparison is much more revealing than any single condition.
- **Target: 90%+ grounding accuracy** (correct answer changes when image changes). Below 80%, the agent is primarily using text shortcuts.
- **Rerun after** model upgrades, prompt changes, or switching VLM providers. Visual grounding capability varies dramatically across models.

---

## 2. Cross-Modal Consistency Under Conflict

Does the agent detect and appropriately handle contradictions between modalities?

### When to Use

Your agent receives information from multiple modalities simultaneously — for example, a document where the text says one thing but an embedded image or table shows something different. You need to verify the agent detects these conflicts rather than silently choosing one modality and ignoring the other.

Use this scenario when:
- Your agent processes documents that combine text, images, tables, and charts
- Information from different sources or modalities may contradict each other
- Silent errors from undetected conflicts would have business consequences (financial reporting, medical records, legal documents)
- Your agent synthesizes information from visual and textual inputs into a single response
- You are building a document extraction pipeline where accuracy is critical

> **Related scenarios:** For single-modality conflict handling, see [Knowledge Grounding & Accuracy, Scenario 3: Conflicting Sources](knowledge-grounding-and-accuracy.md). For multi-component coordination evaluation, see [Tool & Connector Invocations](tool-and-connector-invocations.md). This scenario evaluates conflict detection _across modalities_.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent flags the conflict rather than silently picking one modality |
| Keyword Match (All) | Check for conflict-indicating language ("however," "discrepancy," "the image shows X but the text states Y") |
| Trajectory Match (In-Order) | Verify the agent's reasoning acknowledges both modalities before resolving the conflict |

> **Tip:** The most dangerous failure mode is not getting the wrong answer — it's getting a confidently wrong answer with no indication that conflicting information existed. Your evaluation should specifically test for the _absence_ of conflict acknowledgment, not just answer correctness.

### Setup Steps

1. **Create conflict test cases.** Prepare 10-15 inputs where visual and textual information deliberately contradict each other. Examples: a chart showing declining revenue with text claiming "revenue grew 15%"; a photo of a blue product with text describing it as red.
2. **Vary conflict severity.** Include subtle conflicts (small numerical discrepancies) and obvious conflicts (completely contradictory information). Score detection across both categories.
3. **Include non-conflict controls.** Mix in 5-10 cases where visual and textual information are consistent. The agent should process these normally without false-positive conflict flags.
4. **Define expected behavior tiers.** Tier 1 (minimum): agent mentions the discrepancy. Tier 2 (good): agent identifies which modality is likely correct with reasoning. Tier 3 (excellent): agent provides both interpretations and asks for clarification.
5. **Run the evaluation.** For each conflict case, score: (a) did the agent detect the conflict? (b) did it mention both modalities? (c) did it resolve or escalate appropriately?

### Anti-Pattern

> **Anti-Pattern: Only Testing Obvious Contradictions**
> If you only test cases where text says "up" and the image shows "down," you'll get inflated detection rates. Real-world conflicts are subtle: a chart that shows 14.7% growth while the text rounds to "approximately 15%" — is that a conflict or acceptable rounding? Test a range of conflict magnitudes to find your agent's detection threshold.

### Evaluation Patterns

**Pattern A: Conflict Detection Rate**
Present the agent with known cross-modal conflicts and measure how often it identifies the discrepancy. Score separately for obvious conflicts (target: 95%+) and subtle conflicts (target: 70%+).

**Pattern B: Modality Preference Bias**
When conflicts exist, does the agent consistently prefer one modality over the other? Present 10+ conflicts where the text is correct and 10+ where the image is correct. Measure whether the agent favors text or visual input regardless of which is actually correct. A well-calibrated agent should not have strong modality bias.

**Pattern C: False Positive Rate**
Present consistent (non-conflicting) cross-modal inputs and measure how often the agent incorrectly flags a conflict. Target: fewer than 5% false positives. Over-flagging is disruptive and erodes user trust.

**Pattern D: Resolution Quality**
When the agent detects a conflict, evaluate the quality of its resolution. Does it explain what each modality shows? Does it reason about which is more likely correct? Does it suggest verification steps? Score on a 1-3 scale: 1 = detects only, 2 = detects and reasons, 3 = detects, reasons, and suggests resolution.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Text-chart revenue conflict | Text: "Q3 revenue was $4.2M" + Chart showing Q3 bar at ~$3.8M | Flags discrepancy: "The text states $4.2M but the chart appears to show approximately $3.8M" | Compare Meaning + Keyword Match |
| 2 | Image-text color conflict | Text: "The product comes in blue" + Photo showing a red product | Identifies that the image shows red while the text says blue | Compare Meaning |
| 3 | Subtle numerical discrepancy | Text: "Growth was approximately 15%" + Chart showing 14.2% | Acknowledges minor discrepancy or treats as acceptable rounding (either is acceptable) | Compare Meaning |
| 4 | Consistent input (control) | Text: "The team has 12 members" + Org chart showing 12 people | Processes normally without flagging a false conflict | Keyword Match (absence of conflict language) |
| 5 | Multi-conflict document | Invoice with three mismatches between line items (text) and totals (table) | Identifies all three discrepancies, not just the first one | Compare Meaning |
| 6 | Ambiguous conflict requiring escalation | Legal document with clause text that may or may not contradict an attached diagram | Agent notes the potential conflict and recommends human review rather than resolving unilaterally | Compare Meaning |

### Tips

- **Test both conflict detection AND resolution.** Detection alone is table stakes — the agent should also explain what it found and suggest next steps.
- **Measure modality bias** explicitly. Many agents default to text when conflicts arise, which is correct in some domains (legal documents) but incorrect in others (visual inspection).
- **Target: 95%+ detection for obvious conflicts**, 70%+ for subtle conflicts, fewer than 5% false positives on consistent inputs.
- **Use domain-appropriate conflict severity.** In financial reporting, a $400K discrepancy is critical; in casual product descriptions, minor color shade differences may be acceptable.

---

## 3. GUI Task Completion with Screenshot-Only Observation

Can the agent complete multi-step GUI tasks using only visual observation of the screen?

### When to Use

Your agent automates tasks in graphical user interfaces — web browsers, desktop applications, or mobile apps — by observing screenshots and generating actions (clicks, keystrokes, scrolling). You need to evaluate not just whether the final goal is achieved, but how efficiently and reliably the agent navigates the GUI.

Use this scenario when:
- Your agent performs browser-based or desktop automation tasks
- The agent must interpret UI elements from screenshots rather than structured accessibility data
- You need to evaluate multi-step workflows where each step depends on visual understanding of the current screen state
- You are comparing agents or models for GUI automation capability
- Your agent needs to handle UI variations (different themes, resolutions, or layouts)

> **Related scenarios:** For evaluating tool invocations in general, see [Tool & Connector Invocations](tool-and-connector-invocations.md). For multi-step process evaluation, see [Tool & Connector Invocations, Scenario 6: Multi-Tool Orchestration](tool-and-connector-invocations.md). This scenario specifically evaluates visual GUI navigation where the agent must interpret screenshots to determine its actions.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Trajectory Match (In-Order) | Verify the agent performs GUI steps in the correct sequence |
| Milestone Achievement Rate (checking whether the agent reaches defined intermediate objectives in a task) | Track completion of intermediate sub-goals, not just the final outcome |
| Capability Use (All) | Confirm the agent interacted with the correct UI elements |
| Latency Measurement (tracking wall-clock time per step and end-to-end) | Measure time-to-completion for multi-step workflows |

> **Tip:** Binary pass/fail scoring massively undervalues GUI agents. An agent that completes 4 out of 5 steps gets the same score as one that fails immediately. Use graph-based sub-goal evaluation to measure partial progress — it reveals which steps are bottlenecks and provides actionable improvement signals.

### Setup Steps

1. **Define GUI tasks with sub-goals.** Break each task into a dependency graph of sub-goals. For example, "Book a flight" decomposes to: navigate to booking page -> enter departure -> enter destination -> select dates -> choose flight -> enter passenger details -> confirm booking. Each sub-goal is independently scorable.
2. **Capture ground-truth screenshots.** For each sub-goal, capture the expected screen state after successful completion. These serve as visual checkpoints.
3. **Prepare environment variants.** Set up the same tasks in different visual conditions: light/dark theme, different screen resolutions, with/without pop-ups or overlays. This tests visual robustness.
4. **Instrument action logging.** Record every action the agent takes (click coordinates, text input, scrolling), along with the screenshot that triggered each action.
5. **Define scoring rubric.** For each task: (a) binary end-to-end success, (b) sub-goal completion fraction (0.0-1.0), (c) step efficiency (actual steps / minimum required steps), (d) error recovery (did the agent recover from wrong clicks?).
6. **Run evaluation across multiple trials.** Execute each task at least 3 times to measure consistency (see Scenario 4 for pass@k methodology).

### Anti-Pattern

> **Anti-Pattern: Testing Only on Clean, Static UIs**
> If you only test GUI automation on pristine interfaces with no popups, loading states, or dynamic content, your eval results won't transfer to production. Real GUIs have cookie consent banners, notification popups, loading spinners, and layout shifts. Include these environmental challenges in your test suite.

### Evaluation Patterns

**Pattern A: Sub-Goal Completion Graph**
Model each task as a directed acyclic graph (DAG) of sub-goals. Score the agent on which sub-goals it completes, regardless of whether it achieves the final goal. This reveals exactly where agents fail — for example, "agents consistently complete navigation and form filling but fail at confirmation dialogs."

**Pattern B: Step Efficiency Ratio**
Measure the ratio of actions the agent actually takes to the minimum number of actions required. A ratio of 1.0 means perfect efficiency; 2.0+ means the agent is taking twice as many steps as needed (wasted clicks, unnecessary scrolling, backtracking). Target: under 1.5 for familiar UIs, under 2.0 for novel UIs.

**Pattern C: Visual Robustness Testing**
Run the same task across UI variants: light vs. dark theme, 1080p vs. 1440p resolution, with and without modal overlays. Measure the performance delta. A robust agent shows less than 10% accuracy drop across visual variants.

**Pattern D: Error Recovery**
Deliberately introduce wrong states — navigating to the wrong page, clicking the wrong button — and measure whether the agent detects the error and recovers. Score: percentage of recoverable errors where the agent successfully gets back on track.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Web form completion | "Fill out a job application form" + 3-page form screenshot | Completes sub-goals: navigate to form, fill personal info, upload resume, fill experience, submit (5/5) | Trajectory Match (In-Order) + Milestone Achievement Rate |
| 2 | E-commerce purchase flow | "Find and buy wireless headphones" + store homepage screenshot | Completes sub-goals: search, select item, add to cart, checkout (4/4) | Trajectory Match (In-Order) + Milestone Achievement Rate |
| 3 | Dark mode robustness | Same task with light and dark theme screenshots | Same sub-goals completed in both themes; less than 10% accuracy delta | Compare Meaning + Milestone Achievement Rate |
| 4 | Popup handling | "Search for headphones" + page screenshot with cookie consent banner | Dismisses popup and resumes original task flow | Capability Use (All) + Trajectory Match (In-Order) |
| 5 | Dynamic content navigation | "Find the pricing section" + page with lazy-loaded content | Locates target content by scrolling past dynamically loaded sections | Capability Use (All) + Compare Meaning |
| 6 | Error recovery from wrong page | Agent placed on incorrect page mid-task + wrong-page screenshot | Detects wrong state, navigates back, resumes correct flow | Trajectory Match (In-Order) + Compare Meaning |

### Tips

- **Always use sub-goal scoring**, not just binary pass/fail. It provides 5-10x more information per test run.
- **Test with at least 3 trials per task** to measure consistency. A single successful run is not evidence of reliability.
- **Include at least 20% "noisy" environments** — popups, overlays, loading states — in your test suite.
- **Benchmark against the human-AI gap.** Published OSWorld benchmarks report a substantial human-AI gap on screenshot-based tasks. If your agent scores significantly above published state-of-the-art, verify your eval methodology is sufficiently rigorous.
- **Target: 60%+ sub-goal completion** for production-ready agents, with step efficiency under 1.5x.

---

## 4. Reliability and Consistency (Pass@k vs Pass^k)

Does the agent succeed reliably across repeated attempts, or is it only sometimes correct?

> **Why this is in the multi-modal guide:** While pass@k vs pass^k analysis applies to any agent, multi-modal agents typically exhibit higher variance than text-only agents because vision and GUI interpretation introduce additional sources of non-determinism (image encoding, screenshot rendering, OCR variability). Multi-modal tasks tend to show wider pass@1-to-pass^k gaps, making reliability testing especially critical for multi-modal deployments.

### When to Use

Your agent is deployed in a production environment where inconsistent behavior is as harmful as consistent failure. A customer-facing agent that answers correctly 80% of the time creates a worse experience than one that reliably says "I don't know" — because users lose trust after unpredictable failures. You need to distinguish between agents that sometimes get lucky (high pass@1, low pass^k) and agents that are genuinely reliable (high pass^k).

Use this scenario when:
- Your agent is deployed in production and inconsistent behavior has been reported
- You are selecting between model providers or agent configurations
- Your system requires high reliability (financial, medical, safety-critical)
- You want to understand whether observed accuracy reflects true capability or lucky sampling
- Stakeholders equate "it worked in the demo" with "it's ready for production"

> **Related scenarios:** For evaluating regression across updates, see [Regression Testing](regression-testing.md). For cost implications of retry strategies, see [Tool & Connector Invocations](tool-and-connector-invocations.md). This scenario focuses on measuring inherent reliability independent of the task being performed.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Repeated Trial Execution (running the same input multiple times to measure variance) | Run identical inputs k times (k=8 recommended) to measure consistency |
| Compare Meaning | Verify semantic correctness of each trial independently |
| Statistical Analysis (aggregating trial results into pass@1, pass@k, and pass^k metrics) | Calculate pass@1, pass@k, and pass^k metrics with confidence intervals |

> **Tip:** A key insight from reliability research (e.g., tau-bench): pass@1 and pass^k can diverge substantially. If your agent needs to handle 8 similar requests correctly in a row (a normal day for a customer service agent), pass^8 is the metric that predicts production performance, not pass@1.

### Setup Steps

1. **Select a representative task set.** Choose 20-50 tasks that represent your production workload. Include easy, medium, and hard tasks in proportions matching production distribution.
2. **Define k (number of repetitions).** Start with k=8 for a balance between statistical power and evaluation cost. For high-reliability requirements, use k=16 or k=32.
3. **Control for non-determinism.** Use temperature > 0 (matching your production setting) to capture the natural variance of the model. Log the exact configuration used.
4. **Run each task k times independently.** Do not carry over context between runs. Each run should be a fresh session with identical input.
5. **Score each run independently.** Use your standard evaluation criteria (Compare Meaning, Keyword Match, etc.) to score each of the k runs as pass or fail.
6. **Calculate metrics:**
   - **pass@1** = fraction of tasks where at least 1 run out of k succeeds (optimistic capability measure)
   - **pass@k** = fraction of tasks where all k runs succeed (consistency measure)
   - **pass^k** = (pass@1)^k estimated probability of k consecutive successes (theoretical reliability)
   - **variance per task** = standard deviation of scores across k runs (identifies high-variance tasks)
7. **Compare pass@1 and pass^k.** A large gap signals an agent that is capable but unreliable — the worst case for production.

### Anti-Pattern

> **Anti-Pattern: Reporting Only pass@1**
> Most benchmarks report pass@1 (best of k attempts), which measures peak capability. This dramatically overstates production readiness. An agent with high pass@1 may sound deployment-ready, but pass^k can be substantially lower, meaning users in multi-interaction sessions are likely to encounter at least one failure. Always report both metrics together.

### Evaluation Patterns

**Pattern A: pass@1 vs pass^k Gap Analysis**
Calculate both metrics for your full task set. If the gap is larger than 20 percentage points, your agent has a reliability problem that cannot be solved by prompt engineering alone — it requires either model changes, retry logic, or confidence-based routing.

**Pattern B: Task-Level Variance Identification**
For each task, calculate the variance across k runs. Sort tasks by variance. The highest-variance tasks are your reliability bottleneck — they pass sometimes and fail sometimes, making production behavior unpredictable. Investigate what makes these tasks different (ambiguity, complexity, edge cases).

**Pattern C: Reliability by Task Category**
Group tasks by type (simple lookup, multi-step reasoning, creative generation, etc.) and calculate pass^k per category. This reveals which task types are inherently unreliable for your agent. Use this to set per-category SLAs or to route unreliable categories to human agents.

**Pattern D: Temperature Sensitivity**
Run the full evaluation at multiple temperature settings (e.g., 0.0, 0.3, 0.7, 1.0). Plot pass^k vs temperature. This reveals the reliability-creativity tradeoff and helps you choose the optimal temperature for your production use case.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Customer service reliability | 30 common customer queries, each run k=8 times | pass^8 >= 50% for production readiness | Repeated Trial Execution + Statistical Analysis |
| 2 | Model comparison for reliability | Same 50 tasks on Model A vs Model B, k=8 each | Higher pass^8 model preferred even if pass@1 is lower | Repeated Trial Execution + Statistical Analysis |
| 3 | Task variance analysis | 50 tasks, k=8, sorted by per-task variance | High-variance tasks identified as reliability bottleneck | Repeated Trial Execution + Statistical Analysis |
| 4 | Temperature optimization | Same tasks at temperature 0.0, 0.3, 0.7, 1.0 | pass^8 peaks at optimal temperature per task type | Repeated Trial Execution + Statistical Analysis |
| 5 | Retry strategy justification | Same tasks with retry-on-failure (max 3 attempts) | P(at least 1 success in 3 tries) justifies retry cost | Repeated Trial Execution + Statistical Analysis |

### Tips

- **k=8 is the recommended minimum** for reliability measurement. Smaller k values produce unstable estimates with wide confidence intervals.
- **Always report confidence intervals** with your metrics. pass^8 of 40% +/- 15% is very different from pass^8 of 40% +/- 3%.
- **Use pass^k to set SLAs.** If your SLA requires 99% daily reliability across 50 interactions, you need pass^50 >= 0.99, which requires pass@1 >= 99.98%. This makes reliability requirements concrete.
- **Rerun after** model updates, prompt changes, or system configuration changes. Reliability is the first metric to degrade silently.
- **Target: pass^8 >= 50%** for general production deployment, pass^8 >= 80% for high-reliability applications.

---

## 5. Extended Session Coherence and Meltdown Detection

Does the agent maintain coherent performance over long sessions, or does it degrade suddenly?

### When to Use

Your agent handles long-running sessions — multi-turn conversations, extended document processing, or sequential task execution over many steps. You need to verify it maintains quality throughout the session and detect if and when coherence degrades. Notably, some research (e.g., Vending-Bench) suggests that agent "meltdowns" can be sudden rather than gradual and may not correlate with context window utilization — potentially representing a coherence failure rather than a memory limitation.

Use this scenario when:
- Your agent handles sessions with 20+ turns or steps
- Users report that the agent "loses the plot" partway through long interactions
- You are processing large documents or datasets that require sustained attention
- Your agent performs sequential tasks where later steps depend on earlier context
- You need to define session length limits or circuit breaker policies

> **Related scenarios:** For evaluating conversation quality in individual turns, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md). For error recovery evaluation, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). This scenario evaluates single-agent coherence degradation over time.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Score each response against ground truth at regular intervals throughout the session |
| Consistency Check (comparing current output against the agent's own prior statements for contradictions) | Verify the agent doesn't contradict its own earlier statements or outputs |
| Trajectory Match (In-Order) | Verify the agent follows a logical progression and doesn't loop or regress |
| Keyword Match (Absence) (verifying that specific unwanted terms or patterns do NOT appear in the output) | Detect hallucination markers or incoherent outputs that signal meltdown onset |

> **Tip:** Don't just measure average session quality — plot quality over time. A session with 95% average quality that has one catastrophic meltdown at step 47 is far more dangerous than a session with 85% average quality that's consistent throughout. The spike matters more than the average.

### Setup Steps

1. **Design extended session tasks.** Create tasks that require 30-100+ steps: long document analysis, multi-page form filling, extended research sessions, or sequential customer interactions. Each step should have an independently scorable output.
2. **Establish coherence checkpoints.** At regular intervals (every 5-10 steps), insert a probe question that tests whether the agent remembers and correctly references earlier context. For example, after 30 steps of document analysis: "What was the main finding from the first section?"
3. **Define meltdown indicators.** Catalog the specific failure modes that indicate coherence loss: repeating earlier outputs verbatim, contradicting previous statements, hallucinating information not in the input, losing track of the task structure, or producing incoherent text.
4. **Instrument quality scoring per step.** Score each step independently on a 1-5 scale covering: task relevance (is the output about the right thing?), factual accuracy (is the content correct?), and contextual coherence (does it build on previous steps appropriately?).
5. **Run sessions of varying lengths.** Test at 10, 25, 50, and 100+ steps. Compare quality curves to identify the degradation onset point.
6. **Log context utilization.** Track token count and context window utilization at each step. Compare against quality scores to verify whether degradation correlates with context limits (some research suggests it may not).

### Anti-Pattern

> **Anti-Pattern: Assuming Context Window = Coherence Limit**
> Teams commonly set session length limits based on context window size (e.g., "our model supports 128K tokens, so sessions up to 100K should be fine"). Vending-Bench experiments suggest that meltdowns can occur well before context limits and may not correlate with context utilization. Always empirically test coherence limits — don't derive them from spec sheets.

### Evaluation Patterns

**Pattern A: Quality-Over-Time Curve**
Plot step-by-step quality scores across the full session. Look for three patterns: (1) gradual decline (quality slowly decreases), (2) cliff drop (quality is stable then suddenly collapses), (3) oscillation (quality fluctuates unpredictably). Each pattern requires different mitigation — gradual decline needs session length limits, cliff drops need circuit breakers, oscillation needs investigation of specific failure triggers.

**Pattern B: Coherence Probe Accuracy**
Insert explicit coherence probes at regular intervals: ask the agent to recall, summarize, or reference earlier session content. Score probe responses for accuracy. Plot probe accuracy over time. The point where probe accuracy drops below 80% is your agent's effective coherence limit — regardless of what the context window allows.

**Pattern C: Meltdown Detection and Circuit Breaker Testing**
Define specific meltdown indicators (repetition, contradiction, hallucination, task loss) and test whether your monitoring system detects them. Measure detection latency (how many meltdown steps occur before detection?) and recovery (can the system recover by resetting context or escalating?). Target: detection within 2 steps of meltdown onset.

**Pattern D: Comparative Session Endurance**
Compare multiple models or configurations on the same extended session tasks. Plot quality-over-time curves for each. This reveals which models sustain coherence longest and whether techniques like summarization, context compression, or periodic context resets extend the coherence limit.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Long document analysis | 50-page document, analyze section by section with probes every 5 sections | Coherence probe accuracy remains above 80% through section 40+ | Compare Meaning + Consistency Check |
| 2 | Extended customer service | 40-turn conversation with escalating complexity | No contradictions with earlier turns; quality stays above baseline | Compare Meaning + Consistency Check |
| 3 | Sequential task marathon | 30 independent tasks in one session (no context reset) | Accuracy per task does not drop below baseline over the sequence | Compare Meaning + Consistency Check |
| 4 | Context probe at limits | 100-turn conversation with "recall turn 5" probes at turns 25, 50, 75, 100 | Probe accuracy above 80% at turn 50; declining curve expected | Compare Meaning + Consistency Check |
| 5 | Meltdown circuit breaker | Deliberately run sessions past known coherence limits | Meltdown detected within 2 steps of onset | Keyword Match (Absence) + Consistency Check |
| 6 | Recovery after session reset | At meltdown detection, summarize context and restart session | Quality recovers to within 10% of initial baseline after reset | Compare Meaning + Consistency Check |

### Tips

- **Plot quality over time for every extended session evaluation.** The curve shape tells you more than any aggregate metric.
- **Meltdowns are sudden**, not gradual. A session quality chart may show stable 4.5/5 scores for 40 steps, then drop to 1.0/5 at step 41. Design your monitoring to detect step-level drops, not just rolling averages.
- **Test circuit breaker mechanisms.** When coherence degrades, does your system detect it and take action (session reset, human handoff, context summarization)? A detected meltdown is manageable; an undetected one is a production incident.
- **Your agent's coherence limit is probably shorter than its context window.** Empirically measure it — don't assume.
- **Target: less than 10% quality degradation** over the first 80% of the intended session length. If quality drops more than 10% before the session is 80% complete, your sessions are too long for your agent.

---

## Cross-Scenario Integration

The five scenarios in this guide interact in important ways:

1. **Grounding (Scenario 1) is prerequisite to Conflict Detection (Scenario 2).** An agent that doesn't truly look at images cannot detect cross-modal conflicts. Run grounding evaluation first; only proceed to conflict detection if grounding scores exceed 80%.

2. **GUI Completion (Scenario 3) depends on Reliability (Scenario 4).** A GUI agent that completes tasks 70% of the time is useless for automation. Run reliability evaluation on GUI tasks to distinguish "can do it" from "can do it every time."

3. **Extended Sessions (Scenario 5) amplify all other failure modes.** An agent with 90% grounding accuracy on step 1 may have 60% grounding accuracy on step 50 of a long session. Test critical capabilities at session endpoints, not just session start.

4. **Reliability (Scenario 4) provides the production readiness gate.** Regardless of how well an agent scores on any single scenario run, pass^k determines whether it's ready for deployment. Use pass^k as the final gate after all other scenarios pass.

### Recommended Evaluation Order

```
1. Visual Grounding (MM-01) — Foundation: can the agent actually see?
2. Cross-Modal Consistency (MM-02) — Integration: can it reason across modalities?
3. GUI Task Completion (MM-03) — Application: can it act on visual understanding?
4. Reliability (MM-04) — Production gate: can it do all of the above consistently?
5. Extended Sessions (MM-05) — Endurance: can it sustain performance over time?
```
