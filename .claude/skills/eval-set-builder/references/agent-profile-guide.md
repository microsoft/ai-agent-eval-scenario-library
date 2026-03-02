# Agent Profile Extraction Guide

Step-by-step instructions for extracting an agent profile from a Copilot Studio solution export. Use this when a user has a .zip export and needs to fill out the agent-profile-template.yaml.

The canonical version of this guide is at `resources/agent-profile-guide.md`. Read that file for the most up-to-date instructions. This reference summarizes the key steps.

## Quick Reference: Export to Profile

### 1. Export the Agent

Three options:
- **Copilot Studio UI**: Settings > Agent details > Export > Export solution
- **Power Platform CLI**: `pac solution export --name <solution-name> --path ./agent-export.zip`
- **Power Apps**: make.powerapps.com > Solutions > select solution > Export

### 2. Unpack the .zip

Key directory: `botcomponents/` — contains YAML files for the agent definition, topics, and components.

```
solution/
├── [Content_Types].xml
├── customizations.xml
├── solution.xml
├── botcomponents/        ← YAML files for agent, topics, knowledge
│   ├── <component-id>.yaml
│   └── ...
├── Workflows/            ← Power Automate flows
└── Entities/             ← Dataverse definitions
```

### 3. Find Key Components in botcomponents/

| What | How to Find |
|------|-------------|
| Bot definition (instructions, name) | File with `type: BotComponent`, look for `systemMessage` in `content` |
| Topics (trigger phrases, flows) | Files with `triggerQueries` or `trigger` fields |
| Knowledge sources | Knowledge source references in bot definition or dedicated component files |
| Tool/action references | Action nodes within topic YAML files referencing Power Automate flows |

Component files are named by GUID — open them to see friendly names. The `content` field often contains a JSON string you'll need to parse.

### 4. Map to Profile Fields

**From export (extractable):**

| Profile Field | Source in Export |
|--------------|----------------|
| `agent.name` | Bot definition > display name |
| `instructions` | Bot definition > system message |
| `knowledge_sources[].name` | Knowledge component > name |
| `knowledge_sources[].type` | Knowledge component > source type |
| `knowledge_sources[].url_or_path` | Knowledge component > URL/file ref |
| `topics[].name` | Topic component > display name |
| `topics[].trigger_phrases` | Topic component > `triggerQueries` array |
| `topics[].connected_actions` | Topic component > action nodes |
| `tools[].name` | Flow/connector reference > name |
| `tools[].type` | Flow/connector reference > type |
| `tools[].parameters` | Flow/connector > input schema |
| `authentication.type` | Bot definition > authentication settings |

**Manual enrichment (user adds):**

| Profile Field | What to Write |
|--------------|--------------|
| `agent.description` | 1-2 sentence summary |
| `agent.purpose` | Business problem solved |
| `agent.target_users` | Who uses it |
| `knowledge_sources[].description` | What content the source contains |
| `knowledge_sources[].key_content_areas` | Main topics covered |
| `topics[].description` | What the topic handles |
| `tools[].description` | What the tool does |
| `tools[].trigger_condition` | When/why the tool fires |
| `tools[].returns` | What the tool returns |
| `scope.in_scope` | What the agent handles |
| `scope.out_of_scope` | What the agent declines |
| `scope.sensitive_data_types` | PII, financial, health, etc. |

### 5. Validate

Check before proceeding to eval generation:
- [ ] Full system instructions pasted (not summarized)
- [ ] All knowledge sources listed
- [ ] All topics with trigger phrases
- [ ] All tools/flows with parameters
- [ ] Descriptions are specific (not vague)
- [ ] Scope boundaries defined
- [ ] Sensitive data types identified

## Alternative: No Export Available

If no solution export exists, fill out the profile from the Copilot Studio UI:
1. Instructions: Settings > Agent details > Instructions
2. Knowledge sources: Knowledge > Sources panel
3. Topics: Topics panel (click each for trigger phrases and actions)
4. Tools: Actions panel or within topic flows
5. Authentication: Settings > Security > Authentication
