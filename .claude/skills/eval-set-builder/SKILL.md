---
name: eval-set-builder
description: 'Use this skill when the user wants to build an evaluation set, create test cases, extract or fill out agent information, generate evaluation documents, or use the eval set template. Always use this skill when the user asks: "build eval set", "create test cases", "agent information", "agent profile", "generate evaluation", "eval template", "generate eval set for my agent", or wants to produce eval set documents (.md, .docx, .csv, or .xlsx). Also use when the user has a Copilot Studio solution export (.zip) and wants to turn it into an eval set.'
---

# Eval Set Builder

You guide users through building a complete, production-ready evaluation set for their Copilot Studio agent. Follow the hybrid 4-step workflow: ask questions at each stage, then generate concrete outputs.

## Workflow Overview

```
Step 1: AGENT INFO  →  Step 2: SELECT  →  Step 3: GENERATE  →  Step 4: FORMAT
Capture agent          Choose scenarios     Create tailored       Output as .docx,
information            from the library     test cases            .csv, .xlsx, or all
```

---

## Step 1: Agent Information — Capture Agent Details

This step captures the agent's configuration in a structured format. It's the foundation for everything that follows.

**Ask the user which input they have:**
- **Option A: Solution export (.zip)** — Guide them through extracting details using the agent information guide. Read `references/agent-profile-guide.md` for the full extraction workflow.
- **Option B: They can describe their agent** — Ask structured questions and fill out the information for them.
- **Option C: They already have a YAML agent information file** — Validate it for completeness.
- **Option D: Agent description as a .docx file** — Read the Word document to extract agent details (name, purpose, knowledge sources, topics, tools, scope, instructions). Use the docx skill to read the file, then map the extracted information into the structured YAML format below. Ask clarifying questions for any details not covered in the document.

### If extracting from a .docx file (Option D):

1. Use the docx skill to read the Word document.
2. Extract all agent details: name, purpose, target users, channels, system instructions, knowledge sources, topics/trigger phrases, tools/actions, authentication, scope boundaries, and sensitive data types.
3. Map extracted details into the YAML structure below.
4. Present the completed YAML to the user for review and ask if anything is missing or needs correction.
5. If the document is incomplete (e.g., missing trigger phrases or tool parameters), ask the user targeted questions to fill the gaps.

### If building from description (Option B), ask:

1. **Agent name and purpose** — What does it do? What business problem does it solve?
2. **Target users** — Who uses it? Internal employees, external customers, specific roles?
3. **Knowledge sources** — What documents/sites does it pull answers from? (SharePoint, files, websites, Dataverse)
4. **Topics** — What conversation flows does it support? What trigger phrases activate them?
5. **Tools/actions** — Does it call Power Automate flows, connectors, APIs? What do they do?
6. **Authentication** — How are users identified? (None, Entra ID, custom)
7. **Scope boundaries** — What should it handle? What should it decline? Any sensitive data types?
8. **System instructions** — Can they share the agent's system message/instructions?

### Agent Information Template

Generate the completed agent information in this YAML structure:

```yaml
agent:
  name: ""
  description: ""        # 1-2 sentences
  purpose: ""            # Business problem it solves
  target_users: ""
  channels:
    - ""                 # Teams, Web Chat, etc.

instructions: |
  # Full system instructions here

knowledge_sources:
  - name: ""
    type: ""             # SharePoint | Uploaded File | Website | Dataverse | Custom
    url_or_path: ""
    description: ""
    key_content_areas:
      - ""

topics:
  - name: ""
    description: ""
    trigger_phrases:
      - ""
    connected_actions:
      - ""

tools:
  - name: ""
    type: ""             # Power Automate Flow | Connector | Plugin | HTTP Action
    description: ""
    trigger_condition: ""
    parameters:
      - name: ""
        type: ""
        description: ""
    returns: ""

authentication:
  type: ""               # None | Entra ID | Custom
  user_context_variables:
    - ""

scope:
  in_scope:
    - ""
  out_of_scope:
    - ""
  sensitive_data_types:
    - ""

notes: |
```

The full agent information template with comments is at `resources/agent-profile-template.yaml`. The extraction guide is at `references/agent-profile-guide.md`.

### Validate Completeness

Before proceeding, check:
- Instructions are complete (full system message, not summarized)
- All knowledge sources listed (missing sources = missing test coverage)
- All topics with trigger phrases (drive routing test cases)
- All tools/flows with parameters (drive Capability Use test cases)
- Descriptions are specific ("PTO policy, benefits, leave procedures" not just "HR policies")
- Scope boundaries defined (drive safety test cases)
- Sensitive data types identified (drive PII/safety test cases)

---

## Step 2: Select — Choose Scenarios

Based on the agent information, recommend scenarios using the Quick-Start Map:

| Agent capability | Business-Problem Scenarios | Capability Scenarios |
|-----------------|---------------------------|---------------------|
| Answers questions from knowledge sources | Information Retrieval & Policy Q&A | Knowledge Grounding + Compliance |
| Executes tasks via flows/APIs | Request Submission & Task Execution | Tool Invocations + Safety |
| Troubleshooting workflows | Troubleshooting & Guided Diagnosis | Knowledge Grounding + Graceful Failure |
| Multi-step process guidance | Process Navigation | Trigger Routing + Tone & Quality |
| Routes across topics | Triage & Routing | Trigger Routing + Graceful Failure |
| External customers | (add to rows above) | +Tone & Quality, +Safety, +Compliance |
| Sensitive data | (add to rows above) | +Safety & Boundary, +Compliance |

Target: **3-5 business-problem + 3-5 capability scenarios** (6-12 total).

Present the recommendation to the user and ask if they want to adjust before proceeding.

---

## Step 3: Generate — Create Tailored Test Cases

For each selected scenario:

1. **Read the scenario file** from the repo to get its evaluation patterns, practical examples, and recommended methods
2. **Generate 4-8 test cases per scenario** using the agent's actual configuration — not generic placeholders
3. **Assign evaluation methods** using these rules:

| Quality Signal | Primary Method | Secondary Method | Expected Value Format |
|---------------|---------------|-----------------|----------------------|
| Factual accuracy (specific facts) | Keyword Match (All) | Compare Meaning | List required keywords from agent's knowledge sources |
| Factual accuracy (flexible phrasing) | Compare Meaning | Keyword Match (Any) | Natural language ideal answer |
| Policy compliance (mandatory content) | Keyword Match (All) | Capability Use (All) | Exact phrases that must appear |
| Tool/flow invocation | Capability Use (All) | Keyword Match (Any) | Exact tool/flow name from agent information |
| Correct knowledge source retrieval | Capability Use (All) | Compare Meaning | Expected knowledge source name |
| Personalization | Keyword Match (All) | Compare Meaning | User-context-specific keywords |
| Response quality | General Quality | Compare Meaning | None needed for General Quality |
| Edge case handling | Keyword Match (Any) | General Quality | Acceptable deflection/redirect phrases |
| Hallucination prevention | Compare Meaning | General Quality | What agent should say (including "I don't know") |

**Method rules:**
- Use 2 methods per test case wherever possible
- NEVER use General Quality alone for factual accuracy
- For Capability Use, use the EXACT tool/source names from the agent information
- For negative tests (agent should NOT do something), append "— negative"

For the full eval generation prompt that can be used as a reference, read `references/eval-generation-prompt.md`.

---

## Step 4: Format — Output the Eval Set

**Before generating output, ask the user which format(s) they want:**

- **a) .docx guidance document** — Formatted eval set document with full context: agent summary, scenario descriptions, test case tables, coverage analysis, thresholds, and next steps. Use the docx skill to generate a professionally formatted Word document.
- **b) .csv file(s)** — Lean files structured for direct import into Copilot Studio's evaluation tool. Columns: `Test Case ID`, `Sample Input`, `Expected Value`, `Methods`. One CSV per eval set grouping. Omits metadata columns (Quality Signal, Priority) that the import tool does not use.
- **c) .xlsx file** — Rich eval set spreadsheet with all fields including `Test Case ID`, `Quality Signal`, `Sample Input`, `Expected Value`, `Methods`, `Priority`, `Scenario Source`, and `Category`. Serves as the team's working reference with full context. Use the xlsx skill to generate.
- **d) All of the above** — Generate .docx, .csv files, and .xlsx together.

### Eval Set Structure (used across all formats)

```markdown
# Eval Set: [Agent Name]

## Agent Summary
[1-2 sentences from agent information]

## Selected Scenarios
| # | Scenario | Source | Why Selected |
|---|----------|--------|-------------|
| 1 | ... | Business-Problem: ... | ... |

## Eval Set 1: [Quality Dimension Name]
**Source scenario:** [Library scenario name]
**Pass threshold:** [threshold]

| Test Case ID | Quality Signal | Sample Input | Expected Value | Methods | Priority |
|---|---|---|---|---|---|
| XX-001 | ... | ... | ... | ... | High/Medium |

[Repeat for each eval set grouping]

## Coverage Summary
| Category | Test Cases | % of Total |
|---|---|---|
| Core Business | X | X% |
| Variations | X | X% |
| Architecture & Capability | X | X% |
| Edge Cases & Safety | X | X% |

## Thresholds
| Metric | Threshold |
|---|---|
| Overall pass rate | >= 85% |
| Core business | >= 90% |
| Safety & compliance | >= 95% |
| Capabilities | >= 90% |
| Edge cases | >= 80% |

## Next Steps
1. Adapt sample inputs to match real user language
2. Replace placeholder expected values with content from actual knowledge sources
3. Configure tool/flow names to match Copilot Studio setup exactly
4. Run initial evaluation and calibrate thresholds based on results
```

### Coverage Targets
- Core business happy paths: 30-40% of test cases
- Variations (personalization, ambiguous inputs): 20-30%
- Architecture & capability tests: 20-30%
- Edge cases & safety: 10-20%

### Format-Specific Notes

**For .csv (Copilot Studio import):**
- One CSV file per eval set grouping (e.g., `EvalSet1_InformationRetrieval.csv`)
- Include only the columns required by the Copilot Studio evaluation tool: `Test Case ID`, `Sample Input`, `Expected Value`, `Methods`
- Keep values clean — avoid line breaks or special characters that could break CSV parsing

**For .xlsx (rich reference):**
- Single workbook with one sheet per eval set grouping, plus a Summary sheet
- Include all columns: `Test Case ID`, `Quality Signal`, `Sample Input`, `Expected Value`, `Methods`, `Priority`, `Scenario Source`, `Category`
- The Summary sheet should contain: selected scenarios table, coverage summary, and thresholds

**For .docx (guidance document):**
- Use the docx skill to generate a formatted document with headings, tables, and next steps
- Include the full eval set structure above with context and instructions

---

## Tips

- **The more detailed the agent information, the better the test cases.** Vague descriptions produce generic tests. Specific knowledge source content areas and tool descriptions produce targeted tests.
- **Review and adapt.** The generated eval set is a strong starting point. Validate expected values against actual knowledge sources and adjust sample inputs to match real users' language.
- **Run iteratively.** After the first eval run, add test cases for failure patterns you discover and tighten thresholds as the agent improves.
- **One test case = one primary quality signal.** If a test case fails, you need to know which dimension failed. Mixing signals makes diagnosis harder.
