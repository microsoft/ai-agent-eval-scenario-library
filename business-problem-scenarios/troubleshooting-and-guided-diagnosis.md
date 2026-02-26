# Troubleshooting & Guided Diagnosis

> Scenarios for agents that walk users through diagnostic steps to identify and resolve issues — IT support, product troubleshooting, billing disputes, or technical configuration problems.

[Back to library](../README.md) | [All business-problem scenarios](README.md)

---

## Scenario 1: Verifying Diagnostic Step Accuracy

### When to Use

Your agent provides troubleshooting instructions — reboot steps, configuration changes, clearing caches, verifying settings — and you need to confirm that the steps are correct, in the right order, and will actually resolve the reported issue. Incorrect troubleshooting steps waste user time, can make problems worse, and erode trust in the agent.

This is the foundational scenario for any troubleshooting agent. Start here before testing platform awareness, escalation logic, or edge cases.

> **Related scenarios:** If you also need to verify the agent adapts instructions to the user's platform, see [Scenario 2: Testing Platform-Aware Troubleshooting](#scenario-2-testing-platform-aware-troubleshooting). For verifying the agent uses the correct knowledge source for its instructions, see [Knowledge Grounding & Accuracy](../capability-scenarios/knowledge-grounding-and-accuracy.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms that critical steps, commands, or settings names appear in the response — specific actions the user must take |
| Compare Meaning | Validates that the overall diagnostic flow is correct even when the agent phrases steps differently than the source documentation |
| General Quality | Assesses whether the instructions are clear, actionable, and in the correct sequence — not just factually present but usable |

> **Tip:** Use Keyword Match (All) for steps that have exact commands or specific setting names (e.g., "ipconfig /release"). Use Compare Meaning for steps described in natural language (e.g., "navigate to the network settings panel"). Layer with General Quality to catch ordering and clarity issues.

### Setup Steps

1. Identify your agent's 8-12 most common troubleshooting scenarios. Categorize them by issue type (connectivity, access/permissions, configuration, performance, billing).
2. For each scenario, document the correct troubleshooting steps from your official support documentation. Note the required order if sequence matters.
3. Create **Keyword Match (All)** test cases for steps that include specific commands, paths, menu names, or technical terms (e.g., "Device Manager," "ipconfig /flushdns," "Settings > Network > Reset").
4. Create **Compare Meaning** test cases for scenarios where the troubleshooting approach matters more than exact wording. Write the expected value as a description of the correct diagnostic flow.
5. Add 2-3 **General Quality** test cases with a grading prompt like: "Are these troubleshooting steps correct for the reported issue? Are they in a logical order? Would a non-technical user be able to follow them?"
6. Run the evaluation. For each failure, determine: is the agent giving wrong steps, correct steps in the wrong order, or missing critical steps?

### Anti-Pattern

> **Anti-Pattern: Accepting Any Plausible-Sounding Steps**
> Troubleshooting instructions can sound reasonable even when they are wrong or outdated. An agent that says "Go to Control Panel > Network Settings" may sound correct, but on Windows 11 the setting has moved to Settings > Network & Internet. Always validate steps against your **current** support documentation, not against whether the instructions sound plausible. Use Keyword Match with specific, current paths and commands.

### Evaluation Patterns

**Pattern: Critical Step Presence**
Verify that every required step appears in the response. Some troubleshooting flows have steps that cannot be skipped — for example, "save your work before restarting" or "back up your configuration before resetting." Use Keyword Match (All) to ensure these non-negotiable steps are present.

**Pattern: Step Sequence Validation**
Verify that steps are presented in the correct order. Some sequences are order-dependent — for example, you must disconnect from VPN before changing DNS settings. Use Compare Meaning with an expected value that describes the correct sequence, or use General Quality with a grading prompt that evaluates ordering.

**Pattern: Command and Path Accuracy**
Verify that specific commands, file paths, menu navigation paths, and setting names are accurate for the current version of the product or system. Use Keyword Match (All) with the exact current commands and paths.

**Pattern: Safety Step Inclusion**
Verify that the agent includes safety warnings or precautionary steps where necessary — such as "this will erase all data," "make sure you have administrator privileges," or "disconnect from the corporate network first."

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — VPN troubleshooting | "I can't connect to the company VPN" | "disconnect from current network", "restart VPN client", "check credentials", "verify MFA" | Keyword Match (All) + Compare Meaning |
| 2 | IT helpdesk agent — DNS resolution | "Websites aren't loading but I have internet" | The agent should guide through DNS troubleshooting: flush DNS cache, try alternate DNS, check proxy settings, then escalate if unresolved | Compare Meaning + Keyword Match (Any) |
| 3 | Product support agent — device won't power on | "My device won't turn on" | "hold power button", "10 seconds", "check charging cable", "try different outlet" | Keyword Match (All) + Compare Meaning |
| 4 | Product support agent — firmware update failure | "The firmware update failed and now the device is stuck on the loading screen" | The response should guide through recovery mode steps: hold reset button for 15 seconds, connect to computer via USB, download recovery tool from support site, and follow the recovery wizard — in that order | Compare Meaning + Keyword Match (All) |
| 5 | Billing support agent — incorrect charge | "I was charged twice for my subscription" | The agent should ask the user to verify the charges in their account statement, check for pending vs. completed transactions, and offer to initiate a billing investigation or refund | Compare Meaning + Keyword Match (Any) |
| 6 | IT helpdesk agent — step ordering | "My computer is extremely slow" | Are the steps in a logical diagnostic order? Should start with quick checks (restart, check disk space, close background apps) before deeper diagnostics (check for malware, review event logs, test hardware) | General Quality + Compare Meaning |
| 7 | Product support agent — safety warning | "How do I replace the battery in my device?" | "power off the device", "disconnect from power source", "use anti-static wrist strap", "do not puncture or bend the battery" | Keyword Match (All) + Compare Meaning |
| 8 | IT helpdesk agent — printer setup | "I can't print to the office printer" | Are the troubleshooting instructions clear enough for a non-technical employee to follow? Do they include verifying printer is on the network, checking the print queue, reinstalling the driver, and testing with a different application? | General Quality + Keyword Match (Any) |

### Tips

- Aim for **10-15 test cases** covering your agent's most common support tickets. Prioritize issues with high ticket volume.
- Set a threshold of **>= 90% pass rate** for Keyword Match and Compare Meaning tests. Incorrect troubleshooting steps directly waste user time and can escalate into bigger issues.
- Include at least **2 test cases that validate step ordering**. Sequence errors are a common failure mode that pure keyword matching will not catch.
- Always **validate against current documentation**. If your product ships updates quarterly, refresh your expected values quarterly.
- Include at least **1 safety-critical scenario** (data loss warning, electrical safety, admin privilege requirement) to verify the agent includes necessary caution steps.

---

## Scenario 2: Testing Platform-Aware Troubleshooting

### When to Use

Your agent supports users on multiple platforms, operating systems, browsers, or device types, and the troubleshooting steps differ depending on the user's environment. For example, clearing a browser cache on Chrome requires different steps than on Safari, and resetting network settings on Windows is different from macOS. You need to verify that the agent detects or asks about the platform and provides the correct platform-specific instructions.

This scenario applies when your support documentation includes platform-specific variations, or when the same issue has different resolution paths depending on the user's environment.

> **Related scenarios:** For verifying step accuracy regardless of platform, see [Scenario 1: Verifying Diagnostic Step Accuracy](#scenario-1-verifying-diagnostic-step-accuracy). For testing personalization based on user attributes (not platform), see [Information Retrieval & Policy Q&A — Scenario 2](information-retrieval-and-policy-qa.md#scenario-2-testing-answer-personalization-by-user-context).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Keyword Match (All) | Confirms that platform-specific terms, paths, and commands appear — Windows registry paths, macOS System Preferences, Linux terminal commands |
| Compare Meaning | Validates that the overall approach is correct for the stated platform, even when phrasing varies |
| General Quality | Assesses whether the agent correctly identifies the platform context and provides appropriately scoped instructions without mixing platforms |

> **Tip:** Like personalization tests, platform-aware tests require matched pairs: same issue, different platforms. The expected values must differ per test case to reflect the platform-specific steps.

### Setup Steps

1. Identify which platforms, operating systems, or browsers your agent supports. List the key environment variables (OS, OS version, browser, device type, app version).
2. Select 3-5 troubleshooting issues where the resolution differs meaningfully by platform. Avoid issues where the steps are identical across platforms.
3. For each issue, create **2-3 test cases** with different platform contexts. Set the expected value to the platform-appropriate steps for each.
4. Use **Keyword Match (All)** for platform-specific commands and paths (e.g., "Terminal" for macOS, "Command Prompt" for Windows, "Settings > General > Reset" for iOS).
5. Add **General Quality** tests with a grading prompt like: "Are these instructions appropriate for a [platform] user? Do they reference the correct menus, paths, and tools for this platform?"
6. Include at least 1 test where the user does not specify their platform, to verify the agent asks rather than guesses.
7. Run the evaluation. For failures, determine: did the agent provide wrong-platform steps, generic instructions that don't work on either platform, or fail to detect platform context?

### Anti-Pattern

> **Anti-Pattern: Providing Generic Steps That Work on No Specific Platform**
> Some agents respond to platform-dependent issues with vague, platform-neutral instructions like "Go to your settings and clear the cache." This avoids being wrong for any specific platform but is useless in practice because the user cannot follow instructions without specific paths and menu names. Platform-aware agents should provide concrete, platform-specific guidance. If the platform is unknown, the agent should ask.

### Evaluation Patterns

**Pattern: Same Issue, Different Platform**
Ask the same troubleshooting question with different platform contexts and verify that the steps change appropriately. This is the core platform-awareness pattern.

**Pattern: Platform-Specific Command Accuracy**
Verify that commands, file paths, and menu navigation are accurate for the specified platform and version. A Windows 10 path may differ from Windows 11. An iOS 16 setting may have moved in iOS 17.

**Pattern: Platform Detection via Clarification**
When the user does not specify their platform, verify that the agent asks the right clarifying question rather than guessing. The clarification should offer the specific platforms the agent supports.

**Pattern: No Cross-Platform Contamination**
Verify that platform-specific instructions do not leak steps from other platforms. A macOS user should not see references to the Windows Registry, and a Windows user should not see Terminal commands.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — clear DNS (Windows) | "My browser can't resolve hostnames" (context: Windows 11) | "Command Prompt", "ipconfig /flushdns", "run as administrator" | Keyword Match (All) + Compare Meaning |
| 2 | IT helpdesk agent — clear DNS (macOS) | "My browser can't resolve hostnames" (context: macOS Sonoma) | "Terminal", "sudo dscacheutil -flushcache", "sudo killall -HUP mDNSResponder" | Keyword Match (All) + Compare Meaning |
| 3 | Product support agent — app crash (iOS) | "The app keeps crashing on my phone" (context: iPhone, iOS 17) | "Settings", "General", "iPhone Storage", "Offload App", "reinstall from App Store" | Keyword Match (All) + Compare Meaning |
| 4 | Product support agent — app crash (Android) | "The app keeps crashing on my phone" (context: Samsung Galaxy, Android 14) | "Settings", "Apps", "Clear Cache", "Clear Data", "Force Stop" | Keyword Match (All) + Compare Meaning |
| 5 | IT helpdesk agent — VPN setup (cross-platform) | "How do I set up the company VPN?" (context: not specified) | The agent should ask what operating system the user is on before providing setup instructions | General Quality + Compare Meaning |
| 6 | Billing support agent — browser-specific checkout issue | "The payment page won't load" (context: Safari on macOS) | The response should address Safari-specific steps: check Content Blockers, disable Prevent Cross-Site Tracking for the payment domain, and try disabling Safari extensions — not Chrome or Firefox steps | Compare Meaning + Keyword Match (All) |
| 7 | Product support agent — Bluetooth pairing (Windows) | "I can't pair my device via Bluetooth" (context: Windows 11) | "Settings", "Bluetooth & devices", "Add device", "pairing mode" | Keyword Match (All) + Compare Meaning |
| 8 | IT helpdesk agent — no cross-platform leakage | "How do I check my IP address?" (context: macOS) | The response should not contain "ipconfig" (Windows) or "Settings > Network > Wi-Fi > Advanced" (iOS). It should reference "ifconfig" or "System Settings > Network" for macOS | General Quality + Compare Meaning |

### Tips

- Create **matched pairs** (same issue, different platform) for at least 3-5 issues. This is the only reliable way to confirm platform awareness.
- Set a threshold of **>= 90% pass rate**. Wrong-platform instructions are a common and highly visible failure that immediately undermines user trust.
- **Track platform versions** in your test cases. Instructions for Windows 10 may differ from Windows 11. Update expected values when major OS versions ship.
- Always include at least **1 "platform unknown" test** to verify the agent asks for clarification rather than guessing.
- Check for **cross-platform contamination** — a response that says "On Windows, open Command Prompt; on macOS, open Terminal" when the user already specified their platform is a contamination issue, not a helpful response.
- Re-run platform tests after updating your knowledge sources with new platform documentation.

---

## Scenario 3: Validating Escalation Judgment

### When to Use

Your agent handles initial troubleshooting but should escalate to a human agent when it cannot resolve the issue, when the issue requires account-level access the bot does not have, or when the user is frustrated and needs human intervention. You need to verify that the agent makes the right escalation decision — neither escalating too early (before trying reasonable steps) nor too late (after the user has wasted time on steps that cannot help).

This scenario applies to any support agent that has both self-service troubleshooting capabilities and the ability to hand off to human agents, ticketing systems, or specialized teams.

> **Related scenarios:** For verifying the troubleshooting steps themselves are correct (before escalation is needed), see [Scenario 1: Verifying Diagnostic Step Accuracy](#scenario-1-verifying-diagnostic-step-accuracy). For infrastructure-level escalation mechanics, see [Graceful Failure & Escalation](../capability-scenarios/graceful-failure-and-escalation.md).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Validates that the agent's escalation or non-escalation decision is appropriate for the situation described |
| Keyword Match (All) | Confirms that escalation responses include required elements — ticket number, expected response time, handoff language |
| Capability Use (Any) | Verifies that the agent triggers the correct escalation action (e.g., transfer to live agent, create support ticket, route to specialized team) |
| General Quality | Assesses the overall quality of the escalation decision — timing, empathy, context preservation, and appropriateness |

> **Tip:** Escalation testing requires both "should escalate" and "should NOT escalate" test cases. An agent that escalates everything is just as broken as one that never escalates.

### Setup Steps

1. Define your agent's escalation criteria. Document when the agent should escalate, including: issue severity thresholds, issue types beyond the agent's scope, number of failed troubleshooting attempts, and user sentiment indicators.
2. Create 3-4 test cases where escalation IS the correct action: issues the agent cannot resolve, sensitive situations, repeated failure after troubleshooting steps.
3. Create 3-4 test cases where escalation is NOT the correct action: common issues the agent should be able to resolve with standard troubleshooting.
4. For escalation cases, use **Capability Use (Any)** to verify the correct escalation mechanism fires (live agent transfer, ticket creation, etc.).
5. For escalation cases, use **Keyword Match (All)** to verify the response includes required escalation elements (e.g., ticket reference, estimated wait time, what happens next).
6. For non-escalation cases, use **Compare Meaning** to verify the agent provides troubleshooting steps instead of escalating.
7. Add **General Quality** tests for nuanced situations with a grading prompt like: "Is the agent's decision to escalate (or not) appropriate given the situation? Did the agent try reasonable troubleshooting before escalating?"
8. Run the evaluation. For failures, determine: is the agent escalating too aggressively (never troubleshoots) or too passively (never escalates)?

### Anti-Pattern

> **Anti-Pattern: Only Testing the Happy Escalation Path**
> Some teams only test cases where escalation is clearly correct (e.g., "I need to speak to a manager") and never test cases where the agent should troubleshoot instead of escalating. This misses the more common failure mode: agents that escalate at the first sign of difficulty, bypassing self-service resolution entirely. Always test both directions — "should escalate" and "should NOT escalate."

### Evaluation Patterns

**Pattern: Severity-Based Escalation**
Test that the agent escalates high-severity issues (data breach, account compromise, service outage affecting multiple users) immediately without attempting routine troubleshooting. These issues require human judgment regardless of whether the agent could technically attempt steps.

**Pattern: Exhausted-Options Escalation**
Test that the agent escalates after exhausting its troubleshooting capabilities, rather than looping or repeating the same steps. The agent should acknowledge it has tried what it can and hand off with context.

**Pattern: Should-Not-Escalate Validation**
Test that the agent attempts troubleshooting for common, resolvable issues rather than escalating prematurely. These tests use standard support scenarios where the agent has documented solutions.

**Pattern: Escalation Context Preservation**
When the agent does escalate, verify that it preserves context for the human agent — summarizing the issue, what steps were already attempted, and any diagnostic information gathered. This prevents the user from having to repeat everything.

**Pattern: User Sentiment Escalation**
Test that the agent recognizes when a user is frustrated, upset, or explicitly requesting human help, and escalates even if the technical issue might be resolvable. User experience sometimes trumps technical resolution capability.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — account compromise | "I think someone hacked my account — there are login attempts I don't recognize" | The agent should immediately escalate to the security team without attempting standard troubleshooting | Compare Meaning + Keyword Match (Any) |
| 2 | IT helpdesk agent — account compromise action | "I think someone hacked my account — there are login attempts I don't recognize" | Escalation action: Transfer to Security Incident team | Capability Use (Any) + Compare Meaning |
| 3 | IT helpdesk agent — routine password reset | "I forgot my password and can't log in" | The agent should walk through the self-service password reset process, NOT escalate to a human agent | Compare Meaning + Keyword Match (Any) |
| 4 | Product support agent — defective product | "I've tried all the troubleshooting steps on your website and the device still doesn't work. I've had it for 2 weeks." | "support ticket", "case number", "replacement", "1-2 business days" | Keyword Match (All) + Compare Meaning |
| 5 | Billing support agent — disputed charge | "There's a $500 charge on my account that I didn't authorize" | The agent should escalate to the billing disputes team given the unauthorized charge, and include the charge amount and date in the escalation context | Compare Meaning + Keyword Match (Any) |
| 6 | Product support agent — should not escalate | "How do I connect my device to WiFi?" | The agent should provide WiFi setup instructions rather than escalating. This is a standard self-service scenario | Compare Meaning + Keyword Match (Any) |
| 7 | IT helpdesk agent — frustrated user | "This is the third time I'm asking about this. Your bot is useless. Let me talk to a real person." | The agent should acknowledge the frustration, apologize, and escalate to a human agent — regardless of whether the technical issue is resolvable | General Quality + Compare Meaning |
| 8 | Billing support agent — escalation with context | "I was charged for a subscription I cancelled last month. I already tried re-cancelling through the website." | Does the escalation include context for the human agent: the issue (charge after cancellation), what the user already tried (re-cancellation via website), and account identifiers? | General Quality + Keyword Match (Any) |

### Tips

- Include an **equal number of "should escalate" and "should NOT escalate" test cases** (at least 3-4 of each). This prevents a one-directional bias in your evaluation.
- Set a threshold of **>= 90% pass rate** for escalation judgment. Escalation errors have high user-impact: premature escalation wastes human agent time, and late escalation frustrates users.
- Test **at least 1 high-severity scenario** (security incident, data breach, unauthorized access) to verify the agent escalates immediately without attempting standard troubleshooting.
- Test **at least 1 user-sentiment scenario** where the user is clearly frustrated or explicitly requests a human, even if the technical issue is resolvable.
- Verify that **escalation responses include context preservation** — a summary of what was tried and what the issue is. This is often overlooked but critical for the human agent receiving the handoff.
- Re-run escalation tests after any changes to your escalation logic, topic routing, or the set of issues your agent is trained to handle.

---

## Scenario 4: Testing Multi-Step Diagnosis Completeness

### When to Use

Your agent guides users through multi-step troubleshooting flows — not just a single suggestion, but a structured diagnostic process with multiple checkpoints. You need to verify that the agent walks through ALL necessary steps, does not skip intermediate diagnostics, and reaches the correct conclusion or next action.

This scenario applies when troubleshooting requires a sequence of checks (e.g., "first verify network connectivity, then check VPN client status, then verify credentials, then check firewall rules") and skipping a step could lead to an incorrect diagnosis.

> **Related scenarios:** For verifying individual step accuracy, see [Scenario 1: Verifying Diagnostic Step Accuracy](#scenario-1-verifying-diagnostic-step-accuracy). For testing that the agent escalates when multi-step diagnosis fails, see [Scenario 3: Validating Escalation Judgment](#scenario-3-validating-escalation-judgment).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| Compare Meaning | Validates that the agent's diagnostic flow covers all required checkpoints and reaches the appropriate conclusion |
| Keyword Match (All) | Confirms that specific required diagnostic steps or checkpoints are mentioned — each step the user needs to perform |
| General Quality | Assesses the overall completeness and logical flow of the diagnostic process — are steps in order, is the flow comprehensive, does it reach a resolution or appropriate next step? |

> **Tip:** Multi-step completeness tests differ from accuracy tests. Accuracy asks "are these steps right?" Completeness asks "are ALL the steps here?" An agent that gives 3 correct steps out of a required 6 passes accuracy but fails completeness.

### Setup Steps

1. Identify 5-8 troubleshooting flows where multiple diagnostic steps are required. Document the full expected flow for each, including the number of steps and any branching logic.
2. For each flow, write the complete list of required diagnostic checkpoints. Mark which steps are mandatory vs. conditional.
3. Create **Keyword Match (All)** test cases where each required checkpoint is a keyword. The test fails if any mandatory step is missing from the response.
4. Create **Compare Meaning** test cases where the expected value describes the complete diagnostic flow, including branching (e.g., "If step 2 reveals X, proceed to step 3a; if Y, proceed to step 3b").
5. Add **General Quality** test cases with a grading prompt like: "Does this diagnostic flow include all necessary steps to diagnose the reported issue? Is anything missing that could lead to an incomplete diagnosis?"
6. Run the evaluation. For each failure, determine: did the agent skip steps, collapse multiple steps into one vague instruction, or stop diagnosing prematurely?

### Anti-Pattern

> **Anti-Pattern: Testing Only the First Step**
> Some teams write test cases that check whether the agent starts diagnosing correctly but do not verify whether it completes the full diagnostic flow. An agent that says "First, restart your router" may be correct for step 1 but might never mention steps 2 through 5. Always test for the full diagnostic chain, not just the opening move.

### Evaluation Patterns

**Pattern: Full Checkpoint Coverage**
Verify that every required diagnostic checkpoint appears in the agent's response. Use Keyword Match (All) with a keyword per checkpoint. This is the most direct completeness test.

**Pattern: Diagnostic Flow Completeness**
Verify that the agent's response represents a complete diagnostic pathway — from initial triage through intermediate checks to a conclusion or next action. Use Compare Meaning with an expected value that describes the full intended flow.

**Pattern: Branching Logic Validation**
For diagnostic flows with conditional branches (e.g., "If the light is green, do X; if red, do Y"), verify that the agent presents the appropriate branches rather than only one path. The agent should guide the user through decision points.

**Pattern: Resolution or Next-Action Conclusion**
Verify that the diagnostic flow ends with a clear outcome — either a resolution ("this should fix the issue — try it and let me know"), an escalation ("if none of these steps resolved it, I'll connect you with a specialist"), or a next-step directive. The flow should not trail off without conclusion.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — email not syncing | "My Outlook email stopped syncing on my laptop" | "internet connection", "Outlook restart", "account re-authentication", "clear Outlook cache", "repair Office installation" | Keyword Match (All) + Compare Meaning |
| 2 | IT helpdesk agent — complete VPN flow | "I can't connect to the corporate VPN from home" | The agent should guide through the full diagnostic: verify internet connectivity, restart VPN client, check VPN credentials, verify MFA token, check for VPN client updates, try alternate VPN server, and escalate if all steps fail | Compare Meaning + Keyword Match (Any) |
| 3 | Product support agent — device connectivity | "My smart speaker won't connect to my WiFi network" | "check WiFi password", "restart speaker", "restart router", "verify 2.4GHz vs 5GHz band", "check for firmware update", "factory reset as last resort" | Keyword Match (All) + Compare Meaning |
| 4 | Product support agent — branching diagnosis | "My laptop screen is flickering" | The agent should present branching logic: check if flickering occurs on battery vs. plugged in (hardware vs. power issue), check if it occurs in all apps or just one (software vs. display issue), and provide different resolution paths for each scenario | Compare Meaning + Keyword Match (Any) |
| 5 | Billing support agent — payment failure | "My payment keeps failing when I try to pay my bill" | "verify card details", "check expiration date", "check billing address match", "try different payment method", "check with bank for holds or blocks", "contact support if still failing" | Keyword Match (All) + Compare Meaning |
| 6 | IT helpdesk agent — complete flow with conclusion | "My computer keeps blue-screening" | Does the diagnostic flow end with a clear next step? After diagnostic checks (check recent updates, run memory diagnostic, check disk health, review event logs), the agent should direct the user to either apply a fix or escalate to IT with the collected diagnostic information | General Quality + Compare Meaning |
| 7 | Product support agent — printer not printing | "My printer is connected but nothing prints" | Does the flow cover all diagnostic layers? Should include: check print queue for stuck jobs, verify correct printer is selected, print a test page from printer menu, check driver status in Device Manager, uninstall/reinstall printer, test from a different application | General Quality + Keyword Match (Any) |
| 8 | Billing support agent — overcharge investigation | "I think I'm being overcharged on my monthly bill" | The agent should guide through: verify current plan and pricing, review recent billing statements for changes, check for add-on services, check for promotional pricing expiration, compare charges against plan terms, and offer to open a billing review if discrepancy is confirmed | Compare Meaning + Keyword Match (Any) |

### Tips

- For each troubleshooting flow, count the **required steps** and verify all are present. If a flow has 6 required steps and the agent only produces 4, that is a completeness failure regardless of whether those 4 are correct.
- Set a threshold of **>= 85% pass rate** for completeness tests. Multi-step flows are complex, and some variation in how steps are grouped is acceptable, but missing entire diagnostic categories is not.
- Test at least **2 flows that have branching logic**. Conditional diagnosis is harder for agents and is a common failure point.
- Verify that every diagnostic flow **ends with a conclusion** — either a resolution, an escalation, or a clear next step. Open-ended responses that trail off are a completeness failure.
- Distinguish between **step grouping** (the agent combines two related steps into one instruction — usually acceptable) and **step omission** (the agent skips a diagnostic category entirely — always a failure).
- Re-run completeness tests after any update to your troubleshooting documentation or knowledge sources.

---

## Scenario 5: Handling Intermittent or Unreproducible Issues

### When to Use

Users sometimes report issues that are intermittent ("it happens sometimes"), hard to reproduce ("it worked fine yesterday"), or vaguely described ("something is wrong with the app"). You need to verify that the agent handles these uncertain situations appropriately — gathering more information, suggesting diagnostic logging, or recommending monitoring rather than jumping to a specific fix that may not apply.

This scenario applies when users cannot clearly describe the problem, when the issue does not occur consistently, or when the symptoms described could match multiple root causes.

> **Related scenarios:** For handling broad or ambiguous queries (non-troubleshooting), see [Information Retrieval & Policy Q&A — Scenario 5](information-retrieval-and-policy-qa.md#scenario-5-handling-ambiguous-or-broad-queries). For verifying the agent escalates when it cannot diagnose, see [Scenario 3: Validating Escalation Judgment](#scenario-3-validating-escalation-judgment).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Assesses whether the agent's handling of an uncertain situation is appropriate — does it gather information, suggest monitoring, avoid premature diagnosis? |
| Compare Meaning | Validates that the agent's approach matches the expected strategy for intermittent issues (information gathering, logging setup, monitoring) rather than a definitive fix |
| Keyword Match (Any) | Checks that the response includes at least some diagnostic data-gathering steps — asking about frequency, conditions, error messages, recent changes |

> **Tip:** General Quality is the primary method because handling ambiguous issues is a qualitative judgment. The grading prompt should evaluate the agent's diagnostic strategy, not just whether specific steps appear.

### Setup Steps

1. Identify 4-6 types of intermittent or vague issues your users report. Examples: "the app crashes sometimes," "my connection drops randomly," "I got an error but I don't remember what it said."
2. For each issue, decide what a good diagnostic response looks like. Typically this involves: gathering more information, asking diagnostic questions, suggesting logging or monitoring, and avoiding premature fixes.
3. Create **General Quality** test cases with a grading prompt like: "The user reported an intermittent issue with limited detail. Does the agent appropriately gather more information before suggesting a fix? Does it avoid jumping to a specific solution without sufficient diagnostic data?"
4. Create **Compare Meaning** tests for scenarios where the expected approach is clear (e.g., "the agent should ask about frequency, timing, and error messages before suggesting any fix").
5. Use **Keyword Match (Any)** to verify the response includes information-gathering elements — "how often," "when does it happen," "error message," "recent changes," "which version."
6. Run the evaluation. For failures, determine: did the agent jump to a definitive fix, give a generic response, or fail to ask diagnostic questions?

### Anti-Pattern

> **Anti-Pattern: Prescribing a Specific Fix for a Vague Symptom**
> When a user says "the app is acting weird," the worst response is a confident set of specific troubleshooting steps for a problem the agent has not yet diagnosed. This is the intermittent-issue equivalent of answering the wrong question. The agent should first understand the problem before proposing a solution. Test cases should penalize premature prescriptions and reward information gathering.

### Evaluation Patterns

**Pattern: Information Gathering First**
Verify that the agent asks diagnostic questions before suggesting fixes. For intermittent issues, the agent should ask about frequency, timing, conditions (what was the user doing), error messages, recent changes, and environment. The agent should NOT skip directly to troubleshooting steps.

**Pattern: Monitoring and Logging Guidance**
When the issue cannot be diagnosed from the user's description alone, verify that the agent suggests ways to capture more data — enabling logging, monitoring for recurrence, noting the exact time and conditions when it happens next.

**Pattern: Multiple-Possibility Acknowledgment**
Verify that the agent acknowledges when symptoms could match multiple root causes, rather than confidently diagnosing one. The response should present possibilities with appropriate uncertainty language.

**Pattern: Progressive Narrowing**
For multi-turn scenarios, verify that the agent progressively narrows the diagnosis based on user responses, rather than asking all questions at once or repeating the same questions.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — vague app issue | "The app has been acting weird lately" | The agent should ask clarifying questions: what specific behavior is the user seeing, how often does it occur, which app, and whether anything changed recently | General Quality + Compare Meaning |
| 2 | IT helpdesk agent — intermittent connectivity | "My internet keeps dropping but it comes back after a minute" | The response should ask about frequency, time of day, whether the user is on WiFi or Ethernet, and suggest monitoring the connection pattern before prescribing a fix | Compare Meaning + Keyword Match (Any) |
| 3 | Product support agent — random crash | "My device randomly restarts sometimes" | "how often", "error message", "recent update", "specific app" | Keyword Match (Any) + Compare Meaning |
| 4 | Product support agent — unknown error | "I got an error message but I closed it and don't remember what it said" | The agent should guide the user on how to find the error again: check event logs, look in notification history, try to reproduce the action, and enable error notifications | Compare Meaning + Keyword Match (Any) |
| 5 | Billing support agent — mystery charge | "There's a charge I don't recognize but I'm not sure if it's wrong" | The agent should help the user investigate rather than immediately opening a dispute: ask for the charge amount and date, guide them to check order history, and check for subscription renewals before concluding it is unauthorized | General Quality + Compare Meaning |
| 6 | IT helpdesk agent — possible multiple causes | "My computer is slow" | Does the agent acknowledge that slowness can have many causes (disk space, RAM, malware, background processes, hardware aging) and ask diagnostic questions to narrow down, rather than jumping to a single fix? | General Quality + Keyword Match (Any) |
| 7 | Product support agent — monitoring suggestion | "My smart home device goes offline every few days" | The agent should suggest monitoring: note the time and conditions when it goes offline, check if other devices on the same network are affected, and check the device logs for disconnect events — before suggesting a firmware reset | Compare Meaning + Keyword Match (Any) |
| 8 | IT helpdesk agent — log collection guidance | "Something is wrong with my VPN but I can't describe exactly what" | "event logs", "VPN client logs", "screenshot", "connection log" — the agent should guide the user to collect diagnostic data | Keyword Match (Any) + Compare Meaning |

### Tips

- Use **General Quality as the primary method** for intermittent issue tests, because appropriate handling is a qualitative judgment. The grading prompt should evaluate diagnostic strategy rather than specific keywords.
- Set a threshold of **>= 80% pass rate**. This is lower than accuracy thresholds because handling vague inputs is inherently harder and has more acceptable response patterns.
- Write grading prompts that **penalize premature fixes** and **reward information gathering**. The key evaluation question is: "Did the agent try to understand the problem before solving it?"
- Include at least **3 different levels of vagueness**: slightly vague ("the app crashes sometimes"), very vague ("something is wrong"), and contradictory ("it doesn't work but sometimes it does").
- Test at least **1 scenario where the agent should suggest logging or monitoring** rather than immediate troubleshooting.
- For multi-turn evaluation, test whether the agent **adapts its questions** based on user responses rather than following a rigid script.

---

## Scenario 6: Negative Test — Incorrect Diagnosis Path

### When to Use

Your agent handles multiple types of issues, and you need to verify that it does not suggest troubleshooting steps that are irrelevant to the reported problem, come from the wrong product's documentation, or address a different issue than the one described. This is the negative counterpart to Scenario 1 — instead of checking that the right steps are given, you check that the wrong steps are NOT given.

This scenario is critical when your agent covers multiple products, platforms, or issue categories where steps from one domain could be incorrectly applied to another. For example, a user reporting a software login issue should not receive hardware troubleshooting steps, and a user with a billing question should not be walked through technical diagnostics.

> **Related scenarios:** For verifying the correct steps are provided, see [Scenario 1: Verifying Diagnostic Step Accuracy](#scenario-1-verifying-diagnostic-step-accuracy). For verifying the agent retrieves from the correct knowledge source, see [Information Retrieval & Policy Q&A — Scenario 6](information-retrieval-and-policy-qa.md#scenario-6-negative-test--incorrect-source-retrieval).

### Recommended Test Methods

| Method | Purpose |
|--------|---------|
| General Quality | Assesses whether the troubleshooting steps are relevant to the reported issue and do not include steps from an unrelated problem or product |
| Compare Meaning | Validates that the diagnostic approach addresses the correct issue category, not an adjacent or similar-sounding one |
| Keyword Match (All) | Used as a negative check — verifies that terms, commands, or steps from an incorrect diagnosis path do NOT appear in the response |
| Capability Use (All) | Confirms the agent retrieved from the correct troubleshooting knowledge source and not from documentation for a different product or issue type |

> **Tip:** Negative diagnosis tests require you to anticipate which wrong path the agent is likely to take. Focus on issue types that share symptoms or keywords, because these are the most common misdiagnosis triggers.

### Setup Steps

1. Identify **high-risk confusion pairs** — issue types that share symptoms, keywords, or terminology. Examples: "can't connect" could trigger WiFi troubleshooting or VPN troubleshooting; "slow" could trigger network diagnostics or hardware diagnostics; "error" could trigger software or billing resolution.
2. For each confusion pair, write test cases where the user clearly describes Issue A, and verify the agent does not provide steps for Issue B.
3. Use **General Quality** with a grading prompt like: "Are these troubleshooting steps relevant to the specific issue the user reported? Do they address the right problem, or do they include steps that belong to a different issue type?"
4. Use **Keyword Match (All)** as a negative check: include keywords unique to the wrong diagnosis path and verify they do NOT appear. (If your platform does not support negative matching, use General Quality with a prompt that checks for irrelevant steps.)
5. Use **Capability Use (All)** to verify the agent consulted the correct troubleshooting documentation, not documentation for a different product or issue.
6. Run the evaluation. For failures, determine: what caused the misdiagnosis? Keyword confusion in the query? Retrieval from the wrong source? Incorrect topic routing?

### Anti-Pattern

> **Anti-Pattern: Only Testing Positive Diagnosis, Never Testing for Wrong Paths**
> Most teams test "does the agent give the right answer for issue X?" but never test "does the agent avoid giving the wrong answer for issue X?" These are different tests. An agent can provide the correct VPN troubleshooting steps AND incorrectly add WiFi troubleshooting steps in the same response. Without negative tests, the extra irrelevant steps go undetected, confusing users and wasting their time.

### Evaluation Patterns

**Pattern: Wrong Product Steps**
Test that an agent covering multiple products does not provide troubleshooting steps for Product B when the user reports an issue with Product A. This is common when products share similar feature names or terminology.

**Pattern: Wrong Issue Category**
Test that the agent does not confuse adjacent issue categories. A user reporting a billing error should not receive technical troubleshooting steps. A user reporting a software crash should not receive network connectivity steps.

**Pattern: Symptom-Based Misdiagnosis**
Test that the agent does not diagnose based on surface-level keyword matching. "My screen is black" for a laptop means power/display troubleshooting, but for a software application it means a rendering or compatibility issue. The agent should use the full context, not just matching keywords.

**Pattern: Outdated or Deprecated Resolution Path**
Test that the agent does not suggest troubleshooting steps that applied to a previous version of the product or system but are no longer valid. For example, suggesting a registry fix that was only relevant for Windows 10 to a Windows 11 user.

**Pattern: Irrelevant Diagnostic Data Collection**
Test that the agent does not ask for diagnostic information that is irrelevant to the reported issue. Asking a user with a billing question for their operating system version is a misdiagnosis signal.

### Practical Examples

| # | Scenario | Sample Input | Expected Value / Capability | Method |
|---|----------|-------------|---------------------------|--------|
| 1 | IT helpdesk agent — software vs. hardware | "I can't log in to the HR portal" (software/access issue) | The response should not contain hardware troubleshooting terms like "restart device", "check cables", or "run hardware diagnostics" — this is a login/access issue | General Quality + Compare Meaning |
| 2 | IT helpdesk agent — correct knowledge source | "I can't log in to the HR portal" | Should retrieve from Identity-and-Access-Troubleshooting-KB, NOT from Network-Connectivity-KB or Hardware-Support-KB | Capability Use (All) + Compare Meaning |
| 3 | Product support agent — wrong product | "The camera on my Model X laptop isn't working" | The response should not include steps for the Model Z desktop or external webcam — these have different drivers and settings | General Quality + Compare Meaning |
| 4 | Product support agent — wrong product keywords | "The camera on my Model X laptop isn't working" | The response should not contain "Model Z", "external webcam driver", or "USB camera" — terms from the wrong product's troubleshooting guide | Keyword Match (All) + General Quality |
| 5 | Billing support agent — billing vs. technical | "I was double-charged for my subscription renewal" | The response should address billing investigation (verify charges, check payment method, initiate refund), NOT technical troubleshooting (clear cache, reinstall app, check internet connection) | Compare Meaning + Keyword Match (Any) |
| 6 | IT helpdesk agent — adjacent issue confusion | "My email is slow to load" (performance issue) | The response should focus on email/Outlook performance: check mailbox size, clear cache, test web version. It should NOT suggest general internet troubleshooting like "restart your router" or "run a speed test" unless email-specific steps have been exhausted | General Quality + Compare Meaning |
| 7 | Product support agent — deprecated fix | "My Bluetooth keeps disconnecting" (current device model) | The response should not suggest the legacy "delete Bluetooth registry key" workaround that applied to the 2022 model. It should provide the current troubleshooting path: check firmware version, re-pair device, reset Bluetooth module | Compare Meaning + Keyword Match (Any) |
| 8 | Billing support agent — irrelevant data collection | "Why is my bill higher this month?" | The agent should not ask for the user's operating system, browser version, or device model — these are irrelevant to a billing inquiry. It should ask about the billing period, plan type, and any recent account changes | General Quality + Compare Meaning |

### Tips

- Identify your **top 5 confusion pairs** (issue types most likely to be mixed up) and write at least 2 negative tests for each pair. These are your highest-risk misdiagnosis scenarios.
- Set a threshold of **>= 90% pass rate** for negative diagnosis tests. Irrelevant troubleshooting steps waste user time and can cause users to make unnecessary changes to their systems.
- Use **product-specific or issue-specific keywords** as contamination markers. If your VPN guide mentions "split tunneling" and your WiFi guide mentions "channel interference," these unique terms serve as indicators of which path the agent followed.
- Test negative paths **for your most common issues**, not just edge cases. High-volume troubleshooting topics have the most user impact when misdiagnosed.
- Pay special attention to **symptoms that span categories**. "Slow" could be network, hardware, software, or even billing (slow to process payment). "Error" could be any category. Test these ambiguous symptoms specifically.
- Re-run negative diagnosis tests after adding new products, issue categories, or knowledge sources, since new content can alter retrieval behavior and introduce new confusion risks.

---
