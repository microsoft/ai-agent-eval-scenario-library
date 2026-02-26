# Agent Profile Guide: From Copilot Studio Export to Eval-Ready Profile

This guide walks you through extracting your agent's configuration from a Copilot Studio solution export and filling out the [Agent Profile Template](agent-profile-template.yaml).

---

## Why an Agent Profile?

The evaluation scenario library is agent-agnostic — it provides reusable patterns, but generating tailored eval sets requires knowing your agent's specific knowledge sources, tools, topics, and instructions. The agent profile captures this context in a structured format that works for both human reference and LLM-assisted eval generation.

---

## Step 1: Export Your Agent as a Solution

### Option A: From Copilot Studio UI

1. Open your agent in [Copilot Studio](https://copilotstudio.microsoft.com)
2. Go to **Settings** (gear icon) → **Agent details**
3. Click **Export** → **Export solution**
4. Select or create a solution containing your agent
5. Download the `.zip` file

### Option B: Using Power Platform CLI

```bash
# Install the Power Platform CLI if needed
pac auth create --environment <your-environment-url>

# List solutions to find yours
pac solution list

# Export
pac solution export --name <solution-name> --path ./agent-export.zip
```

### Option C: From Power Apps / Power Platform Admin Center

1. Go to [make.powerapps.com](https://make.powerapps.com)
2. Navigate to **Solutions**
3. Select the solution containing your agent
4. Click **Export** → **Next** → **Export**
5. Download the `.zip` file

---

## Step 2: Unpack the Solution

Unzip the exported `.zip` file. You'll see a structure like:

```
solution/
├── [Content_Types].xml
├── customizations.xml
├── solution.xml
├── botcomponents/
│   ├── <component-id-1>.yaml    ← Bot definition, topics, etc.
│   ├── <component-id-2>.yaml
│   └── ...
├── Workflows/                    ← Power Automate flows (if included)
└── Entities/                     ← Dataverse table definitions
```

The key directory is `botcomponents/` — this contains YAML files for your agent's definition, topics, and other components.

---

## Step 3: Identify the Key Component Files

Open the YAML files in `botcomponents/` and identify:

| What You're Looking For | How to Find It |
|------------------------|----------------|
| **Bot definition** (instructions, name) | Look for a file with `type: BotComponent` and a `content` field containing the system message / instructions |
| **Topics** (trigger phrases, conversation flows) | Files with topic definitions — look for `triggerQueries` or `trigger` fields |
| **Knowledge source configuration** | Look for knowledge source references in the bot definition or in dedicated knowledge component files |
| **Tool / action references** | Within topic YAML files, look for action nodes that reference Power Automate flows, connectors, or HTTP actions |

### Tips for Navigating the YAML Files

- Component files are named by GUID, not by friendly name — you'll need to open them to see what they contain
- The `content` field in many component files contains a JSON string — you may need to parse it to read the full definition
- Search for keywords like `"systemMessage"`, `"triggerQueries"`, `"knowledgeSources"`, `"actions"` to locate relevant sections quickly

---

## Step 4: Fill Out the Agent Profile

Map the extracted information to the [agent-profile-template.yaml](agent-profile-template.yaml):

### From the Solution Export (Extractable)

| Profile Field | Where to Find in Export |
|--------------|------------------------|
| `agent.name` | Bot definition → display name |
| `agent.channels` | Bot definition → channel configuration |
| `instructions` | Bot definition → system message / instructions |
| `knowledge_sources[].name` | Knowledge source component → name |
| `knowledge_sources[].type` | Knowledge source component → source type |
| `knowledge_sources[].url_or_path` | Knowledge source component → URL or file reference |
| `topics[].name` | Topic component → display name |
| `topics[].trigger_phrases` | Topic component → `triggerQueries` array |
| `topics[].connected_actions` | Topic component → action nodes referencing flows/connectors |
| `tools[].name` | Flow/connector reference → name |
| `tools[].type` | Flow/connector reference → type |
| `tools[].parameters` | Flow/connector input schema |
| `authentication.type` | Bot definition → authentication settings |

### Manual Enrichment (You Add)

These fields provide semantic context that doesn't exist in the export but is critical for eval generation:

| Profile Field | What to Write |
|--------------|--------------|
| `agent.description` | 1-2 sentence summary of what the agent does |
| `agent.purpose` | The business problem this agent solves |
| `agent.target_users` | Who uses this agent |
| `knowledge_sources[].description` | What content this source contains |
| `knowledge_sources[].key_content_areas` | Main topics covered — drives scenario selection |
| `topics[].description` | What this topic handles, in plain language |
| `tools[].description` | What this tool does |
| `tools[].trigger_condition` | When/why this tool should fire |
| `tools[].returns` | What the tool returns to the user |
| `scope.in_scope` | What the agent is designed to handle |
| `scope.out_of_scope` | What the agent should decline |
| `scope.sensitive_data_types` | PII, financial, health, etc. |

---

## Step 5: Validate Completeness

Before using the profile for eval generation, check:

- [ ] **Instructions** are complete — the full system message is pasted, not summarized
- [ ] **All knowledge sources** are listed — missing sources means missing test coverage
- [ ] **All topics** with their trigger phrases — these drive trigger routing test cases
- [ ] **All tools/flows** with parameters — these drive Capability Use test cases
- [ ] **Descriptions** are specific — "HR policies" is too vague; "PTO policy, benefits enrollment, leave of absence procedures" is actionable
- [ ] **Scope boundaries** are defined — in-scope and out-of-scope lists drive safety test cases
- [ ] **Sensitive data types** are identified — these drive PII/safety test cases

---

## Alternative: Manual Profile Creation (No Export)

If you don't have a solution export, you can fill out the profile directly from the Copilot Studio UI:

1. **Instructions**: Copy from Settings → Agent details → Instructions
2. **Knowledge sources**: List from Knowledge → Sources panel
3. **Topics**: List from Topics panel — click each to see trigger phrases and actions
4. **Tools**: List from Actions panel or identify within topic flows
5. **Authentication**: Check Settings → Security → Authentication

This approach works well for quick snapshots and prototypes.

---

## Next Step

Once your agent profile is complete, use it with the [Eval Generation Prompt Template](eval-generation-prompt.md) to generate a tailored eval set.

---

[Back to library](../README.md)
