# AI Agent Evaluation Scenario Library

You are an evaluation specialist for Copilot Studio agents. This repo contains 75 reusable evaluation scenarios, templates, and guides for building comprehensive eval sets.

## Project Structure

```
business-problem-scenarios/   5 scenario files — test WHAT the agent delivers
capability-scenarios/         8 scenario files — test HOW the agent's infrastructure works
resources/
  scenario-index.csv          Flat index of all 75 scenarios (filterable by ID, type, category)
  eval-set-template.md        Template for organizing eval sets
  agent-profile-template.yaml Structured agent config snapshot (input for eval generation)
  agent-profile-guide.md      How to extract a profile from a Copilot Studio export
  eval-generation-prompt.md   Reusable LLM prompt for generating tailored eval sets
```

## Key Concepts

**Two scenario types (you need both):**
- **Business-problem scenarios** — what the agent does for users (e.g., "employee asks about PTO policy")
- **Capability scenarios** — how the agent's infrastructure behaves (e.g., "verify tool invocations," "validate knowledge grounding")

**Evaluation methods** (used in test cases):
- **Keyword Match (All)** — all listed keywords must appear. Use for: factual accuracy, compliance.
- **Keyword Match (Any)** — at least one keyword must appear. Use for: deflection phrases, edge cases.
- **Compare Meaning** — response meaning aligns with expected. Use for: flexible-phrasing answers.
- **General Quality** — subjective grading (relevance, groundedness, completeness). Use for: tone, quality.
- **Capability Use (All)** — expected tool/source must be invoked. Use for: tool/knowledge verification.
- **Capability Use (Any) — Negative** — listed capabilities should NOT be invoked.

**Best practice:** Use 2 methods per test case (one specific + one general).

## Scenario Inventory

### Business-Problem Scenarios (5 files, 29 sub-scenarios)
| ID Prefix | Section | File |
|-----------|---------|------|
| BP-IR | Information Retrieval & Policy Q&A (6 patterns) | `business-problem-scenarios/information-retrieval-and-policy-qa.md` |
| BP-TS | Troubleshooting & Guided Diagnosis (6 patterns) | `business-problem-scenarios/troubleshooting-and-guided-diagnosis.md` |
| BP-RS | Request Submission & Task Execution (6 patterns) | `business-problem-scenarios/request-submission-and-task-execution.md` |
| BP-PN | Process Navigation & Multi-Step Guidance (6 patterns) | `business-problem-scenarios/process-navigation-and-multistep-guidance.md` |
| BP-TR | Triage & Routing (5 patterns) | `business-problem-scenarios/triage-and-routing.md` |

### Capability Scenarios (8 files, 46 sub-scenarios)
| ID Prefix | Section | File |
|-----------|---------|------|
| CAP-KG | Knowledge Grounding & Accuracy (6 patterns) | `capability-scenarios/knowledge-grounding-and-accuracy.md` |
| CAP-TI | Tool & Connector Invocations (6 patterns) | `capability-scenarios/tool-and-connector-invocations.md` |
| CAP-TR | Trigger Routing (5 patterns) | `capability-scenarios/trigger-routing.md` |
| CAP-CV | Compliance & Verbatim Content (6 patterns) | `capability-scenarios/compliance-and-verbatim-content.md` |
| CAP-SB | Safety & Boundary Enforcement (6 patterns) | `capability-scenarios/safety-and-boundary-enforcement.md` |
| CAP-TQ | Tone, Helpfulness & Response Quality (6 patterns) | `capability-scenarios/tone-helpfulness-and-response-quality.md` |
| CAP-GF | Graceful Failure & Escalation (5 patterns) | `capability-scenarios/graceful-failure-and-escalation.md` |
| CAP-RT | Regression Testing (5 patterns) | `capability-scenarios/regression-testing.md` |

## Entry Path A: Agent Capability Quick-Start Map

| My agent... | Start with these scenarios |
|------------|--------------------------|
| Answers questions using knowledge sources | Information Retrieval + Knowledge Grounding + Compliance |
| Executes tasks via Power Automate / APIs | Request Submission + Tool Invocations + Safety |
| Walks users through troubleshooting | Troubleshooting + Knowledge Grounding + Graceful Failure |
| Guides users through multi-step processes | Process Navigation + Trigger Routing + Tone & Quality |
| Routes conversations across topics | Triage & Routing + Trigger Routing + Graceful Failure |
| Serves external customers | Add: Tone & Quality + Safety + Compliance |
| Handles sensitive data (PII, financial) | Add: Safety & Boundary + Compliance |
| Is about to be updated/republished | Regression Testing + all previously passing sets |

## Entry Path B: "I Want To..." Routing Table

| I want to... | Go to |
|-------------|-------|
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

## Each Scenario Entry Contains

| Section | What It Provides |
|---------|-----------------|
| When to Use | When this scenario applies |
| Recommended Test Methods | Which eval methods and why |
| Setup Steps | Step-by-step test case creation |
| Anti-Pattern | Common mistake to avoid |
| Evaluation Patterns | Named sub-patterns (different testing angles) |
| Practical Examples | Concrete sample test cases in a table |
| Tips | Coverage targets, thresholds, best practices |

When a user describes their agent, read the relevant scenario files to provide specific, grounded recommendations with examples from those files.

## Available Skills

Two skills are bundled with this repo:

- **eval-scenario-guide** — "What should I evaluate?" Helps navigate the library and select scenarios for a specific agent. Ask about the agent first, then recommend scenarios.
- **eval-set-builder** — "Help me build an eval set." Guides through the full workflow: agent profile, scenario selection, test case generation, formatted output.

## Companion Resource: Failure Triage Playbook

For diagnosing why evaluations fail and finding remediation steps, see the Failure Triage & Remediation Playbook (separate project). It provides score interpretation, root cause classification, and fix guidance.
