# AI Agent Evaluation Scenario Library

Evaluation scenarios, quality signals, and acceptance criteria for AI agents. Browse by agent type or evaluation concern, adapt the examples to your use case, and build a comprehensive eval set.

---

## What This Library Is

This library gives you a head start on agent evaluation. Instead of designing test cases from scratch, you select from proven scenarios that cover business quality, architecture, compliance, and safety — then adapt them to your agent.

Use it to:

1. **Identify** which quality dimensions apply to your agent
2. **Select** evaluation scenarios that match your business needs and architecture
3. **Build** test cases using the examples and patterns each scenario provides
4. **Validate** that your eval set is comprehensive — beyond happy-path testing

### Two Types of Scenarios

The library organizes scenarios into two complementary categories:

- **Business-problem scenarios** — what your agent does for users (e.g., "employee asks about PTO policy," "customer troubleshoots a billing issue"). These capture the outcomes your stakeholders care about.
- **Capability scenarios** — how your agent's architecture behaves (e.g., "verify tool invocations," "validate knowledge grounding"). These confirm that each component works correctly, stays safe, and communicates clearly.

> **You need both.** Business-problem scenarios verify that your agent solves the right problem. Capability scenarios verify that the underlying components work correctly. An agent can return the right answer from the wrong source, or call the right tool with the wrong parameters — only capability testing catches that.

---

## Using This Library with Claude Code

This library includes a built-in AI assistant and two skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). When you open this repo in Claude Code, Claude automatically understands the library's structure, concepts, and scenarios — and can guide you through the entire evaluation workflow interactively.

### Quick Start with Claude Code

1. **Open this repo** in Claude Code (navigate to this directory in your terminal and run `claude`)
2. **Ask Claude** what you need — it will use the right skill automatically:

| What you want to do | Just say... | What happens |
|---------------------|------------|-------------|
| Find the right scenarios for your agent | *"What should I evaluate for my HR policy agent?"* | Claude asks about your agent, then recommends scenarios with rationale |
| Build a complete eval set | *"Build me an eval set for this agent"* | Claude walks you through: agent profile → scenario selection → test case generation → formatted output |
| Understand evaluation methods | *"When should I use Keyword Match vs Compare Meaning?"* | Claude explains the method selection framework with examples |
| Explore a specific scenario | *"Tell me about the safety and boundary enforcement scenarios"* | Claude reads the scenario file and presents the patterns, examples, and tips |

### Step-by-Step: Generate an Eval Set with Claude Code

This is the most common workflow — going from "I have an agent" to "I have a ready-to-use eval set."

**Step 1: Describe your agent** (or provide a file with the agent information)

Tell Claude what your agent does. The more detail you provide, the better the output:
- What business problems it solves
- What knowledge sources it uses (SharePoint, uploaded files, websites)
- What tools/flows it calls (Power Automate, connectors, APIs)
- Who uses it (internal employees, external customers)
- Any sensitive data it handles

Or, if you have a Copilot Studio solution export (.zip), share it and Claude will extract the profile for you.

**Step 2: Review scenario recommendations**

Claude maps your agent's capabilities to the library's scenarios and presents a recommendation like:

> *"Based on your agent, I recommend 8 scenarios: Information Retrieval, Request Submission, Triage & Routing (business-problem), plus Knowledge Grounding, Tool Invocations, Safety, Tone & Quality, Regression Testing (capability). Here's why each was selected..."*

Adjust the selection if needed — add or remove scenarios.

**Step 3: Generate test cases**

Claude generates tailored test cases for each scenario, using your agent's actual knowledge sources, tools, and topics — not generic placeholders. Each test case includes:
- Sample user input
- Expected value (keywords, ideal answer, or tool/source name)
- Evaluation methods (2 per test case)
- Priority level

**Step 4: Get your eval set document**

Claude outputs a structured eval set as a markdown file (or Word document if you ask), ready to import into Copilot Studio's evaluation feature. Includes coverage summary and recommended pass thresholds.

### Available Skills

| Skill | Trigger | What It Does |
|-------|---------|-------------|
| **eval-scenario-guide** | *"What should I evaluate?"*, *"find scenarios"*, *"evaluation methods"* | Navigates the library, recommends scenarios, explains methodology |
| **eval-set-builder** | *"Build eval set"*, *"create test cases"*, *"agent profile"* | Full workflow from agent profile to finished eval set document |

> **Companion skill — Failure Triage:** If your evals are failing and you need to diagnose root causes, the `eval-failure-triage` skill is available in the companion [Failure Triage & Remediation Playbook](https://github.com/TODO-link) project. It helps interpret scores, classify root causes (eval setup / agent config / platform limitation), and find specific remediation actions.

### Without Claude Code

You can use this library entirely without Claude Code — the sections below walk you through the same process manually.

---

## How to Use This Library (Manual)

### Step 1: Orient — Find Your Starting Point

Start with one of the two entry paths below:
- **[Entry Path A](#entry-path-a-agent-capability-quick-start-map)** — "I have an agent that does X — what should I evaluate?"
- **[Entry Path B](#entry-path-b-i-want-to-routing-table)** — "I have a specific evaluation concern — where do I go?"

### Step 2: Select — Pick Your Scenarios

Open the linked scenario files and read the "When to Use" section to confirm relevance. Most agents need **3–5 business-problem scenarios** and **3–5 capability scenarios**.

### Step 3: Build — Create Your Test Cases

Each scenario provides everything you need to create test cases:
- **Recommended Test Methods** — which evaluation methods to use, why, and how to combine them
- **Setup Steps** — step-by-step instructions for creating test cases
- **Evaluation Patterns** — sub-patterns covering different angles of the scenario
- **Practical Examples** — sample test cases you can adapt directly
- **Tips** — coverage targets, thresholds, and best practices

Adapt the examples to your agent's specific knowledge sources, tools, and user base. Use multiple test methods per test case wherever relevant — each method catches a different failure mode, and Copilot Studio supports combining them.

### Step 4: Expand — Check for Gaps

After building your initial eval set, revisit the routing tables for missed dimensions. Common gaps:
- Testing only happy paths — add edge cases from [Safety & Boundary Enforcement](capability-scenarios/safety-and-boundary-enforcement.md) and [Graceful Failure & Escalation](capability-scenarios/graceful-failure-and-escalation.md)
- Skipping compliance testing — see [Compliance & Verbatim Content](capability-scenarios/compliance-and-verbatim-content.md)
- No regression baseline — see [Regression Testing](capability-scenarios/regression-testing.md)

---

## Entry Path A: Agent Capability Quick-Start Map

*"I have an agent that does X — what should I evaluate?"*

| My agent... | Start with these scenarios |
|------------|--------------------------|
| Answers questions using knowledge sources (docs, SharePoint, FAQ) | [Information Retrieval & Q&A](business-problem-scenarios/information-retrieval-and-policy-qa.md) + [Knowledge Grounding](capability-scenarios/knowledge-grounding-and-accuracy.md) + [Compliance](capability-scenarios/compliance-and-verbatim-content.md) |
| Executes tasks via Power Automate, APIs, or connectors | [Request Submission & Task Execution](business-problem-scenarios/request-submission-and-task-execution.md) + [Tool Invocations](capability-scenarios/tool-and-connector-invocations.md) + [Safety](capability-scenarios/safety-and-boundary-enforcement.md) |
| Walks users through diagnostic or troubleshooting steps | [Troubleshooting & Guided Diagnosis](business-problem-scenarios/troubleshooting-and-guided-diagnosis.md) + [Knowledge Grounding](capability-scenarios/knowledge-grounding-and-accuracy.md) + [Graceful Failure](capability-scenarios/graceful-failure-and-escalation.md) |
| Guides users through multi-step processes | [Process Navigation & Multi-Step Guidance](business-problem-scenarios/process-navigation-and-multistep-guidance.md) + [Trigger Routing](capability-scenarios/trigger-routing.md) + [Tone & Quality](capability-scenarios/tone-helpfulness-and-response-quality.md) |
| Routes conversations across multiple topics | [Triage & Routing](business-problem-scenarios/triage-and-routing.md) + [Trigger Routing](capability-scenarios/trigger-routing.md) + [Graceful Failure](capability-scenarios/graceful-failure-and-escalation.md) |
| Serves external customers (not just internal employees) | [Tone & Response Quality](capability-scenarios/tone-helpfulness-and-response-quality.md) + [Safety & Boundary](capability-scenarios/safety-and-boundary-enforcement.md) + [Compliance](capability-scenarios/compliance-and-verbatim-content.md) |
| Handles sensitive data (PII, financial, health) | [Safety & Boundary](capability-scenarios/safety-and-boundary-enforcement.md) + [Compliance](capability-scenarios/compliance-and-verbatim-content.md) |
| Is about to be updated or republished | [Regression Testing](capability-scenarios/regression-testing.md) + all sections previously passing |

> **Tip:** Most agents match multiple rows. An agent that answers HR questions from SharePoint AND submits PTO requests via Power Automate would combine rows 1 and 2.

---

## Entry Path B: "I Want To..." Routing Table

*"I have a specific evaluation concern — where do I go?"*

| I want to... | Go to |
|-------------|-------|
| Test whether my agent answers business questions correctly | [Information Retrieval & Q&A](business-problem-scenarios/information-retrieval-and-policy-qa.md) |
| Verify my agent handles troubleshooting workflows | [Troubleshooting & Guided Diagnosis](business-problem-scenarios/troubleshooting-and-guided-diagnosis.md) |
| Test request submission and task execution | [Request Submission & Task Execution](business-problem-scenarios/request-submission-and-task-execution.md) |
| Evaluate multi-step process guidance | [Process Navigation & Multi-Step Guidance](business-problem-scenarios/process-navigation-and-multistep-guidance.md) |
| Check that my agent triages and routes correctly | [Triage & Routing](business-problem-scenarios/triage-and-routing.md) |
| Confirm my agent doesn't hallucinate or return ungrounded answers | [Knowledge Grounding & Accuracy](capability-scenarios/knowledge-grounding-and-accuracy.md) |
| Check that the right Power Automate flow, connector, or API fires | [Tool & Connector Invocations](capability-scenarios/tool-and-connector-invocations.md) |
| Verify my topic triggers route correctly | [Trigger Routing](capability-scenarios/trigger-routing.md) |
| Confirm a legal disclaimer or policy appears word-for-word | [Compliance & Verbatim Content](capability-scenarios/compliance-and-verbatim-content.md) |
| Test whether my agent handles adversarial or out-of-scope inputs safely | [Safety & Boundary Enforcement](capability-scenarios/safety-and-boundary-enforcement.md) |
| Evaluate tone, empathy, and response quality | [Tone, Helpfulness & Response Quality](capability-scenarios/tone-helpfulness-and-response-quality.md) |
| Confirm my agent escalates or declines appropriately when stuck | [Graceful Failure & Escalation](capability-scenarios/graceful-failure-and-escalation.md) |
| Ensure nothing broke before I publish an update | [Regression Testing](capability-scenarios/regression-testing.md) |
| Make sure my agent tailors answers to user-specific context | [Information Retrieval & Policy Q&A](business-problem-scenarios/information-retrieval-and-policy-qa.md) (personalization scenarios) |

---

## What Each Scenario Entry Contains

Every scenario follows a consistent structure:

| Section | What It Provides |
|---------|-----------------|
| **When to Use** | When this scenario applies, from the customer's perspective |
| **Recommended Test Methods** | Which evaluation methods to use and why |
| **Setup Steps** | Step-by-step instructions for creating test cases |
| **Anti-Pattern** | The most common mistake to avoid |
| **Evaluation Patterns** | Named sub-patterns covering different angles |
| **Practical Examples** | Concrete sample test cases in a table |
| **Tips** | Coverage targets, thresholds, and best practices |

---

## Repository Structure

```
ai-agent-eval-scenario-library/
│
├── README.md                      ← You are here
├── CLAUDE.md                      ← Claude Code agent configuration
│
├── .claude/skills/                ← Claude Code skills
│   ├── eval-scenario-guide/       ← "What should I evaluate?" skill
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── method-selection-guide.md
│   └── eval-set-builder/          ← "Build me an eval set" skill
│       ├── SKILL.md
│       └── references/
│           ├── agent-profile-guide.md
│           └── eval-generation-prompt.md
│
├── business-problem-scenarios/    ← Scenarios grounded in business value
│   ├── README.md
│   ├── information-retrieval-and-policy-qa.md
│   ├── troubleshooting-and-guided-diagnosis.md
│   ├── request-submission-and-task-execution.md
│   ├── process-navigation-and-multistep-guidance.md
│   └── triage-and-routing.md
│
├── capability-scenarios/          ← Scenarios grounded in agent infrastructure
│   ├── README.md
│   ├── knowledge-grounding-and-accuracy.md
│   ├── tool-and-connector-invocations.md
│   ├── trigger-routing.md
│   ├── compliance-and-verbatim-content.md
│   ├── safety-and-boundary-enforcement.md
│   ├── tone-helpfulness-and-response-quality.md
│   ├── graceful-failure-and-escalation.md
│   └── regression-testing.md
│
└── resources/
    ├── scenario-index.csv         ← Flat index of all scenarios (filterable)
    ├── eval-set-template.md       ← Template for building eval sets
    ├── agent-profile-template.yaml ← Structured snapshot of your agent's config
    ├── agent-profile-guide.md     ← How to extract the profile from a solution export
    └── eval-generation-prompt.md  ← Reusable prompt for LLM-based eval set generation
```

---

## How Business-Problem and Capability Scenarios Work Together

```
Example: HR Q&A Agent

BUSINESS-PROBLEM SCENARIOS:
├── "Employee asks about PTO policy" (Information Retrieval)
├── "Employee submits PTO request" (Request Submission)
└── "New hire asks about onboarding steps" (Process Navigation)

CAPABILITY SCENARIOS:
├── Knowledge Grounding — are the right policy docs retrieved?
├── Tool Invocations — does the PTO submission flow execute correctly?
├── Compliance — is legally required language included?
├── Safety — does the agent protect employee PII?
└── Tone — is the agent empathetic for sensitive HR topics?

RESULT: Comprehensive eval set covering business quality + infrastructure +
        compliance + safety + communication quality.
```

Business-problem scenarios test **what** the agent delivers. Capability scenarios test **how** it delivers it. Together they ensure nothing falls through the cracks.

---

## Contributing

This library is a living resource. If you have scenario suggestions, corrections, or feedback, please open an issue or submit a pull request.

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
