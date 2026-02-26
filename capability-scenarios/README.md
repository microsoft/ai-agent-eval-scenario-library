# Capability Scenarios

Evaluation scenarios grounded in **how your agent's infrastructure works** — its knowledge retrieval, tool usage, routing logic, safety properties, and communication quality. These apply to any agent regardless of its business purpose.

## Why These Matter

An agent can give the right answer from the wrong source. It can produce a helpful response while leaking PII. It can sound confident while hallucinating. Capability scenarios catch these infrastructure-level issues that business-problem testing alone won't reveal.

## Scenarios

| # | Category | What It Covers | Link |
|---|----------|---------------|------|
| 1 | **Knowledge Grounding & Accuracy** | Right sources retrieved? Handles missing/outdated/conflicting sources? No hallucination? | [View scenarios](knowledge-grounding-and-accuracy.md) |
| 2 | **Tool & Connector Invocations** | Right tool fires with correct parameters? Response parsed correctly? Error handling? | [View scenarios](tool-and-connector-invocations.md) |
| 3 | **Trigger Routing** | Right topic fires? Handoff works? Disambiguation works? | [View scenarios](trigger-routing.md) |
| 4 | **Compliance & Verbatim Content** | Required disclaimers, exact policy language, mandatory warnings included? | [View scenarios](compliance-and-verbatim-content.md) |
| 5 | **Safety & Boundary Enforcement** | Resists adversarial input? Protects data? Stays in scope? | [View scenarios](safety-and-boundary-enforcement.md) |
| 6 | **Tone, Helpfulness & Response Quality** | Appropriately styled? Empathetic? Well-structured? Complete? | [View scenarios](tone-helpfulness-and-response-quality.md) |
| 7 | **Graceful Failure & Escalation** | When it can't help, does it fail well? Escalates appropriately? | [View scenarios](graceful-failure-and-escalation.md) |
| 8 | **Regression Testing** | After an update, did anything break? Pre-publish validation. | [View scenarios](regression-testing.md) |

## How These Connect to Business-Problem Scenarios

Capability scenarios test the **how** — is the infrastructure working correctly? But they don't test the **what** — is the agent solving the actual business problem?

After selecting your capability scenarios, make sure you also have [business-problem scenarios](../business-problem-scenarios/) covering your agent's core use cases.

---

[Back to main library](../README.md)
