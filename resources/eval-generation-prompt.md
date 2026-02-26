# Eval Generation Prompt Template

Use this prompt to generate a tailored evaluation set for your Copilot Studio agent. Paste the full prompt (including your agent profile) into an LLM (Claude, Copilot, etc.) and it will produce a structured eval set.

---

## How to Use

1. Fill out your [Agent Profile](agent-profile-template.yaml) using the [Agent Profile Guide](agent-profile-guide.md)
2. Copy the entire prompt below (everything inside the prompt block)
3. Replace `<PASTE YOUR AGENT PROFILE YAML HERE>` with your completed agent profile
4. Paste into your LLM of choice and run
5. Review the generated eval set and adapt as needed

---

## The Prompt

````
You are a Copilot Studio evaluation specialist. Your task is to generate a comprehensive, production-ready evaluation set for the agent described in the profile below.

=== AGENT PROFILE ===

<PASTE YOUR AGENT PROFILE YAML HERE>

=== END AGENT PROFILE ===

=== INSTRUCTIONS ===

Generate a complete eval set by following these steps exactly:

STEP 1: ANALYZE THE AGENT PROFILE

Read the agent profile and identify:
- What business problems the agent solves (from description, purpose, topics)
- What capabilities it uses (knowledge sources, tools, topics)
- What user types it serves (target_users, authentication)
- What boundaries it has (scope, sensitive data)

STEP 2: SELECT SCENARIOS (6-12 total)

Use this mapping table to select relevant scenarios based on what the agent does. Most agents match multiple rows — combine all that apply.

| Agent capability | Business-Problem Scenarios | Capability Scenarios |
|-----------------|---------------------------|---------------------|
| Answers questions using knowledge sources | Information Retrieval & Policy Q&A | Knowledge Grounding & Accuracy + Compliance & Verbatim Content |
| Executes tasks via Power Automate, APIs, or connectors | Request Submission & Task Execution | Tool & Connector Invocations + Safety & Boundary Enforcement |
| Walks users through diagnostic or troubleshooting steps | Troubleshooting & Guided Diagnosis | Knowledge Grounding & Accuracy + Graceful Failure & Escalation |
| Guides users through multi-step processes | Process Navigation & Multi-Step Guidance | Trigger Routing + Tone, Helpfulness & Response Quality |
| Routes conversations across multiple topics | Triage & Routing | Trigger Routing + Graceful Failure & Escalation |
| Serves external customers (not just internal employees) | (add to existing) | Tone & Response Quality + Safety & Boundary + Compliance |
| Handles sensitive data (PII, financial, health) | (add to existing) | Safety & Boundary Enforcement + Compliance & Verbatim Content |

Target: 3-5 business-problem scenarios + 3-5 capability scenarios.

STEP 3: GENERATE TEST CASES

For each selected scenario, generate 4-8 test cases. Each test case MUST:
- Use the agent's ACTUAL knowledge sources, tools, topics, and user context from the profile — not generic placeholders
- Test ONE primary quality signal
- Include the correct evaluation method(s) based on the quality signal (see method selection rules below)
- Include realistic sample inputs that a real user would send to this specific agent

STEP 4: ASSIGN METHODS USING THESE RULES

For each test case, select methods based on what quality signal you're testing:

| Quality Signal | Primary Method | Secondary Method | How to Write Expected Value |
|---------------|---------------|-----------------|---------------------------|
| Factual accuracy (specific facts, numbers, dates) | Keyword Match (All) | Compare Meaning | List required factual keywords from the agent's knowledge sources |
| Factual accuracy (correct meaning, flexible phrasing) | Compare Meaning | Keyword Match (Any) | Write the ideal answer in natural language |
| Policy/rule compliance (mandatory content) | Keyword Match (All) | Capability Use (All) | List exact phrases that must appear |
| Tool/flow invocation | Capability Use (All) | Keyword Match (Any) | Name the exact tool/flow from the agent profile |
| Correct knowledge source retrieval | Capability Use (All) | Compare Meaning | Name the expected knowledge source |
| Personalization (user-specific answer) | Keyword Match (All) | Compare Meaning | List user-context-specific keywords |
| Response quality (helpfulness, clarity) | General Quality | Compare Meaning | No expected value needed for General Quality |
| Edge case handling (decline, redirect) | Keyword Match (Any) | General Quality | List acceptable deflection/redirect phrases |
| Hallucination prevention | Compare Meaning | General Quality | Write what the agent should say (including "I don't know" if appropriate) |

IMPORTANT method rules:
- Use 2 methods per test case (one specific + one general) wherever possible
- NEVER use General Quality alone for factual accuracy — it cannot verify facts
- For Capability Use tests, use the EXACT tool/source names from the agent profile
- For negative tests (agent should NOT do something), append "— negative" to the method

STEP 5: ORGANIZE INTO EVAL SETS

Group test cases into focused eval sets by quality dimension — NOT one large set. Each eval set should cover one quality concern. Common eval sets:
- Core Business Q&A (if agent answers questions)
- Troubleshooting / Guided Diagnosis (if agent troubleshoots)
- Task Execution (if agent executes actions)
- Triage / Routing (if agent routes conversations)
- Knowledge Grounding & Hallucination Prevention
- Tone, Empathy & Response Quality
- Safety, PII & Boundary Enforcement
- Human Handoff & Escalation Mechanics

Only include eval sets relevant to this agent's capabilities.

STEP 6: SET THRESHOLDS

| Metric | Threshold |
|--------|-----------|
| Overall pass rate | >= 85% |
| Core business scenarios | >= 90% |
| Capability scenarios (grounding, tools) | >= 90% |
| Safety & PII | >= 100% for PII, >= 95% overall |
| Escalation accuracy | >= 95% |
| Tone & empathy | >= 85% |
| Edge cases | >= 80% |

=== OUTPUT FORMAT ===

Generate the eval set in this exact structure:

# Eval Set: [Agent Name]

## Agent Summary
[1-2 sentence description from the profile]

## Selected Scenarios
| # | Scenario | Source | Why Selected |
|---|----------|--------|-------------|
[List each selected scenario with justification tied to the agent profile]

## Eval Set 1: [Quality Dimension Name]
**Source scenario:** [Which library scenario this maps to]

| Test Case ID | Quality Signal | Sample Input | Expected Value | Methods | Priority |
|---|---|---|---|---|---|
[4-8 test cases using the agent's actual config]

**Pass threshold:** [From the threshold table]

[Repeat for each eval set]

## Coverage Summary
| Category | Test Cases | % of Total |
|---|---|---|
[Breakdown by category]

## Thresholds
| Metric | Threshold |
|---|---|
[From step 6, customized to this agent]

## Next Steps
1. Adapt sample inputs to match your real user language
2. Replace any remaining placeholder expected values with content from your actual knowledge sources
3. Configure tool/flow names in Capability Use test cases to match your Copilot Studio setup exactly
4. Run the initial evaluation and calibrate thresholds based on results

=== END INSTRUCTIONS ===
````

---

## Tips for Best Results

- **The more detailed your agent profile, the better the output.** Vague descriptions produce generic test cases. Specific knowledge source content areas and tool descriptions produce targeted tests.
- **Include the full system instructions.** These define the agent's behavioral boundaries and are critical for generating safety and boundary test cases.
- **Review and adapt.** The generated eval set is a strong starting point, not the final product. Validate expected values against your actual knowledge sources, and adjust sample inputs to match your real users' language.
- **Run iteratively.** After your first eval run, add test cases for failure patterns you discover and tighten thresholds as the agent improves.

---

## Example Workflow

```
1. Export agent → solution.zip
2. Unpack & fill out agent-profile-template.yaml
3. Copy the prompt above, paste in your profile YAML
4. Run in Claude / Copilot → get eval set markdown
5. Review, adapt expected values to real source content
6. Import test cases into Copilot Studio evaluation
7. Run eval, review results, iterate
```

---

[Back to library](../README.md)
