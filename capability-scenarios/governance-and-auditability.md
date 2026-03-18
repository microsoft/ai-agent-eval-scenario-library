# Governance & Auditability Evaluation

> Scenarios for evaluating whether AI agent systems produce reproducible results, maintain complete audit trails, satisfy regulatory conformity assessments, demonstrate fairness across subgroups, and sustain continuous post-market monitoring. Governance evaluation is not about testing what the agent _does_ — it is about testing whether you can _prove_ what the agent did, why it did it, and that it will do the same thing tomorrow.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Why Governance & Auditability Evaluation Matters

The EU AI Act enters full enforcement in August 2026. High-risk AI systems — including agents used in employment, credit, healthcare, and law enforcement — must demonstrate reproducibility, traceability, and bias mitigation with documentary evidence. Failure to comply exposes organizations to fines up to €35 million or 7% of global revenue.

But governance evaluation is not just a European concern. NIST's AI Risk Management Framework (AI RMF), the White House Executive Order on AI, and sector-specific regulations (FDA, SEC, banking supervisors) all converge on the same core requirements:

- **Reproducibility.** Can you run the same evaluation twice and get the same result? If you cannot, you cannot audit. Non-determinism in LLM outputs makes this harder than it sounds — temperature, sampling, version drift, and infrastructure differences all introduce variance. EU AI Act Article 17 requires documented test conditions that produce verifiable results.
- **Audit trails.** Every agent decision must be traceable: what inputs did it receive, what reasoning did it follow, what output did it produce? EU AI Act Article 12 mandates automatic logging capabilities with 10-year retention for high-risk systems. Most agent frameworks produce logs, but few produce _auditable_ logs — tamper-resistant, complete, and machine-parseable.
- **Conformity assessment.** Before deployment, high-risk AI systems must pass a conformity assessment covering technical documentation, risk management, accuracy reporting, and post-market monitoring plans (Articles 9, 11, 15). Evaluation frameworks must map their outputs to these evidence requirements.
- **Fairness and bias.** Agents must perform equitably across demographic subgroups. Disparate impact is not just an ethical concern — it is a legal liability. EU AI Act Article 15 requires bias testing and documentation. NIST AI RMF's MEASURE function calls for disaggregated metrics.
- **Continuous monitoring.** Governance is not a one-time gate. EU AI Act Article 9 and systemic risk provisions require post-market surveillance: drift detection, incident reporting within 72 hours, and re-evaluation triggered by model updates. An evaluation framework that only runs at deployment time is insufficient.

The **Document-Verify-Audit** framework structures governance evaluation in three layers: _document_ what was tested and how, _verify_ that results are reproducible and audit trails are complete, then _audit_ the entire system against regulatory requirements to identify gaps before a regulator does.

> **Key references:** EU AI Act (Regulation 2024/1689), NIST AI RMF (AI 100-1), NIST Test, Evaluation, Verification, and Validation (TEVV) framework, Future of Privacy Forum Conformity Assessment Guide (2025), ISO/IEC 42001:2023 AI Management Systems standard.

---

## 1. Evaluation Reproducibility Verification

Can you run the same evaluation twice and prove you got the same result?

### When to Use

Your agent evaluation pipeline produces results that inform deployment decisions, regulatory filings, or stakeholder reports — and you need to demonstrate that those results are stable, not artifacts of randomness. This scenario is the foundation of all governance: if evaluations are not reproducible, no audit trail, conformity assessment, or bias analysis built on top of them is trustworthy.

Use this scenario when:
- Your organization must demonstrate evaluation reproducibility for regulatory compliance (EU AI Act, FDA, banking supervisors)
- You are comparing agent versions and need confidence that performance differences are real, not noise
- Your evaluation pipeline runs across different environments (dev, CI/CD, cloud regions) and you need parity
- You use LLM-as-judge graders and need to verify their scoring consistency
- Stakeholders have questioned whether evaluation results are reliable

> **Related scenarios:** For testing whether the agent's _outputs_ are consistent, see [Regression Testing](regression-testing.md). For evaluating grader model reliability, see the triage playbook's grader model selection guide. This scenario specifically evaluates whether the _evaluation process itself_ is reproducible.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify that agent outputs under identical conditions are semantically equivalent across runs |
| Exact Match | Confirm that deterministic pipeline components (tool calls, data retrieval) produce identical results |
| Multi-Turn Trajectory Match | Verify that multi-step evaluations produce the same step sequence and scoring |

> **Tip:** The single most impactful action for reproducibility is **pinning everything** — model version, temperature (set to 0 for evaluation), random seeds, prompt templates, grader model versions, and evaluation dataset versions. Then run the same evaluation 10 times and measure variance. If variance exceeds your tolerance threshold, you have a reproducibility problem to diagnose.

### Setup Steps

1. **Pin all evaluation parameters.** Document and freeze: model identifier and version, inference parameters (temperature=0, top_p=1, seed if supported), prompt template version (hash), evaluation dataset version (hash), grader model version, and infrastructure specification (GPU type, runtime version).
2. **Create a reproducibility test suite.** Select 50-100 evaluation cases that represent your typical evaluation workload. These become your reproducibility benchmark — never change them without versioning.
3. **Run the evaluation 10 times.** Execute the full evaluation pipeline 10 times with identical parameters. Record every result with timestamps.
4. **Measure variance.** For each evaluation case, calculate: (a) exact match rate across runs, (b) semantic similarity of outputs across runs, (c) score variance for graded evaluations. Aggregate into an overall reproducibility score.
5. **Test cross-environment parity.** Run the same evaluation on at least two different environments (e.g., local vs. CI/CD, or two cloud regions). Compare results. Any divergence indicates environment-dependent behavior that undermines reproducibility.
6. **Document everything.** Produce a reproducibility report that maps to EU AI Act Article 17 requirements: test conditions, results, variance analysis, and a statement of reproducibility with confidence intervals.

### Anti-Pattern

> **Anti-Pattern: Relying on LLM-as-Judge Without Calibrating the Judge**
> If your evaluation uses an LLM grader (e.g., GPT-4 scoring relevance), the grader itself introduces non-determinism. Running the same agent output through the same grader twice may produce different scores. Always measure grader agreement (intra-rater reliability) separately from agent evaluation. If grader agreement is below 90%, your evaluation reproducibility is capped by your grader's inconsistency, not by your agent's behavior.

### Evaluation Patterns

**Pattern A: Run-to-Run Variance**
Execute the identical evaluation 10 times. For each test case, calculate the standard deviation of scores across runs. Aggregate into a coefficient of variation (CV). Target: CV < 5% for the overall evaluation suite. Flag any individual test cases with CV > 15% for investigation — these are your reproducibility weak points.

**Pattern B: Environment Parity**
Run the evaluation on two or more environments with identical parameters. Calculate the absolute difference in aggregate scores between environments. Target: < 2% absolute difference. If environments diverge, investigate infrastructure differences: GPU type, quantization, batching behavior, or API version drift.

**Pattern C: Grader Consistency**
For LLM-graded evaluations, submit the same 100 agent outputs to the grader 5 times. Calculate Cohen's kappa or Krippendorff's alpha for inter-run agreement. Target: κ > 0.8 (substantial agreement). If below threshold, consider: reducing temperature, switching to a more deterministic grader, or using majority voting across 3+ grader runs.

**Pattern D: Version Drift Detection**
Maintain a frozen "golden" evaluation dataset with known-good results. After any model, prompt, or infrastructure update, re-run the golden evaluation and compare to the baseline. Use a paired statistical test (McNemar's or Wilcoxon signed-rank) to determine whether changes are statistically significant. This catches silent regression introduced by provider-side model updates.

### Practical Examples

**Example 1: Regulatory Filing Reproducibility**
A financial services agent processes loan applications. The regulator asks: "Show me that your evaluation produces the same results when run again." The team pins all parameters, runs their 200-case evaluation suite 10 times, and produces a report showing CV of 2.3% with no individual case exceeding 10% variance. The grader consistency check shows κ = 0.87. This evidence satisfies the reproducibility requirement.

**Example 2: CI/CD Pipeline Parity**
A healthcare agent's evaluation runs in both a developer's local environment and the CI/CD pipeline. Results diverge by 8%. Investigation reveals the CI/CD environment uses a different GPU type, causing different quantization behavior. After standardizing the inference environment, divergence drops to 1.2%.

---

## 2. Decision Audit Trail Completeness

Does the agent produce tamper-resistant logs that trace every decision from input to output?

### When to Use

Your agent makes decisions that affect people — approving applications, recommending treatments, routing support cases, flagging content — and you need to prove, after the fact, exactly what happened and why. This is not just logging; it is producing evidence that would survive a regulatory audit or legal discovery process.

Use this scenario when:
- Your agent operates in a regulated industry where decision traceability is mandated (EU AI Act Article 12, FDA, banking)
- You need to investigate agent failures or complaints and reconstruct the decision chain
- Your organization requires tamper-resistant logs for compliance (SOC 2, ISO 27001, EU AI Act)
- You need to demonstrate that no agent decision was made without a traceable input-reasoning-output chain
- Your retention requirements exceed standard log rotation (EU AI Act mandates 10 years for high-risk systems)

> **Related scenarios:** For evaluating the agent's reasoning quality, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For tool call tracing, see [Tool & Connector Invocations](tool-and-connector-invocations.md). This scenario evaluates whether the logging and audit infrastructure captures enough information to reconstruct any decision.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify audit logs contain required fields: timestamp, input hash, model version, reasoning trace, output, confidence |
| Trajectory Match (In-Order) | Confirm the logged decision sequence matches the actual execution sequence |
| Custom Grader | Evaluate log completeness, tamper resistance, and traceability against a regulatory checklist |

> **Tip:** The acid test for audit trail completeness is the **reconstruction test**: give an auditor only the logs (no access to the agent or its code) and ask them to reconstruct exactly what happened for a random sample of decisions. If they cannot, the audit trail is incomplete.

### Setup Steps

1. **Define the audit schema.** Specify required fields for every logged decision: unique decision ID, timestamp (UTC, millisecond precision), input data hash, full input text, model identifier and version, prompt template version, reasoning trace (chain-of-thought or tool call sequence), output, confidence score, and any human override flags.
2. **Create test transactions.** Design 20-30 agent interactions that exercise different decision paths: simple decisions, multi-step tool use, escalations, edge cases, and error conditions.
3. **Run test transactions.** Execute all test interactions through the agent with audit logging enabled.
4. **Validate log completeness.** For each test transaction, verify that every required field is present in the audit log. Calculate completeness rate: (fields present) / (fields required) across all transactions.
5. **Test traceability.** For each logged decision, verify that the audit trail links input → reasoning → output in an unbroken chain. Attempt to reconstruct the decision from the log alone.
6. **Test tamper resistance.** Attempt to modify a log entry and verify that the modification is detected (e.g., via checksums, append-only storage, or blockchain anchoring).
7. **Test retention compliance.** Verify that logs are stored in a retention-compliant system and that retrieval works for entries across the full retention period.

### Anti-Pattern

> **Anti-Pattern: Logging Outputs Without Reasoning**
> Many agent systems log the final output but not the reasoning chain that produced it. An audit trail that says "the agent approved the application" without recording _why_ (which factors it considered, which tools it called, what intermediate results it saw) is useless for regulatory purposes. EU AI Act Article 12 requires that logs enable "the tracing back of the AI system's operations" — outputs alone do not satisfy this.

### Evaluation Patterns

**Pattern A: Field Completeness Audit**
For each of 30 test transactions, check all required fields against the audit schema. Calculate: (total fields present) / (total fields required). Target: 100% for mandatory fields, 95%+ for recommended fields. Any missing mandatory field is a compliance gap. Report results as a coverage matrix showing which fields are consistently captured and which are missing.

**Pattern B: Decision Reconstruction Test**
Select 10 random decisions from the audit log. Provide only the log entries (no agent access) to an evaluator. Ask them to answer: (a) What was the input? (b) What reasoning did the agent follow? (c) What was the output? (d) What model version produced this? Score: percentage of questions the evaluator can answer correctly from the log alone. Target: 100%.

**Pattern C: Tamper Detection**
Deliberately modify 5 log entries (change an output, alter a timestamp, remove a reasoning step). Run the integrity verification system. Measure: (a) detection rate — how many modifications were caught, (b) false positive rate — how many unmodified entries were flagged. Target: 100% detection, 0% false positives.

**Pattern D: Retention and Retrieval**
Query the audit system for decisions made at various time points (1 day, 1 month, 1 year, approaching retention limit). Measure retrieval latency and completeness. Verify that no log entries have been lost to rotation, compaction, or storage failures. For EU AI Act compliance, verify that entries from day 1 are still retrievable and complete after the retention period.

### Practical Examples

**Example 1: Complaint Investigation**
A customer complains that an insurance agent incorrectly denied their claim. The compliance team queries the audit trail by decision ID and retrieves: the customer's input, the agent's tool calls to the policy database, the reasoning chain that led to the denial, the confidence score, and the model version. They identify that the agent misinterpreted a policy clause. The complete audit trail enables both the complaint resolution and a targeted fix.

**Example 2: Regulatory Spot Check**
A banking regulator requests audit records for 50 randomly selected credit decisions from the past 6 months. The team retrieves all 50 within 4 hours, each containing the full input-reasoning-output chain. The regulator verifies that every decision is traceable and that the model version is documented. No gaps are found.

---

## 3. Conformity Assessment Readiness

Can your evaluation evidence satisfy a regulatory conformity assessment without scrambling?

### When to Use

Your AI agent is classified as high-risk under the EU AI Act (or equivalent regulation), and you need to verify that your evaluation framework produces the evidence a conformity assessment requires. This scenario does not evaluate the agent itself — it evaluates whether your evaluation _process_ produces outputs that map cleanly to regulatory requirements.

Use this scenario when:
- Your agent falls under EU AI Act high-risk classification (Annex III: employment, credit, healthcare, law enforcement, education, migration, critical infrastructure)
- You are preparing for a conformity assessment by a notified body or through internal assessment
- You need to identify gaps between your current evaluation outputs and regulatory evidence requirements
- You want to verify that your technical documentation meets Articles 9 (risk management), 11 (technical documentation), and 15 (accuracy, robustness, cybersecurity)
- You are building an evaluation pipeline that must be "audit-ready" from day one

> **Related scenarios:** For reproducibility verification, see Scenario 1 above. For audit trail completeness, see Scenario 2. For bias testing, see Scenario 4. This scenario integrates all governance capabilities into a single readiness assessment.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom Grader | Score documentation completeness against the EU AI Act conformity assessment checklist |
| Keyword Match (All) | Verify that evaluation reports contain all required regulatory terminology and section headings |
| Compare Meaning | Validate that evaluation descriptions accurately reflect the actual testing methodology |

> **Tip:** The Future of Privacy Forum (FPF) published a detailed conformity assessment mapping guide in 2025. Use it as your checklist. Map each FPF requirement to a specific evaluation output and flag any requirement that has no corresponding evidence.

### Setup Steps

1. **Build the requirements matrix.** Create a spreadsheet or structured document listing every conformity assessment requirement from EU AI Act Articles 9 (risk management), 11 (technical documentation), 15 (accuracy/robustness/cybersecurity), and 17 (quality management). Add columns for: evidence type required, current evaluation output that provides evidence, gap flag.
2. **Map evaluation outputs.** For each requirement, identify which current evaluation output (report, metric, log, document) provides the required evidence. If no output exists, flag it as a gap.
3. **Run a mock assessment.** Have an internal assessor (or external consultant) walk through the conformity assessment process using only your current evaluation outputs. Record every point where evidence is missing, insufficient, or ambiguous.
4. **Score readiness.** Calculate: (requirements with adequate evidence) / (total requirements). Break down by article: risk management readiness, documentation readiness, accuracy readiness, robustness readiness.
5. **Produce a gap report.** For each gap, specify: the requirement, why current evidence is insufficient, what evaluation capability is needed, estimated effort to close the gap, and priority based on regulatory risk.
6. **Validate evidence mapping.** For each mapped requirement, verify that the evaluation output actually provides the evidence it claims to. A metric that exists but does not measure what the regulation requires is worse than a known gap.

### Anti-Pattern

> **Anti-Pattern: Treating Conformity as a Documentation Exercise**
> The most common failure mode is producing a conformity assessment document that describes what you _plan_ to do rather than providing evidence of what you _have done_. Regulators and notified bodies require verifiable evidence: actual evaluation results, actual audit logs, actual bias reports — not plans, policies, or aspirational descriptions. If your conformity evidence consists of policy documents without corresponding evaluation data, you will fail the assessment.

### Evaluation Patterns

**Pattern A: Requirements Coverage Matrix**
Map all EU AI Act conformity assessment requirements to evaluation outputs. Score: percentage of requirements with adequate evidence. Breakdown by article:
- Article 9 (Risk Management): risk identification, mitigation testing, residual risk documentation
- Article 11 (Technical Documentation): system description, design specification, evaluation methodology, evaluation results
- Article 15 (Accuracy): performance metrics with confidence intervals, known limitations, bias testing results
- Article 17 (Quality Management): version control, change management, evaluation reproducibility
Target: 90%+ coverage overall, no article below 75%.

**Pattern B: Mock Assessment Simulation**
Conduct a full mock conformity assessment. Record time to retrieve each piece of evidence. Score: (a) evidence availability — percentage of requests answered completely, (b) retrieval time — average time to produce evidence, (c) evidence quality — assessor's rating of evidence sufficiency on a 1-5 scale. Targets: 95%+ availability, < 30 minutes average retrieval, > 4.0 average quality rating.

**Pattern C: Evidence Freshness and Currency**
For each mapped evidence item, check: (a) when it was last updated, (b) whether it reflects the current model version, (c) whether it reflects the current evaluation methodology. Evidence that refers to a previous model version or deprecated evaluation approach is stale and may not satisfy a conformity assessment. Score: percentage of evidence items that are current. Target: 100%.

**Pattern D: Cross-Reference Integrity**
Verify that evidence items reference each other consistently. The risk management document should reference the same evaluation results cited in the technical documentation. The bias report should reference the same dataset used in accuracy testing. Any inconsistency between documents is an audit red flag. Score: number of cross-reference inconsistencies found. Target: 0.

### Practical Examples

**Example 1: Pre-Assessment Gap Analysis**
A healthcare company deploying an AI triage agent runs the conformity assessment readiness evaluation. Results: Article 9 (risk management) at 82% — missing formal residual risk documentation. Article 11 (technical documentation) at 91%. Article 15 (accuracy) at 68% — no disaggregated performance metrics by patient demographic. Article 17 (quality management) at 73% — no evaluation reproducibility evidence. The gap report prioritizes bias testing (Article 15) as the highest-risk gap and the team closes it within 4 weeks.

**Example 2: Continuous Readiness Monitoring**
A financial services firm embeds the conformity readiness check in their CI/CD pipeline. Every model update triggers a readiness score calculation. When a prompt template change causes the Article 11 documentation score to drop from 93% to 71% (because the template version in the documentation no longer matches production), an alert fires and the deployment is held until documentation is updated.

---

## 4. Bias and Fairness Audit

Does the agent perform equitably across demographic subgroups?

### When to Use

Your agent makes decisions that affect people differently based on demographic characteristics — and you need to measure, document, and mitigate any disparate impact. This is not optional sentiment: EU AI Act Article 15 requires bias testing for high-risk systems, NIST AI RMF's MEASURE function calls for disaggregated metrics, and the EEOC has signaled that AI-driven employment decisions are subject to existing disparate impact law.

Use this scenario when:
- Your agent is used in hiring, lending, insurance, healthcare, education, or other contexts where demographic fairness is legally required
- You need to produce bias testing documentation for regulatory compliance
- You want to measure whether model updates or prompt changes introduce new biases
- You are comparing agent configurations and need fairness as a selection criterion alongside accuracy
- Your organization has fairness commitments and needs quantitative evidence of compliance

> **Related scenarios:** For accuracy testing, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For compliance-specific content evaluation, see [Compliance & Verbatim Content](compliance-and-verbatim-content.md). This scenario specifically evaluates demographic fairness and disparate impact in agent decision-making.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom Grader | Calculate fairness metrics (demographic parity, equalized odds) across subgroups |
| Compare Meaning | Verify that explanations provided to different demographic groups are equally informative |
| Exact Match | Confirm that identical qualifications produce identical outcomes regardless of demographic indicators |

> **Tip:** The most common bias evaluation mistake is testing only for bias in final outcomes without examining the intermediate reasoning. An agent that produces fair outcomes through biased reasoning (e.g., applying stricter scrutiny to one group but more leniency on other factors) is one prompt change away from unfair outcomes. Audit the reasoning chain, not just the result.

### Setup Steps

1. **Define protected attributes and subgroups.** Based on your jurisdiction and use case, identify protected attributes (gender, race, age, disability, etc.) and the specific subgroups you will test. Ensure subgroups are large enough for statistical significance (minimum 30 cases per subgroup, ideally 100+).
2. **Create matched test cases.** For each subgroup, create test cases that are identical in all decision-relevant attributes but differ in demographic indicators. For example, identical résumés with names associated with different demographics, or identical loan applications with different zip codes.
3. **Create intersectional test cases.** Test combinations of protected attributes (e.g., gender × race), not just individual attributes in isolation. Intersectional bias is often invisible when attributes are tested separately.
4. **Run disaggregated evaluation.** Execute the standard evaluation suite and break down results by subgroup. Calculate both overall metrics and per-subgroup metrics.
5. **Calculate fairness metrics.** For each pair of subgroups, calculate: demographic parity ratio (selection rate ratio), equalized odds difference, predictive parity, and calibration across groups. Use the appropriate metric for your context — demographic parity for initial screening, equalized odds for classification tasks.
6. **Document findings.** Produce a bias report that includes: methodology, subgroup definitions, sample sizes, metrics with confidence intervals, identified disparities, and planned mitigations. This report becomes part of your conformity assessment evidence.

### Anti-Pattern

> **Anti-Pattern: Testing Fairness Only on Synthetic Data**
> Synthetic matched test cases (e.g., swapping names on résumés) reveal direct bias but miss systemic bias encoded in the real data distribution. An agent might show no bias on synthetic pairs but still produce disparate impact on real-world data because its learned correlations map non-protected features (zip code, vocabulary, school name) to protected attributes. Always supplement synthetic testing with disaggregated analysis of real evaluation data.

### Evaluation Patterns

**Pattern A: Demographic Parity Test**
For a binary decision (approve/reject), calculate the selection rate for each demographic subgroup. Compute the demographic parity ratio: (lowest selection rate) / (highest selection rate). The four-fifths rule (used by the EEOC) flags ratios below 0.8 as evidence of adverse impact. Target: ratio ≥ 0.8 for all subgroup comparisons. Report: selection rates per subgroup with 95% confidence intervals.

**Pattern B: Equalized Odds Analysis**
For tasks where ground truth is available, calculate true positive rate and false positive rate per subgroup. Equalized odds requires that both TPR and FPR are equal across groups. Calculate the maximum absolute difference in TPR and FPR between any two subgroups. Target: difference < 0.05. If TPR differs significantly, the agent is systematically underperforming for certain groups. If FPR differs, the agent is systematically over-flagging certain groups.

**Pattern C: Counterfactual Fairness**
For each test case, swap demographic indicators and re-run. The counterfactual fairness score is the percentage of cases where the decision remains the same after the swap. Target: 95%+. Cases where the decision changes constitute direct evidence of bias in the agent's reasoning. Log these cases for root cause analysis.

**Pattern D: Intersectional Bias Detection**
Test all pairwise intersections of protected attributes (e.g., gender × race, age × disability). For each intersection, calculate the same fairness metrics as Pattern A. Report which intersections show the largest disparities. Intersectional analysis often reveals bias invisible in single-attribute tests — for example, no bias against women overall, and no bias against a racial minority overall, but significant bias against women of that racial minority.

### Practical Examples

**Example 1: Hiring Agent Fairness Audit**
A recruiting agent shortlists candidates from résumés. The bias audit creates 200 matched résumé pairs (identical qualifications, names swapped to signal different demographics). Results: counterfactual fairness score of 91% for gender, but 76% for race — below the four-fifths threshold. Investigation reveals the agent gives higher scores to candidates with university names associated with certain demographics. The team adds a university-blinding preprocessing step and re-evaluates, achieving 94%.

**Example 2: Lending Agent Intersectional Analysis**
A credit decisioning agent shows demographic parity across gender (ratio 0.87) and across age groups (ratio 0.83) when tested independently. However, intersectional analysis reveals that young women have a selection rate ratio of 0.71 compared to older men. The single-attribute tests masked this intersectional disparity. The team investigates and finds the agent penalizes short credit histories (correlated with youth) more heavily when combined with certain employment patterns (correlated with gender).

---

## 5. Continuous Post-Market Monitoring

Does your evaluation continue running after deployment, or does it stop at the gate?

### When to Use

Your agent is deployed in production and you need to verify that your evaluation framework operates continuously — detecting drift, triggering re-evaluation when conditions change, reporting incidents within regulatory timelines, and linking model updates to new evaluation runs. EU AI Act Article 9 requires post-market surveillance for high-risk systems. Systemic risk provisions require serious incident reporting within 72 hours.

Use this scenario when:
- Your agent is deployed in production and subject to post-market surveillance requirements
- You need to detect when agent performance degrades after deployment (distribution drift, model degradation)
- You need to verify that model or prompt updates trigger automatic re-evaluation
- Your organization must report serious incidents within regulatory timelines (72 hours under EU AI Act)
- You want to ensure that the evaluation framework you built for deployment doesn't go stale

> **Related scenarios:** For production evaluation methodology, see the production monitoring scenarios. For regression testing, see [Regression Testing](regression-testing.md). This scenario specifically evaluates whether your governance and monitoring infrastructure operates continuously and meets regulatory post-market surveillance requirements.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Custom Grader | Score monitoring system coverage, alert latency, and incident response compliance |
| Exact Match | Verify that model updates trigger re-evaluation runs automatically |
| Keyword Match (All) | Confirm incident reports contain all required fields within required timelines |

> **Tip:** The best test of continuous monitoring is the **chaos test**: deliberately inject a performance degradation (swap in a worse model, corrupt a data source, introduce a prompt regression) and measure how long it takes for your monitoring to detect it, alert the right people, and trigger a re-evaluation. If the answer is "we found out when a customer complained," your monitoring is not continuous.

### Setup Steps

1. **Inventory monitoring signals.** List all signals your evaluation framework monitors in production: accuracy metrics, latency, error rates, user feedback, drift indicators, safety flags. For each signal, document: measurement frequency, alerting threshold, and response procedure.
2. **Map to regulatory requirements.** For each EU AI Act post-market surveillance requirement, identify which monitoring signal provides evidence: drift detection (Article 9), incident reporting (systemic risk provisions), change management (Article 17), and ongoing accuracy monitoring (Article 15).
3. **Design chaos tests.** Create controlled degradation scenarios: (a) model performance drop — swap the production model for a weaker version, (b) data drift — change the input distribution, (c) prompt regression — introduce a subtle prompt error, (d) tool failure — disable a key tool or data source.
4. **Execute chaos tests.** Run each degradation scenario and measure: detection latency (time from degradation to alert), false negative rate (degradations that were not detected), alert routing accuracy (did the right team get notified?), and re-evaluation trigger (was a new evaluation automatically launched?).
5. **Test incident reporting.** Simulate a serious incident (e.g., the agent produces a harmful output). Measure: time from incident to report generation, report completeness (all required fields), and escalation path accuracy.
6. **Test change management linkage.** Perform a model update or prompt change. Verify that the change automatically triggers: a new evaluation run, a comparison against the previous baseline, an update to conformity documentation, and an audit log entry linking the change to the new evaluation.

### Anti-Pattern

> **Anti-Pattern: Monitoring Only Aggregate Metrics**
> Monitoring overall accuracy at 95% may mask that accuracy for a specific subgroup has dropped to 60%, or that a specific task type has completely broken. Always monitor disaggregated metrics — by subgroup, by task type, by input source. The EU AI Act requires that post-market monitoring covers the full scope of the system's intended purpose, not just an average.

### Evaluation Patterns

**Pattern A: Drift Detection Latency**
Inject a controlled distribution shift (change 20% of input patterns to out-of-distribution examples). Measure time from injection to alert. Target: < 1 hour for high-risk systems, < 24 hours for standard systems. Also measure: false negative rate (injections not detected) and false positive rate (alerts without actual drift). A monitoring system that misses drift is dangerous; one that alerts constantly is ignored.

**Pattern B: Change-Evaluation Linkage**
Perform 5 different types of changes: model version update, prompt template change, tool configuration change, data source update, and infrastructure change. For each, verify: (a) the change was logged in the audit trail, (b) a re-evaluation was automatically triggered, (c) results were compared to the baseline, (d) conformity documentation was flagged for update. Score: percentage of changes with complete linkage. Target: 100%.

**Pattern C: Incident Response Timing**
Simulate 5 serious incidents of varying severity. For each, measure: (a) time from incident to detection, (b) time from detection to internal notification, (c) time from detection to regulatory report generation, (d) report completeness. EU AI Act requires reporting within 72 hours for serious incidents. Target: < 4 hours from incident to completed report (provides 68 hours of buffer for review and submission).

**Pattern D: Surveillance Coverage Audit**
Map all production monitoring signals against the full scope of the agent's intended purpose. Identify blind spots — task types, user segments, input sources, or failure modes that are not monitored. Score: percentage of intended-purpose scope with active monitoring. Target: 95%+. Any unmonitored scope is a regulatory risk — a regulator can ask "how do you know it's working for [X]?" and you need an answer backed by data.

### Practical Examples

**Example 1: Model Update Chaos Test**
A customer service agent receives a model update (GPT-4o → GPT-4o-mini to reduce costs). The continuous monitoring system detects a 12% accuracy drop on complex multi-step queries within 45 minutes. An alert fires, a re-evaluation is automatically triggered, and the results show the new model fails the regression threshold on 3 of 8 scenario categories. The deployment is automatically rolled back pending review. Without continuous monitoring, this regression would have been discovered through customer complaints days later.

**Example 2: Incident Reporting Pipeline**
A healthcare triage agent produces a recommendation that contradicts clinical guidelines. The safety monitoring system flags the output within 2 minutes (keyword-based safety filter). An incident report is auto-generated with: the input, the agent's output, the safety rule violated, the model version, and the audit trail. The report reaches the compliance team within 15 minutes. They review, confirm the severity, and submit to the regulatory authority within 6 hours — well within the 72-hour requirement.

---

## Summary

| # | Scenario | Key Question | Primary Metrics | Regulatory Mapping |
|---|----------|-------------|-----------------|-------------------|
| 1 | Evaluation Reproducibility | Same eval, same result? | Coefficient of variation, environment parity, grader consistency | EU AI Act Art. 17, NIST TEVV |
| 2 | Decision Audit Trail | Can you reconstruct any decision? | Field completeness, reconstruction success, tamper detection | EU AI Act Art. 12 |
| 3 | Conformity Assessment Readiness | Evidence gaps before the regulator finds them? | Requirements coverage, retrieval time, evidence freshness | EU AI Act Arts. 9/11/15/17 |
| 4 | Bias and Fairness Audit | Equitable across subgroups? | Demographic parity ratio, equalized odds, counterfactual fairness | EU AI Act Art. 15, NIST AI RMF |
| 5 | Continuous Post-Market Monitoring | Evaluation still running after deployment? | Drift detection latency, change-eval linkage, incident response time | EU AI Act Art. 9, systemic risk |

The **Document-Verify-Audit** framework connects these five scenarios into a governance system: Scenarios 1-2 establish the _documentation_ foundation (reproducible results and complete audit trails), Scenario 3 _verifies_ that your evidence maps to regulatory requirements, Scenario 4 _audits_ for fairness, and Scenario 5 ensures governance is _continuous_, not a one-time exercise.

> **Next steps for implementers:** Start with Scenario 1 (reproducibility) — it is the foundation that makes everything else trustworthy. Then build Scenario 2 (audit trails) to create the evidence base. Scenario 3 (conformity readiness) integrates the first two with regulatory requirements. Add Scenario 4 (bias) for fairness compliance. Finally, Scenario 5 (continuous monitoring) ensures none of this atrophies after deployment. Budget 6-8 weeks for a complete governance evaluation framework for a high-risk system.
