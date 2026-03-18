# MCP & Function-Calling Evaluation

> Advanced evaluation scenarios for agents that use Model Context Protocol (MCP) servers, function calling APIs, or plugin-style tool integrations. Covers tool discovery, schema compliance, multi-hop tool chains, fault handling, and function-calling accuracy dimensions that go beyond basic tool invocation testing.

[Back to library](../README.md) | [All capability scenarios](README.md)

> **Prerequisite:** This guide assumes you have already covered the basics in [Tool & Connector Invocations](tool-and-connector-invocations.md). That guide tests whether the right tool fires, parameters are collected, outputs are parsed, and errors are handled. This guide goes further — testing the reliability, accuracy, and robustness of the tool integration layer itself.

---

## 1. Tool Discovery & Registration

Can the agent correctly discover, register, and select from dynamically available tools?

### When to Use

Your agent connects to MCP servers, plugin registries, or dynamic tool catalogs where the set of available tools can change at runtime. Unlike static tool configurations (where every tool is hardcoded), dynamic discovery means the agent must interpret tool descriptions, understand capabilities, and select the right tool from a potentially large and changing catalog.

> **Why this matters:** Research on MCP fault patterns shows that tool discovery and registration issues are the most critical fault category — 32% of all critical-severity MCP faults involve tools that are incorrectly discovered, misregistered, or unavailable when needed. A tool that silently fails to register means the agent has a blind spot it doesn't know about.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the correct tool from the dynamic catalog was selected and invoked |
| Compare Meaning | Verifies the agent explains its tool selection rationale when asked |
| General Quality | Assesses whether the agent handles catalog changes gracefully |

### Setup Steps

1. Configure your agent with a tool catalog containing **at least 10 tools** — enough to test selection from a meaningful set.
2. Create test cases where the user request could plausibly match 2–3 tools with similar descriptions. Verify the agent picks the best match.
3. Test with tool catalog changes mid-session: add a new tool, remove an existing one, or update a tool's description. Verify the agent adapts.
4. Test with a tool that has an identical name but different schema/capabilities to another (namespace collision). Verify the agent distinguishes them.
5. Run the eval set. Failures indicate the agent cannot reliably interpret tool descriptions or handle catalog dynamics.

### Anti-Pattern

> **Anti-Pattern: Testing Only with a Small, Static Tool Set**
> If your test environment has 3 hardcoded tools, you'll never catch discovery failures. Production MCP servers may expose 50+ tools with overlapping descriptions. Test at scale and with catalog changes to expose selection and registration issues.

### Evaluation Patterns

**Pattern A: Description-Based Selection**
Present the agent with tools that have overlapping domains but different capabilities. Example: `search_documents` (keyword search) vs. `semantic_search` (embedding-based) vs. `search_by_date` (temporal filter). Given "find the latest quarterly report," verify the agent selects `search_by_date` — not just the first search tool.

**Pattern B: Catalog Dynamics**
Mid-conversation, a new tool becomes available (e.g., an MCP server reconnects). Verify the agent discovers and can use the new tool without requiring a restart or re-initialization. Conversely, test that the agent handles a tool disappearing gracefully.

**Pattern C: Namespace Collision Resolution**
Two MCP servers expose tools with similar names (e.g., both have a `get_status` tool). Verify the agent uses the correct server-qualified name and routes to the right endpoint.

**Pattern D: Tool Description Quality Sensitivity**
Test with deliberately vague or incomplete tool descriptions. This reveals how dependent the agent is on high-quality tool metadata. Example: a tool described only as "process data" versus one described as "parse CSV files and return structured JSON with column headers as keys."

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Agent selects from overlapping search tools | "Find documents about Q4 revenue from last week" | Capability: `search_by_date` (not generic `search_documents`) | Capability Use (All) + Compare Meaning |
| 2 | Agent uses newly available tool | "Can you check the build status?" (CI server just connected) | Capability: `ci_server.get_build_status` | Capability Use (All) + Compare Meaning |
| 3 | Agent handles tool disappearance | "Run the code formatter" (formatter server disconnected) | Response explains the tool is currently unavailable and suggests alternatives | Compare Meaning + General Quality |
| 4 | Agent distinguishes namespaced tools | "Get the project status" (both PM and CI servers have `get_status`) | Capability: `pm_server.get_status` (for project management context) | Capability Use (All) + Compare Meaning |
| 5 | Agent handles vague tool description | "Analyze this data" (only vaguely described tools available) | Agent asks clarifying question or explains available options | Compare Meaning + General Quality |

### Tips

- Test with **realistic catalog sizes** — 10+ tools minimum, 50+ if your production environment is large.
- Tool description quality is the #1 factor in selection accuracy. If your agent fails selection tests, improve tool descriptions before blaming the model.
- Monitor **tool registration latency** — a tool that takes 5 seconds to register may be missed in fast-paced conversations.
- Target: **90% or higher correct tool selection** when tools have clear, non-overlapping descriptions. Expect lower accuracy (70–80%) with deliberately ambiguous descriptions — use this as a benchmark for description quality improvement.

---

## 2. Schema Compliance & Parameter Validation

Does the agent generate function calls that conform to the tool's declared schema — correct types, required fields, valid enum values, and proper nesting?

### When to Use

Your agent generates structured function calls with typed parameters (JSON Schema, OpenAPI specs, or MCP tool input schemas). This scenario catches a critical failure mode: the agent "calls" the right tool but passes malformed parameters that cause silent failures or unexpected behavior downstream.

> **Why this matters:** Research on function-calling leaderboards (BFCL V4) shows schema compliance is a core evaluation dimension distinct from tool selection. An agent can select the correct tool 95% of the time but still fail 20% of calls due to type mismatches, missing required fields, or invalid enum values. Structured output benchmarks reveal that even top models struggle with deeply nested schemas — accuracy drops significantly beyond 3 levels of nesting.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verifies specific parameter values appear in the function call or confirmation |
| Compare Meaning | Checks that parameter semantics are correct (e.g., "next Friday" correctly resolved to a date) |
| General Quality | Assesses whether the agent handles schema constraints naturally without exposing technical details |

### Setup Steps

1. For each tool, document the full input schema: required fields, types, enums, nested objects, array constraints.
2. Create test cases that exercise each schema constraint:
   - Required field: user omits it → agent should ask
   - Type coercion: user says "five" → agent should pass `5` (integer)
   - Enum validation: user says a synonym → agent should map to the valid enum value
   - Nested objects: user provides flat data → agent should structure it correctly
3. Test with schemas of increasing complexity: flat (3–5 fields), nested (2 levels), deeply nested (3+ levels).
4. Run the eval set. Failures indicate the agent is generating structurally invalid function calls.

### Anti-Pattern

> **Anti-Pattern: Testing Only with Flat Schemas**
> If every test case uses simple tools with 2–3 string parameters, you'll never catch nesting, type coercion, or array handling failures. Real APIs have complex schemas — test with the actual complexity your tools require.

### Evaluation Patterns

**Pattern A: Type Coercion Accuracy**
The user provides parameters in natural language that must be converted to specific types. Test: "Set the priority to high" should map to `priority: 3` (if the schema uses numeric enums) or `priority: "HIGH"` (if string enum). "Schedule it for next Friday at 3pm" should produce valid ISO 8601 datetime.

**Pattern B: Required vs. Optional Field Handling**
Present a request with only required fields provided. Verify the agent calls the tool without asking for optional fields. Then present a request missing a required field — verify the agent asks for it rather than passing null or a default.

**Pattern C: Nested Object Construction**
Tools that accept complex inputs (e.g., `{ "filter": { "date_range": { "start": "...", "end": "..." }, "categories": ["A", "B"] } }`). Verify the agent constructs the nested structure correctly from a flat natural-language request.

**Pattern D: Array Parameter Handling**
The user provides a list of items ("send it to Alice, Bob, and Carol"). Verify the agent constructs a proper array parameter, not a comma-separated string.

**Pattern E: Enum Value Mapping**
The tool accepts specific enum values (`"LOW"`, `"MEDIUM"`, `"HIGH"`) but the user says "not urgent." Verify the agent maps to the correct enum value or asks for clarification if ambiguous.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Natural date converted to ISO format | "Schedule the meeting for next Friday at 2pm" | Parameter `datetime` is valid ISO 8601 | Keyword Match (All) + Compare Meaning |
| 2 | Numeric enum mapped correctly | "Set priority to high" | Parameter `priority` = `3` or `"HIGH"` (per schema) | Keyword Match (All) |
| 3 | Missing required field prompts collection | "Create a ticket" (missing title and category) | Agent asks for title and category before calling tool | Keyword Match (All) + Compare Meaning |
| 4 | Nested filter object constructed correctly | "Find sales reports from January in the EMEA region" | Parameters include nested `filter.date_range` and `filter.region` | Compare Meaning + Keyword Match (All) |
| 5 | Array parameter from natural language list | "Notify Alice, Bob, and Carol" | Parameter `recipients` = `["Alice", "Bob", "Carol"]` (array, not string) | Keyword Match (All) + Compare Meaning |
| 6 | Synonym mapped to valid enum | "Make it low priority" | Parameter `priority` maps to valid enum value, not literal "low priority" | Keyword Match (All) |

### Tips

- Test schemas at **3 complexity tiers**: flat (≤5 fields), nested (2 levels), deeply nested (3+ levels). Accuracy typically drops 15–25% per nesting level.
- For APIs with strict validation, a schema compliance failure causes a hard error. For lenient APIs, it causes silent data corruption — which is worse. Test both.
- Track **schema compliance rate** as a metric: (valid function calls) / (total function calls). Target: **98%+ for flat schemas, 90%+ for nested schemas**.
- If your agent consistently fails on nested schemas, consider simplifying tool interfaces or adding wrapper tools that flatten the input.

---

## 3. Multi-Hop Tool Chains

When the answer requires calling multiple tools where each call depends on the previous result, does the agent plan and execute the full chain correctly?

### When to Use

Your agent handles questions that cannot be answered by a single tool call — the agent must decompose the question, call tools in sequence, pass intermediate results forward, and synthesize a final answer. This goes beyond the multi-tool orchestration in [Tool & Connector Invocations](tool-and-connector-invocations.md) by testing the agent's planning and reasoning across complex dependency chains.

> **Why this matters:** BFCL V4 introduced "multi-step reasoning" as a distinct evaluation dimension, finding that agents which excel at single-tool calls often fail at chains of 3+ dependent calls. The primary failure mode is "chain collapse" — the agent makes the first 1–2 calls correctly but loses track of intermediate state, skips steps, or fabricates results for later steps instead of making the actual tool calls.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms every tool in the chain was invoked — catches dropped steps |
| Keyword Match (All) | Verifies data from each step appears in the final answer — catches lost intermediate results |
| Compare Meaning | Checks that the final synthesized answer correctly combines information from all steps |
| General Quality | Assesses whether the multi-step process feels coherent and complete to the user |

### Setup Steps

1. Map your tool-chain scenarios: identify questions that require 2, 3, and 4+ sequential tool calls.
2. For each chain, document the expected sequence: Tool A → result feeds Tool B → result feeds Tool C → final answer.
3. Create test cases with clear data dependencies — the answer from step 1 *must* be used as input to step 2 (don't allow the agent to skip steps).
4. Add verification for intermediate state: confirm that the correct value from Tool A was passed to Tool B (not a hallucinated or default value).
5. Test with chains that include a branching decision point (the result of Tool A determines whether to call Tool B or Tool C).
6. Run the eval set. The most common failure is "chain collapse" at step 3+.

### Anti-Pattern

> **Anti-Pattern: Accepting Correct Final Answers Without Verifying the Chain**
> An agent might produce a correct-looking answer by skipping intermediate steps and hallucinating results. If you only check the final answer, you miss that the agent called 1 of 4 required tools and made up the rest. Always verify the *full chain* was executed using Capability Use (All).

### Evaluation Patterns

**Pattern A: Linear Chain Execution**
A straightforward sequence: look up customer → get their orders → find the latest order → check shipping status. Verify all 4 tools fire and the final answer contains the shipping status for the correct order (not any order).

**Pattern B: Branching Chain**
The result of an intermediate step determines the next action. Example: check if a user is an admin → if yes, call the admin dashboard API; if no, call the standard dashboard API. Verify the agent takes the correct branch.

**Pattern C: Chain with Aggregation**
Multiple parallel tool calls feed into a synthesis step. Example: get weather for 3 cities → compare them → recommend the best destination. Verify all parallel calls complete and the recommendation references specific data from each.

**Pattern D: Error Recovery Mid-Chain**
One step in the chain fails. Verify the agent reports what it accomplished before the failure and doesn't fabricate results for the remaining steps. Example: customer lookup succeeds, order lookup fails → agent should report the customer was found but orders couldn't be retrieved.

**Pattern E: Chain State Preservation**
In a long chain, verify that values from early steps are still correctly referenced in later steps. This catches "context drift" where the agent forgets or misremembers intermediate results.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | 3-step linear chain: customer → orders → shipping | "What's the shipping status of Jane Smith's most recent order?" | Capabilities (all): `lookup_customer`, `get_orders`, `get_shipping_status` | Capability Use (All) + Keyword Match (All) |
| 2 | Chain passes correct intermediate value | (Same as above) | Order ID from step 2 matches the ID passed to step 3 | Keyword Match (All) + Compare Meaning |
| 3 | Branching chain: role check → appropriate dashboard | "Show me the system dashboard" | Agent checks role first, then calls correct dashboard API | Capability Use (All) + Compare Meaning |
| 4 | Parallel aggregation: multi-city weather | "Compare weather in NYC, London, and Tokyo for next week" | Capabilities (all): 3 weather API calls + synthesized comparison | Capability Use (All) + Compare Meaning |
| 5 | Mid-chain failure handled correctly | "Get Jane Smith's order history and refund status" (refund API down) | Agent reports orders found, explains refund check failed | Compare Meaning + General Quality |
| 6 | State preserved across 4-step chain | "Find the cheapest flight from NYC to London next Friday and book it" | All steps reference consistent dates, airports, and flight IDs | Capability Use (All) + Keyword Match (All) |

### Tips

- Start with **2-step chains** and increase to 3 and 4 steps. Most agents degrade noticeably at 3+ steps.
- Track **chain completion rate**: (fully completed chains) / (total chain attempts). Target: **95%+ for 2-step chains, 85%+ for 3-step chains, 75%+ for 4+ step chains**.
- "Chain collapse" is most common when intermediate results are large or complex. Test with realistic data volumes.
- If your agent consistently fails at step N, consider breaking the chain into smaller, explicit sub-tasks.
- Time-out chains appropriately — a 4-step chain takes 4× the latency of a single call.

---

## 4. Tool Response Handling & Format Sensitivity

Does the agent correctly interpret tool responses across different formats, edge cases, and unexpected structures?

### When to Use

Your agent processes responses from tools that may return data in varying formats (JSON, XML, plain text, markdown), include unexpected fields, return empty results, or produce very large payloads. This scenario catches a critical MCP fault category — tool response handling accounts for 67% of all frequent faults in MCP systems.

> **Why this matters:** Research on MCP fault patterns found that tool response handling is the most frequent fault category. Common issues: the agent ignores unexpected fields, truncates large responses, misparses nested data, or fails silently on format variations. BFCL V4 also identified "format sensitivity" as a key dimension — small changes in tool response structure can cause disproportionate accuracy drops.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verifies the agent correctly interprets tool data regardless of format variations |
| Keyword Match (All) | Confirms specific data values are extracted correctly from various response formats |
| General Quality | Assesses whether the agent presents the data clearly despite format challenges |

### Setup Steps

1. Identify the response formats your tools return: JSON, XML, plain text, CSV, mixed content.
2. Create test cases for each format variation:
   - Standard format: tool returns expected JSON → agent parses correctly
   - Extra fields: tool returns JSON with new/unknown fields → agent ignores them gracefully
   - Missing fields: tool returns JSON with optional fields omitted → agent doesn't hallucinate values
   - Large payload: tool returns 100+ records → agent summarizes or paginates
   - Empty result: tool returns `[]` or `{}` → agent communicates "no results" clearly
3. Test format sensitivity: same data returned as JSON vs. markdown table vs. plain text. Verify consistent interpretation.
4. Run the eval set. Failures indicate the agent is fragile to response format changes.

### Anti-Pattern

> **Anti-Pattern: Testing Only with Perfectly Formatted Responses**
> In production, tool responses are messy. APIs return extra debugging fields, null values where you expect strings, arrays with 0 or 1000 items, and inconsistent date formats. If you only test with clean, predictable tool outputs, you'll miss the response-handling failures that occur in real deployments.

### Evaluation Patterns

**Pattern A: Unexpected Field Robustness**
The tool response includes fields the agent hasn't seen before (e.g., a new `metadata` object). Verify the agent doesn't crash, doesn't hallucinate meanings for unknown fields, and still extracts the known fields correctly.

**Pattern B: Missing Field Graceful Handling**
An optional field that's usually present is missing from the response. Verify the agent doesn't show "null" or "undefined" to the user and handles the absence naturally.

**Pattern C: Large Payload Summarization**
The tool returns 50+ records. Verify the agent summarizes, filters to the most relevant, or offers pagination — rather than dumping all records into a wall of text.

**Pattern D: Format Variation Consistency**
The same semantic data is returned in different formats across test cases. Verify the agent extracts the same meaning regardless of whether the data comes as JSON, a markdown table, or plain text.

**Pattern E: Null and Edge Value Handling**
The tool returns edge values: empty strings, zero values, negative numbers, very long strings, special characters, or Unicode. Verify the agent handles each without corruption.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Agent ignores unexpected fields | "Get my profile" (API returns extra `_debug` field) | Response includes name, email, role — does not reference `_debug` | Keyword Match (All) + Compare Meaning |
| 2 | Agent handles missing optional field | "Get order status" (API omits optional `estimated_delivery`) | Response shows status without mentioning delivery date; no "null" or "N/A" | Compare Meaning + General Quality |
| 3 | Agent summarizes large result set | "List all my transactions" (API returns 200 records) | Response summarizes top/recent items and offers to show more | Compare Meaning + General Quality |
| 4 | Same data extracted from different formats | "Get team members" (JSON in test A, markdown table in test B) | Both produce the same list of team members | Compare Meaning + Keyword Match (All) |
| 5 | Agent handles zero-result response | "Find matching products" (API returns empty array) | "No matching products found" — not an error or empty response | Compare Meaning + General Quality |
| 6 | Special characters preserved | "Get the project name" (API returns `"Müller & Söhne — Q4 (2026)"`) | Response preserves the exact project name with special characters | Keyword Match (All) |

### Tips

- Test with **at least 3 format variations** per tool if the tool's response format could change (API version upgrades, MCP server updates).
- Track a **response handling success rate**: (correctly parsed responses) / (total tool responses). Target: **98%+ for standard formats, 90%+ for unexpected variations**.
- Pay special attention to **date and currency formats** — these are the most common format sensitivity failures.
- Large payload handling is critical for production agents. Test with realistic data sizes, not just 3-record samples.
- If your agent fails on format variations, consider adding a response normalization layer between the tool and the agent.

---

## 5. Relevance Detection & Refusal

When the user's request doesn't match any available tool, does the agent correctly refuse to make a function call — rather than forcing an irrelevant tool or hallucinating a tool that doesn't exist?

### When to Use

Your agent has a defined set of tools, and users will inevitably ask questions or make requests that fall outside what any tool can handle. This scenario tests the agent's ability to recognize when NO tool is appropriate and respond helpfully without making a spurious function call.

> **Why this matters:** BFCL V4 introduced "relevance detection" as a distinct evaluation category. The failure mode is costly: an agent that forces an irrelevant tool call may execute an unintended action (creating a ticket when the user just wanted information), waste API quota, or return confusing results. This is the tool-calling equivalent of hallucination — the agent fabricates an action instead of admitting it can't help.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (Any) | Set as negative check — test passes if NO tool is called |
| Compare Meaning | Verifies the agent provides a helpful response explaining it can't perform the action |
| General Quality | Assesses whether the refusal is constructive and suggests alternatives |

### Setup Steps

1. List your agent's tools and their domains. Identify request categories that fall clearly outside all tools.
2. Create test cases in three categories:
   - **Clearly out-of-scope**: "What's the weather in Paris?" (no weather tool)
   - **Adjacent to a tool's domain**: "How does the ticketing system work?" (has `create_ticket` tool but no informational tool about the system)
   - **Ambiguous but no good match**: vague requests that could be misinterpreted as tool-worthy
3. For each test case, verify NO tool is called AND the response is helpful (not just "I can't do that").
4. Run the eval set. Failures include: forced tool calls, hallucinated tools, or unhelpful refusals.

### Anti-Pattern

> **Anti-Pattern: Never Testing Irrelevant Requests**
> If every test case has a matching tool, you'll never discover that your agent forces irrelevant tool calls on out-of-scope requests. In production, 20–40% of user requests may not match any available tool. Test this explicitly.

### Evaluation Patterns

**Pattern A: Clear Out-of-Scope Request**
Request something completely outside the agent's tool capabilities. Verify the agent responds helpfully without calling any tool.

**Pattern B: Adjacent Domain Confusion**
Request information *about* a tool's domain rather than an action *using* the tool. Example: "What types of tickets can I create?" should not trigger `create_ticket`. The agent should answer from knowledge or explain its limitations.

**Pattern C: Hallucinated Tool Detection**
Present an ambiguous request and verify the agent doesn't invent a non-existent tool. Example: the agent should not claim it will call a `get_weather` function if no such tool exists.

**Pattern D: Helpful Refusal**
When refusing, the agent should explain what it *can* do. Not just "I can't help with that" but "I don't have a tool for weather, but I can help with travel bookings, expense reports, or scheduling."

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | No weather tool, don't force one | "What's the weather like in London?" | NO tool called; helpful response suggesting alternatives | Capability Use (Any) — negative + Compare Meaning |
| 2 | Information about a tool's domain | "How does the expense system work?" | NO tool called (don't call `submit_expense`); informational response | Capability Use (Any) — negative + Compare Meaning |
| 3 | Vague request doesn't force a tool | "Help me with something" | NO tool called; agent asks clarifying question | Capability Use (Any) — negative + General Quality |
| 4 | Agent doesn't hallucinate tools | "Can you translate this to French?" (no translation tool) | Agent does NOT claim to call a translation API | Compare Meaning + General Quality |
| 5 | Refusal includes what agent CAN do | "Play music for me" (no music tool) | Response lists the agent's actual capabilities | Compare Meaning + General Quality |

### Tips

- Aim for a **false positive tool call rate below 5%** — meaning fewer than 1 in 20 irrelevant requests triggers a tool call.
- Test with at least **10 out-of-scope requests** across different domains to establish a reliable baseline.
- The quality of the refusal matters: a helpful refusal ("I can't check weather, but I can help with...") is much better than a dead-end ("I can't do that").
- Monitor this in production — rising false positive rates often indicate tool description creep (descriptions becoming too broad).

---

## 6. Documentation & Server Configuration Faults

Does the agent behave correctly when tool documentation is misleading, incomplete, or when server configuration has issues?

### When to Use

Your agent relies on tool descriptions, API documentation, and server configurations that may be inaccurate, outdated, or ambiguous. This scenario tests resilience to the "other side" of tool integration — not just calling tools correctly, but handling the real-world mess of imperfect documentation and configuration.

> **Why this matters:** MCP fault taxonomy research found that documentation faults account for 7% of all MCP issues and server/tool configuration faults account for 32%. These are often invisible — the tool "works" but produces unexpected behavior because the description doesn't match the implementation, or the server configuration silently affects behavior.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verifies the agent produces correct results despite documentation issues |
| General Quality | Assesses whether the agent handles configuration problems gracefully |
| Keyword Match (Any) | Checks for appropriate warnings when the agent detects inconsistencies |

### Setup Steps

1. Identify tools with documentation gaps: missing parameter descriptions, ambiguous return types, outdated examples.
2. Create test cases with deliberately degraded documentation:
   - Tool with a vague description: "Processes data" (no detail on what data or how)
   - Tool with a misleading parameter name: `id` field that actually expects an email address
   - Tool with missing return type documentation
3. Test server configuration scenarios:
   - Tool that times out inconsistently (simulating flaky MCP server connections)
   - Tool that requires authentication but auth is misconfigured
   - Tool with rate limits that are hit during normal use
4. Run the eval set. Failures indicate the agent blindly trusts documentation or crashes on configuration issues.

### Anti-Pattern

> **Anti-Pattern: Testing Only with Perfect Documentation**
> In production, tool descriptions are often written by developers who assume context the agent doesn't have. API docs go stale. Server configs drift. If your tests assume perfect documentation and configuration, you'll miss the failure modes that matter most in real deployments.

### Evaluation Patterns

**Pattern A: Ambiguous Description Handling**
Tool description is vague. Verify the agent either asks the user for clarification, infers intent from context, or explains uncertainty — rather than blindly calling the tool with guessed parameters.

**Pattern B: Description-Implementation Mismatch**
The tool description says it does one thing, but testing reveals it does something slightly different. Verify the agent surfaces the discrepancy to the user when results don't match expectations.

**Pattern C: Timeout and Retry Behavior**
An MCP server responds slowly or intermittently times out. Verify the agent retries appropriately (not infinitely), communicates delays to the user, and eventually gracefully degrades.

**Pattern D: Authentication and Permission Errors**
The tool call fails due to auth or permission issues. Verify the agent explains this clearly without leaking credential details, and suggests remediation.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Vague tool description → agent clarifies | "Process this data" (tool desc: "Processes data") | Agent asks what kind of processing is needed | Compare Meaning + General Quality |
| 2 | Misleading parameter handled | "Look up user john@example.com" (tool `id` field expects email) | Agent passes email to `id` field correctly based on context | Compare Meaning + Keyword Match (All) |
| 3 | Intermittent timeout with retry | "Get the report" (server times out on first try) | Agent retries and either succeeds or explains the timeout | Compare Meaning + General Quality |
| 4 | Auth failure without credential leak | "Access the admin panel" (misconfigured auth) | Agent explains access issue without mentioning tokens/keys | General Quality + Compare Meaning |
| 5 | Rate limit communicated clearly | "Run all 50 analyses" (hits rate limit after 10) | Agent reports progress (10 completed) and explains the rate limit | Compare Meaning + General Quality |

### Tips

- Include **at least 2 documentation-degradation test cases** per tool to catch documentation-related failures early.
- Server configuration faults are the most common production issues. Simulate timeouts, auth failures, and rate limits in every eval suite.
- Track a **documentation quality score** for each tool: (successful calls with no user confusion) / (total calls). Tools with scores below 85% need documentation improvements.
- Consider creating a **tool health dashboard** that tracks per-tool error rates, timeouts, and user confusion incidents.

---

## Connecting These Scenarios

These six scenarios form a progression from basic to advanced tool evaluation:

```
Tool Discovery (1) → Schema Compliance (2) → Multi-Hop Chains (3)
         ↓                    ↓                       ↓
   Can the agent         Does it call            Can it chain
   find and select       tools with valid         multiple calls
   the right tool?       parameters?              with dependencies?
         ↓                    ↓                       ↓
Response Handling (4) → Relevance Detection (5) → Doc/Config Faults (6)
         ↓                    ↓                       ↓
   Can it parse          Does it know when       Is it resilient to
   tool responses        NOT to call a tool?      imperfect tool metadata?
   robustly?
```

**Start with Scenarios 2 and 5** — schema compliance and relevance detection are the highest-ROI areas. If your agent generates valid function calls and knows when not to call tools, you've addressed the two most common production failures.

**Then add Scenario 4** (response handling) — this catches the 67% of MCP faults related to processing tool responses.

**Finally, add Scenarios 1, 3, and 6** for comprehensive coverage of dynamic environments, complex chains, and real-world configuration challenges.

### Relationship to Existing Tool & Connector Invocations

| Tool & Connector Invocations (basic) | This Guide (advanced) |
|--------------------------------------|----------------------|
| Does the right tool fire? | Can the agent discover and select from 50+ dynamic tools? |
| Are parameters collected from the user? | Do generated parameters conform to complex schemas? |
| Is tool output presented clearly? | Is the agent robust to format variations and large payloads? |
| Does multi-tool orchestration work? | Can the agent plan and execute 4+ step dependency chains? |
| Is error handling graceful? | Is the agent resilient to documentation and configuration faults? |
| — | Does the agent know when NOT to call any tool? |

---

## References

Research and benchmarks referenced in this guide:

- **BFCL V4** (Berkeley Function Calling Leaderboard V4): Multi-dimensional function calling evaluation across 6 dimensions — simple calls, parallel invocations, multiple function selection, relevance detection, multi-turn interactions, and multi-step reasoning. 2000+ test pairs with agentic categories.
- **MCP Fault Taxonomy** (2025): Comprehensive analysis of 419 real faults across MCP software. Five fault categories: Server Setting (27%), Server/Tool Config (32%), Server/Host Config (29%), Documentation (7%), General Programming (5%). Tool response handling most frequent (67%), tool discovery most critical (32% rated critical).
- **MCPVerse** (2025): Benchmark for evaluating language model interaction with MCP-based tool ecosystems. Best model accuracy: 57.77%, highlighting the challenge of real-world tool use.
- **JSONSchemaBench** (2025): 10,000+ real-world JSON schemas for evaluating structured output compliance. Shows accuracy degradation with schema complexity.
