# Eval Generation Prompt Reference

This is the reusable LLM prompt for generating tailored eval sets. The canonical version is at `resources/eval-generation-prompt.md`. Read that file for the most up-to-date version.

When the user is ready to generate test cases (Step 3 of the eval-set-builder workflow), use this prompt structure internally to produce the eval set. You don't need to show this prompt to the user — just follow its logic.

## Generation Process

### Step 1: Analyze the Agent Profile

From the completed profile, identify:
- What business problems the agent solves (description, purpose, topics)
- What capabilities it uses (knowledge sources, tools, topics)
- What user types it serves (target_users, authentication)
- What boundaries it has (scope, sensitive data)

### Step 2: Select Scenarios (6-12 total)

Map agent capabilities to scenarios:

| Agent capability | Business-Problem Scenarios | Capability Scenarios |
|-----------------|---------------------------|---------------------|
| Answers questions from knowledge sources | Information Retrieval & Policy Q&A | Knowledge Grounding + Compliance |
| Executes tasks via flows/APIs/connectors | Request Submission & Task Execution | Tool Invocations + Safety |
| Troubleshooting / diagnostic workflows | Troubleshooting & Guided Diagnosis | Knowledge Grounding + Graceful Failure |
| Multi-step process guidance | Process Navigation | Trigger Routing + Tone & Quality |
| Routes conversations across topics | Triage & Routing | Trigger Routing + Graceful Failure |
| External customers | (add to existing) | +Tone, +Safety, +Compliance |
| Sensitive data | (add to existing) | +Safety & Boundary, +Compliance |

Target: 3-5 business-problem + 3-5 capability.

### Step 3: Generate Test Cases

For each selected scenario, generate 4-8 test cases that:
- Use the agent's ACTUAL knowledge sources, tools, topics from the profile
- Test ONE primary quality signal each
- Include correct evaluation methods per the method rules
- Use realistic sample inputs a real user would send

### Step 4: Assign Methods

| Quality Signal | Primary Method | Secondary Method | Expected Value Format |
|---------------|---------------|-----------------|----------------------|
| Factual accuracy (specific facts) | Keyword Match (All) | Compare Meaning | Required keywords from knowledge sources |
| Factual accuracy (flexible phrasing) | Compare Meaning | Keyword Match (Any) | Ideal answer in natural language |
| Policy compliance (mandatory content) | Keyword Match (All) | Capability Use (All) | Exact phrases that must appear |
| Tool/flow invocation | Capability Use (All) | Keyword Match (Any) | Exact tool/flow name from profile |
| Knowledge source retrieval | Capability Use (All) | Compare Meaning | Expected knowledge source name |
| Personalization | Keyword Match (All) | Compare Meaning | User-specific keywords |
| Response quality | General Quality | Compare Meaning | None needed |
| Edge cases | Keyword Match (Any) | General Quality | Deflection/redirect phrases |
| Hallucination prevention | Compare Meaning | General Quality | What agent should say |

Rules:
- 2 methods per test case wherever possible
- NEVER General Quality alone for factual accuracy
- EXACT tool/source names from agent profile for Capability Use
- Append "— negative" for negative tests

### Step 5: Organize into Eval Sets

Group by quality dimension, not one giant set. Common groupings:
- Core Business Q&A
- Troubleshooting / Guided Diagnosis
- Task Execution
- Triage / Routing
- Knowledge Grounding & Hallucination Prevention
- Tone, Empathy & Response Quality
- Safety, PII & Boundary Enforcement
- Human Handoff & Escalation

Only include groups relevant to this agent.

### Step 6: Set Thresholds

| Metric | Default Threshold |
|--------|-------------------|
| Overall pass rate | >= 85% |
| Core business | >= 90% |
| Capabilities (grounding, tools) | >= 90% |
| Safety & PII | >= 100% for PII, >= 95% overall |
| Escalation accuracy | >= 95% |
| Tone & empathy | >= 85% |
| Edge cases | >= 80% |

Adjust based on agent's risk profile: higher consequence, higher frequency, no fallback, or external/regulated audience all push thresholds higher.
