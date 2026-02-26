# Process Navigation & Multi-Step Guidance

> Scenarios for agents that guide users through multi-step processes — onboarding, configuration setup, compliance workflows, or any procedure that requires sequential steps with conditions and prerequisites.

[Back to library](../README.md) | [All business-problem scenarios](README.md)

---

## 1. Verifying Step Sequence Accuracy

### When to Use

Your agent walks users through processes where the order of steps matters — onboarding checklists, software configuration, compliance procedures, or approval workflows. This scenario tests whether the agent presents steps in the correct sequence and does not skip, reorder, or combine steps in a way that causes downstream failures.

> **Related scenarios:** For verifying that the agent retrieves step information from the correct knowledge source, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md). This scenario focuses on whether the steps are presented in the right order, not whether they are sourced correctly.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verifies that all required step names, actions, or milestones appear in the response |
| Compare Meaning | Validates that the overall guidance matches the authoritative process — right steps in the right order with accurate descriptions |
| Capability Use (All) | Confirms the agent retrieves from the correct process documentation or knowledge source |

> **Tip:** Step sequence errors are subtle — the guidance can contain all the right steps but in the wrong order. Compare Meaning catches this better than Keyword Match alone because it evaluates the logical flow, not just the presence of terms.

### Setup Steps

1. Identify the 3–5 most common multi-step processes your agent guides users through (e.g., new hire onboarding, software setup, compliance certification).
2. Document the authoritative step sequence for each process — include step numbers, names, and any ordering dependencies (e.g., "Step 3 requires Step 2 to be complete").
3. Write test inputs that ask the agent to walk through each process from the beginning.
4. Set expected keywords to the step names or key actions that must appear.
5. Set expected meaning to the complete, correctly ordered process description.
6. Run the test set and flag any cases where steps appear out of order, are missing, or are incorrectly combined.

### Anti-Pattern

> **Anti-Pattern: Checking Only Step Presence, Not Step Order**
> A Keyword Match test that checks for "install," "configure," and "test" will pass even if the agent says "Test the connection, then install the software, then configure the settings." Always use Compare Meaning alongside Keyword Match to validate that steps appear in the correct logical sequence.

### Evaluation Patterns

#### Verify complete step coverage
Ask the agent for the full process and confirm every required step appears. Missing a step — especially an early prerequisite — can cause the entire downstream process to fail.

#### Verify correct sequential ordering
Confirm steps appear in the documented order. Pay special attention to steps that seem interchangeable but are not (e.g., "create account" must come before "configure permissions").

#### Verify step dependency communication
When steps have dependencies, the agent should make those explicit — "Before you can configure SSO, you need to have your Azure AD tenant ID from Step 2."

#### Verify no improper step merging
The agent should not combine distinct steps into one (e.g., merging "install the agent" and "configure the agent" into "install and configure the agent") if the process documentation treats them as separate steps with separate instructions.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | New hire onboarding — full sequence | "What are the steps for new employee onboarding?" | Response must include all onboarding steps in order: accept offer, complete background check, submit I-9 documentation, set up payroll, attend orientation, complete required training; must retrieve from the onboarding knowledge source | Keyword Match (All) + Capability Use (All) + Compare Meaning |
| 2 | Software installation — sequential setup | "Walk me through setting up the development environment" | Response must present steps in order: install prerequisites, clone repository, configure environment variables, install dependencies, run initial build, verify setup with test suite | Compare Meaning + Keyword Match (All) |
| 3 | Compliance certification — ordered workflow | "What do I need to do to complete my annual compliance certification?" | Response must list steps in order: review updated policy documents, complete training module, pass assessment quiz, sign acknowledgment form, submit to compliance team; must not skip the training before the assessment | Compare Meaning + Keyword Match (All) |
| 4 | Step dependency verification | "Can I skip the prerequisites and go straight to configuring the VPN?" | Response should explain that VPN configuration requires the prerequisite steps (account provisioning and certificate installation) and cannot be skipped | Compare Meaning + Keyword Match (Any) |
| 5 | Process with time-sensitive ordering | "How do I submit my quarterly expense reconciliation?" | Response must present steps in the correct order with any deadlines: gather receipts, categorize expenses, submit report by the 5th of the following month, obtain manager approval, finance review; should note the deadline dependency | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 2 test cases per multi-step process — one requesting the full sequence and one asking about a specific step to verify the agent knows where it falls in the order.
- **Threshold:** >= 95% of step-sequence tests should present all steps in the correct order with no missing steps.
- **Test the "start from the middle" case:** Ask the agent about step 4 of a 6-step process and verify it provides context about what should have happened before and what comes next.
- **Document authoritative sequences before testing:** You need a ground truth to compare against. Capture the official process from the source documentation.
- **Rerun after:** Any changes to process documentation, onboarding checklists, or configuration guides in your knowledge sources.

---

## 2. Testing Role-Aware Guidance

### When to Use

Your agent guides users through processes that vary by role, department, or employment status — managers see different onboarding tasks than individual contributors, admins get different configuration steps than end users, US-based employees follow different compliance steps than international employees. This scenario tests whether the agent adapts its guidance based on who is asking.

> **Related scenarios:** For verifying authorization and permission checks during task execution, see [Request Submission & Task Execution](request-submission-and-task-execution.md). This scenario focuses on guidance adaptation, not action authorization.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Validates that the guidance is tailored to the user's role — the right steps, the right tools, the right approvers for their context |
| Keyword Match (All) | Verifies role-specific details appear — specific form names, approval chains, tools, or portals that differ by role |
| Capability Use (Any) | Confirms the agent retrieves from the correct role-specific knowledge source or documentation |

> **Tip:** Role-aware guidance errors are often invisible at first glance. An agent might give a perfectly valid onboarding sequence — just the wrong one for the user's role. Test the same process question from different personas to catch this.

### Setup Steps

1. Identify processes that have role-dependent variations (e.g., onboarding for managers vs. ICs, software setup for admins vs. end users).
2. Document how the process differs by role — different steps, different forms, different approval chains, different tools.
3. Create test personas for each role variation (e.g., "new hire — engineering IC," "new hire — people manager," "new hire — contractor").
4. Write the same process question from each persona and set the expected output to the role-appropriate variation.
5. Verify the agent presents the correct variation for each role — and does not present steps from the wrong role's process.

### Anti-Pattern

> **Anti-Pattern: One-Size-Fits-All Guidance**
> The most common failure is an agent that gives the same process steps regardless of who is asking. If your onboarding process has different steps for managers (complete leadership training, set up direct-report access) and individual contributors (complete technical training, set up development environment), the agent should distinguish between them. Test with multiple roles to catch this.

### Evaluation Patterns

#### Verify role-specific step inclusion
Confirm the agent includes steps that are unique to the user's role and omits steps that belong to other roles.

#### Verify role-specific tool and form references
Different roles often use different forms, portals, or tools. Verify the agent references the correct ones — the manager sees the PeopleSoft manager portal, the IC sees the employee self-service portal.

#### Verify role-specific approval chains
When processes involve approvals, the approval chain often differs by role. A manager's expense report might go directly to the VP; an IC's might go to their manager first. Verify the agent describes the correct chain.

#### Verify agent asks for role when unknown
When the user's role is not already known or inferred, the agent should ask before providing guidance rather than defaulting to a generic or incorrect set of steps.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Manager onboarding vs. IC onboarding | "What do I need to do for onboarding?" (user is a new people manager) | Response must include manager-specific steps: set up direct-report access, complete leadership training, schedule 1:1s with each report; must NOT include IC-specific steps like "set up development environment" | Compare Meaning + Keyword Match (All) |
| 2 | IC onboarding — same question, different role | "What do I need to do for onboarding?" (user is a new engineering IC) | Response must include IC-specific steps: set up development environment, complete technical bootcamp, request repo access; must NOT include manager-specific steps like "set up direct-report access" | Compare Meaning + Keyword Match (All) |
| 3 | Admin software setup vs. end-user setup | "How do I set up the CRM system?" (user is an admin) | Response must include admin steps: configure tenant settings, set up user roles and permissions, configure integrations; must NOT give the end-user instructions (install the app, log in, customize your dashboard) | Compare Meaning + Capability Use (Any) |
| 4 | Compliance workflow — US vs. international | "What are the steps for the annual benefits enrollment?" (user is based in Germany) | Response must present the benefits enrollment process applicable to the user's region, including any region-specific forms, deadlines, or regulatory steps; must NOT present US-specific options like 401(k) enrollment | Compare Meaning + Keyword Match (All) |
| 5 | Role unknown — agent should ask | "Walk me through the onboarding process" (role is not determinable from context) | Agent should ask a clarifying question about the user's role or position before providing guidance — e.g., "Are you joining as a people manager or an individual contributor?" | Compare Meaning + Keyword Match (Any) |

### Tips

- **Coverage target:** For every role-variant process, test at least 2 different roles with the same input question to confirm the agent differentiates.
- **Threshold:** >= 90% of role-aware tests should present the correct role-specific guidance without including steps from the wrong role's process.
- **Test the boundaries between roles:** A "team lead" who is not technically a "people manager" — does the agent know which process applies? Test ambiguous role boundaries.
- **Verify the agent does not over-disclose:** A standard employee asking about onboarding should not see the manager's process, which might reveal information about leadership tools or approval authorities they should not know about.
- **Rerun after:** Any changes to role-specific process documentation, organizational structure, or the agent's user-context detection logic.

---

## 3. Validating Completeness of Guidance

### When to Use

Your agent guides users through processes where missing a step has real consequences — skipping a prerequisite blocks a later step, omitting a safety check creates risk, or forgetting a deadline results in a compliance violation. This scenario tests whether the agent covers ALL required steps, including prerequisites, post-completion actions, and easy-to-miss administrative steps.

> **Related scenarios:** For verifying step order specifically, see Scenario 1 (Verifying Step Sequence Accuracy) in this file. This scenario focuses on completeness — are any steps missing entirely? — rather than ordering.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Verifies every required step, prerequisite, and post-action appears in the response |
| Compare Meaning | Validates the guidance is substantively complete — covering not just the main steps but prerequisites, dependencies, and follow-up actions |
| General Quality | Assesses whether a user following only the agent's guidance could complete the process successfully without needing to look anything up elsewhere |

> **Tip:** Completeness errors are the hardest to catch with Keyword Match alone because you need to know what is missing. Define the full expected checklist before writing the test, then check every item. Compare Meaning can catch missing context that individual keyword checks miss.

### Setup Steps

1. For each process your agent guides, create a complete checklist of every required step — including prerequisites, the main steps, verification steps, and post-completion actions.
2. Write test cases that ask the agent for full process guidance.
3. Set expected keywords to every item on the checklist — step names, prerequisite names, form names, deadlines.
4. Use Compare Meaning with a comprehensive expected description that includes prerequisites and post-actions.
5. Use General Quality to assess self-sufficiency: could a user who has never done this before follow the agent's guidance without needing to consult another source?
6. Flag any test where even one checklist item is missing from the response.

### Anti-Pattern

> **Anti-Pattern: Testing Only the Core Steps**
> Processes are more than their headline steps. Prerequisites ("Make sure you have your employee ID before starting"), verification steps ("Confirm your enrollment before the deadline"), and post-completion actions ("Save your confirmation number for your records") are commonly omitted by agents but critical for user success. Include these in your expected value, not just the main steps.

### Evaluation Patterns

#### Verify prerequisite inclusion
Before the main steps, the agent should mention what the user needs to have ready — documents, access, information, or completed prior steps.

#### Verify main step completeness
Every step in the authoritative process should appear. Use the full checklist as a comparison point.

#### Verify post-completion actions
After the last step, the agent should mention what happens next — confirmation to save, approval timeline to expect, follow-up actions to take.

#### Verify deadline and timing information
If steps have deadlines or time-sensitive dependencies ("must be completed within 30 days of hire date"), the agent should include this information.

#### Verify nothing is assumed
The agent should not assume the user already knows implicit steps. If the process requires logging into a specific portal, the agent should say so — not assume the user knows which portal to use.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | New hire onboarding — complete with prerequisites | "I'm starting next Monday — what do I need to do for onboarding?" | Response must include prerequisites (bring I-9 documents, have bank account info for payroll), all main steps (complete paperwork, attend orientation, set up IT accounts, complete required training), and post-completion actions (confirm benefits enrollment by day 30, save employee ID) | Keyword Match (All) + Compare Meaning |
| 2 | Software setup — including verification step | "How do I set up the project management tool for my team?" | Response must include prerequisites (obtain admin credentials, confirm license count), main steps (create workspace, configure project templates, invite team members, set permissions), and verification ("Create a test project to confirm the setup is working correctly") | Keyword Match (All) + General Quality + Compare Meaning |
| 3 | Compliance workflow — including deadlines | "Walk me through the SOX compliance audit preparation" | Response must include all preparation steps, the documentation that needs to be gathered, the review meetings to schedule, the submission deadline, and the post-submission follow-up (retain copies, expect auditor questions within 2 weeks) | Compare Meaning + Keyword Match (All) |
| 4 | Configuration with implicit prerequisites | "How do I configure single sign-on for the application?" | Response must mention prerequisites that are easy to miss: obtain the IdP metadata URL, ensure the admin has directory access, confirm the DNS records are updated; should not assume the user has already completed these | Compare Meaning + General Quality |
| 5 | Process with easy-to-miss administrative step | "What do I need to do to offboard an employee?" | Response must include not just the obvious steps (revoke access, collect equipment) but also commonly missed administrative steps: update org chart, transfer document ownership, close open tickets, notify payroll, schedule exit interview | Keyword Match (All) + Compare Meaning |

### Tips

- **Coverage target:** At least 1 completeness test per process, with the full checklist defined as the expected value.
- **Threshold:** >= 90% of completeness tests should include all prerequisite steps and all post-completion actions. 100% should include all main steps.
- **Create the checklist from the source documentation, not from the agent's output.** If you build your checklist based on what the agent already says, you will miss whatever the agent is already missing.
- **Test "what else do I need to know?" follow-ups:** After the agent gives its initial guidance, ask "Is there anything else I should know?" or "Am I missing anything?" and see if it surfaces additional steps.
- **Rerun after:** Any changes to process documentation, the addition of new steps to existing processes, or changes to prerequisite requirements.

---

## 4. Testing Branching Paths

### When to Use

Your agent guides users through processes that have conditional branches — "if you are a US employee, do X; if international, do Y" or "if your request is under $5,000, self-approve; if over $5,000, route to VP." This scenario tests whether the agent follows the correct branch based on the user's situation rather than presenting all branches or defaulting to one.

> **Related scenarios:** For trigger routing at the topic/flow level, see [Trigger Routing](../capability-scenarios/trigger-routing.md). This scenario focuses on within-process branching — the agent is already in the right topic but needs to follow the correct conditional path within it.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Validates the agent follows the correct branch for the user's situation — presenting only the relevant path, not all possible paths |
| Keyword Match (All) | Verifies branch-specific details appear — the forms, approvers, tools, or steps unique to the correct branch |
| Capability Use (Any) | Confirms the agent retrieves from the correct branch-specific knowledge source if branches are documented separately |

> **Tip:** The most common branching failure is not choosing the wrong branch — it is presenting ALL branches and letting the user figure out which one applies. Test that the agent selects and presents only the relevant path.

### Setup Steps

1. Map the branching points in each process your agent guides — identify the condition (role, location, amount, status) and the different paths.
2. For each branching point, write test cases that clearly establish which branch the user should follow (e.g., "I'm based in the UK" establishes the international branch).
3. Set expected meaning to the guidance for the correct branch only — the test should fail if the agent presents the wrong branch or all branches.
4. Set expected keywords to branch-specific terms that only appear in the correct path.
5. Test at least 2 branches per branching point to confirm the agent differentiates.

### Anti-Pattern

> **Anti-Pattern: Presenting All Branches Instead of Selecting One**
> An agent that responds with "If you're in the US, do A. If you're international, do B. If you're a contractor, do C" is not guiding — it is dumping the decision tree on the user. When the agent has enough context to determine the branch (the user already said they are in the UK), it should present only the relevant path. Test that the agent selects the right branch rather than listing all options.

### Evaluation Patterns

#### Verify correct branch selection
Given clear user context (role, location, amount, status), the agent should present only the steps from the correct branch.

#### Verify no cross-branch contamination
Steps from the wrong branch should not appear in the guidance. If the US branch requires Form W-4 and the international branch requires a tax residency certificate, a UK employee should only see the tax residency certificate.

#### Verify branching condition inquiry
When the agent does not have enough information to determine the branch, it should ask the relevant question — not guess or default.

#### Verify complex multi-branch navigation
Some processes have nested branches (e.g., international → EU vs. APAC → country-specific). Test that the agent navigates through multiple levels of branching correctly.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Benefits enrollment — US employee path | "How do I enroll in benefits? I'm based in Chicago." | Response must present the US benefits enrollment path: select health plan, choose 401(k) contribution, designate beneficiaries; must NOT include international-specific steps like pension scheme enrollment or social insurance registration | Compare Meaning + Keyword Match (All) |
| 2 | Benefits enrollment — international path | "How do I enroll in benefits? I'm based in London." | Response must present the UK benefits enrollment path: enroll in pension scheme, select private medical insurance, designate beneficiaries under UK regulations; must NOT include US-specific steps like 401(k) or HSA enrollment | Compare Meaning + Keyword Match (All) |
| 3 | Expense approval — under threshold | "I need to submit a $2,500 expense for conference registration" | Response must present the self-approval path (expenses under $5,000): submit through the expense portal with receipt, direct manager approval is sufficient; must NOT mention VP approval or finance committee review | Compare Meaning + Keyword Match (Any) |
| 4 | Expense approval — over threshold | "I need to submit a $12,000 expense for the vendor engagement" | Response must present the high-value approval path: submit through the expense portal, requires VP approval, may require finance committee review; should mention additional documentation requirements for high-value expenses | Compare Meaning + Keyword Match (All) |
| 5 | Branching condition unknown — agent asks | "How do I get set up for remote access?" | Agent should ask a clarifying question to determine the branch: "Are you using a company-managed device or a personal device?" because the setup process differs significantly | Compare Meaning + Keyword Match (Any) |
| 6 | Nested branching — multi-level conditions | "I'm a contractor based in Singapore — how do I complete tax documentation?" | Response must navigate two branch levels (contractor → APAC → Singapore-specific) and present the correct tax documentation process for Singapore-based contractors; must NOT present employee tax forms or US/EU contractor requirements | Compare Meaning + Keyword Match (All) |

### Tips

- **Coverage target:** At least 2 test cases per branching point — one per branch — to confirm the agent differentiates.
- **Threshold:** >= 90% of branching tests should present the correct branch without cross-branch contamination.
- **Test the "I already told you" case:** If the user established their context earlier in the conversation ("I mentioned I'm in the UK"), the agent should remember and not ask again at a later branching point.
- **Map your decision trees explicitly:** Before writing tests, diagram the branching logic. This reveals branches you might not have thought to test.
- **Test boundary conditions:** An expense at exactly $5,000 — does it follow the under-threshold or over-threshold path? Test values at branch boundaries.
- **Rerun after:** Any changes to process branching logic, threshold values, or the addition of new branches to existing processes.

---

## 5. Handling Mid-Process Questions

### When to Use

Users rarely follow a multi-step process from start to finish without asking questions along the way. They ask "What does this field mean?", "Can I go back and change step 2?", "How long does this take?", or "What if I don't have the information you need?" This scenario tests whether the agent can answer mid-process questions without losing its place in the process flow — and then resume guidance from where it left off.

> **Related scenarios:** For testing knowledge retrieval accuracy of the answers themselves, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md). This scenario focuses on the agent's ability to handle interruptions within a multi-step flow without losing context.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Validates the agent answers the mid-process question accurately and then resumes guidance at the correct step |
| General Quality | Assesses whether the agent handles the interruption smoothly — the user should not feel they have been redirected, restarted, or ignored |
| Keyword Match (All) | Verifies that after the interruption, the agent resumes with the correct next step — not back at step 1 |

> **Tip:** The two failure modes are equally bad: ignoring the question and plowing ahead with the next step, or answering the question but losing track of where the user was in the process. Test for both.

### Setup Steps

1. Choose a multi-step process with at least 4 steps.
2. Write a conversation flow where the agent is guiding through the process and the user interrupts with a question after step 2 or 3.
3. The test should evaluate two things: (a) the answer to the mid-process question is accurate, and (b) the agent resumes at the correct step after answering.
4. Set expected meaning to a response that both answers the question and continues from the right step.
5. Write at least 2 types of mid-process questions: factual clarification ("What does this field mean?") and process-level questions ("Can I do this step later?").

### Anti-Pattern

> **Anti-Pattern: Restarting the Process After Every Question**
> Some agents lose their conversational context when the user asks a mid-process question and restart the guidance from step 1. This is frustrating for users who are partway through a 6-step process. Test that the agent answers the question and resumes from the correct point — not the beginning.

### Evaluation Patterns

#### Verify accurate answer to the mid-process question
The agent should answer the clarifying question correctly — not deflect, ignore, or give a generic "please complete the current step" response.

#### Verify process resumption at the correct step
After answering, the agent should continue from where the user left off. If the user was on step 3 and asked a question, the agent should resume with step 3 or step 4 — not step 1.

#### Verify no step skipping after resumption
The agent should not accidentally skip a step when resuming. If the user was between step 3 and step 4, the agent should resume at step 4 — not jump to step 5.

#### Verify handling of "can I skip this?" questions
When the user asks whether a step can be skipped, the agent should accurately answer based on the process requirements — "yes, this step is optional" or "no, this step is required because step 5 depends on it."

#### Verify handling of "can I go back?" questions
When the user asks to revisit a previous step, the agent should either provide the information for that step or explain any implications of going back (e.g., "You can update your selection from step 2, but it may change the options available in step 4").

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Clarification question during onboarding | Agent is on step 3 (set up payroll). User asks: "What's a W-4 form and where do I find it?" | Response must explain what a W-4 form is, where to obtain it, and then resume guidance at step 3 (payroll setup) or advance to step 4 — not restart from step 1 | Compare Meaning + Keyword Match (All) |
| 2 | "Can I skip this?" during software setup | Agent is on step 4 (configure notifications). User asks: "Do I really need to set up notifications? Can I skip this?" | Response must accurately state whether notifications configuration is required or optional; if optional, should offer to move to step 5; if required, should explain why and continue with step 4 | Compare Meaning + General Quality |
| 3 | Process-level question during compliance workflow | Agent is on step 2 (complete training module). User asks: "How long does this whole process usually take?" | Response must provide a reasonable time estimate for the full process, then resume at step 2 (the training module) without restarting | Compare Meaning + General Quality |
| 4 | "Can I go back?" during configuration | Agent is on step 5 (test the integration). User asks: "Wait, I think I entered the wrong API key in step 3. Can I go back and fix it?" | Response should explain how to update the API key from step 3, note any implications for steps 4–5 (may need to redo), and provide clear guidance to resume | Compare Meaning + Keyword Match (All) |
| 5 | Off-topic question mid-process | Agent is on step 3 (submit documents). User asks: "By the way, what's the office holiday schedule this year?" | Response should either answer briefly if the information is available or direct the user to the appropriate resource, then resume at step 3; must not abandon the current process flow | General Quality + Compare Meaning |

### Tips

- **Coverage target:** At least 2 mid-process interruption tests per multi-step process — one factual clarification and one process-level question (skip, go back, how long).
- **Threshold:** >= 90% of mid-process question tests should correctly answer the question AND resume at the right step.
- **Test interruptions at different points:** Interruptions in early steps, middle steps, and late steps to verify context retention throughout the process.
- **Test rapid successive questions:** Ask 2–3 clarifying questions in a row before continuing — verify the agent does not lose track of which step the user was on.
- **Verify the agent signals where it is resuming:** A good agent says "Now, back to step 3 — setting up your payroll..." so the user knows exactly where they are in the process.
- **Rerun after:** Any changes to the agent's conversational memory, system prompt, or topic-switching behavior.

---

## 6. Negative Test: Incorrect Step Order

### When to Use

This is the adversarial counterpart to step sequence verification. Instead of testing that the agent presents steps in the right order, this scenario tests that the agent actively resists presenting steps out of order, skipping mandatory steps, or letting users jump ahead to steps that require prerequisites. The goal is to confirm the agent protects users from process failures by enforcing step dependencies.

> **Related scenarios:** For testing safety boundary enforcement against adversarial inputs, see [Safety & Boundary Enforcement](../capability-scenarios/safety-and-boundary-enforcement.md). This scenario focuses specifically on process-integrity enforcement — the agent protecting users from skipping steps that will cause failures.

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Verifies the agent does not comply with out-of-order requests — it should redirect the user to the correct step or explain the dependency |
| Keyword Match (All) | Confirms the response mentions the prerequisite step that must be completed first |
| General Quality | Assesses whether the agent's redirection is helpful and educational — explaining WHY the step order matters, not just refusing |

> **Tip:** Users who ask to skip ahead usually do so for a reason — they think they already completed the prerequisite, or they are in a hurry. The agent should be firm about step dependencies but also helpful — explaining why the order matters and offering to check whether the prerequisite is actually done.

### Setup Steps

1. Identify steps in your processes that have hard dependencies — steps that will fail or cause downstream problems if prerequisites are not completed.
2. Write test cases where the user explicitly asks to skip to a later step or asks for a step out of order.
3. Set expected meaning to a response that (a) does not provide the out-of-order step as if it can be done now and (b) explains what needs to be completed first.
4. Set expected keywords to the prerequisite step names that the agent should reference.
5. Test both direct skip requests ("Can I just do step 5?") and implicit skip attempts ("I don't want to do step 2 — what's next?").

### Anti-Pattern

> **Anti-Pattern: Compliant Skip Execution**
> When a user says "Skip the training and go straight to the certification exam," an agent that simply complies is setting the user up for failure. If the training is a prerequisite for the exam, the agent should explain the dependency. Test that the agent pushes back on skip requests for mandatory steps rather than blindly complying.

### Evaluation Patterns

#### Verify resistance to explicit skip requests
When the user says "Skip to step 5," the agent should not jump to step 5 if prerequisites are unmet. It should explain what needs to happen first.

#### Verify resistance to implicit ordering violations
When the user's question implies skipping steps ("I already know how to do the first part — just tell me how to configure SSO"), the agent should verify the prerequisites are met rather than assuming they are.

#### Verify dependency explanation
The agent should explain WHY steps must be done in order — "You need to complete the security training before accessing the compliance portal because your access is provisioned after training completion." This education builds user trust.

#### Verify offer to check prerequisite status
Instead of a flat refusal, the agent should offer to verify whether the prerequisite is actually complete — "Let me check whether your security training is marked as complete. If it is, we can move to the next step."

#### Verify handling of legitimately skippable steps
Not all steps are mandatory. The agent should distinguish between hard dependencies (cannot be skipped) and optional steps (can be skipped). Test both to confirm the agent is not overly rigid about optional steps.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | Skip mandatory training for certification | "I already know this material — can I just take the certification exam and skip the training?" | Response must explain that the training module is a prerequisite for the certification exam and must be completed first; should offer to check if training is already recorded as complete | Compare Meaning + Keyword Match (All) |
| 2 | Jump to deployment before testing | "Let's skip testing and go straight to deploying the integration to production" | Response must explain that testing is a required step before production deployment; should describe the risks of skipping (untested integration, potential data issues) and recommend completing the test step | Compare Meaning + General Quality |
| 3 | Attempt to configure before prerequisites | "I want to set up SSO — skip the part about Azure AD setup" | Response must explain that Azure AD configuration is a prerequisite for SSO setup; the IdP metadata, tenant ID, and directory configuration from the Azure AD step are required inputs for SSO configuration | Compare Meaning + Keyword Match (All) |
| 4 | Skip optional step — should be allowed | "Can I skip the custom dashboard setup and move to the next step?" (dashboard setup is optional) | Response should confirm that the dashboard setup is optional, explain what the user will miss by skipping it, and proceed to the next step without blocking | Compare Meaning + General Quality |
| 5 | Implicit skip — user claims completion | "I already set up my development environment. What's next after that?" (no way to verify) | Response should proceed to the next step but remind the user of key prerequisites from the dev environment setup that the next step depends on — e.g., "Make sure you have Node.js v18+ installed and your environment variables configured, as the next step requires both" | Compare Meaning + Keyword Match (All) |
| 6 | Reverse order request | "Walk me through the offboarding process, but start with the exit interview and work backwards" | Response should explain that the offboarding process has a specific order for a reason (e.g., access revocation must happen after knowledge transfer, not before) and present the steps in the correct forward sequence | Compare Meaning + General Quality |

### Tips

- **Coverage target:** At least 1 negative skip test per mandatory step in each process, plus 1 test for an optional step to verify the agent is not overly rigid.
- **Threshold:** >= 95% of mandatory-step skip requests should be declined with a clear explanation. 100% of optional-step skip requests should be allowed.
- **Test the "I already did it" claim:** Users often say "I already completed that step" without it being verified. The agent should either verify through the system or provide a reminder of key outputs from that step that the next step depends on.
- **Balance firmness with helpfulness:** The agent should never be confrontational. A good refusal educates the user about why the order matters and offers to help them through the prerequisite.
- **Test escalation for persistent skip requests:** If the user insists on skipping a mandatory step after being told it is required, the agent should maintain its position but may offer to connect the user with a human who can make an exception.
- **Rerun after:** Any changes to step dependencies, the introduction of new prerequisite requirements, or changes to which steps are optional vs. mandatory.
