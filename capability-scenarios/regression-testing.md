# Regression Testing

> Scenarios for validating that agent updates don't break existing behavior. Run these before publishing any change — knowledge source updates, topic modifications, tool configuration changes, or prompt adjustments.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Scenario 1: Baseline Comparison After Knowledge Source Updates

### When to Use

Use this when you have updated, added, or removed knowledge sources — SharePoint documents, website URLs, uploaded files, or Dataverse data — and you need to verify that previously passing test cases still pass. Knowledge source changes are the most common trigger for regressions because they can alter retrieval results in unexpected ways: a new document might outrank an existing one, a deleted document might leave a question unanswered, or an updated document might change the correct answer without you realizing it affects other questions.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Exact Match | Re-run existing test cases that previously passed with exact expected values — any deviation is a regression signal |
| Compare Meaning | Re-run test cases where the answer should remain semantically equivalent even if wording changes due to the updated source |
| Keyword Match (All) | Re-verify that critical keywords, policy names, or data points that appeared in previous passing responses still appear |
| General Quality | Re-evaluate overall response quality for a sample of previously passing test cases to catch quality degradation |

### Setup Steps

1. Before making any knowledge source changes, run your full existing test suite and record the results. This is your baseline. Save a snapshot of pass/fail status for every test case.
2. Make your knowledge source changes (add, update, or remove documents).
3. Re-run the full test suite against the updated agent without changing any test case inputs or expected values.
4. Compare results: any test case that previously passed and now fails is a regression. Investigate each failure to determine whether it is a true regression (the agent's behavior worsened) or an expected change (the correct answer genuinely changed due to the updated knowledge).
5. For expected changes, update the test case's expected value to reflect the new correct answer. For true regressions, investigate the knowledge source change that caused the degradation.

### Anti-Pattern

> **"Test After You Forget"** — Updating knowledge sources and publishing without running regression tests, then discovering the regression when a user reports a wrong answer. Regression testing is a pre-publish activity, not a post-incident investigation. Run your regression suite before every publish, not after every complaint.

### Evaluation Patterns

#### Pattern: Full Suite Re-Run
The simplest regression approach: re-run every existing test case after the knowledge change. Any new failure is a regression candidate. This is the most thorough approach but can be time-consuming for large test suites. Use this when the knowledge change is broad (e.g., replacing a major policy document).

#### Pattern: Targeted Re-Run by Knowledge Domain
If the knowledge change is scoped to a specific domain (e.g., updating the benefits document), re-run only the test cases that are likely affected — benefits-related questions, eligibility checks, enrollment procedures. This is faster than a full re-run and appropriate for narrow changes.

#### Pattern: Before-and-After Comparison
For each regression candidate, compare the pre-change response and the post-change response side by side. Determine whether the change is a genuine improvement (the updated document provides a better answer), a neutral change (different wording, same meaning), or a degradation (the answer is now wrong or incomplete). Use Compare Meaning for neutral-change detection and Exact Match or Keyword Match (All) for degradation detection.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Previously correct answer remains correct after doc update | "What is the PTO accrual rate?" (PTO doc was reformatted but not substantively changed) | Response still contains "15 days per year" or the previously verified correct answer | Exact Match + Compare Meaning |
| 2 | Answer reflects new information from updated source | "What is the deadline for open enrollment?" (date changed in updated doc) | Response contains the new deadline date, not the old one — update the test case expected value | Keyword Match (All) + Compare Meaning |
| 3 | Unrelated question is unaffected by knowledge change | "How do I reset my password?" (the password FAQ was not changed) | Response is semantically equivalent to the pre-change baseline response | Compare Meaning + Keyword Match (Any) |
| 4 | Deleted document does not leave a gap | "What is the company's social media policy?" (social media policy doc was removed and replaced with a broader communications policy) | Response still provides relevant guidance — the replacement document is being retrieved | General Quality + Compare Meaning |
| 5 | New document does not outrank existing authoritative source | "What is the maximum expense reimbursement amount?" (a new travel policy doc was added alongside the existing expense policy) | Response still cites the correct authoritative source and amount, not information from the new document that may have different context | Exact Match + Compare Meaning |
| 6 | Quality does not degrade for previously high-quality responses | "Walk me through the performance review process." (knowledge base was updated) | Response maintains the same level of structure, completeness, and clarity as the baseline response | General Quality + Compare Meaning |

### Tips

- Always capture a baseline before making changes. Without a pre-change snapshot, you cannot distinguish regressions from pre-existing failures.
- Knowledge source changes have a blast radius that is often wider than expected. Updating one document can affect retrieval results for questions that seem unrelated, because retrieval ranking is competitive — a new document can push an existing one out of the top results.
- After a broad knowledge update (e.g., annual policy refresh), run the full test suite. After a narrow update (e.g., correcting a typo in one document), a targeted re-run is sufficient.
- Not every test failure after a knowledge change is a regression. If you corrected inaccurate information, tests checking for the old (wrong) answer should fail — update those test cases to reflect the new correct answer.
- Track your regression rate over time. If more than 10% of test cases fail after a typical knowledge update, your test suite or your knowledge architecture may need restructuring.

---

## Scenario 2: Regression Check After Topic or Flow Changes

### When to Use

Use this when you have modified topics, conversation flows, trigger phrases, or routing logic in Copilot Studio — and you need to verify that these changes did not break existing conversation behavior. Topic changes can cause regressions in several ways: a modified trigger phrase might steal routing from another topic, a restructured flow might skip a previously included step, or a deleted node might remove a capability the agent previously had. These regressions are particularly insidious because they affect conversation structure, not just content.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Re-verify that the correct topics, flows, and actions still fire for previously tested inputs |
| Keyword Match (All) | Re-verify that expected content from specific topics still appears in the responses |
| Compare Meaning | Re-verify that the overall response meaning is preserved even if the flow structure changed |
| General Quality | Re-evaluate response quality for conversations that traverse the modified topics |

### Setup Steps

1. Before making topic or flow changes, document the current routing behavior: which inputs trigger which topics, what actions each topic takes, and what the expected outputs are. Run your existing test suite to establish the baseline.
2. Make your topic or flow changes.
3. Re-run all test cases that could be affected by the change. At minimum, re-run: (a) all test cases that directly test the modified topic, (b) all test cases for topics with similar trigger phrases that might be affected by routing changes, and (c) a sample of general test cases to catch unexpected side effects.
4. For Capability Use (All) checks, verify that the same topics and actions fire as before. A change in which topic fires is a routing regression.
5. For conversation flow changes, re-run multi-turn conversation scripts to verify that the conversation still follows the expected path and does not skip steps or enter unexpected branches.

### Anti-Pattern

> **"Localized Change, Localized Testing"** — Only testing the specific topic you changed and assuming nothing else was affected. Topic changes can cause routing conflicts, trigger phrase collisions, and flow disruptions in other topics. Always test adjacent topics and a general sample, not just the changed topic.

### Evaluation Patterns

#### Pattern: Routing Stability Check
After changing a topic's trigger phrases, verify that the change did not steal routing from other topics. Test inputs that previously routed to other topics and confirm they still route correctly. This is especially important when adding new trigger phrases that are semantically similar to existing ones in other topics.

#### Pattern: Flow Completeness Check
After modifying a conversation flow (adding, removing, or reordering nodes), re-run multi-turn conversation scripts that traverse that flow. Verify that all expected steps still occur in the correct order and that no steps were accidentally removed or skipped.

#### Pattern: Cross-Topic Interaction Check
If your agent has topics that reference or redirect to each other (e.g., a PTO topic that redirects to the Leave of Absence topic for extended leave), verify that these cross-topic interactions still work after the change. A renamed or restructured topic can break incoming references.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Modified topic still fires for its primary input | "I need to submit a PTO request." (PTO topic was restructured) | PTO topic still fires and the response addresses PTO submission | Capability Use (All) + Keyword Match (Any) |
| 2 | Adjacent topic routing is not disrupted | "What is the leave of absence policy?" (PTO topic trigger phrases were expanded) | Leave of Absence topic fires — not the PTO topic — since this is a different intent | Capability Use (All) + Compare Meaning |
| 3 | Multi-turn flow still completes all steps | Multi-turn PTO submission conversation (flow was modified to add a new confirmation step) | All original steps still occur, plus the new confirmation step, in the correct order | General Quality + Compare Meaning |
| 4 | Cross-topic redirect still works | "Can I extend my PTO into a leave of absence?" (PTO topic redirects to Leave of Absence topic) | Conversation correctly transitions from PTO topic to Leave of Absence topic | Capability Use (All) + Compare Meaning |
| 5 | Deleted topic content does not create a gap | "How do I submit a facilities request?" (old Facilities topic was merged into a General Requests topic) | The General Requests topic fires and handles the facilities request with the same quality as the old topic | Compare Meaning + General Quality |
| 6 | General routing sample passes | Random sample of 10 previously passing test cases from unrelated topics | All 10 test cases still pass with the same routing and response quality | Keyword Match (All) + Compare Meaning |

### Tips

- Topic and flow changes have cascading effects that are difficult to predict. Even a small trigger phrase addition can change routing for inputs you did not anticipate. Always test more broadly than the change itself.
- Keep a routing map that documents which inputs route to which topics. Update this map with every change and use it to identify which test cases to re-run.
- Multi-turn conversation scripts are essential for flow regression testing. Single-turn tests cannot catch flow-order regressions where steps are skipped or reordered.
- If you use Copilot Studio's topic routing, test with inputs that are ambiguous between topics — these are the inputs most likely to change routing after a trigger phrase modification.
- Aim for a regression suite that covers at least the top 3–5 inputs for every topic, plus a random sample of 10–15 general inputs to catch unexpected routing changes.

---

## Scenario 3: Regression Check After Tool Configuration Changes

### When to Use

Use this when you have modified tool, connector, or Power Automate flow configurations — changing parameters, updating authentication, modifying response mappings, enabling or disabling tools, or altering how the agent invokes external systems. Tool configuration changes can cause regressions where the agent either fails to invoke the tool, invokes it with wrong parameters, or misinterprets the tool's response. These regressions are particularly impactful because they affect actions the agent takes on behalf of the user, not just information it provides.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Re-verify that the correct tools and connectors still fire with the correct parameters for previously tested inputs |
| Exact Match | Re-verify that tool responses are parsed and presented correctly — especially for structured data like dates, amounts, and statuses |
| Keyword Match (All) | Re-verify that key data points from tool responses still appear in the agent's answer |
| General Quality | Re-evaluate the overall quality of responses that depend on tool invocations |

### Setup Steps

1. Before making tool configuration changes, run all test cases that involve tool invocations and record the baseline results: which tools fired, what parameters were passed, and what the agent's final response was.
2. Make your tool configuration changes.
3. Re-run all tool-related test cases. Focus on: (a) test cases for the specific tool you changed, (b) test cases for tools that share authentication or parameters with the changed tool, and (c) test cases for workflows that chain multiple tools where the changed tool is one link in the chain.
4. For Capability Use (All) checks, verify that the tool is still invoked with the correct parameters. A parameter change is a regression if it causes incorrect behavior.
5. For response parsing, use Exact Match or Keyword Match (All) to verify that the agent still correctly extracts and presents data from the tool's response.

### Anti-Pattern

> **"Works in Isolation"** — Testing the tool configuration change in isolation (e.g., confirming the Power Automate flow runs correctly in its own test environment) without testing how the agent invokes it. The tool may work perfectly when called directly but fail when the agent calls it — because the agent sends different parameters, uses different authentication, or parses the response differently.

### Evaluation Patterns

#### Pattern: Invocation Parameter Verification
After changing a tool's configuration, verify that the agent still sends the correct parameters. Compare the parameters in the post-change invocation against the pre-change baseline. Any parameter change that was not intentional is a regression.

#### Pattern: Response Parsing Stability
Tool configuration changes can alter the structure of the tool's response (different field names, different data format, additional or removed fields). Verify that the agent's response parsing still works correctly — data points from the tool's response should still appear accurately in the agent's final answer.

#### Pattern: Error Handling Preservation
If the tool configuration change affects error cases (new error codes, changed error messages, different timeout behavior), verify that the agent's error handling still works. Test with inputs that are designed to trigger tool errors and verify the agent handles them gracefully.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Tool still fires after configuration change | "Submit a PTO request for next Monday." (PTO submission flow was reconfigured) | PTO submission tool/flow is still invoked with the correct date parameter | Capability Use (All) + Keyword Match (Any) |
| 2 | Tool parameters are correct post-change | "Check the status of my order #12345." (order lookup connector authentication was updated) | Order lookup tool is invoked with parameter "12345" and returns the correct status | Capability Use (All) + Keyword Match (Any) |
| 3 | Response data is parsed correctly | "What is my current PTO balance?" (HR system connector response format was updated) | Response still contains the correct numeric PTO balance from the tool's response | Exact Match + Compare Meaning |
| 4 | Chained tool workflow still completes | "Book a conference room for tomorrow at 2 PM." (calendar connector was updated, room booking flow depends on it) | Both the calendar lookup and room booking tools fire in sequence with correct parameters | Capability Use (All) + Compare Meaning |
| 5 | Error handling still works | "Submit a PTO request for a date in the past." (flow was reconfigured) | Agent still handles the error gracefully — explains the date is invalid rather than crashing or returning a raw error | General Quality + Compare Meaning |
| 6 | Tool response data is current and complete | "How many open tickets are in my queue?" (ticketing system connector was updated) | Response contains the correct count and matches the data from the tool's response, not stale data | Keyword Match (All) + Compare Meaning |

### Tips

- Tool configuration changes have a higher risk-per-change ratio than knowledge or topic changes because they affect agent actions, not just agent words. A broken tool invocation can submit incorrect data, not just display incorrect information.
- Always test the full invocation chain: agent prompt to tool invocation to tool response to agent response. A change anywhere in this chain can cause a regression.
- If you use chained tools (Tool A's output feeds into Tool B's input), test the full chain even if you only changed Tool B. The change might alter what Tool B expects from Tool A.
- Test tool error scenarios specifically. Configuration changes often break error handling before they break happy-path behavior, because error handling is less commonly tested.
- Aim for at least 3–5 test cases per tool: 2–3 happy-path invocations and 1–2 error-path invocations.

---

## Scenario 4: Full-Suite Regression Before Publishing

### When to Use

Use this as a comprehensive pre-publish checkpoint before promoting any agent change to production. Regardless of what changed — knowledge sources, topics, tools, prompts, or any combination — a full-suite regression run provides confidence that the agent's overall behavior has not degraded. This is the final gate before users see the updated agent. It covers all quality dimensions: accuracy, routing, tool invocations, compliance, safety, tone, and escalation behavior.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Exact Match | Re-verify all test cases with exact expected values — factual answers, specific numbers, verbatim text |
| Compare Meaning | Re-verify all test cases where meaning equivalence is the standard — policy explanations, guidance, advice |
| Keyword Match (All) | Re-verify all compliance and content-completeness checks — required terms, disclaimers, data points |
| Capability Use (All) | Re-verify all tool invocation and topic routing checks — correct capabilities fire for correct inputs |
| General Quality | Re-evaluate a representative sample of response quality, tone, and helpfulness test cases |

### Setup Steps

1. Maintain a master regression test suite that includes test cases from all quality dimensions: knowledge accuracy, tool invocations, topic routing, compliance, safety, tone, and escalation. This suite should grow over time as you add new scenarios.
2. Before publishing, run the full master regression suite against the updated agent.
3. Compare results against the previous run's baseline. Categorize any new failures as: (a) true regressions (behavior worsened), (b) expected changes (correct answer changed due to intentional updates), or (c) flaky tests (intermittent failures unrelated to the change).
4. For true regressions, investigate the root cause before publishing. Do not publish with known regressions unless they are explicitly accepted and documented.
5. For expected changes, update the test case expected values and re-run to confirm the updated test passes.
6. After publishing, save the test results as the new baseline for the next regression cycle.

### Anti-Pattern

> **"Ship and Hope"** — Publishing agent changes without running a regression suite because "the change was small" or "we only updated one document." Small changes cause regressions. One-line prompt changes can alter behavior across dozens of scenarios. The cost of a 30-minute regression run is always less than the cost of a production incident.

### Evaluation Patterns

#### Pattern: Full-Dimensional Coverage
The regression suite should include at least one test case from each quality dimension: factual accuracy, tool invocation, topic routing, compliance, safety, tone, and escalation. If any dimension is missing, you have a coverage gap that could hide regressions.

#### Pattern: Pass Rate Monitoring
Track the overall pass rate of your regression suite over time. A healthy agent maintains a stable or improving pass rate across releases. A declining pass rate across successive releases — even if each individual release passes a threshold — signals cumulative quality degradation.

#### Pattern: Regression Triage Workflow
Not all regression failures are equal. Establish a triage workflow: critical regressions (safety, compliance, data accuracy) block publishing; important regressions (tone, structure, completeness) require review before publishing; minor regressions (formatting, phrasing) can be documented and fixed in the next cycle.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Factual accuracy baseline holds | "What is the company's 401k match percentage?" | Response still contains "4%" (or whatever the verified correct value is) | Exact Match + Compare Meaning |
| 2 | Topic routing baseline holds | "I need to file a complaint." | Complaint topic still fires, not the general FAQ topic | Capability Use (All) + Keyword Match (Any) |
| 3 | Tool invocation baseline holds | "Submit my timesheet for this week." | Timesheet submission flow is still invoked with correct parameters | Capability Use (All) + Keyword Match (Any) |
| 4 | Compliance content baseline holds | "What is the procedure for reporting harassment?" | Response still contains required legal disclaimer language | Keyword Match (All) + Compare Meaning |
| 5 | Tone and quality baseline holds | "I'm having a really difficult time with my manager." | Response maintains empathetic tone and provides helpful guidance comparable to the baseline response | General Quality + Compare Meaning |
| 6 | Escalation behavior baseline holds | "I want to speak with a human agent." | Handoff capability is still invoked, with appropriate handoff communication | Capability Use (All) + Keyword Match (Any) |

### Tips

- Build your master regression suite incrementally. Every time you create test cases for a new scenario, add the most important ones to the master regression suite. Over time, this suite becomes your agent's comprehensive quality baseline.
- Set a pass rate threshold for publishing — for example, 95% of regression test cases must pass. Document exceptions for the remaining 5% and include a plan to address them.
- Full-suite regression takes time. For large test suites (50+ test cases), allocate 30–60 minutes for the run and triage. Build this into your release process, not as an afterthought.
- Keep your regression suite curated. Remove test cases that are no longer relevant (e.g., for deprecated features) and add test cases for new capabilities. A stale regression suite gives false confidence.
- Track regression metrics across releases: pass rate, number of new regressions, time to triage, and categories of regression. These metrics reveal quality trends that individual test runs cannot show.

---

## Scenario 5: Targeted Regression for Specific Capability

### When to Use

Use this when you have made a narrow, well-scoped change to the agent — a single topic modification, one knowledge document update, or a specific tool configuration change — and you want to run a focused regression check on just the affected capability rather than the full suite. Targeted regression is faster than full-suite regression and is appropriate when the change is isolated and the blast radius is predictable. Use full-suite regression (Scenario 4) when the change is broad or when you are not confident about the blast radius.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Exact Match | Re-run the specific test cases for the changed capability with their existing expected values |
| Compare Meaning | Check that the changed capability's responses are still semantically correct after the update |
| Capability Use (All) | For tool or routing changes, verify the specific capability still fires correctly |
| Keyword Match (All) | For knowledge changes, verify that key information from the affected domain still appears |

### Setup Steps

1. Identify the specific capability affected by your change: which topic, knowledge domain, or tool was modified.
2. From your master regression suite, select the test cases that directly test this capability. Also select 3–5 test cases from adjacent capabilities that could be affected by the change (e.g., if you changed the PTO topic, also test Leave of Absence and Time Tracking topics).
3. Run this targeted subset against the updated agent.
4. Compare results against the previous baseline for these specific test cases.
5. If all targeted test cases pass, the narrow change is likely safe. If any fail, investigate whether the failure is related to the change or is a pre-existing issue.
6. Optionally, run 5–10 random test cases from unrelated capabilities as a spot check. This provides a lightweight safety net without the cost of a full-suite run.

### Anti-Pattern

> **"Targeted Only, Always"** — Always running targeted regression instead of full-suite regression, even for broad changes. Targeted regression is appropriate for narrow, isolated changes with a predictable blast radius. For broad changes (prompt rewrites, major knowledge refreshes, multiple topic modifications), targeted regression gives false confidence. Know when to escalate from targeted to full-suite.

### Evaluation Patterns

#### Pattern: Direct Impact Re-Run
Re-run all test cases that directly test the changed capability. If you changed the benefits knowledge document, re-run all benefits-related test cases. This is the minimum regression check for any change.

#### Pattern: Adjacent Impact Re-Run
Beyond the directly affected test cases, re-run test cases for capabilities that interact with or are semantically similar to the changed capability. Topic routing changes can affect adjacent topics. Knowledge changes can affect retrieval ranking for related queries. This broader check catches indirect regressions.

#### Pattern: Random Spot Check
After running the targeted and adjacent test cases, run 5–10 randomly selected test cases from unrelated capabilities. This provides a lightweight safety net against unexpected cross-capability regressions. If all spot checks pass, you have reasonable confidence that the blast radius was limited to the targeted area.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Direct test case for changed capability passes | "How many sick days do I get per year?" (sick leave document was updated) | Response reflects the updated sick leave policy with correct numbers | Exact Match + Compare Meaning |
| 2 | Related test case for adjacent capability passes | "What is the difference between sick leave and PTO?" (only sick leave doc was changed) | Response accurately distinguishes both policies — the PTO information is unchanged | Compare Meaning + Keyword Match (Any) |
| 3 | Routing for changed topic still works | "I need information about sick leave." (sick leave topic triggers were modified) | Sick leave topic fires correctly, not a different topic | Capability Use (All) + Keyword Match (Any) |
| 4 | Adjacent topic routing is unaffected | "I want to request PTO." (sick leave topic was modified, not PTO) | PTO topic still fires correctly — no routing interference from the sick leave change | Capability Use (All) + Compare Meaning |
| 5 | Random spot check from unrelated capability | "How do I connect to the VPN?" (unrelated to the sick leave change) | Response is correct and consistent with the baseline — no unexpected impact | Compare Meaning + General Quality |
| 6 | Changed capability still handles edge cases | "Can I use sick leave for a mental health day?" (sick leave policy was updated) | Response accurately reflects the updated policy's position on mental health days | Keyword Match (All) + Compare Meaning |

### Tips

- Targeted regression works when you can confidently define the blast radius of your change. If you are unsure what else might be affected, default to full-suite regression.
- Maintain your test cases organized by capability or domain so you can quickly pull the targeted subset for any given change. A well-organized test suite makes targeted regression fast and reliable.
- The adjacent-impact re-run is the most valuable step in targeted regression. Direct test cases for the changed capability are obvious; it is the adjacent capabilities where surprise regressions hide.
- The random spot check is optional but cheap. Running 5–10 random test cases adds minimal time and occasionally catches regressions that the targeted approach would miss.
- Document which regression approach you used for each release (targeted vs. full-suite) and the results. This creates an audit trail that is valuable for understanding quality trends and debugging future regressions.
