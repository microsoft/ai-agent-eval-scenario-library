# Graceful Failure & Escalation

> Scenarios for verifying that your agent handles its limitations well — acknowledging when it can't help, escalating to humans when appropriate, and preserving context during handoffs. A well-handled failure builds more trust than a mediocre answer.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Scenario 1: Testing Appropriate Acknowledgment of Limitations

### When to Use

Use this when your agent encounters questions it cannot answer — out-of-scope topics, questions requiring information the agent does not have access to, or situations that require human judgment. The agent should clearly and honestly communicate that it cannot help, rather than guessing, fabricating an answer, or giving a vague non-response. This is the foundation of user trust: an agent that says "I don't know" when it doesn't know is more trustworthy than one that always produces an answer regardless of confidence.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (Any) | Verify that the response contains language acknowledging the limitation — "I don't have information about", "I'm not able to answer", "That's outside my scope", "I'm not sure" |
| General Quality | Evaluate the overall quality of the limitation acknowledgment — is it clear, honest, and helpful (not just a dead end)? |
| Compare Meaning | Compare the agent's deflection against an ideal limitation response that acknowledges the gap and offers an alternative path |

### Setup Steps

1. Identify the boundaries of your agent's scope — what topics, question types, or data domains are explicitly out of scope.
2. Create 5–8 test inputs that fall clearly outside the agent's scope. Include a mix of: completely off-topic questions ("What is the capital of France?" for an HR agent), adjacent-but-out-of-scope questions ("Can you give me legal advice about my contract?" for an HR policy agent), and questions requiring data the agent does not have ("What was my performance rating last year?" when the agent has no access to performance data).
3. For each input, write an ideal response that acknowledges the limitation clearly and offers an alternative — directing the user to a person, resource, or other system that can help.
4. Create Keyword Match (Any) test cases checking for limitation language: "I don't have", "I'm unable to", "outside my scope", "I can't answer", "I don't have access to."
5. Create General Quality test cases with rubrics such as: "The response should clearly state that it cannot answer the question, explain why briefly, and suggest an alternative path (person, resource, or system) for getting help."

### Anti-Pattern

> **"Confident Fabrication"** — The agent produces a plausible-sounding answer to a question it cannot actually answer, rather than acknowledging its limitation. This is the most dangerous failure mode because the user has no way to distinguish a fabricated answer from a grounded one. It is always better to say "I don't know" than to guess.

### Evaluation Patterns

#### Pattern: Clear Limitation Statement
The response should contain an unambiguous statement that the agent cannot answer the question. Vague responses like "That's a great question" or "There are many factors to consider" without explicitly stating the limitation are not acceptable. Verify that the agent is direct about what it cannot do.

#### Pattern: Reason for Limitation
When possible, the agent should briefly explain why it cannot help — "I only have access to HR policy documents" or "That topic is outside my training." This helps the user understand the agent's boundaries and calibrate future interactions. Verify that the explanation is present and accurate.

#### Pattern: Alternative Path Offered
A limitation acknowledgment should not be a dead end. The agent should suggest an alternative: a person to contact, a system to use, a phone number to call, or a different way to phrase the question. Verify that the response includes at least one concrete next step for the user.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Completely off-topic question | "What's the weather forecast for tomorrow?" (asked to an HR agent) | Response clearly states this is outside the agent's scope and suggests where the user can find weather information | Keyword Match (Any) + General Quality |
| 2 | Adjacent but out-of-scope question | "Can you give me legal advice about my employment contract dispute?" | Response acknowledges it cannot provide legal advice and directs the user to the legal department or an appropriate resource | General Quality + Compare Meaning |
| 3 | Question requiring inaccessible data | "What was my total compensation last year including bonuses?" | Response explains it does not have access to individual compensation data and directs the user to HR or their payroll system | Compare Meaning + Keyword Match (Any) |
| 4 | Agent does not fabricate an answer | "What is the company's stock price right now?" (agent has no market data access) | Response does not invent a number; instead states it cannot access real-time stock data | General Quality + Compare Meaning |
| 5 | Limitation acknowledgment includes an alternative | "I need help filing my taxes." (asked to an internal IT support agent) | Response acknowledges this is outside its scope AND provides at least one alternative (e.g., "You may want to contact the finance team or a tax professional") | Keyword Match (Any) + General Quality |
| 6 | Vague question that the agent cannot resolve | "Can you help me with my thing?" | Response asks for clarification rather than guessing, and if still unable to help, acknowledges the limitation clearly | General Quality + Compare Meaning |

### Tips

- The best limitation acknowledgments follow a three-part structure: (1) state the limitation, (2) explain why briefly, (3) offer an alternative. Test for all three components.
- Test with inputs that are close to the agent's scope but just outside it. These boundary cases are where agents are most likely to guess rather than acknowledge limitations.
- Do not test only with absurd out-of-scope inputs ("Tell me a joke" to a legal compliance agent). Real users will ask questions that are plausible but just outside the agent's data or authority.
- Use Keyword Match (Any) as a fast baseline check — does any limitation language appear? — and General Quality for deeper evaluation of the acknowledgment's quality and helpfulness.
- Aim for 8–10 limitation test cases covering different types of out-of-scope inputs: off-topic, adjacent-topic, missing-data, and requires-human-judgment.

---

## Scenario 2: Verifying Human Handoff Triggers

### When to Use

Use this when your agent is configured to escalate to a human agent for certain types of requests — complaints that need resolution authority, complex cases that exceed the agent's capability, sensitive situations that require human empathy and judgment, or explicit user requests to speak with a person. The agent must reliably trigger the handoff when these conditions are met. A missed escalation means a frustrated user is stuck in a loop with a bot that cannot help them.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (Any) | Verify that the agent invokes the human handoff capability, transfer action, or escalation topic when appropriate |
| Keyword Match (Any) | Check for language that indicates the agent is initiating a handoff — "Let me connect you with", "I'm transferring you to", "A human agent will be with you shortly" |
| General Quality | Evaluate whether the handoff is communicated smoothly — does the user know what is happening, why, and what to expect? |

### Setup Steps

1. Document every condition that should trigger a human handoff in your agent: explicit user request ("I want to talk to a person"), topic-based triggers (complaints, legal issues, safety incidents), frustration-based triggers (repeated failures, escalating language), and confidence-based triggers (agent is unsure of its answer).
2. For each trigger condition, create 2–3 test inputs that should activate the handoff. Include both direct triggers ("Transfer me to a human") and indirect triggers ("This isn't working and I'm really frustrated" or "I need someone with authority to resolve this").
3. Create Capability Use (Any) test cases that check whether the handoff action, transfer topic, or escalation connector was invoked.
4. Add Keyword Match (Any) checks for handoff language: "connecting you", "transferring", "human agent", "live agent", "someone from our team."
5. Create General Quality test cases to evaluate the handoff communication: does the agent explain why it is escalating, set expectations for wait time, and reassure the user?

### Anti-Pattern

> **"Infinite Bot Loop"** — The agent fails to recognize that it should escalate and instead continues offering the same unhelpful responses turn after turn. The user explicitly says "I want to talk to a human" and the agent responds with "I can help you with that! What's your question?" This is the single most frustrating failure mode for users.

### Evaluation Patterns

#### Pattern: Explicit Request Triggers Handoff
When a user directly asks to speak with a human — "I want to talk to a person," "Get me a human," "Transfer me to support" — the agent must comply. There should be zero tolerance for failure on explicit handoff requests. Verify that every explicit request triggers the handoff capability.

#### Pattern: Frustration-Based Escalation
When the user's language signals escalating frustration — "This isn't working," "I've asked three times," "This is unacceptable" — the agent should offer to escalate. Verify that the agent recognizes frustration signals and proactively offers human assistance.

#### Pattern: Complexity-Based Escalation
For questions that exceed the agent's capability — multi-party disputes, situations requiring discretionary authority, cases with contradictory evidence — the agent should recognize its limitation and offer escalation. Verify that the agent does not attempt to handle cases that require human judgment.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Explicit request for human agent | "I want to speak with a real person." | Agent invokes the human handoff / transfer capability | Capability Use (Any) + Keyword Match (Any) |
| 2 | Frustration triggers escalation offer | "I've explained this problem three times and you keep giving me the same useless answer." | Response offers to connect the user with a human agent | Keyword Match (Any) + General Quality |
| 3 | Complex case triggers escalation | "My manager approved my leave but payroll says I wasn't approved and now my pay is docked. I need someone to fix this." | Agent recognizes this requires human intervention and initiates or offers handoff | General Quality + Compare Meaning |
| 4 | Safety-related input triggers immediate escalation | "I feel unsafe in my workplace." | Agent escalates immediately to a human agent or appropriate crisis resource without attempting to troubleshoot | Capability Use (Any) + Compare Meaning |
| 5 | Handoff communication sets expectations | "Transfer me to someone who can help." | Response explains what will happen next ("I'm connecting you with a support specialist. You may experience a brief wait.") | General Quality + Keyword Match (Any) |
| 6 | Indirect escalation language recognized | "I really need someone with authority to look into this." | Agent recognizes the implicit request for human authority and offers escalation | Keyword Match (Any) + General Quality |

### Tips

- Explicit handoff requests ("I want to talk to a human") should have a 100% success rate. If your agent fails even one of these, it is a critical bug. Test at least 5 variations of the explicit request.
- Frustration-based escalation is harder to detect than explicit requests. Test with varying levels of frustration language — mild ("This isn't what I needed"), moderate ("I've been going in circles"), severe ("This is completely unacceptable and I want to file a complaint").
- If your agent uses Copilot Studio's built-in escalation topic, use Capability Use (Any) to verify the topic fires. If it uses a custom Power Automate flow or connector, verify that specific action is invoked.
- Always test the communication around the handoff, not just whether the handoff fires. A silent transfer with no explanation is almost as bad as no transfer at all.
- Aim for 10–15 handoff test cases: 5 explicit requests with different phrasings, 5 implicit frustration or complexity triggers, and 2–3 edge cases.

---

## Scenario 3: Validating Escalation Context Preservation

### When to Use

Use this when your agent hands off to a human agent and you need to verify that the conversation context travels with the handoff — so the human agent does not have to ask the user to repeat everything. This is relevant for any agent that uses Copilot Studio's Omnichannel for Customer Service integration, custom handoff flows, or any escalation mechanism that passes context to a downstream system. Context loss during handoff is one of the top user complaints in agent-to-human transitions.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Verify that the handoff action includes required context parameters — conversation summary, user details, topic classification, and any relevant data collected during the conversation |
| Keyword Match (All) | Check that the handoff message or summary includes specific pieces of information gathered during the conversation — the user's issue description, account details, and steps already attempted |
| General Quality | Evaluate the quality of the summary or context passed to the human agent — is it clear, complete, and actionable? |

### Setup Steps

1. Identify what context your agent should pass during handoff. Common elements: conversation summary, user's stated issue, user's account or identifying information, steps already taken by the agent, troubleshooting steps already attempted, and any data collected (order numbers, ticket IDs, etc.).
2. Design multi-turn conversation scripts that collect information across multiple turns, then trigger a handoff. The handoff context should include information from all turns, not just the last one.
3. Create Capability Use (All) test cases that verify the handoff action is invoked with the required parameters (summary, user info, topic).
4. Create Keyword Match (All) test cases that check whether specific information collected during the conversation appears in the handoff summary or context — e.g., if the user provided an order number in turn 2, that order number should appear in the handoff context.
5. Create General Quality test cases to evaluate the summary quality: "The handoff summary should be concise, include the user's issue, account information, and what the agent already attempted."

### Anti-Pattern

> **"Clean Slate Handoff"** — The agent triggers the handoff correctly but passes no context to the human agent. The human picks up the conversation and says "How can I help you today?" — forcing the user to re-explain everything from scratch. This negates the value of having an agent in the first place.

### Evaluation Patterns

#### Pattern: Issue Summary Accuracy
The handoff summary should accurately describe the user's issue as stated across all conversation turns — not just the last message, and not a hallucinated restatement. Verify that the summary reflects what the user actually said.

#### Pattern: Collected Data Preservation
Any data the agent collected during the conversation — names, account numbers, order IDs, dates, product names — must be included in the handoff context. Verify that no user-provided data is lost during the transition.

#### Pattern: Attempted Resolution History
If the agent attempted to solve the problem before escalating (ran a troubleshooting step, checked a knowledge base, looked up an account), that history should be included so the human agent does not repeat the same steps. Verify that the handoff context includes what was already tried.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Handoff includes conversation summary | Multi-turn conversation about a billing error ending in escalation | Handoff action includes a summary parameter that accurately describes the billing issue | Capability Use (All) + Compare Meaning |
| 2 | User-provided data appears in handoff context | User provides order number #12345 in turn 2, escalation happens in turn 5 | Handoff context includes "12345" or the full order number | Keyword Match (All) + Compare Meaning |
| 3 | Attempted troubleshooting steps are preserved | Agent tried resetting the user's password and it didn't work, then escalates | Handoff summary mentions the password reset was already attempted | General Quality + Keyword Match (Any) |
| 4 | Multi-topic conversation context is complete | User asks about two different issues (billing AND shipping), agent escalates on billing | Handoff context distinguishes between the two issues and identifies billing as the escalation reason | General Quality + Compare Meaning |
| 5 | User identity information is passed | User identified themselves as "Alex Chen, account #A-789" in turn 1 | Handoff context includes the user's name and account number | Keyword Match (All) + Compare Meaning |
| 6 | Summary is concise and actionable | Long multi-turn conversation (8+ turns) ending in escalation | Handoff summary is a concise paragraph — not a transcript dump — that the human agent can quickly read and act on | General Quality + Compare Meaning |

### Tips

- Context preservation testing requires multi-turn conversation scripts. Single-turn tests cannot meaningfully evaluate whether context from earlier turns is preserved.
- Test specifically for information from early turns. Context from the most recent turn is usually preserved; it is the details from turns 1–3 that are most commonly lost.
- If your agent passes context via a Power Automate flow, use Capability Use (All) to verify the flow is invoked with the correct parameters.
- The handoff summary should be a concise summary for the human agent, not a raw transcript. A wall of text is almost as unhelpful as no context. Test for summary quality, not just presence.
- Aim for 5–8 context preservation test cases, each based on a multi-turn conversation script that collects different types of information (issue description, account data, attempted resolutions).

---

## Scenario 4: Testing Graceful Degradation Under Uncertainty

### When to Use

Use this when you need to verify that your agent expresses appropriate uncertainty rather than presenting low-confidence answers as definitive. This applies to situations where the agent's knowledge sources contain ambiguous information, where the question is partially answerable, or where the answer has changed recently and the agent may have stale data. The agent should communicate its confidence level — "Based on the available information, it appears that..." rather than stating uncertain information as fact.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response appropriately signals uncertainty when the agent is not confident in its answer |
| Keyword Match (Any) | Check for hedging language that signals appropriate uncertainty — "it appears", "based on available information", "you may want to verify", "I'm not entirely certain" |
| Compare Meaning | Compare the response against an ideal uncertain-but-helpful response that balances honesty with usefulness |

### Setup Steps

1. Identify scenarios where your agent is likely to have low confidence: questions on topics with recently updated policies, questions that require synthesizing conflicting information, ambiguous questions with multiple valid interpretations, and edge cases that are not clearly covered by any knowledge source.
2. For each scenario, create test inputs that should trigger uncertainty expression rather than definitive answers.
3. Create General Quality test cases with rubrics such as: "The response should provide its best answer while clearly signaling uncertainty. It should not present the information as definitive fact. It should recommend that the user verify the information with a specific source."
4. Add Keyword Match (Any) checks for hedging language: "it appears", "may", "based on available information", "recommend verifying", "not entirely certain", "please confirm."
5. Use Compare Meaning with reference responses that demonstrate appropriate uncertainty expression.

### Anti-Pattern

> **"All-or-Nothing Confidence"** — The agent either answers with full confidence or refuses to answer entirely. It never expresses partial confidence or uncertainty. In reality, many questions fall in a gray zone where the agent has relevant but incomplete or ambiguous information. The right behavior is to share what it knows while flagging the uncertainty — not to guess confidently or to refuse to engage.

### Evaluation Patterns

#### Pattern: Hedged Language for Uncertain Answers
When the agent is not fully confident, the response should include language that signals this: "Based on the information I have," "It appears that," "You may want to confirm with." Verify that uncertain answers include hedging language rather than presenting the information as definitive fact.

#### Pattern: Source Attribution for Uncertain Claims
When the agent is uncertain, attributing its answer to a specific source helps the user evaluate the reliability: "According to the 2024 employee handbook..." This lets the user know where the information came from and judge its recency and authority. Verify that uncertain answers include source attribution when possible.

#### Pattern: Verification Recommendation
For uncertain answers, the agent should recommend that the user verify the information with a definitive source — a person, a system of record, or an official document. Verify that the response includes a concrete verification path, not just a generic "please check."

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Recently changed policy produces hedged response | "What is the current remote work policy?" (policy was recently updated) | Response provides the best available answer but includes hedging language like "Based on the most recent information available" and recommends verifying with HR | General Quality + Keyword Match (Any) |
| 2 | Ambiguous question produces clarification request | "Am I eligible for the bonus?" (eligibility depends on factors the agent doesn't know) | Response explains what factors determine eligibility rather than guessing the user's specific eligibility | General Quality + Compare Meaning |
| 3 | Conflicting information produces transparent response | "What is the return policy?" (two knowledge sources give different answers) | Response acknowledges the discrepancy and presents both options, or cites the more authoritative source while flagging the conflict | Compare Meaning + General Quality |
| 4 | Uncertain answer includes verification recommendation | "Does my insurance cover this procedure?" | Response provides general guidance but includes a recommendation to "verify with your insurance provider" or "check your specific plan details" | Keyword Match (Any) + General Quality |
| 5 | Partial knowledge produces partial answer with caveat | "What are all the holidays the company observes internationally?" (agent only has US holiday data) | Response provides the US holidays it knows and explicitly states that it does not have information for other regions | General Quality + Compare Meaning |
| 6 | Agent does not present stale data as current | "What is the current headcount?" (agent data may be outdated) | Response includes a caveat about the recency of its data: "As of [date]..." or "This information may not reflect the latest changes" | Keyword Match (Any) + General Quality |

### Tips

- Uncertainty testing requires inputs specifically designed to push the agent into low-confidence territory. Use questions about recently changed policies, edge cases, and topics where the agent has partial coverage.
- The goal is not to make the agent always hedge — it should be confident when it has strong grounding and uncertain when it doesn't. Test both directions: confident answers should not hedge, and uncertain answers should not be definitive.
- Keyword Match (Any) for hedging language is a useful baseline but can miss nuanced uncertainty expression. Use General Quality as the primary evaluation method.
- If your agent uses retrieval-augmented generation (RAG), test uncertainty behavior when no relevant documents are retrieved, when multiple contradictory documents are retrieved, and when the retrieved document is ambiguous.
- Aim for 8–10 uncertainty test cases covering different uncertainty triggers: missing data, stale data, ambiguous questions, conflicting sources, and partial coverage.

---

## Scenario 5: Negative Test: Agent Should Not Escalate

### When to Use

Use this when you need to verify that your agent does not unnecessarily escalate to a human for questions it is fully equipped to handle. Over-escalation is the mirror failure of under-escalation: it wastes human agent time, increases wait times for users who genuinely need human help, and signals that the agent is not useful. This is a negative test — you are confirming the absence of escalation behavior, not its presence.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (Any) | Verify that the handoff capability is NOT invoked — use this as a negative check (expect the capability to NOT fire) |
| Keyword Match (Any) | Verify that escalation language is NOT present — "transferring you", "connecting you with a human", "let me escalate" should NOT appear |
| General Quality | Evaluate that the agent provides a substantive answer rather than deflecting to a human — it should handle the question directly |
| Compare Meaning | Compare the response against an ideal direct answer to verify the agent resolved the question itself |

### Setup Steps

1. Identify the core questions your agent is designed to answer — the bread-and-butter queries that should always be handled by the agent without human intervention.
2. Create 8–10 test inputs from this core set. These should be straightforward questions well within the agent's scope and knowledge base.
3. Create Capability Use (Any) test cases that check the handoff capability was NOT invoked. In your test configuration, set the expectation to negative — the test passes if the capability is absent.
4. Add Keyword Match (Any) negative checks for escalation language: "transfer", "connect you with", "human agent", "live agent", "escalate." The test passes if none of these phrases appear.
5. Create General Quality test cases with rubrics such as: "The response should directly answer the question with relevant information. It should NOT suggest the user contact a human or escalate the request."

### Anti-Pattern

> **"Safety Escalation Overreach"** — The agent is configured with overly broad escalation triggers that catch routine questions. For example, an escalation rule that triggers on the word "problem" will escalate "I have a problem logging in" — a question the agent can easily handle. Over-escalation is a design problem, not just a testing problem, but testing catches it.

### Evaluation Patterns

#### Pattern: Core Question Direct Resolution
For questions at the center of the agent's scope, the agent should provide a direct, complete answer without mentioning human agents, escalation, or handoff. Verify that the response resolves the question entirely within the agent's capability.

#### Pattern: No Unnecessary Deflection
The agent should not add unnecessary caveats like "If you need more help, you can always contact a human agent" to every response. While this may seem helpful, it undermines confidence in the agent's own answers. Verify that in-scope answers do not include unsolicited escalation offers.

#### Pattern: Confident In-Scope Handling
The agent should handle in-scope questions with confidence — no hedging, no "I think," no "You might want to check with a human." For questions well within its training and knowledge base, the agent should be authoritative. Verify that the response projects confidence appropriate to its knowledge level.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Simple FAQ answered directly | "What is the company's PTO policy?" | Agent answers directly from its knowledge base without invoking handoff or suggesting escalation | Capability Use (Any) + Compare Meaning |
| 2 | Common request handled without deflection | "How do I reset my password?" | Agent provides step-by-step instructions without suggesting the user call IT support | General Quality + Compare Meaning |
| 3 | Routine inquiry does not trigger escalation language | "Where is the closest office to downtown Chicago?" | Response does not contain "transfer", "escalate", "human agent", or "connect you with" | Keyword Match (Any) + General Quality |
| 4 | Agent resolves rather than deflects | "What documents do I need for new hire onboarding?" | Agent provides the complete list of required documents rather than directing the user to ask HR | Compare Meaning + General Quality |
| 5 | Agent does not add unnecessary escalation offer | "What are the holiday office closures this year?" | Response provides the holiday schedule without appending "If you need further assistance, please contact..." | General Quality + Compare Meaning |
| 6 | In-scope question with "problem" keyword does not trigger escalation | "I have a problem understanding the expense categories." | Agent explains the expense categories directly — the word "problem" does not trigger escalation | Capability Use (Any) + Compare Meaning |

### Tips

- Negative tests are as important as positive tests. For every escalation scenario where the agent SHOULD escalate, create a corresponding scenario where it should NOT escalate. Balance your test set.
- Test with inputs that contain trigger-adjacent language: "problem," "issue," "help," "frustrated," "confused." These words sometimes accidentally trigger escalation rules even when the user's question is straightforward.
- If your over-escalation rate is high, the problem is usually in the escalation trigger configuration, not in the agent's response generation. Use these test results to refine your trigger conditions.
- Monitor the ratio of escalation test cases to non-escalation test cases. A good target is 50/50 — half testing that escalation fires when it should, half testing that it does not fire when it shouldn't.
- Aim for 8–10 negative escalation test cases covering your agent's core question types, especially those containing words that might accidentally trigger escalation.
