# Tone, Helpfulness & Response Quality

> Scenarios for evaluating the subjective quality of your agent's communication — clarity, empathy, tone, completeness, and structure. These are the qualities that determine whether users trust and continue using the agent, even when factual accuracy is perfect.

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## Scenario 1: Evaluating Response Clarity and Structure

### When to Use

Use this when your agent produces long or multi-part answers and you need to verify that responses are well-organized, easy to follow, and appropriately formatted. This is especially important for agents that explain policies, walk through processes, or return results that include multiple data points. A factually correct answer that is poorly structured will frustrate users and erode trust.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response is logically organized, uses appropriate formatting (lists, headings, paragraphs), and is easy to scan |
| Keyword Match (Any) | Verify that structural markers are present — numbered steps, bullet points, section breaks — when the content warrants them |
| Compare Meaning | Compare the agent's response structure against an ideal reference response that demonstrates the expected formatting and organization |

### Setup Steps

1. Identify 5–8 questions that typically produce multi-part answers from your agent (e.g., "What are the steps to submit an expense report?" or "Explain the differences between Plan A and Plan B").
2. For each question, write an ideal reference response that demonstrates the formatting and structure you expect — numbered lists for sequential steps, bullet lists for non-sequential items, bold for key terms, short paragraphs for explanations.
3. Create test cases using General Quality.
4. Add Compare Meaning test cases that compare the agent's response against your ideal reference, focusing on structural similarity rather than word-for-word match, with rubric instructions such as: "The response should be organized with clear sections, use numbered steps for sequential instructions, and avoid walls of unbroken text."
5. Optionally add Keyword Match (Any) checks for structural indicators like "Step 1", "First", "1.", or "**" (bold markers) when the content type demands them.

### Anti-Pattern

> **"Wall of Text" Acceptance** — Marking a response as passing because the information is factually correct, even though it is delivered as a single dense paragraph with no formatting. Users scan before they read; an unstructured response is a failed response regardless of accuracy.

### Evaluation Patterns

#### Pattern: Sequential Instruction Formatting
When the user asks "how do I..." or "what are the steps to...", the response should use numbered steps. Each step should be a single action. Verify that the agent does not embed a multi-step process inside a single paragraph.

#### Pattern: Comparison and Options Layout
When the user asks the agent to compare options or list alternatives, the response should use a parallel structure — either a table, a bullet list with consistent sub-points per option, or clearly labeled sections. Verify that the agent does not describe Option A in three sentences and Option B in one.

#### Pattern: Key Information Prominence
Critical details — deadlines, dollar amounts, eligibility requirements — should appear early in the response or be visually distinct (bold, separate line). Verify that the agent does not bury the most important information in the middle of a paragraph.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Multi-step process returns numbered steps | "How do I submit a travel reimbursement?" | Response uses numbered steps (1, 2, 3...) with one action per step | General Quality + Compare Meaning |
| 2 | Comparison question returns parallel structure | "What's the difference between the Standard and Premium plans?" | Response presents both plans with comparable detail using a consistent structure (table, parallel bullets, or labeled sections) | General Quality + Compare Meaning |
| 3 | Policy explanation highlights key terms | "What is the company's parental leave policy?" | Key details (duration, eligibility, deadlines) are prominent — bolded, listed, or placed at the beginning of the response | Compare Meaning + Keyword Match (Any) |
| 4 | FAQ answer is concise and scannable | "What are the office hours for the downtown location?" | Response is brief and directly answers the question without unnecessary preamble | General Quality + Compare Meaning |
| 5 | Complex answer includes section breaks | "Walk me through the annual enrollment process for benefits." | Response uses headings or clear section breaks for distinct phases of the process | Keyword Match (Any) + General Quality |
| 6 | Short answer does not over-format | "What is the Wi-Fi password for the guest network?" | Response provides the answer directly without unnecessary bullet points, numbered lists, or excessive formatting | General Quality + Compare Meaning |

### Tips

- Not every response needs heavy formatting. A one-sentence answer to a simple question should be a single sentence — not a bulleted list with one bullet. Test for appropriate formatting, not maximum formatting.
- Use General Quality rubrics that are specific: "Response should use numbered steps for sequential instructions" is better than "Response should be well-formatted."
- Create a small set of "gold standard" reference responses for your most common question types. Use these as Compare Meaning targets.
- Test with real user questions from your chat logs, not just questions you invented. Real users phrase things in ways that can trip up formatting logic.
- Aim for at least 8–10 structure-focused test cases covering different response types (steps, comparisons, simple lookups, explanations).

---

## Scenario 2: Testing Empathy in Sensitive Contexts

### When to Use

Use this when your agent handles topics where users may be experiencing stress, frustration, grief, or anxiety — complaint resolution, health information, bereavement policies, job loss, financial hardship, or safety concerns. The agent's tone in these moments matters as much as its accuracy. A correct but cold response to "my family member just passed away and I need to know about bereavement leave" will damage trust in the agent and the organization behind it.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response demonstrates appropriate empathy, acknowledgment, and warmth for the emotional context |
| Keyword Match (Any) | Check for the presence of empathetic language — "I'm sorry", "I understand", "That must be difficult" — when the context warrants it |
| Keyword Match (All) | Verify that both empathetic acknowledgment AND the factual answer are present (not just one or the other) |
| Compare Meaning | Compare the agent's response against an ideal empathetic response to verify overall tone alignment |

### Setup Steps

1. Identify the sensitive topics your agent handles — bereavement, complaints, termination, health issues, financial hardship, safety incidents, harassment, disability accommodations.
2. For each topic, write 2–3 test inputs that express emotional distress: one direct ("I need bereavement leave"), one emotional ("My mother just passed away and I don't know what to do about work"), and one frustrated ("This is the third time I've been overcharged and nobody is helping me").
3. Write ideal reference responses that demonstrate the right empathetic tone — acknowledging the situation before delivering information.
4. Create General Quality test cases with rubrics such as: "The response should acknowledge the user's emotional situation before providing procedural information. It should not jump straight to policy details without empathetic acknowledgment."
5. Add Keyword Match (Any) checks for empathetic phrases appropriate to each context.
6. Add Keyword Match (All) checks to ensure the response includes both empathy and the requested information — empathy without an answer is not helpful.

### Anti-Pattern

> **"Robotic Accuracy"** — The agent provides a perfectly accurate policy answer to a grief-related question without any acknowledgment of the user's emotional state. "Bereavement leave is 5 days for immediate family members. Submit form HR-204 to your manager." This is technically correct but communicatively harmful.

### Evaluation Patterns

#### Pattern: Acknowledge-Then-Inform
For sensitive topics, the response should follow an acknowledge-then-inform structure: first recognize the user's situation, then provide the information they need. Verify that the agent does not skip the acknowledgment step.

#### Pattern: Tone Calibration by Severity
Not all sensitive topics require the same level of empathy. A billing complaint needs professional acknowledgment. A bereavement question needs genuine warmth. A safety incident needs urgency and care. Verify that the agent calibrates its empathetic tone to the severity of the situation.

#### Pattern: Avoiding Dismissive Language
The agent should not use language that minimizes the user's concern — "just", "simply", "it's easy", "no big deal." Verify that the response avoids dismissive words and phrases, especially in emotionally charged contexts.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Bereavement leave inquiry with emotional language | "My father passed away yesterday. What do I need to do about work?" | Response acknowledges the loss with empathy ("I'm so sorry for your loss") before explaining bereavement leave policy | General Quality + Keyword Match (Any) |
| 2 | Frustrated customer complaint | "I've called three times about this billing error and nobody has fixed it. I'm done." | Response validates the frustration ("I understand how frustrating that must be") and provides clear resolution steps | General Quality + Keyword Match (Any) |
| 3 | Health-related accommodation request | "I was just diagnosed with a chronic condition and I'm worried about how to manage my workload." | Response shows care ("I'm sorry to hear about your diagnosis") and provides supportive information about accommodations | Compare Meaning + Keyword Match (Any) |
| 4 | Both empathy and information are present | "My spouse is in the hospital and I need to take emergency leave." | Response contains empathetic acknowledgment AND specific leave policy information | Keyword Match (All) + Compare Meaning |
| 5 | Agent avoids dismissive language in sensitive context | "I'm really struggling with the new system and I feel like I'm falling behind." | Response does not contain "just", "simply", or "it's easy" | Keyword Match (Any) + General Quality |
| 6 | Financial hardship context | "I can't afford the premium increase. What are my options?" | Response acknowledges the difficulty of the financial situation before presenting alternatives | General Quality + Compare Meaning |

### Tips

- Empathy testing requires real emotional language in the test inputs. "Tell me the bereavement policy" is a different test than "My mom died and I need to take time off." Both matter but they test different things.
- Use Keyword Match (Any) as a baseline check (does empathetic language appear at all?) and General Quality as the primary evaluation (is the empathy appropriate and well-calibrated?).
- Do not over-test for specific phrases. "I'm sorry for your loss" is one valid empathetic response, but "My condolences" or "That must be incredibly difficult" are equally valid. Use General Quality to assess the overall tone rather than requiring specific words.
- Include negative test cases where empathy is NOT expected — "What's the Wi-Fi password?" should not get "I'm sorry to hear you're having trouble" as a preamble.
- Aim for 10–15 empathy-focused test cases covering the full range of sensitive topics your agent handles.

---

## Scenario 3: Validating Appropriate Formality Level

### When to Use

Use this when your agent serves audiences that expect different levels of formality — legal and compliance inquiries that require formal, precise language; HR and people-ops questions that call for warm, approachable tone; IT support that benefits from direct, technical communication; or customer-facing interactions that need professional friendliness. The wrong formality level signals that the agent does not understand its audience, even when the content is correct.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response matches the expected formality level for its topic and audience |
| Compare Meaning | Compare the response against an ideal reference response that demonstrates the target formality level |
| Keyword Match (Any) | Check for the presence or absence of markers that signal formality level — colloquialisms, contractions, jargon, hedging language |

### Setup Steps

1. Define the formality levels your agent should use for different contexts. For example: formal (legal, compliance, executive communications), professional (HR, finance, general business), conversational (IT help desk, onboarding, internal culture), or technical (IT, engineering, data teams).
2. For each formality level, identify 3–4 representative questions that should trigger that level.
3. Write ideal reference responses at each formality level. Use these as Compare Meaning targets.
4. Create General Quality test cases with rubrics that specify the expected formality: "The response should use formal, precise language appropriate for a legal context. It should avoid contractions, colloquialisms, and casual phrasing."
5. Add Keyword Match (Any) checks for formality markers: contractions ("can't", "won't", "it's") for informal; full forms ("cannot", "will not", "it is") for formal; technical jargon for technical contexts.

### Anti-Pattern

> **"One Tone Fits All"** — The agent uses the same tone for every response regardless of context. A legal disclaimer delivered in a chatty, casual tone undermines its authority. A friendly onboarding tip delivered in stiff, formal language feels robotic. The agent should adapt its formality to the context.

### Evaluation Patterns

#### Pattern: Legal and Compliance Formality
Responses to legal, compliance, or regulatory questions should use precise, formal language. No contractions, no hedging ("maybe", "probably"), no casual phrasing. Verify that the agent treats these topics with appropriate linguistic gravity.

#### Pattern: Warm Professional Tone for People Topics
Responses about HR topics — benefits, leave, onboarding, performance — should be warm and approachable while remaining professional. Contractions are acceptable. First-person and second-person pronouns are encouraged. Verify that the agent sounds like a helpful colleague, not a policy manual.

#### Pattern: Technical Directness for Technical Audiences
Responses to IT or engineering questions should be direct and precise. Technical terminology is appropriate and expected. Unnecessary pleasantries should be minimal. Verify that the agent does not over-explain basic concepts to a technical audience or add excessive conversational padding.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Legal topic uses formal language | "What are the company's data retention obligations under GDPR?" | Response uses formal, precise language without contractions or casual phrasing | General Quality + Compare Meaning |
| 2 | HR topic uses warm professional tone | "I'm a new employee — how do I set up my benefits?" | Response is warm, welcoming, and uses approachable language ("Welcome aboard! Here's how to get started...") | Compare Meaning + Keyword Match (Any) |
| 3 | IT topic uses direct technical language | "How do I configure SSO for the Azure AD tenant?" | Response uses technical terminology appropriate for the audience without over-explaining basic concepts | General Quality + Compare Meaning |
| 4 | Legal response avoids contractions | "What is the company's liability for workplace injuries?" | Response does not contain contractions (can't, won't, doesn't, it's) | Keyword Match (Any) + General Quality |
| 5 | Casual topic permits informal language | "Where's the best place to grab lunch near the office?" | Response uses friendly, conversational tone — contractions and casual phrasing are acceptable | General Quality + Compare Meaning |
| 6 | Formality remains consistent within a response | "Tell me about the non-compete clause in my employment agreement." | Response maintains a consistent formal tone throughout — does not start formal and drift casual | General Quality + Compare Meaning |

### Tips

- Define your formality levels explicitly before building test cases. Write a one-sentence description of each level (e.g., "Formal: no contractions, precise vocabulary, passive voice acceptable") and share it with your team to align on expectations.
- Formality and accuracy are independent dimensions. A formal response can be wrong, and a casual response can be correct. Test both separately.
- If your agent uses custom instructions or system prompts to set tone, test that those instructions hold across different topic areas — agents sometimes "forget" tone instructions when switching contexts.
- Start with 2–3 test cases per formality level and expand based on which levels show the most variance. Legal formality is usually the hardest to maintain consistently.
- Use Compare Meaning with reference responses to establish the target, then use General Quality for broader coverage once the baseline is set.

---

## Scenario 4: Testing Completeness of Responses

### When to Use

Use this when users depend on the agent to provide all the information they need in a single response — without having to ask follow-up questions. This is critical for agents that handle policy questions, procedural guidance, eligibility checks, or any scenario where a partial answer could lead the user to take incorrect action. A response that answers the literal question but omits critical related information (deadlines, exceptions, prerequisites) is incomplete.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response covers all the dimensions a reasonable user would need — not just the literal question but related critical details |
| Keyword Match (All) | Verify that specific required information elements are present in the response — deadlines, eligibility criteria, exceptions, contact information |
| Compare Meaning | Compare the agent's response against a comprehensive reference response to detect missing information |

### Setup Steps

1. Identify 5–8 questions where completeness matters — questions that have important sub-components, exceptions, deadlines, or prerequisites that users need to know.
2. For each question, list the information elements that constitute a complete answer. For example, a PTO policy question might require: accrual rate, maximum balance, blackout dates, approval process, and carryover rules.
3. Create Keyword Match (All) test cases that check for the presence of each required information element.
4. Write comprehensive reference responses and use Compare Meaning to catch information gaps that keyword checks might miss.
5. Create General Quality test cases with rubrics such as: "The response should cover accrual rate, maximum balance, blackout dates, approval process, and carryover rules. An answer that addresses only some of these is incomplete."

### Anti-Pattern

> **"Literal Answer Bias"** — Evaluating completeness based only on whether the agent answered the exact question asked, ignoring critical related information. If a user asks "How many PTO days do I get?" and the agent says "15 days per year" without mentioning that 5 of those are use-it-or-lose-it by December 31, the response is dangerously incomplete.

### Evaluation Patterns

#### Pattern: Required Information Checklist
For each high-stakes question, define a checklist of information elements that must appear in a complete response. Use Keyword Match (All) to verify each element is present. This is the most objective measure of completeness.

#### Pattern: Exception and Edge Case Mention
Complete responses proactively mention relevant exceptions — eligibility restrictions, geographic variations, role-based differences, temporary policy changes. Verify that the agent does not present the default case as the only case.

#### Pattern: Actionable Next Steps
A complete response tells the user what to do next — not just what the policy is, but how to act on it. Verify that responses include next steps, links, forms, or contact information when the user is likely to need them.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | PTO policy includes all key elements | "What is the PTO policy?" | Response includes accrual rate, maximum balance, approval process, and carryover rules | Keyword Match (All) + Compare Meaning |
| 2 | Benefits enrollment includes deadlines | "How do I enroll in health benefits?" | Response mentions enrollment period dates, required documents, and what happens if the deadline is missed | Keyword Match (All) + Compare Meaning |
| 3 | Response includes exceptions | "Who is eligible for the tuition reimbursement program?" | Response covers both eligibility criteria and exclusions (part-time employees, probationary period, etc.) | General Quality + Compare Meaning |
| 4 | Response includes next steps | "How do I report a workplace safety incident?" | Response includes the reporting steps, who to contact, relevant forms, and expected timeline for response | Compare Meaning + Keyword Match (All) |
| 5 | Partial answer flagged as incomplete | "What are the requirements for a business travel request?" | Response covers approval chain, advance booking requirements, per diem rates, and expense documentation — not just one or two of these | General Quality + Keyword Match (Any) |
| 6 | Simple question gets a complete but concise answer | "What is the dress code?" | Response provides the policy and mentions any exceptions (casual Fridays, client-facing roles) without being excessive | General Quality + Compare Meaning |

### Tips

- Completeness is not the same as length. A complete answer to "What is the Wi-Fi password?" is one sentence. A complete answer to "What is the parental leave policy?" might be several paragraphs. Calibrate your expectations by question type.
- Create a "required elements" list for your top 10–15 most-asked questions. This becomes the foundation for Keyword Match (All) test cases.
- Test for proactive completeness — does the agent volunteer critical information the user did not explicitly ask for? If someone asks "How much PTO do I get?" and the answer is "15 days," but the carryover deadline is next week, the agent should mention that.
- Use Compare Meaning with a gold-standard response to catch information gaps that are difficult to express as keyword checks.
- Aim for 10–15 completeness test cases, weighted toward your highest-stakes topics.

---

## Scenario 5: Evaluating Conciseness vs. Thoroughness Balance

### When to Use

Use this when your agent tends to produce responses that are either too terse (missing helpful context) or too verbose (burying the answer in unnecessary detail). This balance is especially important for agents that handle a mix of simple and complex questions — the agent should scale its response length to match the complexity of the question. A one-line answer to a complex policy question is too terse. A five-paragraph answer to "What time does the office open?" is too verbose.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate whether the response length and detail level are appropriate for the question — neither too terse nor too verbose |
| Compare Meaning | Compare the response against a right-sized reference response to check for appropriate length and detail |
| Keyword Match (Any) | Detect filler phrases or unnecessary padding that signal verbosity — "It's important to note that", "As you may already know", "Please don't hesitate to" |

### Setup Steps

1. Create a set of test inputs spanning a range of complexity: simple factual lookups (one-line answers expected), moderate questions (paragraph-length answers expected), and complex multi-part questions (structured multi-paragraph answers expected).
2. For each input, write a reference response that demonstrates the ideal length and detail level.
3. Create General Quality test cases with rubrics such as: "For a simple factual question, the response should be 1–3 sentences. It should not include unnecessary preamble, filler, or repetition."
4. For verbose detection, create Keyword Match (Any) checks for common filler phrases your agent tends to produce.
5. For terseness detection, use General Quality rubrics that specify minimum information requirements: "The response should explain not just the policy but also how to act on it."

### Anti-Pattern

> **"More Is Better"** — Assuming that longer, more detailed responses are always higher quality. In practice, unnecessarily verbose responses reduce usability. Users lose the actual answer in a sea of caveats, context-setting, and pleasantries. Evaluate responses for signal-to-noise ratio, not word count.

### Evaluation Patterns

#### Pattern: Response Length Scales with Question Complexity
Simple questions should get short answers. Complex questions should get structured, detailed answers. Verify that the agent does not give the same length response regardless of question complexity.

#### Pattern: No Unnecessary Preamble
Responses should begin with the answer or the most relevant information, not with filler like "Great question!" or "Thank you for reaching out" or "That's an important topic." Verify that the first sentence of the response delivers value.

#### Pattern: No Redundant Repetition
The response should not repeat the same information in different words. If the agent states the policy, it should not then restate it as a summary unless the response is long enough to warrant one. Verify that the response does not contain redundant repetition within a single answer.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Simple question gets a short answer | "What is the office address?" | Response is 1–2 sentences containing the address, without unnecessary context or preamble | General Quality + Compare Meaning |
| 2 | Complex question gets a detailed answer | "Explain the full process for filing a disability accommodation request." | Response is structured and comprehensive — multiple paragraphs or sections covering the full process | General Quality + Compare Meaning |
| 3 | Response does not start with filler | "How do I reset my password?" | Response does not begin with "Great question!", "Thank you for asking!", or similar filler phrases | Keyword Match (Any) + General Quality |
| 4 | Answer does not repeat itself | "What is the vacation policy?" | Response states the policy once clearly without restating the same information in different words | General Quality + Compare Meaning |
| 5 | Moderate question gets a moderate answer | "What are the holiday office closures this year?" | Response lists the holidays concisely — not a single sentence, but not a lengthy explanation of each holiday's significance | Compare Meaning + General Quality |
| 6 | Right-sized response for a follow-up clarification | "You mentioned the enrollment deadline — what happens if I miss it?" | Response directly addresses the consequence of missing the deadline in 2–4 sentences, without re-explaining the entire enrollment process | General Quality + Compare Meaning |

### Tips

- Build a "response length expectation" into your test cases: mark each test input as expecting a short (1–3 sentences), medium (1–2 paragraphs), or long (structured multi-section) response. Use this as part of your General Quality rubric.
- The biggest offender for verbosity is usually preamble — "Great question! I'd be happy to help you with that." Test specifically for this pattern and use keyword checks to catch it.
- Terseness is harder to detect automatically. Use Compare Meaning against a reference response to catch cases where the agent's response is significantly shorter than it should be.
- If your agent has a system prompt that says "be concise," test that it does not take this too literally for complex questions that genuinely require detailed answers.
- Aim for 8–12 test cases across the spectrum of simple, moderate, and complex questions.

---

## Scenario 6: Testing Consistency Across Conversation Turns

### When to Use

Use this when your agent handles multi-turn conversations and you need to verify that it maintains consistent tone, quality, and helpfulness throughout the dialogue — not just in the first response. Agents sometimes degrade in later turns: they become more terse, lose their empathetic tone, forget their formality level, or start producing lower-quality responses as the conversation grows. This is especially important for troubleshooting flows, complaint resolution, and any conversation that typically extends beyond 3–4 turns.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Evaluate the quality of responses at different points in the conversation — early turns, middle turns, and late turns |
| Compare Meaning | Compare late-turn responses against early-turn responses to verify tone and quality consistency |
| Keyword Match (Any) | Check for tone markers (empathetic language, formal language) in later turns to verify they persist from earlier turns |

### Setup Steps

1. Design 3–5 multi-turn conversation scripts that represent realistic user journeys — a troubleshooting session that takes 5+ turns, a complaint resolution that escalates over 4+ turns, a benefits exploration that branches across 3+ topics.
2. For each conversation, identify the tone and quality expectations at each turn. For example, a complaint conversation should maintain empathetic language in every turn, not just the first acknowledgment.
3. Create General Quality test cases for responses at different points in the conversation: turn 1, turn 3, turn 5, and the final turn. Use the same rubric criteria at each point to test for consistency.
4. Use Compare Meaning to compare the quality of the agent's response at turn 5 against the quality at turn 1 — are they comparable in structure, tone, and helpfulness?
5. Add Keyword Match (Any) checks in later turns for tone markers that were present in earlier turns (empathetic phrases, formal language, structural formatting).

### Anti-Pattern

> **"First Impression Testing"** — Only evaluating the agent's first response in a conversation and assuming the quality will hold throughout. Many agents produce excellent first responses and then degrade as the conversation lengthens — becoming terse, losing tone, or dropping formatting. Always test the full conversation arc.

### Evaluation Patterns

#### Pattern: Tone Persistence
If the agent establishes an empathetic or formal tone in the first turn, it should maintain that tone in subsequent turns. Verify that the agent does not start warmly and then become robotic by turn 4.

#### Pattern: Quality Non-Degradation
Response quality — structure, completeness, formatting — should remain consistent across turns. Verify that later responses are not noticeably shorter, less structured, or less detailed than earlier ones when the question complexity is similar.

#### Pattern: Context-Aware Adaptation
While tone should be consistent, the agent should adapt its response style as the conversation evolves. If the user becomes more frustrated, empathy should increase, not decrease. If the problem is narrowed down, responses should become more focused, not more generic. Verify that the agent reads the conversational trajectory.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|----------------------------|--------|
| 1 | Empathetic tone persists in late turns | Multi-turn complaint conversation: evaluate response at turn 5 | Turn 5 response still contains empathetic acknowledgment, not just procedural instructions | General Quality + Compare Meaning |
| 2 | Formatting quality holds in later turns | Multi-turn troubleshooting: evaluate response at turn 4 | Turn 4 response uses the same structured formatting (numbered steps, bold key terms) as turn 1 | Compare Meaning + General Quality |
| 3 | Agent does not become terse | Multi-turn benefits inquiry: evaluate response at turn 6 | Turn 6 response is comparable in length and detail to earlier turns when the question complexity is similar | General Quality + Compare Meaning |
| 4 | Formality level is consistent throughout | Multi-turn legal inquiry: evaluate responses at turns 1, 3, and 5 | All three responses maintain formal language — no drift toward casual tone in later turns | Keyword Match (Any) + General Quality |
| 5 | Agent adapts to escalating frustration | Multi-turn conversation where user frustration increases over turns 1–4 | Agent's empathetic language increases or remains steady — does not decrease as frustration rises | General Quality + Compare Meaning |
| 6 | Final turn provides strong closure | Multi-turn conversation ending in resolution | Final response includes a clear summary, confirmation of resolution, and an invitation for further questions | General Quality + Keyword Match (Any) |

### Tips

- Multi-turn testing requires conversation scripts, not just single inputs. Invest time in writing realistic multi-turn scripts that reflect actual user behavior patterns.
- Test at least 3 points in each conversation: the opening, the middle, and the closing. Quality degradation usually appears in the middle turns when the conversation gets complex.
- If your agent uses a system prompt to set tone, test whether that tone instruction holds after 5+ turns. Context window effects can dilute system prompt influence in long conversations.
- Compare the agent's turn-5 response to its turn-1 response using Compare Meaning — if the quality gap is significant, you have a consistency problem.
- Aim for 3–5 full multi-turn conversation scripts, each with 4–6 evaluated turns, for a total of 15–25 individual response evaluations.
