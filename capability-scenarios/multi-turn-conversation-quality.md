# Multi-Turn Conversation Quality

> Scenarios for validating that your agent maintains context, coherence, and quality across multiple conversation turns. Single-turn testing misses a class of failures that only emerge when the agent must track state, resolve references, and adapt to evolving user intent across a conversation.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## 1. Context Retention Across Turns

### When to Use

Your agent conducts multi-turn conversations and must remember information the user provided in earlier turns — names, preferences, constraints, or details about their situation. This scenario tests whether the agent retains and correctly applies earlier context without asking the user to repeat themselves.

Use this scenario when:
- Your agent collects information incrementally (e.g., gathering details to file a request or troubleshoot an issue)
- Users provide constraints or preferences early in the conversation that should influence later responses
- The agent references earlier user inputs when formulating later answers
- You have observed users complaining that the agent "forgot" what they said

> **Related scenarios:** For testing whether the agent retrieves from the correct knowledge source, see [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md). For testing whether the agent hands off context correctly during escalation, see [Graceful Failure & Escalation](graceful-failure-and-escalation.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's later responses reflect details provided in earlier turns |
| Keyword Match (All) | Confirm specific details from earlier turns (names, dates, IDs) appear correctly in later responses |
| General Quality | Assess whether the overall conversation feels coherent and the agent does not ask for information already provided |

> **Tip:** The most common context retention failure is not total amnesia — it is **partial retention** where the agent remembers some details but drops others. Test with conversations that provide 3–4 distinct details across early turns, then verify all of them are reflected in the final response.

### Setup Steps

1. Design multi-turn test conversations where the user provides key details across the first 2–3 turns (e.g., turn 1: name and department, turn 2: issue description, turn 3: preferred resolution).
2. In a later turn, ask a question or request an action that requires the agent to use those earlier details.
3. Set expected values that reference the specific details from earlier turns. The agent should not ask for them again and should use them correctly.
4. Use **Compare Meaning** to check the agent's final response against an expected response that incorporates all earlier details.
5. Use **Keyword Match (All)** for specific factual details (employee ID, dates, amounts) that must be reproduced exactly.
6. Run the evaluation. Any case where the agent asks the user to repeat information, uses the wrong detail, or ignores a previously stated constraint is a context retention failure.

### Anti-Pattern

> **Anti-Pattern: Testing Only Adjacent Turns**
> If your test conversation provides a detail in turn 3 and checks for it in turn 4, you are testing short-term memory — which most agents handle well. The real test is whether the agent retains details from turn 1 or 2 when it needs them in turn 5 or 6. Always include at least one test where the critical detail is separated from its use by two or more intervening turns.

### Evaluation Patterns

**Pattern: Detail Persistence Across Distance**
Provide a specific detail early (e.g., "I'm in the London office") and verify the agent still uses it correctly 3–5 turns later when context is relevant (e.g., giving UK-specific advice rather than US-specific). The farther apart the detail and its application, the harder the test.

**Pattern: Multi-Detail Aggregation**
Provide different details across multiple turns, then ask the agent to summarize or act on all of them together. For example, the user provides their role, department, and issue across three turns, and the agent should generate a support ticket that includes all three.

**Pattern: Implicit Detail Usage**
The agent should use earlier context without being explicitly reminded. If the user said "I'm a contractor" in turn 1, the agent should not present full-time-employee-only options in turn 4 — without the user needing to say "remember, I'm a contractor."

**Pattern: No Redundant Questions**
The agent should never ask for information the user has already provided in the same conversation. This is both a context retention test and a user experience test — repeated questions signal the agent has lost context.

### Practical Examples

| # | Scenario | Sample Conversation | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Detail persistence: location | Turn 1: "I'm based in the Singapore office." Turn 2: "What's the process for booking a conference room?" Turn 3: "What are the available rooms for tomorrow?" | Agent provides Singapore-specific room availability — not headquarters rooms | Compare Meaning + Keyword Match (Any) |
| 2 | Multi-detail aggregation | Turn 1: "My employee ID is E-4521." Turn 2: "I need to request equipment — a standing desk." Turn 3: "Please submit this request." | Agent generates a request referencing employee E-4521 for a standing desk — both details from earlier turns | Keyword Match (All) + Compare Meaning |
| 3 | No redundant questions | Turn 1: "I'm in the Marketing department." Turn 2: "What training courses are available?" Turn 3: [Agent asks department-specific follow-up] | Agent does NOT ask "What department are you in?" again — it uses the earlier answer | General Quality + Compare Meaning |
| 4 | Implicit detail usage | Turn 1: "I'm a part-time employee." Turn 2: "What benefits am I eligible for?" Turn 3: "Tell me more about the health insurance options." | Agent presents part-time-eligible health insurance options — does not list full-time-only plans | Compare Meaning + General Quality |
| 5 | Constraint retention | Turn 1: "My budget is under $500." Turn 2: "What laptop options are available?" Turn 3: "Can you show me something with more storage?" | Agent recommends options within the $500 budget — does not present options exceeding the stated constraint | Compare Meaning + Keyword Match (Any) |
| 6 | Long-distance retention | Turn 1: "I prefer email communication over phone calls." Turns 2–4: [Agent helps with an issue]. Turn 5: "How will I receive the confirmation?" | Agent states confirmation will be sent via email — not phone — reflecting the turn 1 preference | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 4 multi-turn context retention test cases, with at least one testing retention across 4+ turns.
- **Context retention threshold:** Target at least 95% — occasional failures on very distant details are understandable, but failures on details from 2–3 turns ago are critical.
- **Vary the detail types:** Test retention of names, numbers, preferences, constraints, and stated conditions — different detail types may have different retention rates.
- **Rerun after:** Changes to system prompt, conversation flow design, context window settings, or memory/summarization features.
- These tests require multi-turn test conversations. If your testing framework only supports single-turn input, you may need to simulate earlier turns as part of the conversation history in the test setup.

---

## 2. Coreference and Follow-Up Resolution

### When to Use

The user refers back to something mentioned in a previous turn using pronouns ("it," "that," "the one you mentioned"), ellipsis ("and what about pricing?"), or shorthand references. This scenario tests whether the agent correctly resolves these references to the right entity or topic.

Use this scenario when:
- Your agent has conversations where users naturally use pronouns and shorthand
- Users ask follow-up questions that only make sense in the context of the previous turn
- The agent handles topics with multiple entities that could be confused (e.g., multiple products, policies, or people)

> **Related scenarios:** For testing whether the agent retains context broadly, see [Context Retention Across Turns](#1-context-retention-across-turns). For testing topic routing, see [Trigger Routing](trigger-routing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent resolves the reference to the correct entity and provides relevant information |
| Keyword Match (All) | Confirm the response includes details specific to the correct referent, not a different entity |
| General Quality | Assess whether the agent handles ambiguous references gracefully — asking for clarification when genuinely ambiguous |

> **Tip:** The most revealing tests involve conversations with **two or more candidate referents**. If the agent discussed both Plan A and Plan B, and the user says "tell me more about that one," the agent must determine which "that one" refers to — or ask for clarification.

### Setup Steps

1. Design conversations where the agent discusses one or more entities (products, policies, options) in early turns.
2. In a follow-up turn, use a pronoun or elliptical reference to one of them.
3. Set expected values based on the agent correctly identifying which entity the user means.
4. For ambiguous cases (where the reference could reasonably apply to multiple entities), the expected behavior is to ask for clarification — not to guess.
5. Run the evaluation. Any case where the agent answers about the wrong entity or ignores the reference entirely is a resolution failure.

### Anti-Pattern

> **Anti-Pattern: Testing Only Unambiguous References**
> If your test conversations only include cases where there is one obvious referent ("What does the health plan cost?" after discussing only one plan), you are not testing coreference resolution — you are testing basic comprehension. Include cases with multiple candidate referents to test whether the agent can disambiguate, and cases where it should ask for clarification.

### Evaluation Patterns

**Pattern: Pronoun Resolution with Single Referent**
The conversation establishes one main topic, and the user refers to it with a pronoun. The agent should resolve the pronoun to the correct entity and respond with relevant information.

**Pattern: Pronoun Resolution with Multiple Referents**
The conversation mentions two or more entities, and the user's follow-up could apply to either. The agent should either use contextual cues (the most recently discussed entity, the entity most relevant to the user's question) or ask for clarification.

**Pattern: Elliptical Follow-Up**
The user asks a question that omits the subject, relying on the previous context: "And what about the cost?" after discussing a specific product. The agent should provide cost information for the previously discussed product — not a generic response about costs.

**Pattern: Topic Shift Detection**
The user shifts topics but uses a reference that sounds like a follow-up: "What about returns?" after a conversation about product features. The agent should recognize this as a new question about the same product (return policy), not a continuation of the features discussion.

### Practical Examples

| # | Scenario | Sample Conversation | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Pronoun with single referent | Turn 1: "Tell me about the Premium health plan." Turn 2: "How much does it cost?" | Agent provides the cost of the Premium health plan — not a different plan | Compare Meaning + Keyword Match (Any) |
| 2 | Pronoun with multiple referents | Turn 1: "Compare the Basic and Premium plans." Turn 2: "I want to go with that one." | Agent asks for clarification: "Would you like the Basic plan or the Premium plan?" | General Quality + Compare Meaning |
| 3 | Elliptical follow-up | Turn 1: "What are the features of the Pro subscription?" Turn 2: "And the pricing?" | Agent provides pricing for the Pro subscription specifically | Compare Meaning + Keyword Match (Any) |
| 4 | Topic shift with same entity | Turn 1: "How do I set up the VPN client?" Turn 2: "Is there a mobile version?" | Agent discusses the mobile version of the VPN client — recognizes the entity carries over | Compare Meaning + Keyword Match (Any) |
| 5 | Nested reference | Turn 1: "I have a problem with my monitor." Turn 2: "The one I reported last week." Turn 3: "Has there been any update?" | Agent retrieves status of the previously reported monitor issue — not a new issue | Compare Meaning + Keyword Match (Any) |
| 6 | Correction follow-up | Turn 1: "I need help with the Standard plan." Turn 2: "Actually, I meant the Enterprise plan." Turn 3: "What are the limits?" | Agent provides limits for the Enterprise plan — honoring the correction | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 4 coreference resolution test cases, including at least one with multiple candidate referents.
- **Resolution accuracy threshold:** Target at least 90%. Ambiguous references where the agent asks for clarification should be scored as successes, not failures.
- **Vary reference types:** Test pronouns ("it," "that"), ellipsis ("and the cost?"), and deictic references ("the first one").
- **Rerun after:** Changes to conversation flow, topic structure, or system prompt instructions about handling follow-ups.

---

## 3. Conversation Coherence Under Topic Switching

### When to Use

The user changes topics mid-conversation — intentionally or tangentially. This scenario tests whether the agent can cleanly switch context to the new topic while still being able to return to the previous topic if needed.

Use this scenario when:
- Your agent covers multiple domains or topics and users may jump between them
- Users frequently digress, ask tangential questions, and then return to the original topic
- You need to verify the agent does not conflate details from different topics within the same conversation

> **Related scenarios:** For testing context retention within a single topic, see [Context Retention Across Turns](#1-context-retention-across-turns). For testing how the agent routes between topics, see [Trigger Routing](trigger-routing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent responds to the current topic and does not bleed details from the previous topic |
| General Quality | Assess the smoothness of topic transitions and whether the agent can resume the original topic coherently |
| Keyword Match (All) | Confirm topic-specific details are correct for the active topic |

> **Tip:** The hardest test for topic switching is the **return test** — the user switches away from Topic A to Topic B, discusses Topic B for a few turns, then says "going back to my earlier question." The agent must cleanly resume Topic A with the correct context, not carry over details from Topic B.

### Setup Steps

1. Design conversations where the user starts on one topic, switches to a different topic, and optionally returns to the first topic.
2. For each topic, include details that are specific and distinguishable — so you can detect if the agent confuses them.
3. After the topic switch, verify the agent responds to the new topic correctly and does not carry over irrelevant details from the previous topic.
4. If the user returns to the first topic, verify the agent resumes with the correct context.
5. Run the evaluation. Watch for two failure modes: context bleeding (details from Topic A appearing in Topic B answers) and context loss (the agent cannot resume Topic A after discussing Topic B).

### Anti-Pattern

> **Anti-Pattern: Testing Only Clean Breaks**
> If the user explicitly says "Let me ask about something else entirely," the topic switch is easy for the agent. Real conversations involve gradual drifts, tangential questions, and ambiguous shifts. Test with transitions like "Oh, that reminds me — what about..." or "While I have you, can you also check..." where the boundary between topics is less clear.

### Evaluation Patterns

**Pattern: Clean Topic Switch**
The user explicitly changes topics. The agent should respond to the new topic without referencing the old one unless the user makes a connection. Test that no irrelevant details from the old topic appear in the new response.

**Pattern: Context Bleeding Detection**
After a topic switch, the agent should not confuse details from the two topics. If the user was discussing a $500 budget for equipment and then switches to asking about travel reimbursement, the $500 should not appear in the travel reimbursement answer (unless it is coincidentally the correct value).

**Pattern: Resume After Digression**
The user returns to the original topic after a digression. The agent should pick up where it left off — remembering what was discussed, what was resolved, and what was still pending. Test that the agent does not start the original topic from scratch.

**Pattern: Parallel Topic Tracking**
In some conversations, the user is effectively managing two issues at once, weaving between them. The agent should keep the details of each issue separate and respond to whichever one the user is currently addressing.

### Practical Examples

| # | Scenario | Sample Conversation | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Clean topic switch | Turns 1–2: Discussing PTO policy. Turn 3: "Can you also help me find the Wi-Fi password?" | Agent responds about Wi-Fi — no PTO details bleed into the response | Compare Meaning + Keyword Match (Any) |
| 2 | Context bleeding check | Turns 1–2: Equipment budget of $500. Turn 3: "What's the limit for conference travel reimbursement?" | Agent provides the travel reimbursement limit from its sources — does NOT reference the $500 equipment budget | Compare Meaning + Keyword Match (All) |
| 3 | Resume after digression | Turns 1–2: Setting up a software request. Turn 3: "Quick question — what are the office hours?" Turn 4: "OK back to my software request." | Agent resumes the software request where it left off — remembers what was already discussed | General Quality + Compare Meaning |
| 4 | Gradual drift | Turns 1–2: Discussing health insurance. Turn 3: "Does the dental plan cover orthodontics?" Turn 4: "What about vision — are contacts covered?" | Agent addresses each sub-topic correctly, treating dental and vision as related but distinct questions with different answers | Compare Meaning + Keyword Match (Any) |
| 5 | Parallel topic tracking | Turn 1: "I have two issues — my monitor is broken and I need VPN access." Turn 2: "Let's start with the monitor." Turn 3: "OK, now what about the VPN?" | Agent keeps monitor and VPN issues separate; does not mix up details or resolution steps | General Quality + Compare Meaning |
| 6 | Ambiguous return reference | Turns 1–2: Discussing expenses. Turns 3–4: Discussing PTO. Turn 5: "So how do I submit that?" | Agent asks for clarification (submit what — an expense report or a PTO request?) OR correctly infers from context | General Quality + Compare Meaning |

### Tips

- **Coverage target:** At least 3 topic-switching test cases, including one with a return to the original topic.
- **Context bleeding rate:** Target 0% — the agent should never confuse details between topics.
- **Topic switching is especially important for generalist agents** that cover multiple domains. Specialist agents with narrow scope are less likely to encounter topic switches.
- **Rerun after:** Changes to topic routing, conversation flow structure, or memory/summarization settings.

---

## 4. Long Conversation Degradation

### When to Use

Your agent's conversations may extend beyond 8–10 turns in real usage. This scenario tests whether the agent maintains response quality, accuracy, and coherence as the conversation grows longer — or whether performance degrades as the context window fills up.

Use this scenario when:
- Your agent handles complex issues that require many exchanges to resolve
- Users have reported that the agent "gets confused" later in long conversations
- You want to validate agent behavior at the edges of its context window
- Your agent uses summarization or truncation strategies for long conversations

> **Related scenarios:** For testing context retention specifically, see [Context Retention Across Turns](#1-context-retention-across-turns). For testing regression after updates, see [Regression Testing](regression-testing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Assess overall response quality at different conversation lengths — compare turn 3 quality to turn 10 quality |
| Compare Meaning | Verify accuracy of responses in later turns against what the sources actually say |
| Keyword Match (All) | Check that specific factual details remain correct in later turns |

> **Tip:** The best way to test for degradation is to ask **the same type of question** at different points in the conversation and compare the quality. If the agent answers a knowledge grounding question well in turn 2 but poorly in turn 12, that is degradation — not a different scenario.

### Setup Steps

1. Design test conversations that are intentionally long — 10+ turns of realistic interaction.
2. Include a mix of information-gathering turns, topic exploration, and follow-up questions to simulate real usage patterns.
3. At intervals (e.g., turns 3, 7, and 12), ask questions of comparable difficulty and type.
4. Set expected values based on correct answers at each interval — the expected quality should not decrease.
5. Use **General Quality** to score the overall coherence and helpfulness at each interval.
6. Run the evaluation. Compare quality scores across intervals. A consistent decline indicates degradation.

### Anti-Pattern

> **Anti-Pattern: Testing Only Short Conversations**
> If your test conversations are 3–4 turns, you will never observe degradation. Most agents perform well in short conversations. The failures emerge at 8+ turns when the context window contains more information, when earlier details may have been summarized or truncated, and when the model must manage more competing context. Your tests must reach the conversation lengths your real users experience.

### Evaluation Patterns

**Pattern: Quality Parity Across Length**
Compare response quality metrics at turn 3 vs. turn 10 for equivalent questions. Both should meet the same quality bar. If turn 10 responses are noticeably shorter, vaguer, or less accurate, that is degradation.

**Pattern: Early Detail Accuracy in Late Turns**
Provide a detail in the first 1–2 turns and test whether the agent still uses it correctly in turns 8–12. This is a stress test for context retention under the pressure of a filled context window.

**Pattern: Instruction Following Persistence**
If the agent is given behavioral instructions early in the conversation (e.g., "always provide sources" or "respond in bullet points"), verify it still follows those instructions in later turns. Instruction drift is a common degradation pattern.

**Pattern: No Hallucination Escalation**
Test whether the hallucination rate increases in later turns. As context grows, some models begin to confuse earlier conversation content with their knowledge base, leading to increased fabrication. Compare hallucination rates between early and late turns.

### Practical Examples

| # | Scenario | Sample Test Design | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Quality parity | Ask "What is the return policy?" at turn 2 and a similar question "What about the exchange policy?" at turn 10 | Both responses are equally detailed, accurate, and well-structured | General Quality (compare scores) |
| 2 | Early detail in late turn | Turn 1: User states employee ID. Turns 2–8: Various questions. Turn 9: "Can you confirm my employee ID for the request?" | Agent correctly states the employee ID from turn 1 | Keyword Match (All) + Compare Meaning |
| 3 | Instruction persistence | Turn 1: "Please respond in bullet points." Turns 2–5: Normal questions (agent uses bullets). Turns 6–10: More questions. | Agent still uses bullet point format in turn 10 | General Quality |
| 4 | No hallucination escalation | Turns 1–5: Factual questions (agent answers correctly from sources). Turns 6–10: Similar factual questions. | Hallucination rate does not increase — turn 10 answers are as grounded as turn 2 answers | Compare Meaning + Keyword Match (All) |
| 5 | Context window pressure | Design a 15-turn conversation with substantial content each turn. Turn 15: Ask a question requiring information from turn 2. | Agent either answers correctly or honestly states it cannot recall the detail — does NOT fabricate | Compare Meaning + General Quality |
| 6 | Repetition detection | In turns 8–10, the agent should not repeat information it already provided in turns 3–4 unless asked | Agent provides new, relevant information — does not loop back to earlier responses unprompted | General Quality |

### Tips

- **Coverage target:** At least 2 long-conversation test cases (10+ turns each), with quality checks at early, middle, and late points.
- **Degradation threshold:** Response quality at turn 10 should be within 10% of quality at turn 3. A drop greater than 20% is a critical finding.
- **Track degradation over model updates** — different model versions may have different degradation profiles for long conversations.
- **Rerun after:** Changes to context window settings, summarization strategies, system prompt length, or model version.
- If your agent uses conversation summarization or truncation, specifically test the boundary where summarization kicks in — this is often where degradation begins.
- **Consider conversation reset strategies** — if degradation is consistent, it may be better to design conversation flows that naturally close and restart rather than extending indefinitely.

---

## 5. Conversation Recovery After Misunderstanding

### When to Use

The agent misunderstands the user's intent, gives an incorrect or off-topic response, and the user corrects the agent or rephrases. This scenario tests whether the agent can recover gracefully — acknowledging the misunderstanding, adjusting its understanding, and proceeding correctly.

Use this scenario when:
- Your agent handles ambiguous inputs where misunderstanding is likely
- Users sometimes rephrase or correct the agent mid-conversation
- You want to verify the agent does not "double down" on a misunderstanding when corrected
- Recovery from errors is important for user trust

> **Related scenarios:** For testing coreference resolution, see [Coreference and Follow-Up Resolution](#2-coreference-and-follow-up-resolution). For testing tone during difficult interactions, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's post-correction response aligns with the user's corrected intent |
| General Quality | Assess whether the recovery is graceful — the agent acknowledges the error, does not become defensive, and moves forward |
| Keyword Match (All) | Confirm the corrected details appear in subsequent responses |

> **Tip:** Good recovery has three parts: (1) acknowledge the misunderstanding without excessive apology, (2) demonstrate understanding of the corrected intent, and (3) proceed with the correct information. Over-apologizing ("I'm so sorry, I completely misunderstood you, I apologize for the confusion...") is almost as bad as not acknowledging the error at all.

### Setup Steps

1. Design conversations where the agent's first response is plausibly wrong — either because the input was ambiguous or because the agent misinterprets.
2. Have the user correct the agent explicitly: "No, I meant..." or "That's not what I asked."
3. Set expected values for the post-correction response that reflect the corrected intent.
4. Use **General Quality** to evaluate the recovery itself — was it graceful and efficient?
5. In subsequent turns after recovery, verify the agent uses the corrected understanding — not the original misunderstanding.
6. Run the evaluation. Key failure modes: agent doubles down, agent over-apologizes without actually correcting, agent corrects in the immediate response but reverts to the misunderstanding later.

### Anti-Pattern

> **Anti-Pattern: Testing Only Explicit, Detailed Corrections**
> If the user says "No, I don't mean the Basic plan, I mean the Premium plan — the one that costs $99/month with the advanced features," the correction is so detailed that even a weak agent can recover. Real corrections are often terse: "No, the other one" or "Premium, not Basic." Test with minimal corrections that require the agent to do inference work.

### Evaluation Patterns

**Pattern: Explicit Correction Recovery**
The user clearly states what the agent got wrong and what the correct interpretation is. The agent should immediately switch to the correct interpretation and proceed.

**Pattern: Implicit Correction Recovery**
The user rephrases their question without explicitly calling out the error. The agent should recognize the rephrasing as a correction and adjust — not treat it as a new, unrelated question.

**Pattern: No Reversion After Correction**
After the agent recovers, verify it maintains the corrected understanding in subsequent turns. A common failure is the agent correcting itself in turn N but reverting to the original misunderstanding in turn N+2.

**Pattern: Proportional Acknowledgment**
The agent should acknowledge the misunderstanding proportionally — a brief "Got it" or "Thanks for clarifying" is appropriate for a minor miscommunication. Lengthy apologies for small corrections waste the user's time and erode confidence.

### Practical Examples

| # | Scenario | Sample Conversation | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Explicit correction | Turn 1: "Help me with my order." Turn 2: Agent asks about a product order. Turn 3: "No, I mean a food order for the office event." | Agent smoothly pivots to the food ordering context and proceeds appropriately | Compare Meaning + General Quality |
| 2 | Implicit correction (rephrase) | Turn 1: "What's the deadline?" Turn 2: Agent provides the project deadline. Turn 3: "I meant the enrollment deadline for benefits." | Agent provides the benefits enrollment deadline without the user needing to explicitly say "you were wrong" | Compare Meaning + Keyword Match (Any) |
| 3 | Minimal correction | Turn 1: Agent discusses the Standard plan. Turn 2: "Premium, not Standard." | Agent switches to the Premium plan and provides relevant details — does not ask "Do you mean the Premium plan?" | Compare Meaning + Keyword Match (All) |
| 4 | No reversion | Turn 1: Misunderstanding. Turn 2: User corrects. Turn 3: Agent recovers correctly. Turn 4: Follow-up question. | Turn 4 response is based on the corrected understanding — does not revert to the original misunderstanding | Compare Meaning + Keyword Match (Any) |
| 5 | Proportional acknowledgment | Turn 1: Minor misunderstanding. Turn 2: User corrects. | Agent says something like "Got it, let me look into that" — not a five-sentence apology | General Quality |
| 6 | Correction during multi-step process | Turns 1–3: Agent guides user through a process. Turn 4: "Wait, I should have chosen option B, not A." | Agent adjusts the process to reflect the corrected choice and does not require starting over from scratch | General Quality + Compare Meaning |

### Tips

- **Coverage target:** At least 3 recovery test cases, including both explicit and implicit corrections.
- **Recovery success rate:** Target at least 90%. Some edge cases may be genuinely ambiguous and require additional clarification.
- **Test the full recovery arc** — not just the immediate response but also subsequent turns to verify no reversion.
- **Rerun after:** Changes to system prompt, conversation flow, or error handling instructions.
- Over-apologizing is a common anti-pattern in corrected agents. If your agent produces three sentences of apology for every correction, that is a conversation quality issue to address in your system prompt.

---

## 6. Consistent Persona and Behavior Across Turns

### When to Use

Your agent is configured with specific behavioral guidelines — a persona, communication style, response format preferences, or behavioral boundaries. This scenario tests whether the agent maintains those behaviors consistently throughout the entire conversation, not just in the first response.

Use this scenario when:
- Your agent has a defined persona (e.g., professional, friendly, formal) that should persist across the conversation
- Response format requirements (e.g., bullet points, structured answers) should apply to every turn
- Behavioral boundaries (e.g., "never provide medical advice") must hold throughout the conversation
- You have observed inconsistency in persona or behavior as conversations progress

> **Related scenarios:** For tone and helpfulness specifically, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md). For safety boundaries, see [Safety & Boundary Enforcement](safety-and-boundary-enforcement.md). For quality degradation in long conversations, see [Long Conversation Degradation](#4-long-conversation-degradation).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Assess persona/style consistency across multiple turns |
| Compare Meaning | Verify behavioral boundaries are maintained in later turns |
| Keyword Match (Any) | Check for consistent use of required language patterns or formatting |

> **Tip:** Persona drift is subtle and hard to catch in single-turn testing. An agent that is professional and formal in turn 1 may become increasingly casual by turn 8 — especially if the user's tone is casual. Design tests where the user's tone is different from the expected agent persona to see if the agent holds its ground.

### Setup Steps

1. Define the specific persona traits and behavioral requirements your agent should maintain (tone, formality, format, boundaries).
2. Design multi-turn conversations that naturally test persona consistency — include turns where the user's behavior might pull the agent away from its persona (e.g., very casual language when the agent should be formal).
3. At multiple points in the conversation, evaluate the agent's adherence to its persona requirements.
4. Use **General Quality** with rubric criteria specific to persona consistency.
5. Run the evaluation. Compare persona adherence in early turns vs. late turns.

### Anti-Pattern

> **Anti-Pattern: Testing Persona Only in the First Turn**
> The first turn is the easiest — the system prompt is fresh, there is no competing context, and the model is at its most compliant. Real persona drift happens in turns 5–10 as the conversation takes on its own momentum. Always test persona in later turns, not just the opening.

### Evaluation Patterns

**Pattern: Tone Persistence Under User Influence**
The user uses a tone different from the agent's configured persona (e.g., very casual while the agent should be formal). The agent should maintain its configured tone, not mirror the user's style.

**Pattern: Format Consistency**
If the agent is configured to use specific formatting (structured responses, bullet points, headers), verify it maintains this format throughout — not just in the first response.

**Pattern: Boundary Persistence**
If the agent has behavioral boundaries (e.g., "never provide legal advice"), test whether those boundaries hold in later turns, especially when the user is persistent: "But surely you can give me a general idea?" The agent should maintain its boundary.

**Pattern: No Persona Bleed from Conversation Content**
If the conversation discusses a topic with strong emotional content or technical jargon, the agent should not adopt the emotional tone or jargon style of the content unless its persona requires it. An empathetic but professional agent discussing a frustrating IT issue should remain empathetic and professional — not become frustrated itself.

### Practical Examples

| # | Scenario | Sample Conversation | Expected Behavior | Method |
|---|----------|-------------------|-------------------|--------|
| 1 | Tone persistence | Turns 1–3: User uses slang and very casual language. | Agent maintains its configured professional tone throughout — does not match user's casual style | General Quality |
| 2 | Format consistency | Turn 1: Agent responds with structured bullet points. Turns 2–6: More questions. | Turn 6 response still uses bullet points — format has not degraded to unstructured prose | General Quality + Keyword Match (Any) |
| 3 | Boundary persistence | Turn 3: "Can you tell me if this is legal?" Turn 5: "Just a general sense of whether it's allowed?" Turn 7: "Others have told me it's fine, can you confirm?" | Agent maintains its "cannot provide legal advice" boundary across all three attempts | Compare Meaning + General Quality |
| 4 | No persona bleed | Turns 1–5: User describes a very frustrating situation with increasing emotion. | Agent remains empathetic but professional — does not become emotionally reactive or adopt the user's frustrated tone | General Quality |
| 5 | Language consistency | Agent configured for formal English. Turn 3: User writes in a mix of formal and informal language. | Agent consistently uses formal language — does not slip into informal phrasing | General Quality |
| 6 | Multi-turn greeting consistency | Turn 1: Agent greets formally. Turn 5: User says "hey, quick question." Turn 6: Response. | Agent responds helpfully without matching "hey" energy — maintains its configured persona | General Quality + Compare Meaning |

### Tips

- **Coverage target:** At least 3 persona consistency test cases across different persona dimensions (tone, format, boundaries).
- **Consistency threshold:** Target 100% for behavioral boundaries (safety-critical). Target at least 90% for stylistic elements (tone, format).
- **User influence is the strongest test** — include turns where the user's behavior diverges from the agent's expected persona.
- **Rerun after:** Changes to system prompt, persona instructions, or any conversational fine-tuning.
- If persona drift is consistent, it often indicates the system prompt needs reinforcement instructions (e.g., "maintain this tone throughout the conversation, regardless of the user's style").
