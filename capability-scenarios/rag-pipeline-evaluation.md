# RAG Pipeline Evaluation

> Scenarios for evaluating the retrieval-augmented generation pipeline itself — chunk-level retrieval quality, query robustness, context relevance, citation accuracy, and behavior under retrieval stress. Applies to any agent that uses generative answers over configured knowledge sources.

[Back to library](../README.md) | [All capability scenarios](README.md)

> **How this relates to Knowledge Grounding & Accuracy:** The [Knowledge Grounding & Accuracy](knowledge-grounding-and-accuracy.md) scenarios test whether the agent retrieves from the correct **source** and stays faithful to it. This file goes deeper into the RAG pipeline — testing whether the retrieval step finds the right **passages**, whether the generation step uses retrieved context effectively, and whether the pipeline holds up under challenging queries. Use both: Knowledge Grounding for source-level correctness, RAG Pipeline Evaluation for pipeline-level quality.

---

## 1. Chunk-Level Retrieval Relevance

### When to Use

Your agent retrieves passages (chunks) from knowledge sources to generate answers. This scenario tests whether the **retrieved chunks actually contain the information needed to answer the question** — not just whether the right document was selected, but whether the right section within that document was found.

Use this scenario when:
- Your knowledge sources are long documents where the answer is in a specific section
- Users ask precise questions that require specific passages, not general document summaries
- You suspect the agent sometimes gives vague or incomplete answers because retrieval found adjacent but not quite right content
- You have adjusted chunking settings (chunk size, overlap) and want to verify retrieval quality

> **Related scenarios:** For verifying the correct document is used, see [Knowledge Grounding — Verifying Retrieval from Correct Sources](knowledge-grounding-and-accuracy.md#1-verifying-retrieval-from-correct-sources). For testing what happens when no chunk has the answer, see [Behavior When Retrieval Returns Low-Relevance Chunks](#4-behavior-when-retrieval-returns-low-relevance-chunks).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes specific details that only appear in the correct passage — not just the correct document |
| Compare Meaning | Confirm the answer reflects the specific passage content, not a general summary of the document |
| Capability Use (All) | If your agent surfaces source references, verify the cited passage is the one that contains the answer |

> **Tip:** The key difference from source-level testing is granularity. A document might have 50 pages. Source-level testing confirms the right document; chunk-level testing confirms the right paragraph. Design test cases where the correct answer is in one specific section and plausible-but-wrong answers exist in other sections of the same document.

### Setup Steps

1. Identify long knowledge source documents where different sections answer different questions.
2. For each document, find 3–5 distinct facts that live in specific, identifiable sections — facts that do not appear elsewhere in the same document.
3. Write test cases targeting each fact. Use **section-unique markers**: details that only appear in one part of the document (a specific number, date, name, or condition).
4. Set expected values to include the section-unique marker.
5. Write distractor test cases: questions where a plausible-sounding answer exists in a different section of the same document but the correct answer requires a specific passage.
6. Run the evaluation. A vague or partially correct answer often indicates the right document was found but the wrong chunk was retrieved.

### Anti-Pattern

> **Anti-Pattern: Testing Only Short Documents**
> If your test documents are one or two pages long, every chunk covers most of the content and chunk-level retrieval is trivially correct. The value of this scenario is in testing retrieval precision within **long, multi-topic documents** — 10+ pages with distinct sections. If all your knowledge sources are short, this scenario adds less value.

### Evaluation Patterns

**Pattern: Section-Unique Marker Verification**
Each test targets a fact found in only one section of a long document. If the response includes the section-unique marker, retrieval found the correct chunk. If the response is vague or uses information from a different section, the wrong chunk was retrieved.

**Pattern: Same-Document Distractor**
Ask a question where Section A has the answer but Section B has related (but different) information. For example, if a policy document has separate sections for full-time and contractor benefits, ask about contractor benefits and verify the answer comes from the contractor section — not the full-time section.

**Pattern: Specificity as a Signal**
When chunk retrieval is working well, answers are specific and detailed. When the wrong chunk is retrieved, answers tend to become vague or generic. Compare the specificity of the response to what the correct passage actually says. A vague answer to a question with a specific answer in the source suggests a retrieval quality issue.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Section-unique marker: specific policy clause | "What is the approval workflow for purchases over $10,000?" (answer is in Section 4.2 of procurement policy; Section 3.1 covers purchases under $10,000) | Response includes "VP-level approval" and "three business days" (details unique to Section 4.2) — not the manager-approval process from Section 3.1 | Keyword Match (All) + Compare Meaning |
| 2 | Same-document distractor: employee type | "What is the probation period for contract employees?" (answer is in Section 7 of HR handbook; Section 2 covers full-time probation periods) | Response includes "60 days" (contract) — not "90 days" (full-time from a different section of the same document) | Keyword Match (All) + Compare Meaning |
| 3 | Specificity signal: detailed vs. vague | "What are the required fields for an international wire transfer request?" (specific list in Section 12 of finance procedures) | Response includes the specific field names from Section 12 — not a generic statement about "required documentation" | Keyword Match (All) + Compare Meaning |
| 4 | Deep retrieval: buried answer | "What is the data retention period for customer support tickets?" (answer is on page 23 of a 40-page compliance document) | Response includes "18 months" (the specific retention period from that section) — not a vague statement about data retention policies | Keyword Match (All) + Compare Meaning |
| 5 | Adjacent chunk: nearby but wrong | "What are the overtime rules for exempt employees?" (answer is in Section 5.3; Section 5.2 covers non-exempt overtime rules) | Response correctly states exempt employees are not eligible for overtime per Section 5.3 — not the overtime calculation formula from Section 5.2 | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 3 chunk-level test cases for each long knowledge source document (10+ pages).
- **Chunk retrieval accuracy threshold:** Target at least 90% — expect slightly lower accuracy than document-level retrieval.
- **Pair with source-level tests.** If a test fails, diagnose whether the failure is at the document level (wrong source) or chunk level (right source, wrong section). Source-level tests from Knowledge Grounding help isolate this.
- **Rerun after:** Changes to chunk size, chunk overlap, embedding models, or document structure. These settings directly affect retrieval quality.
- If chunk retrieval is consistently poor, consider restructuring documents with clearer section headings, or adjusting chunk size to match the granularity of questions users ask.

---

## 2. Query Reformulation Robustness

### When to Use

Different users ask the same question in different ways — using different vocabulary, sentence structure, or level of formality. This scenario tests whether the RAG pipeline retrieves the same (correct) content regardless of how the question is phrased.

Use this scenario when:
- Your user base has diverse vocabulary (e.g., technical vs. non-technical users, different departments)
- You have observed inconsistent answers to semantically identical questions
- Your knowledge sources use specific terminology that users may not use (e.g., the document says "personal time off" but users say "vacation days")
- You want to verify the pipeline handles typos, abbreviations, or colloquial phrasing

> **Related scenarios:** For testing retrieval from the correct source, see [Knowledge Grounding — Verifying Retrieval from Correct Sources](knowledge-grounding-and-accuracy.md#1-verifying-retrieval-from-correct-sources). For testing tone adaptation across user types, see [Tone, Helpfulness & Response Quality](tone-helpfulness-and-response-quality.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify all reformulations produce responses with the same core meaning |
| Keyword Match (All) | Confirm the same key facts appear in responses regardless of query phrasing |
| General Quality | Assess whether responses remain equally helpful and complete across reformulations |

> **Tip:** Write 3–5 reformulations of the same question. Run them as separate test cases with the same expected value. If some reformulations pass and others fail, the retrieval pipeline has a vocabulary gap — it works for certain phrasings but not others.

### Setup Steps

1. Select 5 important questions your agent should handle reliably.
2. For each question, write 3–5 reformulations:
   - **Formal:** "What is the organization's policy regarding remote work arrangements?"
   - **Casual:** "Can I work from home?"
   - **Keyword-style:** "remote work policy"
   - **Synonym-based:** "What are the telecommuting guidelines?"
   - **Misspelled/abbreviated:** "whats the wfh policy"
3. Set the same expected value for all reformulations of a given question.
4. Run the evaluation. Group results by question to see which reformulations succeed and which fail.
5. If certain reformulations consistently fail, this indicates a retrieval vocabulary gap that may need synonym configuration or document enrichment.

### Anti-Pattern

> **Anti-Pattern: Testing Only Clean, Formal Queries**
> If all your test inputs use the same vocabulary as the knowledge source documents, you are testing the easiest case. Real users misspell words, use abbreviations, mix languages, and ask questions in fragments. Include at least one messy, informal reformulation per question group.

### Evaluation Patterns

**Pattern: Vocabulary Gap Detection**
The knowledge source uses term A (e.g., "personal time off") but the user uses term B (e.g., "vacation days"). Test that the agent bridges this gap and retrieves the correct content regardless of which term the user uses.

**Pattern: Abbreviation and Acronym Handling**
Users often use abbreviations ("PTO," "WFH," "OOO") that may not appear in the knowledge sources. Test that the pipeline resolves common abbreviations to their full terms and retrieves appropriately.

**Pattern: Query Complexity Invariance**
A simple "PTO policy?" and a complex "Can you explain the full policy around personal time off including accrual rates, carryover limits, and the request process?" should both retrieve relevant content. Test that query length and complexity do not cause the pipeline to miss relevant chunks.

**Pattern: Consistency Scoring**
Run all reformulations and compare the responses. Calculate a consistency score: what percentage of reformulations produce responses with the same core facts? Low consistency indicates retrieval fragility.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Formal vs. casual: same answer | Formal: "What is the process for submitting a request for paid leave?" / Casual: "How do I ask for time off?" | Both responses include the same core process steps and approval requirements | Compare Meaning + Keyword Match (All) |
| 2 | Vocabulary gap: user term vs. doc term | "What are the telecommuting rules?" (document uses "remote work" not "telecommuting") | Response includes the remote work policy content despite the vocabulary mismatch | Compare Meaning + Keyword Match (Any) |
| 3 | Abbreviation: WFH | "What's the WFH policy?" (document says "work from home" or "remote work," not "WFH") | Response retrieves and presents the remote work policy | Compare Meaning + Keyword Match (Any) |
| 4 | Keyword fragment query | "expense report deadline" (not a full question) | Response includes the expense report submission deadline from the finance policy | Compare Meaning + Keyword Match (All) |
| 5 | Misspelled query | "How do I submitt a reimbursment request?" | Response provides the correct reimbursement submission process despite misspellings | Compare Meaning + Keyword Match (All) |
| 6 | Complex multi-part query | "Can you tell me about PTO — how much do I get, can I carry it over, and what's the approval process?" | Response covers accrual, carryover, and approval — all three parts — from the correct source sections | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 3 reformulations per important question, covering formal, casual, and abbreviated/misspelled variants.
- **Consistency threshold:** Target at least 85% — all reformulations of the same question should produce responses with the same core facts.
- **Focus on high-traffic questions.** Reformulation testing is most valuable for questions users ask frequently, since those are the questions most likely to arrive in unexpected phrasings.
- **Rerun after:** Changes to the retrieval model, embedding configuration, or synonym/keyword settings.
- If you find consistent vocabulary gaps, consider adding synonyms to your knowledge source metadata or enriching documents with alternative terms.

---

## 3. Citation and Passage Attribution Accuracy

### When to Use

Your agent provides citations, references, or source attributions in its responses (e.g., "According to the Employee Handbook, Section 3.2..." or includes source links). This scenario tests whether those citations are **accurate** — pointing to the passage that actually supports the claim.

Use this scenario when:
- Your agent is configured to show source references or citations
- Users rely on citations to verify information or navigate to the original source
- Compliance requirements mandate traceable sourcing for agent responses
- You have observed citations that point to the wrong document or section

> **Related scenarios:** For verifying the agent uses the correct source (without citation testing), see [Knowledge Grounding — Verifying Retrieval from Correct Sources](knowledge-grounding-and-accuracy.md#1-verifying-retrieval-from-correct-sources). For compliance scenarios where citing the exact source is legally required, see [Compliance & Verbatim Content](compliance-and-verbatim-content.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the citation includes the correct source name, section, or URL |
| Compare Meaning | Confirm the cited passage actually supports the claim made in the response |
| General Quality | Assess whether citations are helpful and navigable — can the user find the referenced content? |

> **Tip:** Citation accuracy has two dimensions: (1) Does the citation point to a real, existing source? (2) Does the cited source actually contain the information the agent claims it does? Test both. A citation that points to a real document but the wrong section is as misleading as a fabricated citation.

### Setup Steps

1. Identify 5–8 questions where your agent should cite its sources.
2. For each question, note the **exact source** (document name, section, page, or URL) that contains the answer.
3. Write test cases that check whether the agent's citation matches the actual source.
4. For each test case, set expected values that include the correct source identifier (document title, section number, URL path).
5. Run the evaluation. Categorize citation failures as: (a) no citation provided, (b) citation points to wrong source, (c) citation points to right source but wrong section, (d) citation is fabricated (source does not exist).

### Anti-Pattern

> **Anti-Pattern: Accepting Any Citation as Correct**
> If your test only checks whether a citation is present — not whether it is accurate — you are not testing citation quality. An agent that always cites "Company Policy Document" has high citation rate but potentially zero citation accuracy. Verify the specific source, section, or page.

### Evaluation Patterns

**Pattern: Source-Claim Alignment**
For each claim in the agent's response, verify that the cited source actually contains that claim. The citation should support the specific statement, not just be tangentially related to the topic.

**Pattern: Citation Completeness**
When the response draws from multiple sources, verify that all sources are cited — not just the primary one. Missing citations for secondary sources are a common gap.

**Pattern: Fabricated Citation Detection**
Test whether any cited source names, URLs, or section numbers do not actually exist. This is a form of hallucination specific to citations — the agent invents a plausible-sounding reference.

**Pattern: Citation Granularity**
A citation that says "Employee Handbook" is less useful than one that says "Employee Handbook, Section 3.2 — Leave Policies." Test whether citations are specific enough for the user to locate the referenced information.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Source-claim alignment | "What is the expense approval threshold?" (answer is in Finance Policy, Section 5) | Citation references Finance Policy Section 5 (or equivalent) — not the general company handbook | Keyword Match (All) + Compare Meaning |
| 2 | Multi-source citation | "Compare the health insurance options." (Plan A in Benefits Guide, Plan B in Insurance Summary) | Response cites both the Benefits Guide and Insurance Summary — not just one | Keyword Match (All) + General Quality |
| 3 | Fabricated citation check | "What is the policy on conference attendance?" (covered in Professional Development Guide) | Citation references an actual document — not a fabricated or non-existent source name | Keyword Match (Any) + Compare Meaning |
| 4 | Citation granularity | "How do I submit a grievance?" (process in HR Handbook, Section 9.4) | Citation is specific enough to locate the content (section or page level, not just document name) | General Quality + Keyword Match (Any) |
| 5 | Wrong-section citation | "What is the dress code for client meetings?" (answer in Brand Standards Guide, Section 2; company dress code is in Employee Handbook, Section 6) | Citation references Brand Standards Guide — not Employee Handbook, which covers general dress code | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 5 citation accuracy test cases, including at least 1 multi-source and 1 fabricated-citation check.
- **Citation accuracy threshold:** Target at least 90% for source-level accuracy and 80% for section-level accuracy.
- **Check both directions:** the citation supports the claim (forward check), and the claim is in the cited source (backward check).
- **Rerun after:** Changes to knowledge sources, source naming conventions, or citation display configuration.
- If citation accuracy is consistently low, check whether the retrieval pipeline surfaces source metadata correctly and whether the generation prompt instructs the agent to include attribution.

---

## 4. Behavior When Retrieval Returns Low-Relevance Chunks

### When to Use

Sometimes the retrieval step finds content that is only **tangentially related** to the query — the retrieved chunks are not irrelevant enough to trigger a "no information found" response, but not relevant enough to answer the question well. This scenario tests how the agent handles this gray zone.

Use this scenario when:
- Your knowledge sources cover a broad domain where many chunks are tangentially related to any given query
- You have observed the agent giving answers that are technically sourced but do not actually address the user's question
- Users report answers that "sort of" address their question but miss the point
- You want to test the threshold between "answer from context" and "acknowledge the gap"

> **Related scenarios:** For testing when no source has the answer at all, see [Knowledge Grounding — Testing Handling of Missing Sources](knowledge-grounding-and-accuracy.md#2-testing-handling-of-missing-sources). For testing hallucination when the agent tries to fill gaps, see [Knowledge Grounding — Testing Hallucination Prevention](knowledge-grounding-and-accuracy.md#4-testing-hallucination-prevention).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the response actually addresses the user's specific question — not just a related topic |
| General Quality | Assess whether the response is helpful and relevant, or whether it "talks around" the question |
| Keyword Match (Any) | Check for appropriate hedging language when the available context does not fully answer the question |

> **Tip:** The failure mode here is subtle — the agent provides an answer that is sourced, grounded, and non-hallucinated, but does not address what the user actually asked. A question about "parental leave for adoptive parents" answered with general parental leave information is technically grounded but misses the point. Use Compare Meaning with expected values that emphasize the **specific** aspect of the question.

### Setup Steps

1. Identify questions where your knowledge sources have **related but not directly relevant** content. These are questions where:
   - The topic is covered but the specific angle is not
   - A general policy exists but the user asks about a specific exception
   - The terminology is similar but the context is different
2. Write test cases with expected values that describe the **ideal behavior**: the agent should either (a) provide the related information with a clear caveat that it may not fully address the specific question, or (b) acknowledge the gap and suggest where to find the specific answer.
3. Select **Compare Meaning** as the primary method — you are checking whether the response addresses the specific question, not just the general topic.
4. Add **General Quality** to assess overall helpfulness.
5. Run the evaluation. Flag responses that answer a related but different question without acknowledging the mismatch.

### Anti-Pattern

> **Anti-Pattern: Treating Tangentially Related Answers as Correct**
> If the user asks "Does parental leave apply to surrogacy?" and the agent responds with the general parental leave policy without addressing surrogacy, the test should not pass just because the response contains accurate parental leave information. The question has a specific focus; a generic answer that ignores that focus is a retrieval quality failure.

### Evaluation Patterns

**Pattern: Specificity Match**
The response should match the specificity of the question. A specific question deserves a specific answer. If the retrieval pipeline can only find general content, the response should acknowledge the gap between what was asked and what is available.

**Pattern: Transparent Hedging**
When retrieved chunks are only partially relevant, the ideal response uses appropriate hedging: "Based on the available information about parental leave in general..." rather than presenting general information as if it fully answers the specific question.

**Pattern: Redirect to Better Source**
If the agent recognizes that its retrieved content does not fully address the question, it should suggest where the user might find the specific answer (e.g., "For details specific to surrogacy, I'd recommend contacting HR directly").

**Pattern: No Silent Topic Drift**
The agent should not silently shift from the user's specific question to a related but different topic. If the question is about contractor benefits and the retrieval only finds employee benefits, the response should not present employee benefits as if they apply to contractors.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Specific exception: general policy exists | "Does the parental leave policy cover adoption through foster care?" (source covers parental leave generally but not foster care adoption specifically) | Response provides general parental leave information with a caveat that the specific foster care adoption scenario is not explicitly covered, and suggests contacting HR | Compare Meaning + General Quality |
| 2 | Related topic: different audience | "What is the IT equipment policy for interns?" (source covers IT equipment for full-time employees only) | Response does NOT present the full-time employee policy as if it applies to interns; acknowledges the gap or clarifies the available policy's scope | Compare Meaning + Keyword Match (Any) |
| 3 | Similar terminology: different context | "What is the escalation process for customer complaints?" (source covers internal IT escalation, not customer complaint escalation) | Response does NOT present the IT escalation process as the customer complaint process; acknowledges it has information about IT escalation but not customer complaints | Compare Meaning + General Quality |
| 4 | Partial coverage: missing detail | "What is the reimbursement rate for mileage when using a personal vehicle for business travel?" (source covers general travel reimbursement but not per-mile rates) | Response provides general travel reimbursement information with a note that specific per-mile rates are not available in the current sources | Compare Meaning + Keyword Match (Any) |
| 5 | No silent topic drift | "What are the remote work guidelines for the London office?" (source only covers US office remote work) | Response clarifies that available information covers US office guidelines and does not present them as applicable to the London office | Compare Meaning + General Quality |

### Tips

- **Coverage target:** At least 4 low-relevance retrieval test cases, covering specific-exception, wrong-audience, and partial-coverage patterns.
- **Relevance alignment threshold:** Target at least 80% — the agent should recognize and acknowledge the gap between the question and the available content.
- **These tests expose retrieval configuration issues.** If the agent consistently treats tangentially related content as fully relevant, the retrieval relevance threshold may be set too low.
- **Rerun after:** Changes to retrieval confidence thresholds, relevance scoring settings, or knowledge source scope.
- Pair with [Graceful Failure & Escalation](graceful-failure-and-escalation.md) scenarios — low-relevance retrieval is a common trigger for escalation.

---

## 5. Multi-Turn Retrieval Consistency

### When to Use

Your agent handles conversations that span **multiple turns**, where follow-up questions build on previous context. This scenario tests whether the RAG pipeline maintains retrieval quality across a conversation — retrieving relevant content for follow-ups that reference earlier context.

Use this scenario when:
- Users frequently ask follow-up questions ("What about for part-time employees?" after asking about PTO)
- Your agent uses conversational context to resolve ambiguous follow-up queries
- You have observed the agent "forgetting" context in later turns and retrieving irrelevant content
- Follow-up questions use pronouns or references that depend on earlier turns ("What about that?" "And the deadline?")

> **Related scenarios:** For multi-step process guidance, see [Process Navigation & Multi-Step Guidance](../business-problem-scenarios/process-navigation-and-multistep-guidance.md). For testing the agent's ability to route follow-ups to the right topic, see [Trigger Routing](trigger-routing.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify follow-up responses correctly interpret the conversational context and retrieve relevant content |
| Keyword Match (All) | Confirm follow-up responses include specific facts from the correct source section |
| General Quality | Assess whether the conversation flows naturally and each response builds on prior context |

> **Tip:** The most common failure in multi-turn retrieval is the **context collapse** — the follow-up question loses context from the previous turn, and the retrieval pipeline treats it as an independent query. "What about for contractors?" makes no sense without the preceding turn. Test that the pipeline resolves these references correctly.

### Setup Steps

1. Design 3–5 multi-turn conversation sequences. Each sequence should have 2–4 turns where later turns build on earlier context.
2. For each follow-up turn, determine what the correct retrieval target is given the full conversation context.
3. Write test cases for each turn in the sequence. The expected value for follow-up turns should reflect the correct interpretation given conversational context.
4. Include at least one sequence where the follow-up changes the retrieval target (e.g., Turn 1 asks about policy X, Turn 2 asks "How does that compare to policy Y?" — requiring retrieval of both X and Y).
5. Run the evaluation turn by turn. Track whether retrieval quality degrades in later turns.

### Anti-Pattern

> **Anti-Pattern: Testing Only First-Turn Queries**
> If all your test cases are single-turn questions, you are testing the easiest retrieval scenario. Multi-turn conversations introduce referential ambiguity ("that," "it," "the same thing for...") that can confuse the retrieval pipeline. Real users do not restart the conversation for each question — they build on prior context.

### Evaluation Patterns

**Pattern: Pronoun and Reference Resolution**
Follow-up questions often use pronouns ("Does that apply to managers too?") or references ("What about the deadline for that?"). Test that the pipeline resolves these to the correct topic from the conversation history and retrieves the right content.

**Pattern: Topic Shift Within Conversation**
A user starts with one topic and pivots to a related but different one. Test that the pipeline recognizes the shift and retrieves new content rather than continuing to use chunks from the first topic.

**Pattern: Consistency Across Turns**
Information provided in Turn 1 should not contradict information in Turn 3 of the same conversation. Test that the agent maintains factual consistency even as it retrieves different chunks for different questions.

**Pattern: Context Carryover**
When a follow-up narrows the original question (e.g., Turn 1: "What is the leave policy?" Turn 2: "Specifically for bereavement?"), test that the response narrows appropriately — providing bereavement-specific details, not repeating the general policy.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Pronoun resolution | Turn 1: "What is the PTO accrual rate?" / Turn 2: "Does it change after 5 years?" | Turn 2 response addresses PTO accrual rate changes at the 5-year mark — not general information about 5-year milestones | Compare Meaning + Keyword Match (All) |
| 2 | Narrowing follow-up | Turn 1: "What are the company benefits?" / Turn 2: "Tell me more about the dental coverage." | Turn 2 provides specific dental coverage details from the benefits documentation — not a repeat of the general benefits summary | Compare Meaning + Keyword Match (All) |
| 3 | Topic shift | Turn 1: "How do I submit an expense report?" / Turn 2: "Actually, can you explain the travel approval process instead?" | Turn 2 retrieves travel approval content — not continuing with expense report information | Compare Meaning + Keyword Match (All) |
| 4 | Comparison requiring both turns | Turn 1: "What is the standard PTO policy?" / Turn 2: "How does that compare for part-time employees?" | Turn 2 compares full-time and part-time PTO policies, showing understanding of both | Compare Meaning + Keyword Match (Any) |
| 5 | Consistency across turns | Turn 1: "What is the return window for online orders?" / Turn 3 (after unrelated Turn 2): "Going back to the return window — are there exceptions?" | Turn 3 correctly references the return window from Turn 1 and adds exception details from the correct source | Compare Meaning + General Quality |

### Tips

- **Coverage target:** At least 3 multi-turn sequences, each with 2–4 turns.
- **Multi-turn consistency threshold:** Target at least 85% — expect slightly lower accuracy than single-turn because of the added complexity of reference resolution.
- **Test both narrowing and pivoting patterns** — they stress different parts of the pipeline.
- **Rerun after:** Changes to conversation history handling, context window settings, or the number of prior turns the pipeline considers.
- If multi-turn retrieval is consistently poor, check whether the pipeline is sending the full conversation context (not just the latest turn) to the retrieval model.

---

## 6. Retrieval Under Stress: Long Queries and Knowledge Base Scale

### When to Use

This scenario tests RAG pipeline behavior under conditions that stress the retrieval system — very long or multi-part queries, large knowledge bases with many similar documents, and high-ambiguity questions where multiple chunks seem equally relevant.

Use this scenario when:
- Your knowledge base is large (hundreds of documents or more)
- Users sometimes paste long text or ask multi-part questions in a single message
- You have multiple documents covering similar topics (e.g., policies from different years, departments, or regions)
- You want to verify the pipeline scales gracefully as content volume grows

> **Related scenarios:** For testing multi-source synthesis, see [Knowledge Grounding — Verifying Multi-Source Synthesis](knowledge-grounding-and-accuracy.md#5-verifying-multi-source-synthesis). For testing behavior with conflicting sources, see [Knowledge Grounding — Validating Behavior with Conflicting Sources](knowledge-grounding-and-accuracy.md#3-validating-behavior-with-conflicting-sources).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the response addresses all parts of a multi-part query |
| Keyword Match (All) | Confirm specific facts from the most relevant source appear — not blended information from similar but incorrect sources |
| General Quality | Assess whether the response remains coherent and well-organized under stress conditions |

> **Tip:** Retrieval stress failures are often intermittent — the pipeline works most of the time but occasionally retrieves the wrong content when the knowledge base has many similar documents. Run stress tests multiple times (3–5 runs per test case) and check for consistency. A test that passes 80% of the time has a retrieval reliability issue.

### Setup Steps

1. Identify stress conditions relevant to your agent:
   - **Multi-part queries:** Single messages containing 2–4 distinct questions
   - **Similar documents:** Knowledge base sections with overlapping content (e.g., 2024 vs. 2025 policies)
   - **Long queries:** User messages that include pasted text, background context, or detailed scenarios
2. Write test cases for each stress condition with clear expected values.
3. For multi-part queries, expected values should cover all parts — a response that addresses only part of the query is a partial failure.
4. For similar-document stress, expected values should include markers that distinguish the correct document from similar ones (version numbers, dates, department names).
5. Run each test case 3–5 times to check for consistency. Intermittent failures indicate retrieval instability.

### Anti-Pattern

> **Anti-Pattern: Testing Only Simple, Single-Topic Queries**
> If your test cases are all short, single-topic questions against a small knowledge base, you are not stress-testing the pipeline. Retrieval quality can degrade significantly as query complexity and knowledge base size increase. Include test cases that push the boundaries.

### Evaluation Patterns

**Pattern: Multi-Part Query Coverage**
A single user message asks about multiple topics. Test that the response addresses all parts, not just the first or most prominent one. Common failure: the pipeline retrieves chunks for the first question and ignores the rest.

**Pattern: Version Disambiguation**
When multiple versions of a document exist (2024 policy vs. 2025 policy), test that the pipeline retrieves from the correct version. This is especially important when the user does not specify which version they need.

**Pattern: Consistency Under Repetition**
Run the same test case multiple times and check whether the response is consistent. Inconsistency suggests the retrieval scoring is close to a tipping point where small variations (in query embedding, for example) change which chunks are retrieved.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Multi-part query | "I need to know three things: (1) the PTO accrual rate, (2) the process for requesting bereavement leave, and (3) whether I can donate PTO to a colleague." | Response addresses all three parts with specific facts from the correct sources | Keyword Match (All) + Compare Meaning |
| 2 | Version disambiguation | "What is the current travel reimbursement rate?" (2024 and 2025 travel policies both exist with different rates) | Response uses the 2025 rate — not the 2024 rate or a blend of both | Keyword Match (All) + Compare Meaning |
| 3 | Similar documents: different departments | "What is the overtime policy?" (separate policies for engineering, sales, and operations) | Response asks for clarification or provides the general company policy — does not silently pick one department's policy | General Quality + Compare Meaning |
| 4 | Long query with context | "I'm a new employee who started three weeks ago in the Seattle office. I'm planning to take two days off next month for a family event. I'm a full-time employee on the standard benefits plan. What do I need to do to request this time off and am I eligible?" | Response correctly addresses eligibility (considering new employee status) and the request process — does not ignore the tenure context | Compare Meaning + Keyword Match (Any) |
| 5 | Consistency check | Same question run 3+ times: "What are the current health insurance deductibles?" | All runs produce the same deductible amounts from the same source | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 2 multi-part queries, 2 version/similar-document tests, and 1 consistency check.
- **Multi-part coverage threshold:** Target at least 85% — the response should address all parts, not just some.
- **Run consistency checks 3–5 times** per test case. If the response varies, investigate the retrieval scoring.
- **Rerun after:** Knowledge base expansion (new documents added), configuration changes to top-k retrieval settings, or changes to relevance thresholds.
- If version disambiguation fails consistently, consider adding explicit date or version metadata to your knowledge sources and configuring the retrieval pipeline to prefer the most recent version.
