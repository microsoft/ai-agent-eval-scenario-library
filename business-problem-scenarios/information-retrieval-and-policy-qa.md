# Information Retrieval Q&A

> Scenarios for agents that answer user questions by retrieving information from authoritative knowledge sources — SharePoint, uploaded documents, FAQ databases, or other configured data sources.

[Back to library](../README.md) | [All business-problem scenarios](README.md)

---

## Scenario 1: Verifying Answer Accuracy

### When to Use

Your agent answers factual questions by retrieving from knowledge sources (SharePoint pages, uploaded PDFs, FAQ entries), and you need to confirm that the answers it returns are correct — not hallucinated, not outdated, and not pulled from the wrong document.

This is the foundational scenario for any knowledge-grounded agent. Start here before testing personalization, attribution, or edge cases.

> **Related scenarios:** If you also need to verify that the agent retrieves from the *correct* source (not just that the answer is correct), see [Scenario 3: Validating Source Attribution](#scenario-3-validating-source-attribution). For infrastructure-level grounding checks, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms that specific factual details appear in the response — policy names, dollar amounts, eligibility criteria, deadlines |
| Compare Meaning | Validates that the agent conveys the correct meaning even when it paraphrases the source material |
| General Quality | Assesses whether the answer is complete, relevant, and well-structured — not just factually present |

> **Tip:** Use Keyword Match (All) for answers with hard facts (numbers, dates, specific terms). Use Compare Meaning when the agent is expected to summarize or rephrase. Layer both when you need factual precision AND correct interpretation.

### Setup Steps

1. Identify 8-12 questions that represent your agent's most common queries. Include a mix of simple lookups ("What is the PTO accrual rate?") and questions requiring synthesis ("What happens to unused PTO when I transfer departments?").
2. For each question, manually locate the correct answer in your source documents. Write the expected answer as a concise factual statement.
3. Create test cases using **Keyword Match (All)** for questions with specific facts that must appear (e.g., "15 days," "annual," "manager approval").
4. Create test cases using **Compare Meaning** for questions where the agent should summarize or explain a concept rather than quote verbatim.
5. Add 2-3 test cases using **General Quality** with a grading prompt like: "Is this answer factually accurate based on the company's published policy? Is it complete enough for the user to act on?"
6. Run the evaluation and review failures. For each failure, check: is the answer actually wrong, or is the expected value too narrowly defined?

### Anti-Pattern

> **Anti-Pattern: Testing Only Simple Lookups**
> Many teams test questions like "What is our return policy?" but skip synthesis questions like "Can I return an opened item that was a gift?" Simple lookups confirm the agent can retrieve; synthesis questions confirm it can reason over retrieved content. If you only test lookups, you will miss cases where the agent retrieves the right document but draws the wrong conclusion.

### Evaluation Patterns

**Pattern: Direct Fact Verification**
Test questions that have a single, unambiguous factual answer. The expected value contains the specific fact(s) that must appear. Use Keyword Match (All) so the test fails if any required fact is missing.

**Pattern: Synthesis Verification**
Test questions that require the agent to combine information from one or more passages to form an answer. The expected value describes the correct meaning. Use Compare Meaning because the exact wording will vary but the substance must be right.

**Pattern: Completeness Check**
Test questions where a partial answer is misleading or harmful. For example, listing only 2 of 4 eligibility requirements. Use General Quality with a grading prompt that specifies what a complete answer includes.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — PTO accrual | "How many PTO days do I earn per year?" | "15 days", "accrual", "per year" | Keyword Match (All) + Compare Meaning |
| 2 | HR policy bot — PTO carryover synthesis | "What happens to my unused PTO at the end of the year?" | The agent should explain that up to 5 days carry over and the rest is forfeited, with a December 31 deadline | Compare Meaning + Keyword Match (All) |
| 3 | Product FAQ bot — warranty coverage | "Does the warranty cover water damage?" | "not covered", "water damage", "accidental damage exclusion" | Keyword Match (All) + Compare Meaning |
| 4 | Product FAQ bot — repair vs. replace | "If my device is defective, will I get a new one or a repaired unit?" | The agent should explain that devices under 30 days are replaced, and devices over 30 days are repaired, depending on parts availability | Compare Meaning + Keyword Match (Any) |
| 5 | Legal/compliance bot — data retention | "How long do we keep employee records after termination?" | "7 years", "termination", "personnel files" | Keyword Match (All) + Compare Meaning |
| 6 | Legal/compliance bot — cross-border data transfer | "Can we transfer customer data to our EU subsidiary?" | The answer correctly describes the requirement for Standard Contractual Clauses and references the data processing agreement requirement | Compare Meaning + Keyword Match (All) |
| 7 | HR policy bot — completeness of eligibility | "Who is eligible for parental leave?" | Is the answer complete? It should mention full-time employees, 12-month tenure requirement, both birth and adoption, and that part-time employees at 30+ hours also qualify | General Quality + Keyword Match (Any) |
| 8 | Product FAQ bot — return completeness | "How do I return a product I bought online?" | Does the answer include all necessary steps: initiate return in account portal, print shipping label, pack item in original packaging, and drop off at carrier location? | General Quality + Keyword Match (Any) |

### Tips

- Aim for **10-15 test cases** covering your agent's top queries. Prioritize questions users actually ask (check conversation logs if available) over questions you think they might ask.
- Set a threshold of **>= 90% pass rate** for Keyword Match and Compare Meaning tests. If you are below this, investigate whether the issue is retrieval (wrong document) or generation (right document, wrong interpretation).
- Include at least **2-3 synthesis questions** for every 5 direct lookup questions. Synthesis failures are higher-severity because users act on incomplete reasoning.
- When writing expected values for Keyword Match, use the **minimum set of keywords** that uniquely identify the correct answer. Over-specifying causes false failures when the agent uses synonyms.
- Re-run this scenario after any knowledge source update (new document uploaded, SharePoint page edited, FAQ entry changed).

---

## Scenario 2: Testing Answer Personalization by User Context

### When to Use

Your agent tailors its answers based on user attributes — such as role, department, location, employment type, or tenure — and you need to verify that personalization is working correctly. For example, a policy question about parental leave should return different answers depending on whether the user is a full-time employee in the US vs. a contractor in the UK.

This scenario applies when your agent receives user context (via authentication, system variables, or conversation state) and is expected to use that context to filter or adjust its response.

> **Related scenarios:** If personalization is not relevant and you just need answer accuracy, see [Scenario 1: Verifying Policy Answer Accuracy](#scenario-1-verifying-policy-answer-accuracy). If you are testing whether the agent retrieves from the correct source per user context, see [Scenario 3: Validating Source Attribution](#scenario-3-validating-source-attribution).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms that context-specific terms appear — the right policy tier, the right regional benefit, the right role-specific procedure |
| Compare Meaning | Validates that the overall answer is appropriate for the user context, even if wording varies |
| General Quality | Assesses whether the response is correctly scoped to the user's context without leaking information from other segments |

> **Tip:** Personalization tests require careful test case design. You need pairs or groups of test cases with the same question but different user contexts, where the expected answer changes based on context.

### Setup Steps

1. Identify which user attributes your agent uses for personalization (e.g., department, location, employment type, seniority level).
2. Select 3-5 questions where the answer meaningfully changes based on user context. Avoid questions where the answer is the same for everyone.
3. For each question, create **2-3 test cases** with different user contexts. Set the expected value to the context-appropriate answer for each.
4. Include at least 1 **negative personalization test**: a question where the user context should NOT change the answer, to confirm the agent does not over-personalize.
5. Configure test cases using the appropriate system variables or authentication context to simulate each user profile.
6. Use **Keyword Match (All)** for answers where specific context-dependent facts must appear. Use **Compare Meaning** where the overall guidance varies by context.
7. Run the evaluation. For each failure, determine: did the agent ignore the context (returned generic answer), apply the wrong context, or leak information from another segment?

### Anti-Pattern

> **Anti-Pattern: Testing Personalization with Only One User Profile**
> If all your test cases use the same user context (e.g., a US-based full-time employee), you have no evidence that personalization works. You are only confirming the default path. Personalization testing requires **variation in context** — the same question asked by users in different roles, locations, or employment types, with expected answers that differ accordingly.

### Evaluation Patterns

**Pattern: Same Question, Different Context**
Ask the same policy question with two or more user profiles and verify that the answer changes appropriately. The expected values differ per test case. This is the core personalization pattern.

**Pattern: Context-Scoped Accuracy**
Verify that the personalized answer is accurate for that specific context — not just different, but correct. A wrong personalized answer is worse than a generic one because the user trusts it more.

**Pattern: No Leakage Across Segments**
Confirm that the answer for one user context does not include information meant for another segment. For example, a US employee asking about parental leave should not see UK-specific statutory leave information mixed in.

**Pattern: Graceful Fallback When Context Is Missing**
Test what happens when user context is unavailable or incomplete. The agent should either ask for the relevant attribute or provide a general answer with a note that it may vary, rather than guessing.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — US full-time employee | "How many PTO days do I get?" (context: US, full-time, 3 years tenure) | "20 days", "full-time", "3-5 year tier" | Keyword Match (All) + Compare Meaning |
| 2 | HR policy bot — US part-time employee | "How many PTO days do I get?" (context: US, part-time, 3 years tenure) | "10 days", "part-time", "prorated" | Keyword Match (All) + Compare Meaning |
| 3 | HR policy bot — UK employee | "How many PTO days do I get?" (context: UK, full-time, 3 years tenure) | "25 days", "statutory", "plus bank holidays" | Keyword Match (All) + Compare Meaning |
| 4 | Product FAQ bot — enterprise customer | "What support channels are available to me?" (context: Enterprise plan) | The response should mention dedicated account manager, 24/7 phone support, and priority ticket queue | Compare Meaning + Keyword Match (All) |
| 5 | Product FAQ bot — free-tier customer | "What support channels are available to me?" (context: Free plan) | The response should mention community forums and email support with 48-hour SLA, without mentioning phone or dedicated account manager | Compare Meaning + Keyword Match (All) |
| 6 | Legal/compliance bot — manager role | "How do I handle a harassment complaint?" (context: role = manager) | The response should include the manager's obligation to report, escalation steps to HR, and documentation requirements | Compare Meaning + Keyword Match (Any) |
| 7 | Legal/compliance bot — individual contributor role | "How do I handle a harassment complaint?" (context: role = IC) | The response should explain how to file a complaint, available reporting channels, and anti-retaliation protections — without manager-specific obligations | Compare Meaning + Keyword Match (Any) |
| 8 | HR policy bot — no context available | "How many PTO days do I get?" (context: missing employment type) | The agent should either ask whether the user is full-time or part-time, or provide both tiers with a note to check which applies | General Quality + Compare Meaning |

### Tips

- Create **matched pairs** (same question, different context) for at least 3-5 key questions. This is the only reliable way to confirm personalization works.
- Set a threshold of **>= 95% pass rate** for personalization tests, because personalization errors (giving someone the wrong policy for their situation) can lead to real-world consequences.
- Test the **no-context fallback** explicitly. Users will encounter this in production when authentication context is missing or incomplete.
- Check for **information leakage** across segments. A response that says "US employees get 15 days and UK employees get 25 days" when the user is a US employee is a leakage issue — the agent should answer for the user's context only.
- Re-run personalization tests whenever you update user-context logic, add new user segments, or modify role-based access to knowledge sources.

---

## Scenario 3: Validating Source Attribution

### When to Use

Your agent is expected to reference or cite its sources when answering questions — for example, linking to the SharePoint page, naming the policy document, or including a citation. You need to verify that the attributed source is correct and that the answer actually comes from that source.

This scenario also applies when your agent uses multiple knowledge sources and you need to confirm it retrieves from the right one — not just that the answer sounds correct, but that it comes from the authoritative source for that question.

> **Related scenarios:** For answer accuracy without source concerns, see [Scenario 1: Verifying Policy Answer Accuracy](#scenario-1-verifying-policy-answer-accuracy). For infrastructure-level grounding validation, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms that the agent consulted the expected knowledge source(s) to generate its answer |
| Keyword Match (All) | Verifies that the response includes the expected source name, document title, or citation |
| Compare Meaning | Validates that the answer content actually aligns with what the cited source says |

> **Tip:** A response can be factually correct but attributed to the wrong source. Use Capability Use to check which source was actually retrieved, and Keyword Match to verify the citation text in the response. These catch different failure modes.

### Setup Steps

1. Map your agent's knowledge sources: list each source (SharePoint site, PDF, FAQ database) and the topics it covers.
2. For each knowledge source, write 2-3 test cases with questions that should be answered from that specific source.
3. Configure **Capability Use (All)** tests that check whether the expected knowledge source was consulted. Use the knowledge source name or identifier as it appears in your agent's configuration.
4. If your agent surfaces citations in its response, add **Keyword Match (All)** tests that check for the correct source name or document title.
5. Add 1-2 **Compare Meaning** tests to verify that the cited source actually supports the claim in the response — catching cases where the agent cites a source but the content comes from elsewhere.
6. Run the evaluation. For failures, investigate: did the agent use the wrong source, use the right source but cite incorrectly, or fabricate a citation?

### Anti-Pattern

> **Anti-Pattern: Trusting the Citation Without Checking the Source**
> Some teams check only whether a citation appears in the response, not whether the cited source was actually retrieved or whether the answer matches the cited content. An agent can learn to produce plausible-sounding citations that do not correspond to real retrieval. Always verify attribution at the **capability level** (was the source consulted?) and at the **content level** (does the answer match the source?).

### Evaluation Patterns

**Pattern: Correct Source Retrieval**
Verify that the agent retrieves from the expected knowledge source for a given topic. Use Capability Use (All) to check that the right source was consulted. This catches cases where the answer happens to be correct but came from the wrong document.

**Pattern: Citation Accuracy in Response**
Verify that any source reference in the agent's response text (document name, link, page title) is correct and matches the source that was actually retrieved. Use Keyword Match (All) to check for the expected citation text.

**Pattern: Content-Source Alignment**
Verify that what the agent says actually matches what the cited source contains. This catches "citation decoration" — where the agent cites a source but the answer is generated from a different source or fabricated. Use Compare Meaning to check alignment.

**Pattern: Multi-Source Attribution**
For questions that require information from multiple sources, verify that all relevant sources are cited and that the response correctly attributes which information comes from which source.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — source retrieval | "What is the bereavement leave policy?" | Knowledge source: HR-Policies-2025.pdf | Capability Use (All) + Keyword Match (All) |
| 2 | HR policy bot — citation text | "What is the bereavement leave policy?" | "HR-Policies-2025", "Bereavement Leave", "Section 4.3" | Keyword Match (All) + Compare Meaning |
| 3 | Product FAQ bot — correct source | "How do I reset my device to factory settings?" | Knowledge source: Product-Support-KB (not Marketing-Brochures) | Capability Use (All) + Keyword Match (All) |
| 4 | Product FAQ bot — citation in response | "How do I reset my device to factory settings?" | "Support Article KB-4521" or "Factory Reset Guide" | Keyword Match (All) + Compare Meaning |
| 5 | Legal/compliance bot — regulatory source | "What are our GDPR data subject rights obligations?" | Knowledge source: GDPR-Compliance-Handbook-2025.pdf | Capability Use (All) + Keyword Match (All) |
| 6 | Legal/compliance bot — content alignment | "What are our GDPR data subject rights obligations?" | The response content should align with the cited GDPR handbook, describing the right to access, rectification, erasure, and portability as specified in that document | Compare Meaning + Keyword Match (Any) |
| 7 | HR policy bot — multi-source question | "How does parental leave interact with short-term disability?" | Knowledge sources: HR-Policies-2025.pdf AND Benefits-Guide-2025.pdf | Capability Use (All) + Compare Meaning |
| 8 | Product FAQ bot — wrong source negative test | "What are the technical specifications of Model X?" | Knowledge source should be Product-Specs-Sheet, NOT customer-reviews or marketing-copy | Capability Use (All) + Compare Meaning |

### Tips

- Map every knowledge source to its covered topics **before** writing test cases. This mapping is your ground truth for source attribution tests.
- Aim for **2-3 attribution tests per knowledge source** to ensure each source is correctly retrieved for its intended topics.
- Set a threshold of **>= 95% pass rate** for Capability Use tests. Source attribution failures are high-severity — they undermine trust even when the answer happens to be correct.
- Test **cross-source questions** (questions that touch two knowledge sources) separately from single-source questions. These are more likely to have attribution errors.
- Re-run attribution tests whenever you add, remove, or rename knowledge sources, or when you change the agent's knowledge source descriptions.

---

## Scenario 4: Testing Answer Currency

### When to Use

Your knowledge sources are updated periodically (policy revisions, product updates, regulatory changes), and you need to verify that the agent returns information from the **current** version rather than an outdated one. This is critical for agents that answer policy questions, compliance queries, or product-related FAQs where outdated information can cause real harm.

This scenario applies when you have recently updated a knowledge source and want to confirm the agent reflects the change, or when you want to establish a baseline to detect stale-answer issues proactively.

> **Related scenarios:** For general answer accuracy without currency concerns, see [Scenario 1: Verifying Policy Answer Accuracy](#scenario-1-verifying-policy-answer-accuracy). For verifying the agent retrieves from the correct source version, see [Scenario 3: Validating Source Attribution](#scenario-3-validating-source-attribution).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms that the response contains terms, figures, or dates from the current version of the policy or document — not the previous version |
| Compare Meaning | Validates that the answer reflects the current policy intent, especially when the update changed the meaning rather than just specific figures |
| Capability Use (All) | Verifies that the agent retrieved from the current version of the knowledge source rather than a cached or outdated copy |

> **Tip:** Currency tests are most valuable right after a knowledge source update. Create a "currency smoke test" — a small set of 3-5 test cases targeting recently changed content — and run it every time you update a source.

### Setup Steps

1. Identify knowledge sources that were recently updated or are updated on a regular schedule.
2. For each update, note what changed: specific figures, dates, eligibility criteria, process steps, or policy names.
3. Write test cases where the **question is unchanged** but the **expected answer reflects the new content**. This is key: the question the user asks does not change when a policy updates.
4. Use **Keyword Match (All)** with terms or figures from the new version. If the old version had "10 days" and the new version has "15 days," the expected value should include "15 days."
5. Add a **Compare Meaning** test for changes that altered the overall guidance (e.g., a policy that shifted from "manager discretion" to "automatic approval").
6. If your agent has multiple knowledge source versions, use **Capability Use (All)** to confirm retrieval from the current version.
7. Run the evaluation. For failures, investigate: is the agent retrieving a cached version, an old document, or generating from stale training data?

### Anti-Pattern

> **Anti-Pattern: No Expected-Value Update When Sources Change**
> When you update a knowledge source, you must also update your test cases. If your test cases still expect the old answer, they will pass when they should fail. Treat eval set maintenance as part of the knowledge source update process — every document update should trigger a review of test cases referencing that document.

### Evaluation Patterns

**Pattern: Version-Specific Fact Check**
Test a question where a specific fact (number, date, threshold) changed between versions. The expected value uses the new figure. The test fails if the agent returns the old figure.

**Pattern: Policy Direction Change**
Test a question where the overall guidance changed (e.g., a process that used to require approval now does not). Use Compare Meaning to verify the agent reflects the new direction, since the change may be expressed in different words.

**Pattern: Effective Date Awareness**
Test whether the agent correctly communicates effective dates for new policies. If a policy change takes effect on a future date, the agent should not present it as current until that date.

**Pattern: Transition Period Handling**
Test scenarios where old and new policies overlap during a transition period. The agent should explain both the current rules and the upcoming change, with clear dates.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — updated PTO accrual | "How many PTO days do new employees get?" (policy updated from 10 to 15 days in Jan 2026) | "15 days" (not "10 days") | Keyword Match (All) + Compare Meaning |
| 2 | HR policy bot — removed approval step | "Do I need manager approval for PTO under 3 days?" (policy changed: approval no longer required) | The response should state that PTO under 3 days no longer requires manager approval and can be submitted directly through the portal | Compare Meaning + Keyword Match (Any) |
| 3 | Product FAQ bot — updated pricing | "How much does the Pro plan cost?" (price changed from $29 to $39/month) | "$39", "per month", "Pro plan" | Keyword Match (All) + Compare Meaning |
| 4 | Product FAQ bot — new feature in current release | "Does Model X support Bluetooth 5.3?" (added in latest firmware update) | "Bluetooth 5.3", "supported", "firmware update 4.2" | Keyword Match (All) + Compare Meaning |
| 5 | Legal/compliance bot — regulatory update | "What is the breach notification deadline?" (updated from 72 hours to 48 hours per new regulation) | "48 hours" (not "72 hours") | Keyword Match (All) + Compare Meaning |
| 6 | Legal/compliance bot — effective date awareness | "When does the new data retention policy take effect?" | "March 1, 2026", "effective date" | Keyword Match (All) + Compare Meaning |
| 7 | HR policy bot — transition period | "What parental leave am I eligible for?" (new policy effective April 1; current policy still applies until then) | The response should explain the current policy and note the upcoming change effective April 1, so the user knows which applies to their situation | Compare Meaning + Keyword Match (Any) |
| 8 | Product FAQ bot — deprecated feature | "How do I use the legacy import tool?" (feature removed in latest version) | The response should explain that the legacy import tool has been discontinued and describe the replacement workflow | Compare Meaning + Keyword Match (Any) |

### Tips

- Maintain a **currency test set** of 5-10 test cases targeting your most recently changed content. Update this set every time a knowledge source changes.
- Run currency tests **within 24 hours** of updating a knowledge source to catch stale retrieval issues early.
- Set a threshold of **100% pass rate** for currency tests. There is no acceptable tolerance for returning outdated policy information — especially for compliance and legal content.
- Track **which knowledge source version** the agent retrieves from. If your platform supports version identifiers, use Capability Use to verify the current version is being consulted.
- Document a process for updating test cases when sources change. If test case maintenance is not part of the content update workflow, test cases will become stale and give false confidence.

---

## Scenario 5: Handling Ambiguous or Broad Queries

### When to Use

Users often ask vague, broad, or ambiguous questions — "Tell me about benefits," "What's the policy?", "How does shipping work?" — and you need to verify that the agent handles these well. A good agent should either ask a clarifying question, provide a structured overview of the topic, or both. A bad agent guesses at what the user meant and returns a narrow (possibly wrong) answer.

This scenario applies to any knowledge-grounded agent where users may not know the exact terminology, may ask open-ended questions, or may phrase their question ambiguously (e.g., "What's the leave policy?" could refer to PTO, sick leave, parental leave, or bereavement leave).

> **Related scenarios:** For testing whether the agent's answers are correct once the question is clear, see [Scenario 1: Verifying Policy Answer Accuracy](#scenario-1-verifying-policy-answer-accuracy). For agents that should route ambiguous inputs to the right topic, see [Triage & Routing](triage-and-routing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Assesses whether the agent's handling of ambiguity is helpful — does it clarify, provide a structured overview, or guide the user? Requires a grading prompt describing what good handling looks like |
| Compare Meaning | Validates that the agent's response covers the expected range of sub-topics or asks the right clarifying question |
| Keyword Match (Any) | Checks that the response mentions at least some of the relevant sub-topics when a broad overview is expected |

> **Tip:** General Quality is the primary method here because handling ambiguity is a qualitative judgment. Supplement with Compare Meaning or Keyword Match (Any) when you can define what a good response should cover.

### Setup Steps

1. Identify 5-8 broad or ambiguous questions users are likely to ask. Pull these from conversation logs if available — real user queries are almost always broader than test-writer queries.
2. For each question, decide what a good response looks like. Options include: (a) asking a clarifying question to narrow the scope, (b) providing a structured overview with sub-topics the user can explore, or (c) answering the most common interpretation while noting alternatives.
3. Create **General Quality** test cases with a grading prompt that describes the ideal handling. For example: "The agent should either ask a clarifying question to understand which type of leave the user is asking about, or provide a structured overview of all leave types."
4. Add **Compare Meaning** tests for cases where you want to verify a specific handling approach (e.g., the agent should always provide an overview for the query "Tell me about benefits").
5. Use **Keyword Match (Any)** to verify that overviews mention the expected sub-topics (e.g., a response to "Tell me about leave" should mention at least some of: PTO, sick leave, parental leave, bereavement).
6. Run the evaluation. For failures, assess: did the agent guess and answer narrowly, provide an unhelpful generic response, or fail to acknowledge the ambiguity?

### Anti-Pattern

> **Anti-Pattern: Treating Ambiguous Queries as Failures**
> Some teams expect the agent to always give a direct answer, and they mark clarification responses as failures. In reality, asking a clarifying question for an ambiguous input is the correct behavior. If your agent says "Could you tell me which type of leave you're asking about — PTO, sick leave, or parental leave?" that is a pass, not a fail. Write your expected values and grading prompts to reward clarification.

### Evaluation Patterns

**Pattern: Clarification Quality**
When the agent asks a clarifying question in response to ambiguity, evaluate whether the clarification is helpful, specific, and presents the right options. A good clarification narrows the scope efficiently. A bad clarification asks an overly generic question ("Can you be more specific?") that does not guide the user.

**Pattern: Structured Overview**
When the agent provides an overview instead of asking for clarification, evaluate whether the overview is well-structured, covers the major sub-topics, and gives the user a clear path to dig deeper.

**Pattern: Most-Common-Interpretation with Alternatives**
When the agent answers the most likely interpretation of an ambiguous query, evaluate whether it acknowledges that other interpretations exist. For example: "Here's our PTO policy — if you were asking about sick leave or parental leave, let me know and I can help with those too."

**Pattern: Does Not Guess Wrongly**
Verify that the agent does not confidently answer a narrow interpretation of a broad query without acknowledging the ambiguity. This is the failure mode to watch for — it looks like a normal answer but addresses the wrong question.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — broad leave query | "What's the leave policy?" | The agent should ask which type of leave the user is asking about (PTO, sick, parental, bereavement) or provide a structured overview of all leave types | General Quality + Compare Meaning |
| 2 | HR policy bot — vague benefits question | "Tell me about benefits" | The response should cover or list major benefit categories: health insurance, dental/vision, retirement (401k), PTO, wellness programs | Keyword Match (Any) + Compare Meaning |
| 3 | Product FAQ bot — ambiguous product reference | "How do I set it up?" (no product context) | The agent should ask which product the user needs help setting up, or list the available products with setup guides | General Quality + Compare Meaning |
| 4 | Product FAQ bot — broad shipping query | "How does shipping work?" | The response should provide a structured overview covering shipping methods, timelines, tracking, and international shipping — or ask whether the user has a specific question about one of these areas | Compare Meaning + Keyword Match (Any) |
| 5 | Legal/compliance bot — vague compliance query | "What are our obligations?" | The agent should ask which regulatory area the user is asking about (data privacy, workplace safety, financial reporting) rather than guessing | General Quality + Compare Meaning |
| 6 | Legal/compliance bot — broad data privacy question | "Tell me about data privacy" | The response should provide a high-level overview covering data collection, storage, access controls, and user rights — not a narrow answer about just one aspect | Compare Meaning + Keyword Match (Any) |
| 7 | HR policy bot — ambiguous acronym | "What's the policy on FML?" (could mean FMLA or could be garbled input) | The agent should clarify whether the user is asking about FMLA (Family and Medical Leave Act) rather than assuming | General Quality + Compare Meaning |
| 8 | Product FAQ bot — common-interpretation with alternatives | "Can I get a refund?" | The response should address the most common refund scenario (standard return within 30 days) while noting that warranty claims and subscription cancellations have separate processes | Compare Meaning + Keyword Match (Any) |

### Tips

- Pull ambiguous queries from **real conversation logs** if available. Real users phrase things differently than test writers.
- Use **General Quality** as the primary method, with a grading prompt that explicitly states: "Is the agent's handling of this ambiguous query helpful? Does it clarify, provide structure, or answer the most likely interpretation while noting alternatives?"
- Set a threshold of **>= 80% pass rate** for ambiguity handling. This is lower than accuracy thresholds because there are multiple valid ways to handle ambiguity, and grading is more subjective.
- Do NOT penalize the agent for asking a clarifying question. Write your grading prompts and expected values to **reward** appropriate clarification.
- Include at least **3 broad queries and 3 ambiguous queries** in your test set. Broad queries ("Tell me about benefits") and ambiguous queries ("What's the policy on FML?") test different handling skills.

---

## Scenario 6: Negative Test — Incorrect Source Retrieval

### When to Use

Your agent has access to multiple knowledge sources, and you need to verify that it does NOT pull from irrelevant, incorrect, or out-of-scope sources when answering a question. This is the inverse of Scenario 3 (source attribution) — instead of checking that the right source was used, you check that the wrong source was NOT used.

This scenario is critical when your agent has knowledge sources that cover similar but distinct topics (e.g., US policy vs. UK policy, employee handbook vs. contractor handbook, current product vs. discontinued product), where cross-contamination is a real risk.

> **Related scenarios:** For verifying the agent uses the correct source, see [Scenario 3: Validating Source Attribution](#scenario-3-validating-source-attribution). For testing that the agent does not hallucinate beyond any source, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Capability Use (All) | Confirms that the agent retrieved from the expected source AND did not retrieve from a specifically wrong source (use as a negative check) |
| Keyword Match (All) | Verifies that the response does not contain terms, figures, or references that belong to the wrong source — use keywords from the wrong source as a negative expected value |
| General Quality | Assesses whether the response stays scoped to the correct domain and does not include information that clearly comes from an incorrect source |

> **Tip:** Negative tests are harder to write than positive tests because you need to anticipate which wrong source the agent might use. Start with knowledge sources that have overlapping topics — these are the highest-risk pairs.

### Setup Steps

1. Identify **high-risk knowledge source pairs** — sources that cover similar topics but serve different audiences or contexts. Examples: US benefits vs. UK benefits, employee handbook vs. contractor guide, current product line vs. discontinued products.
2. For each high-risk pair, write test cases where the question should be answered from Source A, and verify the agent does not pull from Source B.
3. Use **Capability Use (All)** to check that the correct source was consulted. If your evaluation platform supports it, also verify that the incorrect source was NOT consulted.
4. Use **Keyword Match (All)** as a negative check: include keywords or terms that are unique to the wrong source. If those terms appear in the response, the test should flag it. (Note: configure this as a negative match if your platform supports it, or use General Quality with a prompt like "Does this response contain any UK-specific policy information?")
5. Use **General Quality** with a grading prompt focused on scope: "Is this response correctly scoped to [correct context]? Does it include any information from [incorrect context]?"
6. Run the evaluation. For failures, investigate: what caused the cross-contamination? Is it a retrieval ranking issue, a document overlap issue, or a generation issue?

### Anti-Pattern

> **Anti-Pattern: Only Testing Positive Retrieval, Never Testing Negative**
> Most teams verify that the agent retrieves from the right source but never verify that it avoids the wrong source. This is a major gap. An agent can retrieve from both the right source AND the wrong source, blending information from both. Without negative tests, you will not catch this. Every knowledge-grounded agent with multiple sources should have negative retrieval tests.

### Evaluation Patterns

**Pattern: Cross-Audience Contamination**
Test that the agent does not blend information from sources meant for different audiences. For example, an employee asking about benefits should not receive contractor-specific information, even if the contractor handbook also discusses benefits.

**Pattern: Cross-Region Contamination**
Test that the agent does not mix regional policies. An employee in the US asking about parental leave should not see UK statutory leave entitlements in the response.

**Pattern: Deprecated/Archived Source Avoidance**
Test that the agent does not retrieve from archived or deprecated sources. If you have old versions of documents in your knowledge base (common in SharePoint), verify the agent uses the current version.

**Pattern: Adjacent-Topic Contamination**
Test that the agent does not pull from a source on a related but different topic. For example, a question about "health insurance claims" should not pull from the "life insurance" knowledge source just because both are insurance-related.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | HR policy bot — employee vs. contractor | "What is my health insurance coverage?" (context: full-time employee) | Should NOT retrieve from Contractor-Benefits-Guide.pdf; should retrieve from Employee-Benefits-2025.pdf | Capability Use (All) + Compare Meaning |
| 2 | HR policy bot — US vs. UK policy | "How much parental leave do I get?" (context: US employee) | Response should not contain "statutory", "28 days", or "UK" — terms from the UK policy document | Keyword Match (All) + Compare Meaning |
| 3 | Product FAQ bot — current vs. discontinued | "What are the specifications for Model X?" | Should NOT retrieve from Discontinued-Products-Archive; should retrieve from Current-Product-Catalog | Capability Use (All) + Compare Meaning |
| 4 | Product FAQ bot — product line confusion | "How do I update the firmware on my router?" | Response should not contain instructions for the mesh extender product, which has a different update process | General Quality + Compare Meaning |
| 5 | Legal/compliance bot — jurisdiction scoping | "What are our data breach notification requirements?" (context: California operations) | Response should not include GDPR (EU) notification requirements; should focus on CCPA and California Civil Code | General Quality + Compare Meaning |
| 6 | Legal/compliance bot — old regulation reference | "What is the current fine for non-compliance?" | Should NOT retrieve from the pre-2025 penalty schedule; should retrieve from the 2025-updated regulatory summary | Capability Use (All) + Compare Meaning |
| 7 | HR policy bot — adjacent topic bleed | "How do I file a health insurance claim?" | Response should not include dental insurance claim procedures, even though both are in the benefits knowledge base | General Quality + Compare Meaning |
| 8 | Product FAQ bot — marketing vs. support content | "Why does my device keep disconnecting from WiFi?" | Should NOT retrieve from Marketing-Materials or Sales-Brochures; should retrieve from Technical-Support-KB | Capability Use (All) + Compare Meaning |

### Tips

- Identify your **top 3-5 high-risk knowledge source pairs** and write at least 2 negative tests for each pair. These are your most likely cross-contamination points.
- Set a threshold of **>= 95% pass rate** for negative retrieval tests. Cross-source contamination is a trust-eroding failure that users may not even notice — they just act on wrong information.
- Use **source-specific keywords** as markers. If your US policy says "15 days PTO" and your UK policy says "25 days holiday," those unique terms serve as contamination indicators.
- Test negative retrieval **after every knowledge source addition**. Adding a new source can change retrieval rankings and introduce new contamination risks for existing queries.
- Pay special attention to **archived or versioned documents**. SharePoint in particular may retain old versions that the agent can retrieve from if not properly configured.
- Consider running negative tests as part of a **regression suite** — they protect against retrieval degradation over time.

---
