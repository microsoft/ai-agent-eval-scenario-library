# Trigger Routing

> Scenarios for verifying that your agent routes user inputs to the correct topic or conversation flow. Applies to any agent with multiple topics configured, especially those with similar or overlapping trigger phrases.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## 1. Verifying Direct Topic Match

Does the agent trigger the correct topic for a clear, unambiguous input?

### When to Use

Your agent has multiple topics configured and you need to confirm that clear, straightforward user inputs route to the intended topic. This is the foundational routing test — if the agent can't match unambiguous inputs to the right topic, more nuanced routing will also fail.

> **Related scenarios:** If the input is ambiguous and could match multiple topics, see [Scenario 2: Testing Disambiguation Between Similar Topics](#2-testing-disambiguation-between-similar-topics). If you need to verify the agent calls the right *tool* (not topic), see [Tool & Connector Invocations](tool-and-connector-invocations.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the expected topic was triggered — the primary check for routing accuracy |
| Keyword Match (Any) | Catches topic-specific language in the response as a secondary signal (e.g., response mentions "benefits" for the Benefits Enrollment topic) |

> **Tip:** Capability Use is the authoritative routing check. Response text can appear correct even if the wrong topic fired — for example, the Fallback topic might generate a plausible-sounding answer using generative AI instead of routing to the intended topic with its curated response.

### Setup Steps

1. List every topic in your agent. For each, write 2–3 clear, unambiguous user inputs that should trigger it.
2. Create test cases for each input. Set the **Method** to **Capability Use (All)** with the expected topic name.
3. Use varied phrasing — do not just copy the exact trigger phrases configured in the topic. Real users will paraphrase.
4. Optionally add **Keyword Match (Any)** with topic-specific response language as a secondary check.
5. Run the eval set. Any test case where the expected topic was NOT triggered indicates a routing gap.

### Anti-Pattern

> **Anti-Pattern: Using Only Exact Trigger Phrases as Test Inputs**
> If your test inputs are identical to the trigger phrases configured in your topics (e.g., the topic trigger is "check order status" and your test input is "check order status"), you are testing trigger-phrase matching, not real user routing. Users will say "where's my package?" or "I ordered something last week and want to know if it shipped." Test with the natural language your users actually use.

### Evaluation Patterns

**Pattern A: One-to-One Topic Verification**
For each topic, verify that at least 2–3 different phrasings of the same intent all route to that topic. This confirms the topic's trigger coverage is broad enough to handle natural variation.

**Pattern B: Cross-Domain Clarity**
Test inputs from clearly different domains to confirm there is no cross-contamination. Example: "How do I enroll in benefits?" should route to the Benefits Enrollment topic, not the Payroll Inquiries topic, even though both are HR-related.

**Pattern C: Multi-Language or Regional Phrasing**
If your agent serves users who phrase things differently based on region or language, test those variations. Example: "I'd like to book holiday" (UK English) and "I want to request PTO" (US English) should both route to the Time Off Request topic.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Direct match to Password Reset topic | "I forgot my password and need to reset it" | Capability: `Password Reset` topic | Capability Use (All) + Keyword Match (Any) |
| 2 | Direct match to Order Status topic | "Where's my package? I ordered it three days ago" | Capability: `Order Status` topic | Capability Use (All) + Keyword Match (Any) |
| 3 | Direct match to Benefits Enrollment topic | "How do I sign up for dental insurance?" | Capability: `Benefits Enrollment` topic | Capability Use (All) + Keyword Match (Any) |
| 4 | Paraphrased input routes correctly | "My login credentials aren't working" | Capability: `Password Reset` topic | Capability Use (All) + Keyword Match (Any) |
| 5 | Topic-specific language in response | "I need to change my direct deposit information" | Keywords (any): "payroll", "direct deposit", "bank account" | Keyword Match (Any) + Capability Use (All) |
| 6 | Cross-domain clarity: HR vs. IT | "How do I set up my new laptop?" | Capability: `IT Equipment Setup` topic — NOT `Onboarding` topic | Capability Use (All) + Compare Meaning |

### Tips

- Write at least **3 test cases per topic** using different phrasings. Minimum coverage = 3 x (number of topics).
- Avoid copying trigger phrases verbatim — test with realistic, natural-language variations.
- Target: **100% of unambiguous routing test cases should trigger the expected topic.** Any miss is a critical routing failure.
- Rerun this scenario **every time you add, remove, or modify topic trigger phrases**.
- If a topic consistently fails to trigger, review its trigger phrases and consider adding more variations or switching to semantic matching.

---

## 2. Testing Disambiguation Between Similar Topics

When input could match multiple topics, does the agent disambiguate correctly?

### When to Use

Your agent has topics with overlapping domains, similar trigger phrases, or related subject matter, and you need to verify the agent routes to the right one — or appropriately asks the user to clarify. This scenario is essential for agents with many topics where keyword overlap is inevitable.

> **Related scenarios:** For inputs that clearly match a single topic, see [Scenario 1: Verifying Direct Topic Match](#1-verifying-direct-topic-match). If no topic matches at all, see [Scenario 3: Validating Fallback/Default Routing](#3-validating-fallbackdefault-routing).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the correct topic was selected when the agent can determine intent from context |
| Compare Meaning | Verifies the disambiguation prompt is clear and presents the right options when the agent asks for clarification |
| General Quality | Assesses whether the disambiguation experience is smooth and doesn't feel like an interrogation |

> **Tip:** Some ambiguous inputs have a "right answer" based on context or user history. Others are genuinely ambiguous and the correct behavior is to ask the user. Design test cases for both situations.

### Setup Steps

1. Identify topic pairs (or groups) with overlapping domains. Common overlaps:
   - "Password Reset" vs. "Account Lockout"
   - "Return an Item" vs. "Exchange an Item"
   - "Billing Question" vs. "Payment Issue"
   - "New Hire Onboarding" vs. "IT Equipment Setup"
2. For each overlap, write inputs that sit in the ambiguous zone between the topics.
3. Decide the expected behavior for each: should the agent pick the right topic based on subtle cues, or should it ask the user to clarify?
4. Create test cases using **Capability Use (All)** for cases where the agent should resolve the ambiguity, and **Compare Meaning** for cases where the agent should present a disambiguation prompt.
5. Run the eval set. Failures indicate the agent is picking the wrong topic or failing to ask for clarification when it should.

### Anti-Pattern

> **Anti-Pattern: Treating All Ambiguous Inputs as Failures**
> When the agent asks "Did you mean X or Y?" that is often the *correct* behavior for genuinely ambiguous inputs. Don't mark disambiguation prompts as failures. The failure is when the agent confidently routes to the wrong topic without asking — or when it asks for clarification on inputs that are actually clear.

### Evaluation Patterns

**Pattern A: Subtle Cue Resolution**
The input contains a subtle signal that distinguishes between overlapping topics. Verify the agent picks up on it. Example: "I can't log in" (Password Reset topic) vs. "I've been locked out after too many attempts" (Account Lockout topic). The phrase "locked out after too many attempts" is the distinguishing cue.

**Pattern B: Appropriate Disambiguation Prompt**
The input is genuinely ambiguous with no distinguishing cues. Verify the agent asks a clarifying question that presents the relevant options clearly. Example: "I have a problem with my account" could be Password Reset, Account Lockout, Account Settings, or Billing. The agent should ask the user to specify.

**Pattern C: High-Confidence Override**
Even though two topics overlap, one is statistically much more likely for a given input. Verify the agent routes to the high-confidence match without unnecessary disambiguation. Example: "I need to return something" should route to the Returns topic — not trigger a disambiguation between Returns and Exchanges, since the user explicitly said "return."

**Pattern D: Keyword Overlap Stress Test**
Write inputs that contain keywords from multiple topics simultaneously. Example: "I want to reset my password for my billing account." This contains keywords for both Password Reset and Billing. Verify the agent identifies the primary intent (Password Reset) and does not get confused by the secondary keyword.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Subtle cue distinguishes Account Lockout from Password Reset | "I've been locked out after entering my password wrong too many times" | Capability: `Account Lockout` topic | Capability Use (All) + Compare Meaning |
| 2 | Subtle cue distinguishes Return from Exchange | "I want to send back the shoes I bought — they don't fit" | Capability: `Return an Item` topic | Capability Use (All) + Compare Meaning |
| 3 | Genuinely ambiguous input triggers disambiguation | "I have a problem with my account" | Response asks the user to clarify, presenting options like password issues, billing, or account settings | Compare Meaning + General Quality |
| 4 | Multi-keyword input resolves to primary intent | "I want to reset my password for my billing account" | Capability: `Password Reset` topic | Capability Use (All) + Compare Meaning |
| 5 | High-confidence match avoids unnecessary disambiguation | "I need to return the headphones I ordered" | Capability: `Return an Item` topic (no disambiguation prompt) | Capability Use (All) + Keyword Match (Any) |
| 6 | Disambiguation prompt is well-structured | "I need help with my order" | Response is clear, concise, and offers 2–3 specific options rather than an open-ended question | General Quality + Compare Meaning |

### Tips

- Focus disambiguation tests on your **most overlapping topic pairs**. Audit your topics for keyword overlap before writing test cases.
- For each overlapping pair, write at least **2 test cases**: one where context resolves the ambiguity, one where disambiguation is appropriate.
- Target: **90% or higher of disambiguation test cases should route correctly or present an appropriate clarification prompt.** Incorrect routing without disambiguation is the worst outcome.
- If disambiguation prompts are firing too frequently on clear inputs, your topic trigger phrases may be too broad — tighten them.
- If the agent consistently picks the wrong topic in an overlapping pair, review the trigger phrase confidence scores and consider adding negative examples.

---

## 3. Validating Fallback/Default Routing

When no topic matches, does the agent route to the fallback topic?

### When to Use

Your agent has a configured fallback or default topic (sometimes called the "Fallback" or "Conversational Boosting" topic) that should handle inputs matching no other topic. This scenario verifies the fallback activates appropriately and provides a useful response rather than silence, an error, or a misrouted answer from a low-confidence topic match.

> **Related scenarios:** If your concern is how the agent handles failure beyond fallback routing (e.g., escalation to a human), see [Graceful Failure & Escalation](graceful-failure-and-escalation.md). If the input should match a real topic but doesn't, that's a Scenario 1 failure — not a fallback test.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the Fallback topic was triggered (not a low-confidence match to a real topic) |
| Compare Meaning | Verifies the fallback response is helpful and acknowledges the agent's limitations |
| Keyword Match (Any) | Checks for expected recovery language: suggestions, escalation offers, or scope clarification |
| General Quality | Assesses whether the fallback response feels polished and constructive, not like a dead end |

> **Tip:** The fallback topic is your agent's safety net. A bad fallback makes the agent feel broken; a good fallback makes the agent feel honest and helpful even when it can't answer.

### Setup Steps

1. Write 5–10 out-of-scope inputs that should NOT match any of your configured topics. Include:
   - Completely off-topic requests (e.g., "What's the weather today?" for an HR agent)
   - Gibberish or nonsensical input
   - Inputs in a different language (if your agent is single-language)
   - Vague inputs with no clear intent ("hello" or "I have a question")
2. Create test cases with **Capability Use (All)** set to the Fallback topic name.
3. Add **Compare Meaning** or **Keyword Match (Any)** checks for helpful fallback language (e.g., "I can help with...", "you can reach our support team at...").
4. Add a **General Quality** check to confirm the fallback response is not a dead-end.
5. Run the eval set. Failures indicate the agent is either force-matching out-of-scope inputs to a real topic or providing an unhelpful fallback response.

### Anti-Pattern

> **Anti-Pattern: Fallback Becomes a Catch-All That Answers Everything**
> If your fallback topic uses generative AI to answer any question (even out-of-scope ones), the agent will never say "I don't know." This means out-of-scope inputs get ungrounded, potentially incorrect answers. The fallback should acknowledge limitations and offer constructive next steps — not attempt to answer questions the agent was never designed to handle.

### Evaluation Patterns

**Pattern A: Out-of-Scope Topic Rejection**
The user asks about something entirely outside the agent's domain. Verify the fallback topic fires AND the response clarifies what the agent can help with. Example: "What's the capital of France?" to an IT support agent should trigger the fallback, not a best-guess from knowledge sources.

**Pattern B: Gibberish and Noise Handling**
The user sends nonsensical input, random characters, or accidental messages. Verify the fallback activates gracefully. Example: "asdfghjkl" or "???" should not trigger a real topic.

**Pattern C: Vague Intent Without Enough Signal**
The user provides input that has intent but is too vague to match any topic. Example: "Help" or "I need something." Verify the fallback activates and offers guidance on what the user can ask about.

**Pattern D: Adjacent-Domain Queries**
The user asks about something adjacent to but outside the agent's scope. Example: An HR agent asked "What's the process for filing a workers' compensation claim?" when workers' comp is handled by a separate legal team. The agent should route to fallback and clarify scope — not attempt to answer from HR knowledge sources.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Completely off-topic input hits fallback | "Can you recommend a good restaurant nearby?" | Capability: `Fallback` topic | Capability Use (All) + Compare Meaning |
| 2 | Gibberish input hits fallback | "asdfghjkl 12345" | Capability: `Fallback` topic | Capability Use (All) + Compare Meaning |
| 3 | Vague input hits fallback with guidance | "Help" | Capability: `Fallback` topic | Capability Use (All) + Compare Meaning |
| 4 | Fallback response clarifies scope | "Can you recommend a good restaurant nearby?" | Response mentions what the agent CAN help with (e.g., "I can help with IT support, password resets, and equipment requests") | Compare Meaning + Keyword Match (Any) |
| 5 | Fallback offers escalation or next step | "I have a legal question about my contract" | Keywords (any): "support team", "contact", "help with", "I can assist you with" | Keyword Match (Any) + General Quality |
| 6 | Fallback response is polished and constructive | "What's the meaning of life?" | Response is polite, acknowledges the limitation, and offers a constructive redirect — not a dead-end "I don't understand" | General Quality + Compare Meaning |

### Tips

- Write at least **5 out-of-scope test cases** covering different types of non-matching input (off-topic, gibberish, vague, adjacent-domain).
- Verify the fallback response **lists what the agent CAN help with** — this turns a negative experience into a discovery moment.
- Target: **100% of out-of-scope test cases should trigger the Fallback topic.** If any trigger a real topic, that topic's triggers are too broad.
- Watch for the **"generative fallback" trap**: if your fallback uses generative AI, test that it doesn't answer out-of-scope questions authoritatively. It should redirect, not fabricate.
- Rerun after adding new topics — new topics can accidentally capture inputs that should fall through to the fallback.

---

## 4. Testing Topic Handoff

When mid-conversation the user switches intent, does the agent hand off to the correct topic?

### When to Use

Your agent supports multi-turn conversations and users may change their mind or bring up a new topic mid-conversation. This scenario verifies the agent detects the intent switch and transitions to the correct new topic without losing context or getting stuck in the previous topic's flow.

> **Related scenarios:** If your concern is initial routing on the first message (not mid-conversation switching), see [Scenario 1: Verifying Direct Topic Match](#1-verifying-direct-topic-match). If the user switches to something out of scope, see [Scenario 3: Validating Fallback/Default Routing](#3-validating-fallbackdefault-routing).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the new topic was triggered after the intent switch |
| Compare Meaning | Verifies the response addresses the new intent, not the previous topic |
| General Quality | Assesses whether the transition is smooth and the agent doesn't carry over irrelevant context from the previous topic |

> **Tip:** Topic handoff tests require multi-turn conversations. Design test cases where Turn 1 establishes a topic and Turn 2 switches to a different topic. The evaluation focuses on the Turn 2 response.

### Setup Steps

1. Identify common mid-conversation switches your users make. Examples:
   - User starts asking about PTO policy, then says "Actually, can you help me reset my password?"
   - User is in a password reset flow, then asks "Wait, what are the office hours?"
   - User asks about order status, then says "I also need to return another item"
2. Create multi-turn test cases. Turn 1 establishes Topic A. Turn 2 introduces Topic B.
3. Set the **Expected Capability** on Turn 2 to Topic B (using Capability Use (All)).
4. Add **Compare Meaning** to verify the Turn 2 response addresses Topic B, not Topic A.
5. Run the eval set. Failures indicate the agent is stuck in Topic A, ignoring the switch, or producing a confused response.

### Anti-Pattern

> **Anti-Pattern: Assuming Users Stay in One Topic**
> If you only test single-topic conversations, you miss the common real-world pattern of users switching topics mid-conversation. Users don't think in terms of "topics" — they think in terms of "things I need help with." A user who came for a password reset may suddenly ask about their benefits because they're already talking to the agent. Test topic switches.

### Evaluation Patterns

**Pattern A: Clean Break — User Explicitly Switches**
The user clearly signals they want to talk about something else. Example: "Actually, I have a different question — how do I update my address?" Verify the agent transitions to the Address Update topic without residual context from the previous topic.

**Pattern B: Implicit Switch — User Drifts to a New Topic**
The user doesn't explicitly signal a switch but their message is clearly about a different topic. Example: During a password reset flow, the user says "Oh, by the way, when is open enrollment?" Verify the agent recognizes this as a new topic (Benefits Enrollment) and handles it.

**Pattern C: Return to Previous Topic**
After switching to a new topic, the user wants to return to the original topic. Example: "OK, back to my password reset — where were we?" Verify the agent can return to the original topic and ideally resume where they left off.

**Pattern D: Rapid Multi-Topic Switching**
The user switches topics multiple times in a short conversation. Verify the agent can handle 2–3 switches without confusion or context bleed. Example: Password Reset -> Benefits Question -> Password Reset -> PTO Request.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Explicit switch from PTO to Password Reset | Turn 1: "What is the PTO policy?" / Turn 2: "Actually, can you help me reset my password?" | Capability: `Password Reset` topic | Capability Use (All) + Compare Meaning |
| 2 | Implicit switch during password reset flow | Turn 1: "I need to reset my password" / Turn 2: "When is open enrollment for benefits?" | Capability: `Benefits Enrollment` topic | Capability Use (All) + Compare Meaning |
| 3 | Response addresses the new topic, not the old one | Turn 1: "Check my order status for #88234" / Turn 2: "I also need to return the shoes I bought last week" | Response discusses return process, not order status | Compare Meaning + Keyword Match (Any) |
| 4 | No context bleed from previous topic | Turn 1: "I need to reset my password — my username is jsmith" / Turn 2: "How much PTO do I have left?" | Response does not reference passwords, usernames, or the previous topic — cleanly addresses PTO balance | General Quality + Compare Meaning |
| 5 | Return to previous topic after switch | Turn 1: "Reset my password" / Turn 2: "When is open enrollment?" / Turn 3: "OK, back to my password — what do I do next?" | Capability: `Password Reset` topic | Capability Use (All) + Compare Meaning |
| 6 | Smooth transition experience | Turn 1: "I have a billing question" / Turn 2: "Wait, actually, first I need to update my payment method" | Response transitions smoothly to payment update without confusion or re-asking for context already provided | General Quality + Compare Meaning |

### Tips

- Write at least **3 topic-handoff test cases** covering explicit switches, implicit switches, and returns to previous topics.
- The most common handoff failure is the agent **staying stuck in the previous topic's flow** — it continues asking password reset questions even after the user switched to a benefits question.
- Target: **90% or higher of topic-handoff test cases should trigger the correct new topic.** Mid-conversation routing is harder than first-message routing, so expect slightly lower accuracy.
- If your agent uses slot-filling (collecting parameters), test what happens when the user switches topic mid-collection — the agent should not force the user to finish the original flow.
- Consider testing handoff **both directions** for overlapping topic pairs (A -> B and B -> A) since routing behavior may differ.

---

## 5. Negative Test: Incorrect Topic Triggered

Test cases where a wrong topic commonly fires due to keyword overlap, overly broad triggers, or semantic similarity.

### When to Use

You have observed or suspect that certain inputs consistently route to the wrong topic. This scenario proactively tests known misrouting risks — inputs where keyword overlap between topics causes the agent to pick the wrong one. Unlike Scenario 2 (disambiguation), this scenario focuses on cases where the agent confidently routes to the *wrong* topic without any hesitation.

> **Related scenarios:** If the problem is ambiguity between similar topics, see [Scenario 2: Testing Disambiguation Between Similar Topics](#2-testing-disambiguation-between-similar-topics). If no topic fires at all, see [Scenario 3: Validating Fallback/Default Routing](#3-validating-fallbackdefault-routing). This scenario specifically targets confident misrouting.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms the CORRECT topic fires (positive test for the right topic, which implicitly verifies the wrong topic did NOT fire) |
| Compare Meaning | Verifies the response content matches the intended topic's domain, not the commonly confused topic |
| Keyword Match (All) | Checks for topic-specific terms from the correct topic and absence of terms from the wrong topic |

> **Tip:** Negative routing tests are regression safeguards. Once you fix a misrouting issue (by adjusting trigger phrases), add the test case to your regression suite so it doesn't recur.

### Setup Steps

1. Audit your topics for keyword overlap. Identify pairs where shared keywords could cause misrouting:
   - "Password Reset" and "Account Lockout" share keywords: password, access, locked, login
   - "Return an Item" and "Cancel an Order" share keywords: order, want to, don't want
   - "Payroll" and "Benefits" share keywords: deductions, take-home, employer contribution
2. For each overlap, write inputs that should route to Topic A but are at risk of routing to Topic B.
3. Create test cases with **Capability Use (All)** set to the CORRECT topic (Topic A).
4. Add **Compare Meaning** to verify the response content matches Topic A's domain.
5. Run the eval set. Failures confirm the misrouting is occurring and quantify how often.

### Anti-Pattern

> **Anti-Pattern: Fixing Misrouting by Narrowing Triggers Too Aggressively**
> When you discover a misrouting issue, it's tempting to remove the overlapping keywords from the offending topic's triggers. But if you narrow too much, that topic stops matching legitimate inputs. Instead of removing keywords, add disambiguating context to trigger phrases and use negative examples to teach the topic what NOT to match. Then rerun the test to confirm the fix works without breaking the original topic.

### Evaluation Patterns

**Pattern A: Keyword Overlap Exploitation**
Write inputs that contain keywords from the wrong topic but whose intent clearly belongs to the right topic. Example: "I want to cancel the return I requested" should route to Returns (to cancel a return), not to Cancel Order — even though it contains "cancel."

**Pattern B: Sentence Structure Misdirection**
Test inputs where the sentence structure buries the true intent. Example: "I was trying to reset my password but now I'm locked out" — the primary intent is Account Lockout, but "reset my password" is mentioned first and may trigger the Password Reset topic.

**Pattern C: Domain-Adjacent Phrasing**
Test inputs that use vocabulary from one domain while asking about another. Example: "What's the deduction on my paycheck for dental insurance?" belongs to the Benefits topic (asking about insurance), but "deduction" and "paycheck" are Payroll keywords.

**Pattern D: Known Misrouting Regression**
After fixing a misrouting issue, add the exact input that was misrouting as a permanent test case. This is a regression test — if the misrouting recurs after a future change, this test case will catch it.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | "Cancel" keyword does not misroute to Cancel Order | "I want to cancel the return I submitted last week" | Capability: `Return an Item` topic — NOT `Cancel Order` topic | Capability Use (All) + Compare Meaning |
| 2 | "Password" keyword does not misroute to Password Reset | "I've been locked out after too many password attempts" | Capability: `Account Lockout` topic — NOT `Password Reset` topic | Capability Use (All) + Compare Meaning |
| 3 | "Paycheck" does not misroute to Payroll | "What's the deduction on my paycheck for dental insurance?" | Capability: `Benefits` topic — NOT `Payroll` topic | Capability Use (All) + Compare Meaning |
| 4 | "Onboarding" does not misroute to Onboarding topic | "I'm past onboarding but still need my equipment set up" | Capability: `IT Equipment Setup` topic — NOT `New Hire Onboarding` topic | Capability Use (All) + Compare Meaning |
| 5 | Response content matches correct topic domain | "I was trying to reset my password but now I'm locked out" | Response discusses account lockout recovery (unlock steps, waiting period, admin contact) — not password reset instructions | Compare Meaning + Keyword Match (Any) |
| 6 | Correct topic keywords present, wrong topic keywords absent | "What's the deduction on my paycheck for dental insurance?" | Keywords (all): "dental", "insurance", "coverage" or "plan" | Keyword Match (All) + Compare Meaning |

### Tips

- Audit for misrouting risks **before you launch** — don't wait for user complaints. The keyword overlap analysis in Setup Step 1 is a proactive way to find problems.
- For each identified overlap, write at least **2 misrouting test cases** (one testing each direction: A misrouted to B, and B misrouted to A).
- Target: **100% of negative routing test cases should trigger the correct topic.** Confident misrouting is worse than triggering the fallback — the user gets the wrong answer without any signal that something went wrong.
- Add every fixed misrouting issue to your **regression test suite** so it can't silently recur.
- Common misrouting root causes: (1) topic trigger phrases are too broad, (2) topics share too many keywords without disambiguating phrases, (3) a catch-all topic is absorbing inputs meant for a specific topic.
- Rerun after **any change to trigger phrases, topic names, or topic descriptions** — these changes frequently introduce new misrouting.