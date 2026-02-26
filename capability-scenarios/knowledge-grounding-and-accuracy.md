# Knowledge Grounding & Accuracy

> Scenarios for validating that your agent retrieves information from the correct knowledge sources and does not hallucinate or fabricate answers. Applies to any agent that uses configured data sources (SharePoint, uploaded documents, Dataverse, websites).

[Back to library](../README.md) | [All capability scenarios](README.md)

---

## 1. Verifying Retrieval from Correct Sources

### When to Use

Your agent has one or more configured knowledge sources and must retrieve information from the **right** source for each question. This is the foundational knowledge grounding scenario — before you test for hallucination or conflicting sources, you must first confirm the agent is pulling from the correct place.

Use this scenario when:
- Your agent has multiple knowledge sources (e.g., an HR policy SharePoint site, a product documentation library, and a company FAQ page)
- Different questions should be answered from different sources
- You need to verify the agent does not pull answers from the wrong source (e.g., answering a benefits question using the IT troubleshooting guide)

> **Related scenarios:** For testing what happens when no source has the answer, see [Testing Handling of Missing Sources](#2-testing-handling-of-missing-sources). For verifying the agent does not fabricate answers, see [Testing Hallucination Prevention](#4-testing-hallucination-prevention). For business-level answer accuracy, see [Information Retrieval & Policy Q&A](../business-problem-scenarios/information-retrieval-and-policy-qa.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes specific terms, phrases, or data points that only exist in the correct source |
| Compare Meaning | Confirm the answer's meaning aligns with the content of the correct source document |
| Capability Use (All) | Verify the agent invokes the correct knowledge source or data connector |

> **Tip:** The strongest test for correct source retrieval uses **source-unique markers** — facts, terms, or values that appear in only one source. If the answer includes a data point only found in Document A, you know it retrieved from Document A.

### Setup Steps

1. Inventory your agent's configured knowledge sources and note what content each contains.
2. For each knowledge source, identify **source-unique markers**: facts, terminology, numbers, or phrases that exist only in that source and nowhere else.
3. Write test cases where the correct answer requires retrieving from a specific source. Use questions where you know exactly which source contains the answer.
4. Set the expected value to include the source-unique marker (for Keyword Match) or the source-specific content (for Compare Meaning).
5. If your agent supports source attribution or citation, add test cases verifying the cited source is correct.
6. Run the evaluation. Any test case where the agent answers correctly but from the wrong source is a grounding failure — even if the user would not notice.

### Anti-Pattern

> **Anti-Pattern: Testing Only Answer Correctness**
> An agent can give the correct answer from the wrong source. If your benefits policy and your general FAQ both mention "15 days of PTO," a correct answer does not prove the agent used the right source. Always include source-unique markers in your expected values so you can distinguish between right-answer-right-source and right-answer-wrong-source.

### Evaluation Patterns

**Pattern: Source-Unique Marker Verification**
Each test case targets content that exists in only one knowledge source. The expected value includes a term or data point unique to that source. This is the most reliable way to verify correct retrieval.

**Pattern: Source Attribution Verification**
If your agent cites its sources (e.g., "According to the Employee Handbook..." or provides a link), verify the citation matches the source that actually contains the information. Test cases where the agent names or links the source, with expected values checking the source name.

**Pattern: Cross-Source Interference**
Test inputs where a plausible-sounding answer could come from the wrong source. For example, if your product FAQ and your internal engineering wiki both discuss product features but with different levels of detail, verify the customer-facing agent pulls from the FAQ — not the engineering wiki.

**Pattern: Source Priority Verification**
If your agent has rules about which source takes precedence (e.g., "always prefer the most recently updated document"), test scenarios where multiple sources could answer the question and verify the agent uses the prioritized source.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Source-unique marker: policy document | "What is the approval threshold for travel expenses?" | Response includes "$2,500" (a value that only appears in the Travel Policy document, not in the general FAQ) | Keyword Match (All) + Compare Meaning |
| 2 | Source-unique marker: product docs | "What file formats does the export feature support?" | Response includes "CSV, JSON, XML, and Parquet" (the full list only appears in the technical documentation) | Keyword Match (All) + Compare Meaning |
| 3 | Source attribution check | "Where can I find the company holiday schedule?" | Response meaning aligns with content from the HR SharePoint site and attributes the information to that source | Compare Meaning + Keyword Match (Any) |
| 4 | Cross-source interference | "How do I reset my password?" | Agent retrieves from the IT Self-Service knowledge base — not the developer documentation that also mentions password resets in a different context | Capability Use (All) + Compare Meaning |
| 5 | Source priority: newer vs. older | "What are the current office hours?" | Response includes "8:00 AM to 6:00 PM" (from the updated 2025 policy, not "9:00 AM to 5:00 PM" from the older version) | Keyword Match (All) + Compare Meaning |
| 6 | Multiple sources needed, primary verified | "What is our return policy for international orders?" | Response meaning aligns with the international shipping policy document — not the domestic return policy | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 2 test cases per configured knowledge source, each using a source-unique marker.
- **Source accuracy threshold:** Target at least 95% correct source retrieval. Even one cross-source error may indicate a configuration issue.
- **Identify source-unique markers before writing tests** — this requires reading your actual knowledge source content, not guessing.
- **Rerun after:** Any changes to knowledge source configuration, document updates, or source priority rules.
- If your agent has many sources, prioritize testing the pairs most likely to be confused (sources covering similar topics).

---

## 2. Testing Handling of Missing Sources

### When to Use

The user asks a question that **no configured knowledge source can answer**. This scenario tests whether the agent acknowledges the gap honestly rather than fabricating an answer or pulling from an irrelevant source.

Use this scenario when:
- Your agent has a defined scope of knowledge and users may ask questions outside it
- You need confidence that the agent says "I don't have information about that" rather than guessing
- Incorrect answers to out-of-scope questions could cause real harm (financial, legal, safety)

> **Related scenarios:** For verifying the agent retrieves from the correct source when one exists, see [Verifying Retrieval from Correct Sources](#1-verifying-retrieval-from-correct-sources). For testing fabrication prevention specifically, see [Testing Hallucination Prevention](#4-testing-hallucination-prevention). For testing escalation when the agent cannot help, see [Graceful Failure & Escalation](../capability-scenarios/graceful-failure-and-escalation.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's response acknowledges it does not have the information rather than guessing |
| Keyword Match (Any) | Check for appropriate language indicating uncertainty or scope limitation (e.g., "I don't have," "not available," "I'm unable to find") |
| General Quality | Assess whether the response is helpful despite not answering — does it suggest alternatives, offer to escalate, or redirect? |

> **Tip:** A good "I don't know" response is not just an admission of ignorance — it should be helpful. The agent should acknowledge the gap, avoid guessing, and ideally point the user toward an alternative resource. Use General Quality to evaluate this holistic helpfulness.

### Setup Steps

1. Identify topics that are clearly **outside** your agent's configured knowledge sources. These should be topics your users might realistically ask about.
2. Write test cases for each out-of-scope topic. Use questions that sound reasonable and in-domain but are not covered by any source.
3. Set expected values to describe the appropriate response: acknowledgment that the information is not available, absence of fabricated content, and ideally a suggestion for where to find the answer.
4. Select **Compare Meaning** as the primary method — you are checking the response's intent (acknowledging the gap), not specific wording.
5. Add **Keyword Match (Any)** to verify the presence of uncertainty language.
6. Add **General Quality** test cases for responses where you want to evaluate the overall helpfulness of the "I don't know" response.
7. Run the evaluation. Any test case where the agent provides a confident, specific answer to an out-of-scope question is a failure.

### Anti-Pattern

> **Anti-Pattern: Testing Only Wildly Off-Topic Questions**
> If your out-of-scope test cases are questions like "What is the meaning of life?" or "Tell me a joke," you are testing general boundary enforcement, not knowledge gap handling. The most important "missing source" test cases are questions that are **close to the agent's domain but not covered by any source** — questions a user might reasonably expect the agent to answer. "What is the parental leave policy for contractors?" when your sources only cover full-time employees is a much more revealing test than "What is the weather today?"

### Evaluation Patterns

**Pattern: In-Domain But Uncovered**
Test questions that are within the agent's general domain but not covered by any configured source. These are the most likely to produce hallucination because the agent has enough context to fabricate a plausible-sounding answer.

**Pattern: Adjacent-Topic Gap**
Test questions about topics adjacent to what the agent knows. If the agent has product documentation for Product A, ask about Product B. If it has a US employee handbook, ask about UK policies. These test whether the agent recognizes the boundary of its knowledge.

**Pattern: Helpful Decline**
When the agent correctly identifies that it does not have the information, evaluate whether the response is still helpful — does it suggest where to find the answer, offer to connect with someone who can help, or clarify what it can help with?

**Pattern: No Confident Fabrication**
The critical failure mode is when the agent does not just guess but presents a fabricated answer with confidence — no hedging, no caveats, no mention that the information might not be accurate. Test that the agent does not cross this line.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | In-domain but uncovered: policy gap | "What is the company policy on bringing pets to the office?" (no pet policy exists in any source) | Response meaning aligns with: "The agent acknowledges it does not have information about a pet policy and does not fabricate one." | Compare Meaning + Keyword Match (Any) |
| 2 | Adjacent-topic gap: wrong region | "What are the public holidays for the UK office?" (sources only contain US holiday calendar) | Response includes language like "don't have," "not available," or "unable to find" | Keyword Match (Any) + Compare Meaning |
| 3 | Adjacent-topic gap: wrong product | "Does Product X support integration with Tableau?" (documentation only covers Product Y) | Response meaning aligns with: "The agent indicates it does not have specific information about Product X's Tableau integration and does not guess." | Compare Meaning + Keyword Match (Any) |
| 4 | Helpful decline: suggests alternative | "What are the tax implications of my stock options?" (no tax guidance in any source) | Response acknowledges the gap AND suggests consulting a tax professional or HR benefits team | General Quality + Compare Meaning |
| 5 | Close to domain: recent change | "What is the updated dress code policy?" (sources contain the old policy but no updated version exists) | Agent either provides the existing policy with a note that it may not reflect recent changes, or acknowledges it does not have updated information | General Quality + Compare Meaning |
| 6 | Fabrication trap: specific numbers | "How many vacation days do interns get?" (intern policies not in any source) | Agent does NOT provide a specific number of days; acknowledges it lacks this information | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 5 out-of-scope test cases, with a mix of in-domain-but-uncovered and adjacent-topic-gap patterns.
- **Gap acknowledgment threshold:** Target 100% — the agent should never confidently answer a question when no source supports the answer.
- **Focus on near-miss topics** — the closer the question is to the agent's actual domain, the more likely hallucination becomes.
- **Rerun after:** Any changes to knowledge sources, source scope, or agent instructions about handling unknown topics.
- A response that says "Based on general knowledge..." or "Typically, companies..." when the agent should only use configured sources is a failure — even if the general information is correct.

---

## 3. Validating Behavior with Conflicting Sources

### When to Use

Your agent has multiple knowledge sources that contain **contradictory information** about the same topic. This can happen when documents have different publication dates, different authors, or different scopes — or when a policy has been updated in one source but not another.

Use this scenario when:
- Your agent draws from multiple documents or data sources
- Your content is maintained by different teams who may not coordinate updates
- You have found (or suspect) inconsistencies across your knowledge base
- Users have reported receiving different answers to the same question at different times

> **Related scenarios:** For verifying the agent retrieves from the correct source, see [Verifying Retrieval from Correct Sources](#1-verifying-retrieval-from-correct-sources). For testing source priority rules, see that scenario's "Source Priority Verification" pattern. For compliance-critical content where the wrong version could have legal consequences, see [Compliance & Verbatim Content](../capability-scenarios/compliance-and-verbatim-content.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's response handles the conflict appropriately — uses the correct source, acknowledges the discrepancy, or explains the difference |
| General Quality | Assess whether the response is transparent about the conflict and guides the user appropriately |
| Keyword Match (All) | Verify the response includes the value from the correct (authoritative or most recent) source |

> **Tip:** There are multiple valid ways to handle conflicting sources: prefer the most recent document, defer to the authoritative source, surface both and let the user decide, or acknowledge the discrepancy and recommend verification. Your expected value should reflect your agent's designed behavior — there is no universal right answer.

### Setup Steps

1. Identify known conflicts in your knowledge sources. If none are known, create test scenarios based on plausible conflicts (e.g., "Source A says 15 days PTO, Source B says 20 days PTO").
2. For each conflict, determine what the **correct agent behavior** is: use the newer source? Use the more authoritative source? Acknowledge the discrepancy?
3. Write test cases that ask questions where the conflicting sources give different answers.
4. Set expected values based on your designed conflict resolution behavior.
5. Select **Compare Meaning** as the primary method — you are checking how the agent handles the conflict, not just which answer it gives.
6. Add **General Quality** tests to evaluate whether the agent is transparent about uncertainty when sources conflict.
7. Run the evaluation. Pay attention to whether the agent silently picks one source without acknowledging the conflict.

### Anti-Pattern

> **Anti-Pattern: Expecting the Agent to Merge Conflicting Information**
> If two sources disagree on a specific fact (e.g., different PTO allowances), the agent should not blend them into a compromise answer ("PTO is approximately 15–20 days"). This creates a response grounded in no actual source. The agent should either choose the authoritative source, present both with context, or flag the discrepancy — never average or merge conflicting facts.

### Evaluation Patterns

**Pattern: Recency-Based Resolution**
When your agent is configured to prefer the most recent source, test that it does so consistently. Ask questions where the older and newer source give different answers and verify the agent uses the newer one.

**Pattern: Authority-Based Resolution**
When your agent is configured to prefer a specific source over others (e.g., the official policy document over a department FAQ), test that it defers to the authoritative source when there is a conflict.

**Pattern: Discrepancy Acknowledgment**
For agents designed to surface conflicts transparently, test that the agent mentions both sources and the discrepancy rather than silently picking one. The response should help the user understand that the information may vary and where to verify.

**Pattern: No Silent Merge**
Verify the agent does not create a blended answer that does not match any source. This is a subtle form of hallucination — the agent is not fabricating from nothing, but it is fabricating a synthesized answer that no source actually states.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Recency-based: updated policy vs. old | "How many vacation days do employees get per year?" (old policy says 15, new policy says 20) | Response includes "20" from the updated policy — not "15" from the outdated one | Keyword Match (All) + Compare Meaning |
| 2 | Authority-based: official vs. informal | "Can I work remotely on Fridays?" (official HR policy says "with manager approval," department FAQ says "yes, Fridays are remote days") | Response meaning aligns with the official HR policy requiring manager approval | Compare Meaning + Keyword Match (Any) |
| 3 | Discrepancy acknowledgment | "What is the deadline for submitting expense reports?" (Source A says 30 days, Source B says 45 days) | Response acknowledges the discrepancy or recommends the user verify with their manager/finance team | General Quality + Compare Meaning |
| 4 | No silent merge | "What is the maximum reimbursement for professional development?" (one source says $2,000, another says $3,000) | Agent provides one specific value from the authoritative source OR acknowledges the discrepancy — does NOT say "$2,000–$3,000" or "$2,500" | Compare Meaning + Keyword Match (Any) |
| 5 | Scope-based conflict: different audiences | "What is the return window for products?" (consumer policy says 30 days, enterprise agreement says 90 days) | Response asks the user which context applies or explains both with clear distinction | General Quality + Compare Meaning |
| 6 | Date-sensitive conflict | "What are the enrollment dates for benefits?" (2024 document says October, 2025 document says November) | Response uses the 2025 enrollment dates | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 3 conflict scenarios, covering recency-based, authority-based, and genuine ambiguity patterns.
- **Conflict resolution consistency:** The agent should handle conflicts the same way every time — if it is configured to prefer recency, it should always prefer recency. Target 100% consistency.
- **Audit your knowledge sources** for conflicts before writing tests. Real conflicts produce more valuable test cases than hypothetical ones.
- **Rerun after:** Any knowledge source updates, additions, or removals — these are the most likely times for new conflicts to appear.
- If your agent cannot reliably handle conflicting sources, consider cleaning up the sources rather than tuning the agent.

---

## 4. Testing Hallucination Prevention

### When to Use

You need to verify that your agent does **not fabricate information** that is not present in any of its configured knowledge sources. This includes inventing facts, generating plausible-sounding but unsupported details, or extrapolating beyond what the sources actually state.

Use this scenario when:
- Your agent answers questions using configured knowledge sources and should not go beyond them
- The consequences of fabricated information are significant (financial guidance, legal advice, safety procedures, medical information)
- You have observed or suspect the agent sometimes adds details not present in any source
- You want to verify the agent stays grounded across a range of question types

> **Related scenarios:** For verifying the agent retrieves from the correct source, see [Verifying Retrieval from Correct Sources](#1-verifying-retrieval-from-correct-sources). For testing what the agent does when no source exists, see [Testing Handling of Missing Sources](#2-testing-handling-of-missing-sources). For explicit tests designed to elicit hallucination, see [Negative Test: Ungrounded Claims](#6-negative-test-ungrounded-claims).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent's response aligns with and does not exceed what the source documents state |
| Keyword Match (All) | Check that specific facts in the response (numbers, dates, names) match the source content exactly |
| General Quality | Assess whether the response stays within the bounds of the available information without adding unsupported claims |

> **Tip:** Hallucination testing is not just about wildly fabricated answers — the most dangerous hallucinations are **plausible additions** that sound correct and are close to what the source says but include details the source does not actually contain. A response that says "The policy allows 15 days of PTO and unused days roll over to the next year" when the source only mentions the 15 days is a hallucination — even though rollover policies are common.

### Setup Steps

1. Select 5–8 questions your agent should be able to answer from its configured sources.
2. For each question, read the actual source content carefully and note **exactly** what the source says — no more, no less.
3. Write test cases with expected values that reflect the source content precisely, including specific numbers, dates, conditions, and qualifications.
4. Add expected values that explicitly call out what the source does NOT say — details the agent might be tempted to add.
5. Select **Compare Meaning** to verify the response aligns with source content. Layer with **Keyword Match (All)** for factual details.
6. Run the evaluation. Flag any response that includes specific facts, numbers, conditions, or qualifications not present in the source.

### Anti-Pattern

> **Anti-Pattern: Checking Only for Correct Information**
> If your expected value is "The response should mention 15 days of PTO" and the agent responds "Employees receive 15 days of PTO, which can be rolled over for up to two years, with a maximum accrual of 30 days," the test passes — but the agent hallucinated the rollover and accrual details. Your expected values must check for both the **presence of correct information** and the **absence of fabricated additions**.

### Evaluation Patterns

**Pattern: Source-Faithful Response**
The agent's response should convey the information from the source without adding, embellishing, or extrapolating. Compare the response to the actual source content line by line. Additions that are not in the source are hallucinations, even if they are factually plausible.

**Pattern: Numerical Precision**
Numbers are the most verifiable type of hallucination. If the source says "up to $500," the agent should not say "$500 per person" or "$500 per year" unless the source specifies those qualifications. Test with questions that have specific numerical answers and verify the agent reproduces the numbers with the correct units, conditions, and qualifiers.

**Pattern: Conditional and Qualification Accuracy**
Sources often include conditions ("if you have been employed for at least one year") or qualifications ("subject to manager approval"). Test that the agent includes these conditions when they exist in the source and does not add conditions that do not exist.

**Pattern: Absence of Extrapolation**
The agent should not infer or extrapolate beyond the source. If the source describes how to submit an expense report, the agent should not add estimated processing times unless the source states them. Test with questions where extrapolation would be tempting and verify the agent stays within bounds.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Numerical precision | "What is the maximum amount for equipment reimbursement?" (source says "$1,500 per calendar year") | Response includes "$1,500" and "per calendar year" — does NOT add per-item limits or category breakdowns not in the source | Keyword Match (All) + Compare Meaning |
| 2 | Source-faithful response | "What is the process for requesting a leave of absence?" (source lists 3 steps) | Response meaning aligns with the 3 steps in the source — does NOT add a 4th step or embellish the existing steps | Compare Meaning + Keyword Match (All) |
| 3 | Conditional accuracy | "Am I eligible for tuition reimbursement?" (source says "full-time employees who have completed one year of service") | Response includes both conditions: "full-time" and "one year of service" — does NOT add conditions like GPA requirements or degree type restrictions | Keyword Match (All) + Compare Meaning |
| 4 | No extrapolation: processing time | "How long does it take for my expense report to be processed?" (source says "submit to your manager for approval" but does not state a timeline) | Response does NOT fabricate a specific number of days; acknowledges the source describes the process but not the timeline | Compare Meaning + General Quality |
| 5 | No embellishment: policy details | "What does the company health insurance cover?" (source lists "medical, dental, and vision") | Response includes "medical, dental, and vision" — does NOT add "prescription," "mental health," or other categories not in the source | Keyword Match (All) + Compare Meaning |
| 6 | Qualification accuracy: eligibility | "Can part-time employees use the company gym?" (source says "all employees" without specifying full-time or part-time) | Response aligns with "all employees" — does NOT add a restriction to "full-time only" or fabricate part-time specific rules | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 5 hallucination prevention test cases, focusing on questions where plausible additions are most tempting.
- **Hallucination rate threshold:** Target 0% — any hallucinated content is a failure, even if the core answer is correct.
- **Read the actual source content** before writing expected values. You cannot test for hallucination if you do not know exactly what the source says.
- **Pay special attention to numbers, dates, conditions, and lists** — these are the most commonly hallucinated details.
- **Rerun after:** Any changes to knowledge sources, agent instructions, or system prompts — especially changes to temperature or creativity settings.
- Layer multiple methods: use Keyword Match (All) for specific facts and Compare Meaning for overall faithfulness.

---

## 5. Verifying Multi-Source Synthesis

### When to Use

The answer to the user's question requires combining information from **two or more knowledge sources**. This scenario tests whether the agent correctly synthesizes across sources without distorting, omitting, or fabricating information in the process.

Use this scenario when:
- Your agent has multiple knowledge sources that cover different aspects of a topic
- Users ask questions that no single source fully answers
- You need to verify the agent combines information accurately rather than using only one source and ignoring the other

> **Related scenarios:** For verifying retrieval from the correct single source, see [Verifying Retrieval from Correct Sources](#1-verifying-retrieval-from-correct-sources). For handling cases where sources conflict, see [Validating Behavior with Conflicting Sources](#3-validating-behavior-with-conflicting-sources). For preventing fabrication during synthesis, see [Testing Hallucination Prevention](#4-testing-hallucination-prevention).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verify the response includes key details from EACH required source |
| Compare Meaning | Confirm the synthesized answer correctly represents the combined information |
| General Quality | Assess whether the synthesis is coherent, well-organized, and does not introduce contradictions |

> **Tip:** Multi-source synthesis is where hallucination risk is highest. When the agent combines information from two sources, it may fill gaps with plausible-sounding details that exist in neither source. Always test that the synthesis only includes content actually present in the contributing sources.

### Setup Steps

1. Identify questions in your agent's domain that require information from more than one source to answer fully. Common patterns: eligibility (from one source) + process steps (from another), product features (from one source) + pricing (from another).
2. For each question, identify which specific detail comes from which source.
3. Write test cases where the expected value includes details from each contributing source, clearly noted.
4. Use **Keyword Match (All)** to verify details from each source are present.
5. Use **Compare Meaning** to verify the combined answer is coherent and accurate.
6. Use **General Quality** to check the synthesis is clear and does not contradict itself.
7. Run the evaluation. Watch for three failure modes: missing details from one source, distorted details from either source, and fabricated "glue" information connecting the two.

### Anti-Pattern

> **Anti-Pattern: Testing Synthesis Without Knowing the Sources**
> If you write expected values based on what you think the answer should be rather than what each source actually states, you cannot distinguish between correct synthesis and hallucination. For every multi-source test case, document which piece of the expected answer comes from which source. This traceability is essential.

### Evaluation Patterns

**Pattern: Completeness Across Sources**
The response should include relevant information from each required source, not just the first source retrieved. Test that details from the second (or third) source appear in the response.

**Pattern: Accurate Attribution Per Source**
Each detail in the response should match the source it came from. If Source A says "15 days" and Source B says "30 days" for different things, verify the agent does not swap or blend these values.

**Pattern: Coherent Integration**
The synthesized response should read as a coherent whole, not as two separate paragraphs pasted together. However, coherence should come from organization, not from fabricated connecting statements. Test that the agent integrates information logically.

**Pattern: No Gap-Filling Fabrication**
When sources cover different aspects of a topic but leave gaps between them, the agent should not fill those gaps with fabricated information. If Source A explains eligibility and Source B explains the process but neither mentions a deadline, the agent should not invent a deadline to make the answer feel complete.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Eligibility + process | "How do I apply for education reimbursement and am I eligible?" (eligibility in HR policy doc, application process in benefits guide) | Response includes eligibility criteria from the HR policy AND the application steps from the benefits guide | Keyword Match (All) + Compare Meaning |
| 2 | Product features + limitations | "Can I use the data export feature on the Basic plan?" (feature description in product docs, plan limits in pricing page) | Response meaning aligns with: "The export feature works as described in the product docs, but the Basic plan has specific limitations as noted in the pricing documentation." | Compare Meaning + Keyword Match (Any) |
| 3 | General info + department-specific rules | "What is the dress code for the engineering team?" (company-wide dress code in employee handbook, engineering-specific exceptions in department page) | Response includes both the general dress code AND the engineering-specific exceptions | Keyword Match (All) + Compare Meaning |
| 4 | No gap-filling | "What is the process and timeline for internal transfers?" (process in HR guide, but no source mentions a timeline) | Response includes the process steps from the HR guide but does NOT fabricate a specific timeline | Compare Meaning + General Quality |
| 5 | Three-source synthesis | "What should I know about our parental leave policy?" (eligibility in HR policy, process in benefits guide, pay details in compensation doc) | Response is coherent, covers eligibility, process, and pay, and does not contradict any of the three sources | General Quality + Compare Meaning |
| 6 | Accurate attribution under synthesis | "Compare our health insurance options." (Plan A details in one document, Plan B details in another) | Response accurately represents Plan A details from the correct source and Plan B details from the correct source — no detail swapping | Compare Meaning + Keyword Match (All) |

### Tips

- **Coverage target:** At least 3 multi-source synthesis test cases, prioritizing the most common cross-source questions your users ask.
- **Source traceability:** For every expected value, document which source each detail comes from. Without this, you cannot diagnose whether failures are retrieval errors or synthesis errors.
- **Synthesis accuracy threshold:** Target at least 90% — multi-source answers are inherently harder, so expect slightly lower accuracy than single-source answers.
- **Watch for gap-filling** — the most common synthesis hallucination is inventing details to make the combined answer feel more complete.
- **Rerun after:** Any changes to the knowledge sources involved in cross-source questions, or changes to the agent's retrieval configuration.
- Start with two-source synthesis before testing three or more sources — get the basics right first.

---

## 6. Negative Test: Ungrounded Claims

### When to Use

This scenario provides test cases **specifically designed to elicit hallucination** — obscure questions, edge cases, plausible-sounding but unsupported premises, and questions that push the boundaries of the agent's knowledge. The goal is to verify the agent **declines rather than invents**.

Use this scenario when:
- You have high confidence in your agent's performance on common questions and want to stress-test edge cases
- The consequences of hallucination are significant and you want to verify the agent fails safely
- You are preparing for adversarial users who might deliberately try to extract fabricated information
- You want to establish a hallucination resistance baseline before publishing

> **Related scenarios:** For verifying the agent stays faithful to sources on normal questions, see [Testing Hallucination Prevention](#4-testing-hallucination-prevention). For testing what the agent does when no source exists, see [Testing Handling of Missing Sources](#2-testing-handling-of-missing-sources). For adversarial inputs designed to bypass safety boundaries, see [Safety & Boundary Enforcement](../capability-scenarios/safety-and-boundary-enforcement.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verify the agent declines to answer or acknowledges uncertainty rather than fabricating a response |
| Keyword Match (Any) | Check for appropriate uncertainty language (e.g., "I don't have information," "I'm not sure," "I cannot confirm") |
| General Quality | Assess whether the decline is graceful — not just refusing to answer but being helpful about what the agent can do |

> **Tip:** The best negative tests for hallucination use **plausible-sounding false premises** — questions that assume something that is not true but sound like they could be. "What is the deadline for the Q3 wellness reimbursement program?" when no such program exists will test whether the agent invents a deadline or pushes back on the premise.

### Setup Steps

1. Design test cases across these categories:
   - Questions about topics not in any source but adjacent to the agent's domain
   - Questions with false premises (assuming something that is not true)
   - Extremely specific questions that no source addresses (specific dates, names, amounts)
   - Questions that could be answered with general knowledge but should only be answered from configured sources
2. For each test case, the expected behavior is that the agent **declines, hedges, or acknowledges uncertainty** rather than providing a specific, confident answer.
3. Set expected values to describe the appropriate decline behavior, not a specific answer.
4. Select **Compare Meaning** as the primary method — you are checking that the response conveys uncertainty, not that it uses specific wording.
5. Add **Keyword Match (Any)** to check for uncertainty indicators.
6. Run the evaluation. **Any test case where the agent provides a confident, specific answer is a critical failure** — it has fabricated information.

### Anti-Pattern

> **Anti-Pattern: Using Obviously Absurd Questions**
> Questions like "What is the company policy on time travel?" or "Can I expense a trip to Mars?" are too obviously absurd. The agent will correctly decline, but this does not test real hallucination risk. Effective hallucination tests use **plausible, in-domain questions that are just barely outside the agent's sources** — the kind of question a real user might ask and a real agent might be tempted to answer.

### Evaluation Patterns

**Pattern: False Premise Rejection**
The question assumes something that is not true — a program that does not exist, a policy that was never implemented, or a feature that is not available. Test that the agent pushes back on the premise rather than accepting it and building a fabricated answer on top.

**Pattern: Hyper-Specific Detail Resistance**
Ask for extremely specific details that no source provides — exact dollar amounts for obscure scenarios, specific people's names for unnamed roles, exact dates for unscheduled events. The agent should not invent these details.

**Pattern: General Knowledge Suppression**
Questions the agent could answer from its training data but should only answer from configured sources. "What is the standard notice period for resignation?" has a general answer, but if the agent is configured to use company-specific sources, it should only answer from those sources — not from general knowledge about typical notice periods.

**Pattern: Leading Question Resistance**
Questions that guide the agent toward a specific fabricated answer: "Since the reimbursement limit was raised to $5,000, do I need to resubmit my request?" when the limit was not raised. Test that the agent does not accept the leading premise.

**Pattern: Confident Decline**
The agent should decline with appropriate confidence — not apologetically or vaguely, but clearly. "I don't have information about that specific program in my available sources" is better than "Um, I'm not entirely sure, maybe..." Test that declines are clear and actionable.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | False premise: nonexistent program | "What is the deadline to enroll in the company's student loan forgiveness program?" (no such program exists) | Response meaning aligns with: "The agent does not have information about a student loan forgiveness program and does not fabricate enrollment details." | Compare Meaning + Keyword Match (Any) |
| 2 | False premise: leading question | "Now that the remote work policy allows four days from home, do I need my manager to approve the fifth day?" (policy was not changed) | Response does not accept the premise; clarifies what the actual remote work policy states or acknowledges it cannot confirm the change | Compare Meaning + Keyword Match (Any) |
| 3 | Hyper-specific detail: unnamed role | "Who is the current head of the employee wellness committee?" (no source names this person) | Response includes uncertainty language; does NOT fabricate a name | Keyword Match (Any) + Compare Meaning |
| 4 | Hyper-specific detail: exact amount | "What is the exact per-meal reimbursement rate for business travel in Japan?" (sources only have a general travel policy, no per-country rates) | Agent does not invent a specific dollar amount for Japan; acknowledges the limitation | Compare Meaning + Keyword Match (Any) |
| 5 | General knowledge suppression | "How many days of notice should I give before resigning?" (agent should only use company-specific sources, not general knowledge) | Response either cites the company's specific notice period from a configured source or declines to answer if no company-specific source exists — does NOT cite general industry standards | General Quality + Compare Meaning |
| 6 | Obscure edge case | "If I adopt a child internationally while on a temporary work assignment abroad, does parental leave still apply?" (sources do not address this specific combination) | Agent acknowledges this is a specific situation not covered in its available information and suggests contacting HR directly | General Quality + Compare Meaning |
| 7 | Fabrication trap: plausible process | "Walk me through the steps to appeal a denied expense claim." (no appeal process exists in any source) | Response does NOT fabricate a multi-step appeal process; acknowledges it does not have information about an appeal process | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** At least 6 negative test cases spanning false premise, hyper-specific detail, general knowledge suppression, and leading question patterns.
- **Hallucination resistance threshold:** Target 100% — every one of these test cases should result in the agent declining or hedging. A single confident fabrication is a critical finding.
- **Design tests that are tempting** — the value of negative testing is in finding the boundary where the agent starts to fabricate. If all your tests are easy to decline, you have not found the boundary.
- **Include at least 2 false-premise tests** — these are the most revealing because they test whether the agent can push back on assumptions, not just admit ignorance.
- **Rerun after:** Any changes to system prompts, temperature settings, knowledge source configuration, or agent instructions about handling uncertainty.
- **Track fabrication patterns** — if the agent consistently fabricates in one category (e.g., always invents dates but correctly declines on names), that reveals a specific prompting or configuration issue.
- These tests are complementary to [Testing Hallucination Prevention](#4-testing-hallucination-prevention) — Scenario 4 tests that normal answers stay grounded, while this scenario actively tries to break grounding.
