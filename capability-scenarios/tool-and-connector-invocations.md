# Tool & Connector Invocations

> Scenarios for verifying that your agent calls the correct tools, Power Automate flows, connectors, or APIs with the right parameters and handles the results appropriately. Applies to any agent that executes actions beyond answering questions.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## 1. Verifying Tool Invocation

Does the right tool, flow, connector, or API fire for the given user request?

### When to Use

Your agent has one or more tools configured (Power Automate flows, custom connectors, APIs, plugins) and you need to confirm the correct one fires when a user makes a request. This is the most fundamental tool-testing scenario — if the wrong tool fires (or none fires), everything downstream is wrong.

> **Related scenarios:** If your concern is whether the agent routes to the correct *topic* (not tool), see [Trigger Routing](trigger-routing.md). If you need to verify the tool's *output* is presented correctly, see [Scenario 4: Verifying Tool Output in Response](#4-verifying-tool-output-in-response) below.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the expected tool/flow/connector was invoked — the primary check for this scenario |
| Keyword Match (Any) | Catches surface-level confirmation language in the response (e.g., "your request has been submitted") as a secondary signal |

> **Tip:** A response can sound correct ("Your PTO request has been submitted!") without actually calling the Submit PTO Request flow. Capability Use is the only method that verifies the tool was actually invoked — never rely on response text alone.

### Setup Steps

1. List every tool, flow, connector, and API your agent can invoke. For each, write a clear user request that should trigger it.
2. Create one test case per tool. Set the **Sample Input** to a natural-language request that should invoke that tool.
3. For each test case, set the **Method** to **Capability Use (All)** and the **Expected Capability** to the exact tool/flow name as configured in your agent.
4. Optionally add a **Keyword Match (Any)** check with confirmation language you expect in the response.
5. Run the eval set and review results. Any test case where the expected capability was NOT used indicates a routing or tool-selection failure.

### Anti-Pattern

> **Anti-Pattern: Testing Only by Response Text**
> If you only check whether the response *says* "your PTO request has been submitted" (Keyword Match), you will miss cases where the agent generates a plausible-sounding response without actually calling the flow. Always pair response-text checks with Capability Use to confirm the tool was truly invoked.

### Evaluation Patterns

**Pattern A: Single-Tool Match**
For agents with multiple tools, verify that each user request maps to exactly the right tool. Write one test case per tool with a clear, unambiguous request. This is your baseline coverage.

**Pattern B: Ambiguous Request Resolution**
Write requests that could plausibly match more than one tool (e.g., "I need help with my order" could trigger Get Order Status API or Create Support Ticket connector). Verify the agent selects the correct one based on context or asks a clarifying question.

**Pattern C: Tool Invocation with Contextual Clues**
Test whether the agent uses conversational context to select the right tool. For example, if the user previously said "I placed an order yesterday" and then says "can you check on that?", the agent should invoke Get Order Status API — not Create Support Ticket connector.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | PTO request triggers the correct flow | "I'd like to submit a PTO request for next Friday" | Capability: `Submit PTO Request flow` | Capability Use (All) + Keyword Match (Any) |
| 2 | Order status triggers the correct API | "Where is my order #88234?" | Capability: `Get Order Status API` | Capability Use (All) + Keyword Match (Any) |
| 3 | Ticket creation triggers the correct connector | "I need to open a support ticket for a billing issue" | Capability: `Create Support Ticket connector` | Capability Use (All) + Keyword Match (Any) |
| 4 | Expense report triggers the correct flow | "Submit my expense report for the NYC trip" | Capability: `Submit Expense Report flow` | Capability Use (All) + Keyword Match (Any) |
| 5 | Ambiguous request resolves correctly | "I need to update my address" | Capability: `Update Employee Profile flow` | Capability Use (All) + Compare Meaning |
| 6 | Confirmation language appears in response | "I'd like to submit a PTO request for next Friday" | Keywords (any): "submitted", "confirmed", "PTO request" | Keyword Match (Any) + Capability Use (All) |

### Tips

- Write at least **1 test case per tool** configured in your agent. If your agent has 8 tools, you need at minimum 8 test cases in this scenario.
- Use natural, varied phrasing — don't copy trigger phrases verbatim from your tool descriptions. Real users won't say "invoke the Submit PTO Request flow."
- Rerun this scenario **every time you add, remove, or rename a tool** in your agent configuration.
- Target: **100% of unambiguous tool-invocation test cases should invoke the expected capability.** Any miss here is a critical failure.

---

## 2. Negative Test: Tool Should NOT Be Called

For informational or conversational queries, does the agent avoid unnecessary tool invocation?

### When to Use

Your agent has tools configured but also handles informational queries that should be answered from knowledge sources or general conversation — without calling any tool. This scenario catches over-eager tool invocation, where the agent calls a flow or API when it should just answer the question.

> **Related scenarios:** If you want to verify the agent answers informational queries accurately, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). This scenario focuses specifically on the absence of tool invocation, not the quality of the response itself.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (Any) | Set as a **negative check** — the test passes if NONE of the listed capabilities are used |
| Compare Meaning | Verifies the agent still provides a semantically correct informational response (without needing a tool) |

> **Tip:** A negative Capability Use check is the inverse of Scenario 1. You are confirming the agent did NOT call any tool, not that it called the right one.

### Setup Steps

1. Identify informational queries your users commonly ask that do NOT require tool invocation (e.g., "What is the PTO policy?" vs. "Submit a PTO request").
2. Create test cases with these informational inputs.
3. For each test case, set the **Method** to **Capability Use (Any)** configured as a negative check, listing the tools that should NOT fire.
4. Add a **Compare Meaning** check with the expected informational answer to confirm the agent still provides a useful response.
5. Run the eval set. Any test case where a tool WAS invoked indicates over-eager tool calling.

### Anti-Pattern

> **Anti-Pattern: Skipping Negative Tests Entirely**
> Many teams only test "does the right tool fire?" and never test "does no tool fire when it shouldn't?" This leads to agents that call the Submit PTO Request flow every time someone mentions "PTO" — even for "What is the PTO policy?" Negative tests are essential for agents that mix informational and action-oriented capabilities.

### Evaluation Patterns

**Pattern A: Informational Query About a Tool's Domain**
The user asks about the same domain a tool covers, but in an informational way. Example: "What are the steps to submit an expense report?" should be answered from knowledge — not by invoking the Submit Expense Report flow.

**Pattern B: Casual or Conversational Input**
The user makes small talk, asks for clarification, or says something that doesn't warrant any action. Example: "Thanks, that's helpful" or "Can you explain that again?" should not trigger any tool.

**Pattern C: Partial Intent Without Commitment**
The user expresses interest but does not commit to an action. Example: "I'm thinking about taking time off next month" is exploratory — the agent should not invoke the Submit PTO Request flow without explicit confirmation.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Policy question should not call PTO flow | "What is the company's PTO policy?" | Should NOT use: `Submit PTO Request flow` | Capability Use (Any) — negative + Compare Meaning |
| 2 | Informational query about expenses | "What are the expense report submission deadlines?" | Should NOT use: `Submit Expense Report flow` | Capability Use (Any) — negative + Compare Meaning |
| 3 | Exploratory statement should not trigger action | "I'm thinking about taking Friday off" | Should NOT use: `Submit PTO Request flow` | Capability Use (Any) — negative + Compare Meaning |
| 4 | Conversational reply should not trigger any tool | "Thanks, that makes sense" | Should NOT use: any configured tool | Capability Use (Any) — negative + Compare Meaning |
| 5 | Clarification question should not trigger tool | "What information do I need to submit a ticket?" | Should NOT use: `Create Support Ticket connector` | Capability Use (Any) — negative + Compare Meaning |
| 6 | Informational answer is still correct | "What is the company's PTO policy?" | "Employees receive 15 days of paid time off per year..." | Compare Meaning + Keyword Match (Any) |

### Tips

- For every action-oriented tool, write at least **1 negative test** with an informational query in the same domain.
- Pay special attention to queries that contain tool-related keywords but are informational in nature ("What is..." / "How does..." / "Tell me about...").
- Target: **100% of informational test cases should NOT invoke any tool.** Over-eager tool calling is a user-trust issue — the agent takes action the user did not request.
- If you find your agent is too eager to call tools, review the tool descriptions in your agent configuration — vague descriptions cause false matches.

---

## 3. Verifying Input Collection

Does the agent gather all required parameters before calling the tool?

### When to Use

Your tools require specific input parameters (e.g., dates, employee IDs, order numbers) and you need to verify the agent collects all of them before invoking the tool. This scenario catches premature tool invocation — where the agent calls a flow with missing or default parameters instead of asking the user.

> **Related scenarios:** If your concern is whether the agent presents the tool's results correctly after calling it, see [Scenario 4: Verifying Tool Output in Response](#4-verifying-tool-output-in-response). If the tool call itself fails, see [Scenario 5: Testing Error Handling on Tool Failure](#5-testing-error-handling-on-tool-failure).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms the agent's response asks for ALL required parameters before proceeding |
| General Quality | Assesses whether the agent's parameter-collection conversation is natural and complete |
| Capability Use (All) | Confirms the tool is NOT invoked prematurely (use as negative check on the first turn), then IS invoked after parameters are collected |

> **Tip:** Input collection often spans multiple conversational turns. Design multi-turn test cases where the first turn provides an incomplete request and subsequent turns supply the missing parameters.

### Setup Steps

1. For each tool, list the required input parameters (e.g., Submit PTO Request flow requires: start date, end date, PTO type).
2. Create test cases where the user's initial request is **missing one or more required parameters**.
3. Set the **Expected Value** to confirm the agent asks for the missing parameters (using Keyword Match (All) with the parameter names).
4. Create follow-up test cases where the user provides the missing parameters, and verify the tool is then invoked with Capability Use (All).
5. Run the eval set. Failures indicate the agent either skipped parameter collection or asked for parameters it didn't need.

### Anti-Pattern

> **Anti-Pattern: Testing Only Complete Requests**
> If every test case provides all required parameters upfront ("Submit PTO for Dec 23–27, vacation type"), you never test whether the agent handles incomplete requests. Real users often say "I want to take time off" without specifying dates. Test with partial inputs to verify the agent's collection behavior.

### Evaluation Patterns

**Pattern A: Single Missing Parameter**
The user provides most parameters but omits one. Verify the agent asks for exactly the missing parameter — not all of them again.

**Pattern B: Multiple Missing Parameters**
The user provides only the intent (e.g., "I want to submit an expense report") with no parameters. Verify the agent asks for all required parameters, either in one prompt or through a guided sequence.

**Pattern C: Parameter Validation**
The user provides a parameter in an invalid format (e.g., "next Friday" instead of "2026-03-06"). Verify the agent either interprets it correctly or asks for clarification — not that it passes the ambiguous value to the tool as-is.

**Pattern D: Optional vs. Required Parameter Distinction**
The user provides required parameters but omits optional ones. Verify the agent proceeds with the tool call without unnecessarily asking for optional parameters.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Agent asks for missing dates | "I want to submit a PTO request" | Keywords (all): "start date", "end date" | Keyword Match (All) + Compare Meaning |
| 2 | Agent asks for missing order number | "Can you check on my order?" | Keywords (all): "order number" | Keyword Match (All) + Compare Meaning |
| 3 | Agent asks for missing ticket details | "I need to create a support ticket" | Keywords (all): "description", "category" or "issue type" | Keyword Match (All) + Compare Meaning |
| 4 | Agent does not invoke tool prematurely | "I want to submit a PTO request" (first turn, no dates given) | Should NOT use: `Submit PTO Request flow` | Capability Use (All) — negative + Compare Meaning |
| 5 | Agent invokes tool after parameters collected | "Start date March 23, end date March 27, vacation type" (follow-up turn after initial request) | Capability: `Submit PTO Request flow` | Capability Use (All) + Keyword Match (Any) |
| 6 | Agent handles ambiguous date format | "I want PTO starting next Friday through the following Wednesday" | Response is clear, natural, and either confirms interpreted dates or asks for clarification | General Quality + Compare Meaning |

### Tips

- Write at least **2 test cases per tool that requires parameters**: one with all parameters provided, one with parameters missing.
- Test with **realistic incomplete requests** — most users don't provide all parameters in their first message.
- For tools with 3+ required parameters, test with 1 missing, 2 missing, and all missing to verify the agent handles each case.
- Target: **100% of incomplete-request test cases should prompt for the missing parameters** rather than invoking the tool with missing data.
- Rerun after changing tool parameter definitions or descriptions.

---

## 4. Verifying Tool Output in Response

Does the agent correctly parse and present the tool's returned data in its response to the user?

### When to Use

Your tools return structured data (JSON, tables, status codes, record details) and the agent must extract the relevant fields and present them in a clear, user-friendly response. This scenario catches cases where the agent ignores tool output, misinterprets fields, or presents raw data instead of a human-readable answer.

> **Related scenarios:** If your concern is whether the correct tool fired in the first place, see [Scenario 1: Verifying Tool Invocation](#1-verifying-tool-invocation). If the tool returns an error, see [Scenario 5: Testing Error Handling on Tool Failure](#5-testing-error-handling-on-tool-failure).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms specific data values from the tool output appear in the response (e.g., order status, tracking number, dates) |
| Compare Meaning | Verifies the response accurately conveys the meaning of the tool's returned data |
| General Quality | Assesses whether the response is well-formatted, readable, and free of raw JSON or technical artifacts |

> **Tip:** Use Keyword Match (All) for objective data fields (dates, IDs, status values) and Compare Meaning for cases where the agent should summarize or interpret the data rather than quote it verbatim.

### Setup Steps

1. For each tool, identify the key data fields returned in its output (e.g., Get Order Status API returns: order status, estimated delivery date, tracking number).
2. Create test cases that trigger the tool and set the **Expected Value** to the specific data values the tool should return.
3. Use **Keyword Match (All)** with the exact values you expect to appear in the response (e.g., "shipped", "March 5", "TRACK-88234").
4. Add a **General Quality** check to confirm the response is formatted for a human reader — not raw JSON or unprocessed output.
5. Run the eval set. Failures indicate the agent is dropping, misinterpreting, or poorly formatting tool output.

### Anti-Pattern

> **Anti-Pattern: Checking Only That a Response Exists**
> Some teams verify the tool was called (Capability Use) and that the agent responded (non-empty response) but never check whether the response contains the actual data the tool returned. An agent can call Get Order Status API and respond "I checked your order — let me know if you need anything else" without including the status, date, or tracking number. Always verify the specific data fields appear in the response.

### Evaluation Patterns

**Pattern A: Key Field Extraction**
The tool returns structured data with multiple fields. Verify the agent extracts and presents the fields that matter most to the user. Not every field needs to appear — but critical ones (status, dates, amounts, IDs) must.

**Pattern B: Data Formatting**
The tool returns data in a technical format (ISO dates, status codes, nested JSON). Verify the agent converts these into human-readable language ("March 5, 2026" not "2026-03-05T00:00:00Z"; "Shipped" not "STATUS_SHIPPED").

**Pattern C: Multi-Record Results**
The tool returns a list of records (e.g., multiple open tickets, several upcoming PTO entries). Verify the agent presents them in a structured, scannable format — not as a wall of text.

**Pattern D: No-Result Handling**
The tool returns successfully but with empty or null results (e.g., no matching orders found). Verify the agent communicates this clearly rather than presenting an empty or confusing response.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Order status fields appear in response | "Where is my order #88234?" | Keywords (all): "shipped", "March 5", "TRACK-88234" | Keyword Match (All) + Compare Meaning |
| 2 | PTO balance details presented correctly | "How much PTO do I have left?" | Keywords (all): "12 days", "vacation", "2026" | Keyword Match (All) + Compare Meaning |
| 3 | Support ticket confirmation includes ID | "Create a ticket for my login issue" | Keywords (all): "TKT-", "created", "login" | Keyword Match (All) + Compare Meaning |
| 4 | Response conveys meaning of returned data | "What is the status of my expense report?" | "Your expense report for the NYC trip has been approved by your manager and is pending finance review." | Compare Meaning + Keyword Match (Any) |
| 5 | No raw JSON or technical artifacts in response | "Where is my order #88234?" | Response is clear, well-formatted, and free of raw JSON, status codes, or technical field names | General Quality + Keyword Match (Any) |
| 6 | Empty result communicated clearly | "Do I have any open support tickets?" | "You don't currently have any open support tickets." or similar clear statement | Compare Meaning + General Quality |

### Tips

- For each tool, identify the **3–5 most important output fields** and write test cases that check for them specifically.
- Watch for **data type conversion issues** — dates, currency amounts, and status codes are common culprits for poor formatting.
- Test with both **single-result** and **multi-result** tool responses if your tool can return lists.
- Target: **95% or higher of tool-output test cases should include all expected data fields** in the response. Missing data is a functional failure.
- If your agent uses Generative Answers to format tool output, pay extra attention to hallucinated or embellished data that was not in the tool's actual return.

---

## 5. Testing Error Handling on Tool Failure

When a tool call fails (timeout, API error, invalid credentials, downstream service unavailable), does the agent handle it gracefully?

### When to Use

Your agent calls external tools, APIs, or connectors that may fail due to network issues, service outages, rate limits, invalid inputs, or authentication errors. This scenario verifies the agent communicates the failure clearly, avoids exposing technical error details, and offers a constructive next step.

> **Related scenarios:** If your concern is broader graceful failure beyond tool errors (e.g., the agent doesn't know the answer), see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). This scenario focuses specifically on failures originating from tool/API calls.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verifies the agent communicates the failure in a user-friendly way that matches the expected error-handling message |
| Keyword Match (Any) | Checks for the presence of helpful recovery language (e.g., "try again", "contact support", "we're experiencing an issue") |
| General Quality | Assesses whether the error response is empathetic, constructive, and free of technical jargon or raw error codes |

> **Tip:** Error-handling tests require you to simulate failure conditions. Depending on your setup, you may need to temporarily misconfigure a connector, use a test environment with a failing endpoint, or test against known error-triggering inputs.

### Setup Steps

1. Identify the failure modes for each tool: timeout, invalid input, authentication failure, downstream service error, rate limiting.
2. For each failure mode, create a test case that triggers the condition (e.g., use an invalid order number to trigger a "not found" API error).
3. Set the **Expected Value** to the user-friendly error message you want the agent to deliver (using Compare Meaning or Keyword Match (Any)).
4. Add a **General Quality** check to confirm the response does not expose raw error codes, stack traces, or technical details.
5. Run the eval set. Failures indicate the agent either crashes silently, exposes technical errors, or provides no recovery guidance.

### Anti-Pattern

> **Anti-Pattern: Only Testing the Happy Path**
> If every test case assumes the tool call succeeds, you have no coverage for the inevitable moment a tool fails in production. APIs go down. Connectors time out. Credentials expire. Test at least one failure mode per tool to ensure your agent degrades gracefully rather than presenting a blank screen or a cryptic error message.

### Evaluation Patterns

**Pattern A: User-Friendly Error Communication**
The tool fails and the agent must explain what happened without technical jargon. Verify the response says something like "I wasn't able to retrieve your order status right now" — not "HTTP 500 Internal Server Error" or "null reference exception."

**Pattern B: Recovery Guidance**
After a failure, the agent should offer a constructive next step: retry, contact a human, try a different approach, or check back later. Verify the response includes actionable guidance rather than just stating the failure.

**Pattern C: No Data Leakage on Error**
Error responses from APIs sometimes contain internal system information (server names, database paths, API keys). Verify the agent does not echo raw error payloads to the user.

**Pattern D: Partial Failure in Multi-Step Flows**
If the agent is executing a multi-step process and one step fails, verify it communicates which step failed and what (if anything) was completed successfully, rather than failing silently or reporting total failure.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | API timeout produces friendly message | "Where is my order #88234?" (with API configured to time out) | "I'm unable to retrieve your order status at the moment. Please try again in a few minutes or contact support." | Compare Meaning + Keyword Match (Any) |
| 2 | Invalid input produces helpful guidance | "Check order status for order INVALID" | Response mentions the input may be incorrect and suggests checking the order number | Compare Meaning + General Quality |
| 3 | Recovery language is present | "Submit my expense report" (with flow failing) | Keywords (any): "try again", "contact", "support", "assistance", "unable" | Keyword Match (Any) + General Quality |
| 4 | No raw error codes in response | "Where is my order #88234?" (with API returning 500 error) | Response is empathetic, constructive, and does not contain HTTP status codes, stack traces, or system error messages | General Quality + Compare Meaning |
| 5 | Partial failure communicated clearly | "Book the conference room and send invites to the team" (room booking succeeds, invite sending fails) | Response confirms what succeeded and explains what failed: "I've booked the conference room, but I was unable to send the meeting invites..." | Compare Meaning + Keyword Match (Any) |
| 6 | Authentication failure handled gracefully | "Pull up my benefits information" (with connector credentials expired) | Response does not mention "authentication" or "credentials" — instead says something like "I'm having trouble accessing your benefits information right now" | General Quality + Compare Meaning |

### Tips

- Write at least **1 error-handling test case per tool**. If a tool has multiple failure modes, test the most likely ones.
- To simulate failures, consider using test environments, intentionally invalid inputs, or temporarily misconfigured connectors.
- Verify the agent **never exposes raw error payloads, API keys, internal URLs, or stack traces** to the user.
- Target: **100% of error-handling test cases should produce a user-friendly response** with recovery guidance. A blank response or a raw error message is always a critical failure.
- Coordinate with [Safety & Boundary Enforcement](safety-and-boundary-enforcement.md) to ensure error responses don't leak sensitive system information.

---

## 6. Verifying Multi-Tool Orchestration

When a request requires multiple sequential tool calls, does the agent execute them in the right order and combine the results?

### When to Use

Your agent handles requests that require calling two or more tools in sequence or in coordination — where the output of one tool feeds into the input of the next, or where the agent must synthesize results from multiple tools into a single response. This scenario catches ordering errors, dropped intermediate results, and incomplete orchestration.

> **Related scenarios:** If each tool call is independent and you just want to verify each one fires, see [Scenario 1: Verifying Tool Invocation](#1-verifying-tool-invocation). This scenario specifically covers cases where tool calls depend on each other or their results must be combined.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms ALL required tools in the sequence were invoked — not just the first or last one |
| Compare Meaning | Verifies the final response synthesizes results from all tool calls into a coherent answer |
| Keyword Match (All) | Checks for specific data values from each tool's output in the final response |
| General Quality | Assesses whether the combined response is well-structured and doesn't feel fragmented or incomplete |

> **Tip:** Capability Use (All) is critical here because it verifies that *every* tool in the sequence was called. A common failure mode is the agent calling the first tool, getting a result, and responding without proceeding to the second tool.

### Setup Steps

1. Identify user requests that require multiple tool calls. Map the expected tool sequence (e.g., "Cancel my order" requires: Get Order Status API to check eligibility, then Cancel Order flow to execute cancellation).
2. Create test cases for each multi-tool request. Set the **Expected Capability** using **Capability Use (All)** with every tool in the sequence listed.
3. Add **Keyword Match (All)** checks for data from each tool (e.g., order status from tool 1, cancellation confirmation from tool 2).
4. Add a **Compare Meaning** check to verify the final response combines the information coherently.
5. Run the eval set. Failures indicate the agent dropped a step, called tools in the wrong order, or failed to synthesize results.

### Anti-Pattern

> **Anti-Pattern: Testing Multi-Tool Flows as Individual Tool Calls**
> If you test "Get Order Status" and "Cancel Order" as separate, independent test cases, you validate each tool in isolation but never test whether the agent chains them correctly. A user who says "Cancel my order" expects the agent to check status, verify eligibility, and then cancel — as one continuous flow. Test the full orchestration, not just the parts.

### Evaluation Patterns

**Pattern A: Sequential Dependency**
Tool B requires output from Tool A. Verify that (a) Tool A is called first, (b) its output is passed correctly to Tool B, and (c) the final response reflects the combined outcome. Example: "Cancel my order #88234" requires checking order status (Tool A) before executing cancellation (Tool B).

**Pattern B: Parallel Aggregation**
The agent needs data from multiple independent tools and must combine them into a single response. Verify all tools are called and their results are synthesized. Example: "Give me a summary of my account" requires calling Get PTO Balance, Get Open Tickets, and Get Pending Expense Reports, then combining the results.

**Pattern C: Conditional Branching**
The result of the first tool determines which tool to call next. Verify the agent takes the correct branch. Example: "Cancel my order" — if the order status is "shipped," the agent should explain it can't be cancelled instead of calling the Cancel Order flow.

**Pattern D: Error Recovery in Orchestration**
One tool in the sequence fails. Verify the agent handles the partial failure rather than silently dropping the remaining steps or reporting complete success. Example: The agent successfully retrieves the order status but the cancellation flow fails — it should report the status and explain the cancellation couldn't be processed.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Cancel order requires status check then cancellation | "Cancel my order #88234" | Capability (all): `Get Order Status API`, `Cancel Order flow` | Capability Use (All) + Compare Meaning |
| 2 | Account summary requires multiple data sources | "Give me a summary of my account" | Capability (all): `Get PTO Balance flow`, `Get Open Tickets API`, `Get Pending Expenses API` | Capability Use (All) + Compare Meaning |
| 3 | Combined results appear in final response | "Cancel my order #88234" | Keywords (all): "order #88234", "cancelled", "refund" | Keyword Match (All) + Compare Meaning |
| 4 | Final response synthesizes multi-tool data | "Give me a summary of my account" | Response includes PTO balance, open ticket count, and pending expense status in a coherent summary | Compare Meaning + Keyword Match (Any) |
| 5 | Conditional branch: shipped order cannot be cancelled | "Cancel my order #88234" (order status = shipped) | Response explains order has already shipped and cannot be cancelled, offers alternatives | Compare Meaning + General Quality |
| 6 | Multi-tool response is well-structured | "Give me a summary of my account" | Response is clearly organized with distinct sections for each data type, not a jumbled paragraph | General Quality + Compare Meaning |

### Tips

- Map out every **multi-tool user journey** in your agent before writing test cases. Draw the flow: Tool A -> Tool B -> response. This prevents gaps.
- Test at least **1 sequential dependency**, **1 parallel aggregation**, and **1 conditional branch** if your agent supports them.
- Always use **Capability Use (All)** to verify the full tool chain — not just the first or last tool.
- Target: **95% or higher of multi-tool test cases should invoke ALL expected capabilities.** Dropped tool calls in a sequence are functional failures.
- Watch for **timeout issues** in multi-tool flows — if the total execution time exceeds limits, the agent may abandon later tool calls. Test realistic data volumes.
- Rerun after any changes to tool descriptions, flow logic, or orchestration configuration.