# Triage & Routing

> Scenarios for agents that classify user requests, determine urgency, and route to the appropriate team, department, or resource — support ticket routing, complaint triage, service request classification.

[Back to library](../README.md) | [All business-problem scenarios](README.md)

---

## 1. Verifying Classification Accuracy

### When to Use

Your agent receives user requests and must categorize them into the correct type, category, or department before routing. This applies whenever the agent's first job is to understand **what kind of issue** the user has — whether that means tagging an IT ticket as "hardware" vs. "software," identifying a customer complaint as "billing" vs. "shipping," or recognizing an HR request as "payroll" vs. "benefits."

Use this scenario when:
- Your agent classifies requests into predefined categories
- Misclassification leads to delays, rework, or poor customer experience
- The classification drives downstream routing or prioritization decisions

> **Related scenarios:** If your agent also assigns urgency levels, see [Validating Priority Assignment](#3-validating-priority-assignment). If you want to verify that the agent routes to the right destination after classifying, see [Negative Test: Misrouted Request](#5-negative-test-misrouted-request). For verifying the underlying topic triggers, see [Trigger Routing](../capability-scenarios/trigger-routing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes the correct category label or classification term |
| Compare Meaning | Confirm the agent's classification explanation aligns with the expected category definition |
| General Quality | Assess whether the classification rationale is clear and well-reasoned |

> **Tip:** Use Keyword Match (All) as the primary check for the classification label itself, then layer Compare Meaning to verify the agent explains the classification correctly. A response can name the right category but describe it incorrectly.

### Setup Steps

1. Identify the full set of categories your agent classifies into (e.g., Hardware, Software, Network, Access, Other).
2. For each category, write 2–3 test cases with unambiguous user inputs that clearly belong to that category.
3. For each test case, set the **expected value** to the category label the agent should assign.
4. Select **Keyword Match (All)** as the test method and include the category label as the required keyword.
5. Add 1–2 test cases per category using **Compare Meaning** where the expected value is a brief explanation of why that category applies.
6. Run the evaluation and review results by category — look for categories with consistently low accuracy, which may indicate unclear category definitions or overlapping boundaries.

### Anti-Pattern

> **Anti-Pattern: Testing Only Clean-Cut Inputs**
> If every test input is a textbook example of its category ("My monitor is broken" for Hardware, "I can't install the app" for Software), you will overestimate classification accuracy. Real users describe problems in messy, incomplete, or overlapping ways. Include at least 30% of inputs that use informal language, describe symptoms rather than root causes, or mention multiple topics.

### Evaluation Patterns

**Pattern: Exact Category Match**
The agent must assign the request to the correct predefined category. Test with inputs that clearly belong to one category and verify the agent names it. This is the baseline — if the agent cannot classify clean-cut inputs, nothing else will work.

**Pattern: Informal and Ambiguous Language**
Real users rarely state their problem in formal category terms. Test with inputs like "nothing's working" or "I'm having a situation with my paycheck" and verify the agent still classifies correctly or asks a clarifying question (see [Handling Ambiguous Triage Scenarios](#4-handling-ambiguous-triage-scenarios)).

**Pattern: Multi-Issue Classification**
Some user messages contain more than one issue. Test whether the agent identifies the primary issue, acknowledges both, or asks the user which to address first. The expected behavior depends on your agent's design — the key is that it does not silently drop one issue.

**Pattern: Edge-of-Category Inputs**
Test inputs that sit on the boundary between two categories. For example, "My VPN won't connect" could be Network or Software depending on your taxonomy. Verify the agent classifies according to your defined rules, not its own assumptions.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT ticket: clear hardware issue | "My laptop screen is flickering and has a crack in the corner." | Response includes "Hardware" | Keyword Match (All) + Compare Meaning |
| 2 | IT ticket: clear software issue | "Excel keeps crashing every time I open a file larger than 10 MB." | Response includes "Software" | Keyword Match (All) + Compare Meaning |
| 3 | Customer complaint: billing category | "I was charged twice for my February subscription." | Classification meaning aligns with: "This is a billing issue involving a duplicate charge that should be routed to the billing team." | Compare Meaning + Keyword Match (All) |
| 4 | Customer complaint: shipping category | "My package was supposed to arrive three days ago and tracking shows no updates." | Response includes "Shipping" | Keyword Match (All) + Compare Meaning |
| 5 | HR request: benefits vs. payroll | "I need to change my direct deposit information." | Response includes "Payroll" | Keyword Match (All) + Compare Meaning |
| 6 | IT ticket: informal language | "Hey, something weird is happening — I can't get into any of my accounts since this morning." | Response includes "Access" | Keyword Match (All) + Compare Meaning |
| 7 | Multi-issue input | "My laptop is overheating and I also need a software license for Adobe." | Response acknowledges both a hardware concern and a software/licensing request | General Quality + Compare Meaning |

### Tips

- **Coverage target:** At least 2 test cases per classification category, with a mix of clear-cut and borderline inputs.
- **Accuracy threshold:** Target at least 90% correct classification across all test cases. Categories below 80% likely have definition or prompt issues.
- **Rerun after:** Any changes to category definitions, agent instructions, or the classification prompt.
- **Track category-level accuracy** separately — overall accuracy can mask that one category is consistently misclassified.
- Include at least one test case per category that uses informal, non-technical language.

---

## 2. Testing Context Preservation During Handoff

### When to Use

Your agent classifies a request and then routes it to another team, resource, or agent — and the receiving side needs context about the original request. This applies whenever there is a **handoff boundary**: the user talks to the triage agent, and then a different team or system takes over.

Use this scenario when:
- Your agent transfers conversations to human agents, other Copilot Studio agents, or external systems
- The receiving team or system needs a summary, category, or relevant details from the original interaction
- Users complain about "having to repeat themselves" after being transferred

> **Related scenarios:** For verifying the classification itself is correct, see [Verifying Classification Accuracy](#1-verifying-classification-accuracy). For validating the topic trigger that initiates the handoff, see [Trigger Routing](../capability-scenarios/trigger-routing.md). For graceful failure when the handoff target is unavailable, see [Graceful Failure & Escalation](../capability-scenarios/graceful-failure-and-escalation.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify that key details from the user's request appear in the handoff summary or transfer message |
| Compare Meaning | Confirm the handoff summary accurately represents the user's issue |
| Capability Use (All) | Verify the agent invokes the correct handoff action or transfer topic |

> **Tip:** Context preservation has two dimensions: **completeness** (all relevant details are passed) and **accuracy** (the details are correct). Use Keyword Match (All) to check completeness and Compare Meaning to check accuracy. A handoff can include all the right keywords but misrepresent the situation.

### Setup Steps

1. Identify all handoff points in your agent — where does it transfer to another team, agent, or system?
2. For each handoff point, write test cases that include specific details the receiving side would need (e.g., account numbers, error messages, steps already attempted).
3. Set expected values to include the **key details** that must be preserved: user name, issue category, specific problem details, and any information gathered during the conversation.
4. Select **Keyword Match (All)** to verify essential details appear in the handoff summary.
5. Add **Compare Meaning** test cases to verify the overall summary accurately reflects the conversation.
6. If your agent triggers a specific action or topic for handoff, add **Capability Use (All)** test cases to verify the correct handoff mechanism fires.
7. Run the evaluation and review any cases where details were lost, distorted, or misattributed.

### Anti-Pattern

> **Anti-Pattern: Testing Handoff Without Multi-Turn Context**
> If you only test handoff with a single user message, you miss the most common failure: the agent collects information across multiple turns but only passes the last message to the receiving team. Test with multi-turn conversations where critical details are mentioned early in the conversation, and verify those early details survive the handoff.

### Evaluation Patterns

**Pattern: Key Detail Preservation**
The agent must pass along specific, identifiable details from the user's request. Test that account numbers, error codes, product names, dates, and other concrete details appear in the handoff summary or transfer message.

**Pattern: Conversation Summary Accuracy**
When the agent generates a summary for the receiving team, test that the summary accurately represents the user's issue — not just keywords, but the correct meaning. A summary that says "user cannot access email" when the user said "my email is sending duplicates" has the right keywords but the wrong meaning.

**Pattern: Gathered Information Transfer**
If the agent asked clarifying questions and the user provided answers, test that those answers are included in the handoff. For example, if the agent asked "What operating system are you using?" and the user said "Windows 11," verify "Windows 11" appears in the context passed to the next team.

**Pattern: Handoff Action Invocation**
Verify that the correct handoff mechanism fires — the right transfer topic, the correct API call, or the appropriate escalation action. The context can be perfect but delivered to the wrong destination.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT support: error code preserved | "I keep getting error code 0x800F0922 when trying to update Windows." | Handoff summary includes "0x800F0922" and "Windows update" | Keyword Match (All) + Compare Meaning |
| 2 | Customer complaint: order details preserved | "Order #98421 arrived damaged — the box was crushed and two items inside were broken." | Handoff summary meaning aligns with: "Customer reports order #98421 arrived with physical damage to packaging and two broken items." | Compare Meaning + Keyword Match (All) |
| 3 | HR request: employee details preserved | "I'm in the Seattle office, Engineering department, and I need to update my emergency contact." | Handoff summary includes "Seattle," "Engineering," and "emergency contact" | Keyword Match (All) + Compare Meaning |
| 4 | Multi-turn: early details survive | Turn 1: "I need help with my benefits." Turn 2 (agent asks which benefit): "My dental coverage — I had a claim denied last week." Turn 3 (agent routes): handoff fires. | Handoff summary includes "dental" and "claim denied" | Keyword Match (All) + Compare Meaning |
| 5 | Correct handoff action fires | "I need to speak with someone about a workplace safety concern." | Agent invokes the HR escalation transfer topic | Capability Use (All) + Keyword Match (Any) |
| 6 | Summary accuracy check | "I was told my refund would arrive in 5 business days but it's been two weeks and nothing has posted to my account." | Handoff meaning aligns with: "Customer is following up on a delayed refund that was promised within 5 business days but has not arrived after two weeks." | Compare Meaning + Keyword Match (All) |

### Tips

- **Coverage target:** At least 2 test cases per handoff point — one with simple context and one with multi-turn context.
- **Completeness threshold:** 100% of critical details (account numbers, error codes, dates) should be preserved. Target 90% for secondary details.
- **Test multi-turn conversations** — this is where context loss most commonly occurs.
- **Rerun after:** Any changes to handoff topics, transfer actions, or agent instructions about summarization.
- If your agent uses a system message or adaptive card for handoff, inspect the actual payload — not just the user-facing message.

---

## 3. Validating Priority Assignment

### When to Use

Your agent assesses the urgency or priority of a request and assigns a level (e.g., Critical / High / Medium / Low, or P1 / P2 / P3 / P4). This applies to any agent that must distinguish between issues that need immediate attention and those that can wait.

Use this scenario when:
- Your agent assigns severity, priority, or urgency levels to incoming requests
- Priority levels drive SLAs, queue ordering, or escalation paths
- Misassigned priority leads to either over-escalation (wasted resources) or under-escalation (delayed resolution of critical issues)

> **Related scenarios:** For verifying the category is correct, see [Verifying Classification Accuracy](#1-verifying-classification-accuracy). For testing whether the agent asks clarifying questions when priority is unclear, see [Handling Ambiguous Triage Scenarios](#4-handling-ambiguous-triage-scenarios). For safety-critical inputs that should always be high priority, see [Safety & Boundary Enforcement](../capability-scenarios/safety-and-boundary-enforcement.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes the correct priority label (e.g., "High," "P1") |
| Compare Meaning | Confirm the agent's urgency assessment rationale matches the expected reasoning |
| General Quality | Assess whether the priority assignment explanation is logical and well-supported |

> **Tip:** Priority assignment is one of the hardest triage tasks because it requires judgment, not just classification. Use General Quality alongside Keyword Match to catch cases where the agent assigns the right label but for the wrong reason — which means it will fail on similar inputs with slight variations.

### Setup Steps

1. Document your priority levels and the criteria for each (e.g., P1 = system-wide outage or safety issue, P2 = single user blocked from critical work, P3 = inconvenience with workaround, P4 = enhancement request).
2. Write 2–3 test cases per priority level, covering the full range from critical to low.
3. Include at least 2 test cases where the priority is not immediately obvious — the user's language is calm but the issue is critical, or the language is urgent but the issue is low-priority.
4. Set expected values to the correct priority label and, for Compare Meaning tests, the reasoning behind the assignment.
5. Select **Keyword Match (All)** for label verification and **Compare Meaning** for reasoning verification.
6. Run the evaluation and analyze errors by priority level — look for systematic under- or over-escalation patterns.

### Anti-Pattern

> **Anti-Pattern: Letting Tone Drive Priority**
> If your test cases only include urgent-sounding language for high-priority issues and calm language for low-priority issues, you are testing tone detection, not priority assessment. Include test cases where a calm user describes a critical issue ("Just a heads up, the payment processing system seems to be down") and an urgent user describes a minor issue ("THIS IS EXTREMELY FRUSTRATING — my email signature font is wrong!!!"). The agent should assess priority based on business impact, not emotional intensity.

### Evaluation Patterns

**Pattern: Clear-Cut Priority Assignment**
Test inputs where the priority is unambiguous based on your defined criteria. A system-wide outage is P1. A font preference request is P4. This is the baseline — the agent must get these right before you trust it with borderline cases.

**Pattern: Tone vs. Impact Separation**
Test whether the agent distinguishes between the user's emotional tone and the actual business impact of the issue. An all-caps message about a cosmetic issue should still be low priority. A matter-of-fact message about a security breach should be critical.

**Pattern: Escalation Trigger Recognition**
Certain keywords or situations should automatically trigger high priority regardless of other factors — safety incidents, data breaches, complete system outages, legal threats. Test that the agent recognizes these escalation triggers.

**Pattern: Priority with Incomplete Information**
When the user does not provide enough information to assess priority, the agent should either assign a default priority and note the uncertainty, or ask a clarifying question. Test that it does not confidently assign a priority when it lacks the information to do so.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT support: system-wide outage | "The entire CRM system is down and no one on the sales team can access customer records." | Response includes "Critical" or "P1" | Keyword Match (All) + Compare Meaning |
| 2 | IT support: single-user inconvenience | "My default printer keeps switching back to the one on the third floor." | Response includes "Low" or "P4" | Keyword Match (All) + Compare Meaning |
| 3 | Customer complaint: calm tone, high impact | "I wanted to let you know that I've noticed unauthorized transactions on my account." | Priority meaning aligns with: "This is a high-priority security issue involving potential unauthorized access to the customer's account, despite the user's calm tone." | Compare Meaning + Keyword Match (Any) |
| 4 | Customer complaint: urgent tone, low impact | "I am VERY upset — the colors on your website look different on my phone than on my laptop!!" | Response includes "Low" or "P3" or "P4" | Keyword Match (Any) + Compare Meaning |
| 5 | HR request: safety escalation trigger | "I witnessed a coworker slip and fall in the warehouse and they're having trouble standing up." | Response includes "Critical" or "P1" or "Urgent" | Keyword Match (Any) + Compare Meaning |
| 6 | IT support: incomplete information | "Something is wrong with the system." | Agent asks a clarifying question to assess urgency, or assigns a default priority and notes incomplete information | General Quality + Compare Meaning |
| 7 | HR request: standard process | "I'd like to request a name change on my employee badge and email address." | Response includes "Medium" or "Low" or "P3" or "P4" | Keyword Match (Any) + Compare Meaning |

### Tips

- **Coverage target:** At least 2 test cases per priority level, plus 2–3 tone-vs-impact mismatch cases.
- **Accuracy threshold:** Target at least 95% accuracy for the highest and lowest priority levels (where the stakes of misassignment are greatest). Target at least 85% for middle tiers.
- **Watch for priority inflation** — if the agent assigns High or Critical to more than 30% of test cases, it may be over-escalating.
- **Rerun after:** Any changes to priority definitions, escalation criteria, or agent instructions about urgency assessment.
- Include at least 2 test cases where the user's emotional tone contradicts the actual business impact.

---

## 4. Handling Ambiguous Triage Scenarios

### When to Use

The user's request is unclear, could belong to multiple categories, or lacks enough information for the agent to confidently classify, prioritize, or route. This scenario tests whether the agent **asks clarifying questions** instead of guessing.

Use this scenario when:
- Your agent receives requests that could reasonably belong to more than one category
- Users often provide vague or incomplete descriptions
- Incorrect guessing leads to misrouting, wasted time, or poor user experience
- You want to verify the agent knows when it does not have enough information

> **Related scenarios:** For testing clear-cut classification, see [Verifying Classification Accuracy](#1-verifying-classification-accuracy). For testing what happens when the agent guesses wrong, see [Negative Test: Misrouted Request](#5-negative-test-misrouted-request). For general failure handling, see [Graceful Failure & Escalation](../capability-scenarios/graceful-failure-and-escalation.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's clarifying question addresses the right ambiguity |
| General Quality | Assess whether the clarifying question is helpful, specific, and moves the conversation forward |
| Keyword Match (Any) | Check that the response includes key terms related to the possible categories (indicating the agent recognizes the ambiguity) |

> **Tip:** The goal is not that the agent always asks a question — it is that the agent asks a question **when it should**. If the input is clear, the agent should classify directly. The evaluation pattern here is about recognizing the boundary between "I can classify this" and "I need more information."

### Setup Steps

1. Identify the most common ambiguity points in your triage workflow — where do categories overlap? Where do users provide insufficient detail?
2. Write test cases that sit on category boundaries (e.g., a request that could be billing or shipping) or provide minimal information (e.g., "I need help with my account").
3. For each test case, set the expected value to describe the **type of clarifying question** the agent should ask — not the exact wording, but the ambiguity it should address.
4. Select **Compare Meaning** to verify the clarifying question targets the right ambiguity.
5. Add **General Quality** test cases to verify the question is well-formed and actionable.
6. Also include 2–3 clear-cut inputs alongside the ambiguous ones — the agent should NOT ask clarifying questions for these. Verify with Keyword Match (All) that it classifies directly.
7. Run the evaluation and look for two failure modes: asking unnecessary questions for clear inputs, and guessing instead of asking for ambiguous inputs.

### Anti-Pattern

> **Anti-Pattern: Rewarding Any Question**
> Not all clarifying questions are equal. An agent that responds to "I need help" with "Can you tell me more?" is technically asking a clarifying question, but it is not helpful. Test that the agent asks **specific, targeted** questions that narrow the ambiguity efficiently. "Are you experiencing a billing issue or a problem with your order delivery?" is far better than "Can you provide more details?"

### Evaluation Patterns

**Pattern: Targeted Clarifying Question**
When the input is ambiguous, the agent should ask a question that directly addresses the ambiguity. If the request could be billing or shipping, the agent should ask which one — not ask an open-ended "tell me more." Test that the clarifying question names the possible categories or narrows the issue.

**Pattern: Appropriate Confidence Threshold**
The agent should not ask clarifying questions for every input — only when ambiguity is genuine. Test with a mix of clear and ambiguous inputs and verify the agent classifies directly when it has enough information and asks only when it does not.

**Pattern: Progressive Clarification**
If the user's answer to the first clarifying question is still ambiguous, the agent should ask a follow-up — not guess. But it should also not ask more than 2–3 clarifying questions before either routing with its best assessment or escalating. Test multi-turn clarification sequences.

**Pattern: Graceful Handling of Uncooperative Responses**
Sometimes users respond to clarifying questions with "I don't know" or "Just help me." Test that the agent either routes to a general queue, provides a best-effort classification, or escalates to a human — rather than looping indefinitely.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Ambiguous IT request | "My system isn't working right." | Agent asks a clarifying question that distinguishes between hardware, software, network, or access issues | Compare Meaning + Keyword Match (Any) |
| 2 | Boundary: billing vs. shipping | "There's a problem with my order." | Agent asks whether the issue is about the charge/payment or about the delivery/shipment | Compare Meaning + Keyword Match (Any) |
| 3 | Minimal information HR request | "I have a question about my benefits." | Agent asks which type of benefit (health, dental, vision, retirement) or what specifically the user needs help with | General Quality + Compare Meaning |
| 4 | Clear-cut input — no question needed | "I need to reset my password for the Salesforce application." | Response includes "Access" or "Password Reset" — agent classifies directly without unnecessary clarification | Keyword Match (Any) + Compare Meaning |
| 5 | Uncooperative response to clarification | Turn 1: "I need help." Turn 2 (agent asks clarifying question): "I don't know, just help me." | Agent routes to a general support queue or escalates to a human rather than looping | General Quality + Compare Meaning |
| 6 | Multi-issue ambiguity | "I'm having problems with my computer and also need to ask about my timesheet." | Agent acknowledges both issues and asks which the user would like to address first, or addresses the more urgent one | General Quality + Compare Meaning |
| 7 | Progressive clarification | Turn 1: "Something is wrong with the network." Turn 2 (agent asks if it is VPN, Wi-Fi, or internal): "The internet one." | Agent narrows to Wi-Fi or external connectivity and either classifies or asks one more targeted question | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 3–5 ambiguous inputs plus 2–3 clear-cut inputs (to verify the agent does not over-clarify).
- **Clarification quality threshold:** At least 80% of clarifying questions should be specific and targeted (not generic "tell me more" questions), assessed via General Quality.
- **Watch for two failure modes:** over-clarifying (asking questions for clear inputs) and under-clarifying (guessing when the input is genuinely ambiguous).
- **Rerun after:** Any changes to category definitions, agent instructions, or disambiguation prompts.
- Test at least one "uncooperative user" scenario where the user does not provide a clear answer to the clarifying question.

---

## 5. Negative Test: Misrouted Request

### When to Use

This scenario is specifically designed to catch **incorrect routing** — cases where the agent sends the request to the wrong team, department, or resource. While other scenarios test that the agent gets classification and priority right, this scenario focuses on verifying the agent does NOT make common routing errors.

Use this scenario when:
- Your agent routes requests to specific teams, queues, or systems
- Misrouting has real consequences — delayed resolution, privacy violations (sensitive data sent to wrong team), or customer frustration
- You want to test that the agent avoids known failure patterns specific to your routing logic

> **Related scenarios:** For verifying correct classification, see [Verifying Classification Accuracy](#1-verifying-classification-accuracy). For verifying correct handoff actions, see [Testing Context Preservation During Handoff](#2-testing-context-preservation-during-handoff). For safety implications of misrouting sensitive data, see [Safety & Boundary Enforcement](../capability-scenarios/safety-and-boundary-enforcement.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes the CORRECT routing destination (and does not include the wrong one) |
| Compare Meaning | Confirm the routing rationale matches the expected destination logic |
| Capability Use (All) | Verify the correct transfer topic or routing action fires — not the wrong one |
| General Quality | Assess whether the routing decision is defensible and well-reasoned |

> **Tip:** Negative testing requires thinking about what SHOULD NOT happen, not just what should. For each test case, document both the expected destination and the likely incorrect destination. Then verify the agent avoids the wrong one.

### Setup Steps

1. Review your agent's routing map — all the possible destinations and the rules for each.
2. Identify the **most commonly confused pairs** of destinations. Which teams' responsibilities overlap? Which routing rules have the most exceptions?
3. For each confused pair, write test cases with inputs that might trick the agent into routing to the wrong one. Focus on inputs with misleading keywords (e.g., "billing" mentioned in a shipping issue).
4. Set expected values to the CORRECT destination. Also note the INCORRECT destination the agent might choose.
5. Select **Keyword Match (All)** to verify the correct destination appears, and use **Capability Use (All)** if routing involves triggering a specific action.
6. Run the evaluation and pay special attention to test cases where the agent chose the wrong destination — these reveal systematic routing logic problems.

### Anti-Pattern

> **Anti-Pattern: Only Testing With Correct-Destination Inputs**
> If your test cases only include inputs that clearly belong to one destination, you are testing classification — not misrouting prevention. Negative tests must include inputs **designed to mislead**: requests that mention the wrong team's keywords, requests that have been previously misrouted in production, or requests that require understanding nuance beyond surface-level keyword matching.

### Evaluation Patterns

**Pattern: Misleading Keyword Resistance**
Test inputs that contain keywords associated with the wrong destination. "I have a billing question about my shipping costs" mentions "billing" but is about shipping. "My software license is expired and I need new hardware" mentions both Software and Hardware. The agent should route based on the actual issue, not keyword frequency.

**Pattern: Previously Misrouted Scenarios**
If you have production data showing common misroutes, turn those into test cases. These are the highest-value negative tests because they represent real failure patterns.

**Pattern: Adjacent-Team Boundary**
For teams with overlapping responsibilities (e.g., IT Security vs. IT Infrastructure, Customer Success vs. Customer Support), test inputs that sit on the boundary and verify the agent routes to the correct side based on your defined rules.

**Pattern: Sensitive Data Routing Guard**
Some misroutes have privacy or security implications — a request containing health information sent to the general IT queue, or a complaint containing financial details sent to an unrelated team. Test that sensitive requests are routed to teams authorized to handle that data type.

**Pattern: Cascade Misroute Prevention**
After the agent routes, verify it does not trigger a secondary, unintended routing action. For example, routing to Team A should not also send a copy to Team B unless that is the intended design.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Misleading keyword: billing mentioned in shipping issue | "I have a billing question — why does my receipt show a shipping charge when I selected free shipping?" | Response routes to "Shipping" or "Order Fulfillment" — NOT "Billing" | Keyword Match (All) + Compare Meaning |
| 2 | Misleading keyword: software mentioned in hardware issue | "The software update fried my laptop and now it won't turn on at all." | Response routes to "Hardware" — NOT "Software" | Keyword Match (All) + Compare Meaning |
| 3 | Adjacent-team boundary: IT Security vs. IT Infrastructure | "I got an email asking me to click a link and verify my password — is this legit?" | Response routes to "IT Security" or "Phishing" — NOT general "IT Support" | Keyword Match (Any) + Compare Meaning |
| 4 | Sensitive data routing: health information | "I need to submit my doctor's note for my medical leave of absence." | Agent routes to "HR" or "Benefits/Leave" — NOT general support queue | Capability Use (All) + Keyword Match (Any) |
| 5 | Previously misrouted: complaint about process, not product | "Your return process is the worst I've ever experienced — I've been waiting a month for my refund." | Response routes to "Returns/Refunds" — NOT "Product Quality" or "Complaints" | Compare Meaning + Keyword Match (Any) |
| 6 | Multi-keyword confusion | "I need help accessing the billing portal — my login isn't working." | Response routes to "Access/IT Support" for the login issue — NOT "Billing" for the portal content | Compare Meaning + Keyword Match (Any) |
| 7 | Cascade prevention: single route only | "I want to cancel my subscription effective immediately." | Agent invokes only the "Cancellations" routing action — does not also trigger "Billing" or "Retention" unless designed to | Capability Use (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 2 negative test cases per commonly confused destination pair. If you have 5 possible routing destinations, target 8–10 negative test cases total.
- **Misroute rate threshold:** Target a 0% misroute rate for sensitive data scenarios and less than 5% for all other scenarios.
- **Use production data** — if you have logs of previously misrouted requests, these are your highest-priority negative test cases.
- **Rerun after:** Any changes to routing rules, team responsibilities, or the set of available routing destinations.
- **Pair with context preservation testing** — a request can be routed to the right team but with wrong or missing context, which is a different failure mode.
- Document both the correct AND incorrect expected destination for each test case — this makes results easier to interpret.
