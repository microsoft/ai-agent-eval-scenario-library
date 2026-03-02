---
name: eval-scenario-guide
description: >
  Use this skill when the user wants to find evaluation scenarios, understand what to test
  for their Copilot Studio agent, navigate the scenario library, learn about evaluation
  methodology, or explore specific scenario categories. Always use this skill when the user
  asks: "what should I evaluate", "find scenarios for my agent", "which scenarios cover X",
  "how do I test Y", "evaluation methods", "what quality signals should I measure", or
  anything related to discovering and selecting evaluation scenarios from this library.
---

# Eval Scenario Guide

You help users navigate the AI Agent Evaluation Scenario Library and select the right evaluation scenarios for their agent. Follow the hybrid workflow below: ask about the agent first, then generate tailored recommendations.

## Workflow

### Step 1: Understand the Agent

Before recommending scenarios, ask the user about their agent. You need to know:

1. **What does the agent do?** (answer questions, execute tasks, troubleshoot, route conversations, guide processes)
2. **What capabilities does it use?** (knowledge sources, Power Automate flows, connectors, topic routing)
3. **Who are the users?** (internal employees, external customers, specific roles)
4. **Any specific evaluation concerns?** (hallucination, PII, compliance, tone, tool firing)

If the user provides an agent profile YAML or describes their agent, proceed directly. If they're unsure, ask these questions one at a time.

### Step 2: Map to Scenarios

Use the Quick-Start Map to identify recommended scenarios based on the agent's capabilities:

| My agent... | Business-Problem Scenarios | Capability Scenarios |
|------------|--------------------------|---------------------|
| Answers questions using knowledge sources | Information Retrieval & Policy Q&A | Knowledge Grounding + Compliance |
| Executes tasks via Power Automate / APIs | Request Submission & Task Execution | Tool Invocations + Safety |
| Walks users through troubleshooting | Troubleshooting & Guided Diagnosis | Knowledge Grounding + Graceful Failure |
| Guides users through multi-step processes | Process Navigation & Multi-Step Guidance | Trigger Routing + Tone & Quality |
| Routes conversations across topics | Triage & Routing | Trigger Routing + Graceful Failure |
| Serves external customers | (add to existing rows) | +Tone & Quality, +Safety, +Compliance |
| Handles sensitive data (PII, financial, health) | (add to existing rows) | +Safety & Boundary, +Compliance |
| Is about to be updated/republished | (add to existing rows) | +Regression Testing |

Most agents match multiple rows. Combine all that apply. Target: **3-5 business-problem + 3-5 capability scenarios** (6-12 total).

If the user has a specific concern rather than a capability description, use the "I Want To..." table:

| I want to... | Scenario |
|-------------|----------|
| Test business question accuracy | Information Retrieval & Policy Q&A |
| Verify troubleshooting workflows | Troubleshooting & Guided Diagnosis |
| Test task execution | Request Submission & Task Execution |
| Evaluate multi-step guidance | Process Navigation & Multi-Step Guidance |
| Check triage and routing | Triage & Routing |
| Confirm no hallucination | Knowledge Grounding & Accuracy |
| Check tool/flow fires correctly | Tool & Connector Invocations |
| Verify topic routing | Trigger Routing |
| Confirm required disclaimers appear | Compliance & Verbatim Content |
| Test adversarial/out-of-scope handling | Safety & Boundary Enforcement |
| Evaluate tone and empathy | Tone, Helpfulness & Response Quality |
| Confirm appropriate escalation | Graceful Failure & Escalation |
| Validate before publishing | Regression Testing |

### Step 3: Deep-Dive into Selected Scenarios

For each recommended scenario, **read the actual scenario file from the repo** to pull specific, grounded details. The scenario files are at:

**Business-Problem Scenarios:**
- `business-problem-scenarios/information-retrieval-and-policy-qa.md`
- `business-problem-scenarios/troubleshooting-and-guided-diagnosis.md`
- `business-problem-scenarios/request-submission-and-task-execution.md`
- `business-problem-scenarios/process-navigation-and-multistep-guidance.md`
- `business-problem-scenarios/triage-and-routing.md`

**Capability Scenarios:**
- `capability-scenarios/knowledge-grounding-and-accuracy.md`
- `capability-scenarios/tool-and-connector-invocations.md`
- `capability-scenarios/trigger-routing.md`
- `capability-scenarios/compliance-and-verbatim-content.md`
- `capability-scenarios/safety-and-boundary-enforcement.md`
- `capability-scenarios/tone-helpfulness-and-response-quality.md`
- `capability-scenarios/graceful-failure-and-escalation.md`
- `capability-scenarios/regression-testing.md`

Read the files and extract: evaluation patterns, recommended methods, practical examples, and tips relevant to this user's agent.

### Step 4: Generate a Scenario Selection Report

Present the output as a structured recommendation:

```
## Recommended Scenarios for [Agent Name/Description]

### Business-Problem Scenarios
| # | Scenario | Why Selected | Priority | Key Patterns to Test |
|---|----------|-------------|----------|---------------------|
| 1 | ... | ... | High/Medium | ... |

### Capability Scenarios
| # | Scenario | Why Selected | Priority | Key Patterns to Test |
|---|----------|-------------|----------|---------------------|
| 1 | ... | ... | High/Medium | ... |

### Coverage Assessment
- Core business coverage: [which use cases are covered]
- Infrastructure coverage: [which components are validated]
- Gaps to consider: [anything not covered and why it might matter]

### Recommended Evaluation Methods
[Brief summary of which methods apply to this agent's scenarios]

### Next Step
Use the eval-set-builder skill to generate a complete eval set from these scenarios.
```

## Scenario Inventory Quick Reference

All 75 scenarios with their IDs (for lookup in `resources/scenario-index.csv`):

**Business-Problem (BP):**
- BP-IR-01 to BP-IR-06: Information Retrieval & Policy Q&A
- BP-TS-01 to BP-TS-06: Troubleshooting & Guided Diagnosis
- BP-RS-01 to BP-RS-06: Request Submission & Task Execution
- BP-PN-01 to BP-PN-06: Process Navigation & Multi-Step Guidance
- BP-TR-01 to BP-TR-05: Triage & Routing

**Capability (CAP):**
- CAP-KG-01 to CAP-KG-06: Knowledge Grounding & Accuracy
- CAP-TI-01 to CAP-TI-06: Tool & Connector Invocations
- CAP-TR-01 to CAP-TR-05: Trigger Routing
- CAP-CV-01 to CAP-CV-06: Compliance & Verbatim Content
- CAP-SB-01 to CAP-SB-06: Safety & Boundary Enforcement
- CAP-TQ-01 to CAP-TQ-06: Tone, Helpfulness & Response Quality
- CAP-GF-01 to CAP-GF-05: Graceful Failure & Escalation
- CAP-RT-01 to CAP-RT-05: Regression Testing

## Evaluation Methods Quick Reference

| Method | When to Use | Expected Value Format |
|--------|------------|----------------------|
| Keyword Match (All) | Specific facts, numbers, compliance phrases must appear | List of required keywords |
| Keyword Match (Any) | At least one of several valid phrasings | List of acceptable keywords |
| Compare Meaning | Correct meaning, flexible wording | Natural language ideal answer |
| General Quality | Subjective quality assessment | None needed |
| Capability Use (All) | Verify specific tool/source was invoked | Tool/source name from agent config |
| Capability Use (Any) — Negative | Verify tool/source was NOT invoked | Tool/source that should not fire |

**Multi-method best practice:** Use 2 methods per test case. Primary method tests the specific requirement; secondary catches unexpected failures.

| Quality Signal | Primary Method | Secondary Method |
|---------------|---------------|-----------------|
| Factual accuracy | Keyword Match (All) | Compare Meaning |
| Policy compliance | Keyword Match (All) | Capability Use (All) |
| Tool invocation | Capability Use (All) | Keyword Match (Any) |
| Knowledge grounding | Capability Use (All) | Compare Meaning |
| Personalization | Keyword Match (All) | Compare Meaning |
| Response quality | General Quality | Compare Meaning |
| Edge cases | Keyword Match (Any) | General Quality |

For a deeper dive into method selection, read `references/method-selection-guide.md`.
