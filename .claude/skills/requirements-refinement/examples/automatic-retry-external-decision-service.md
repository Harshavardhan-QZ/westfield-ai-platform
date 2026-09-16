# Requirements Refinement — Analysis Output (Reference Example)

*(per `.claude/skills/requirements-refinement/SKILL.md`)*

**Source:** User Story — "Automatic Retry on External Decision Service Failure (Generalized)"

This document is a reference example of the expected output format and depth for the requirements-refinement-agent / skill. Use it as a formatting and thoroughness benchmark for future refinement passes — not as a template to copy content from.

## 1. Business Objective

Prevent transient failures in the external eligibility/decision rules service integration from unnecessarily blocking policy/quote processing, while ensuring that persistent (unresolved) integration failures are not silently ignored — they are routed to a human Underwriter for manual review instead of leaving the Agent stuck or the quote processed incorrectly.

## 2. Actors and Roles

- **Agent** — the primary user/actor initiating quote processing; the "I want" role in the story.
- **External Eligibility/Decision Rules Service** — external system being called; source of the failure condition.
- **Underwriter** — downstream human actor who receives the referral for manual review.
- **System (implicit)** — the platform component responsible for orchestrating retries, setting the failure flag, and generating the referral.

## 3. Functional Behavior

- The system calls an external eligibility/decision rules service at one or more points in quote processing.
- On failure (unavailable or invalid response), the system retries the call some number of times (count unconfirmed — see §16).
- If all retries are exhausted without a valid response, a failure flag is set to `true`.
- When the failure flag is `true`, the system generates a referral to an Underwriter, with a specific customer-facing message.
- The Agent is prevented from continuing to process the quote as normal once this referral condition is reached (per Additional Context).

## 4. Business Rules

- A referral must be generated when the failure flag is `true` (retries exhausted, no valid response).
- The Agent must not be able to continue normal quote processing once persistent failure occurs (referral is the required alternate path).
- The referral must carry the confirmed customer-facing message: "We'd like to take a closer look at this quote before determining if we can offer coverage."
- Retry attempt count is a business rule that governs when the failure flag transitions to `true` — but its exact value is not yet settled (see §16, §19).

## 5. Inputs and Outputs

**Inputs:**
- Call/response (or failure) from the external eligibility/decision rules service.
- Implicit input: which "failure scenario" / integration step triggered the failure (eligibility questions, hard-stop-after-eligibility, hard-stop-after-exposure/classification, referral-or-hard-stop-after-underwriting-questions).

**Outputs:**
- A `failure flag` (boolean state, internal).
- A referral record/event directed to an Underwriter.
- The customer-facing referral message text.
- (Unconfirmed) an internal reason code tied to the referral.

## 6. Validation Rules

None explicitly stated. The only implied validation is service-response validity ("fails to return a valid response") — but "valid response" is not defined (schema, status code, content check, timeout threshold, etc.). This is a gap (see §16).

## 7. Error and Exception Handling

- Explicitly in scope: Failure to call/receive a valid response from the external eligibility/decision rules service, across multiple integration points in the quote flow (eligibility questions, post-eligibility hard-stop check, post-exposure/classification hard-stop check, post-underwriting-questions referral/hard-stop check).
- Handling defined: retry the call (count TBD) → if still failing, set failure flag → generate Underwriter referral with the confirmed message → block normal continuation of quote processing.
- Not defined: what happens to any partially-completed quote state at the point of referral; whether the Agent receives any explicit UI/system feedback beyond the referral being generated; whether the referral is retryable/re-openable later.

## 8. Positive Scenarios

- External service responds successfully on the first attempt at any of the four integration points — quote processing continues normally, no referral generated. (Implied by contrast, not explicitly stated — flag as inferred.)
- External service fails initially but succeeds on a subsequent retry (before exhaustion) — quote processing continues normally, no referral generated, failure flag remains `false`. (Explicitly implied by "so that temporary integration issues do not unnecessarily interrupt policy processing.")

## 9. Negative Scenarios

- External service fails on every attempt up to and including the final configured attempt → failure flag set `true` → referral generated (this is AC1, the only explicitly confirmed scenario).
- External service returns a response that is not "valid" (malformed, unexpected schema, etc.) rather than an outright unavailability/error — whether this is treated identically to a hard failure is not explicitly confirmed, only implied by "fails to return a valid response" in the story's "I want" clause.

## 10. Boundary and Edge Cases

- Exact retry count boundary (2 vs. 3 total attempts) is unresolved — this directly affects when the "last attempt" boundary is crossed and the flag flips to `true` (see §16).
- Behavior if the external service fails at more than one of the four listed integration points within the same quote lifecycle (e.g., fails after eligibility questions, is retried and later also fails after exposure/classification) — not addressed.
- Behavior if the service is intermittently slow/timing out vs. returning explicit error responses — whether both count identically toward the retry/failure flag is not confirmed.
- Timing edge case: what happens if a referral is in the process of being generated and the underlying service recovers mid-process — not addressed (likely irrelevant depending on architecture, but unconfirmed).

## 11. Dependencies

- Depends on the external eligibility/decision rules service and its availability/response contract.
- Depends on an internal "failure flag" mechanism, which must be settable and readable across whatever retry logic exists.
- Depends on an Underwriter referral/queueing mechanism being able to receive and display this referral with the associated message.
- Depends on internal error-handling standards documents referenced but not fully reconciled (conflicting attempt counts).

## 12. Integrations

- External eligibility/decision rules service (primary integration under test).
- Underwriter referral system/workflow (downstream integration receiving the referral).

## 13. Security and Authorization

Not addressed in the source material. No mention of who can view/action the referral, whether the Underwriter role requires specific permissions, or whether the referral message/data contains anything requiring access control. Gap.

## 14. Non-Functional Requirements

- Retry interval/backoff strategy — explicitly called out as "not yet defined" in the source (Open Questions).
- No performance, SLA, or timeout thresholds specified for what constitutes a "failure" of the external call (e.g., timeout duration).
- No volume/concurrency expectations stated (e.g., how many simultaneous quotes could be retrying against the external service at once).

## 15. Ambiguities

- "fails to return a valid response" — "valid" is undefined (schema validity? business-rule validity? non-error HTTP status?).
- "retry calls... when unavailable or fails to return a valid response" — unclear if unavailability and invalid-response are treated as the same failure category for retry/counting purposes, or handled differently.
- AC1's Given/When structure treats "failure flag = true" as if retries are a black box — per the Testing Scope Notes, this is intentional (tests should not depend on retry mechanics), but it means the trigger condition for AC1 is itself not independently testable without assuming the flag-setting logic works correctly elsewhere.
- "referred to an Underwriter for review" — no confirmation whether this is one universal referral type regardless of which of the four failure points triggered it, or differentiated (explicitly flagged as unconfirmed in source, "Referral differentiation by failure point").

## 16. Missing Information

- Exact total retry/attempt count — source material conflicts ("2" vs. "3 total (1 initial + 2 retries)" vs. a stray "3, 5, configurable" reference). This is explicitly flagged by the source as unresolved.
- Retry interval / backoff strategy — explicitly stated as "not yet defined."
- Whether a distinct internal reason code accompanies the referral message — explicitly unconfirmed.
- Whether referral reason/messaging differs by which of the four failure points triggered it — explicitly unconfirmed.
- Definition of "valid response" from the external service.
- What "unavailable" means operationally (connection refused, timeout, 5xx, circuit breaker open, etc.).
- Any SLA/timeout value governing when a call attempt is considered failed.

## 17. Contradictions

- Direct numeric contradiction in source material regarding total attempt count: "2 attempts" vs. "3 attempts total (1 initial + 2 retries)" vs. an ambiguous "3, 5, configurable" annotated with "2." These cannot all be true simultaneously and must be resolved by the story owner before attempt-count-specific behavior (though not testing, per Testing Scope Notes) can be implemented with confidence.

## 18. Assumptions

*(All items below are assumptions, not confirmed facts — labeled explicitly per the "never invent business decisions" rule.)*

- Assumption: "Unavailable" and "fails to return a valid response" are treated as the same failure category for the purpose of triggering a retry and eventually the failure flag. (Not confirmed — could be handled differently.)
- Assumption: The failure flag is scoped per quote/per integration call, not a global system-wide flag. (Reasonable given context, but not explicitly stated.)
- Assumption: A successful response on any attempt prior to exhaustion resets/prevents the failure flag from being set, allowing normal processing to continue. (Implied by the story's benefit clause, not explicitly stated as a rule.)
- Assumption: The four failure-point scenarios listed under "Referral differentiation by failure point" are the exhaustive set of integration points where this retry/referral behavior applies. (Presented as "failure scenarios to consider," not confirmed as complete or final.)

## 19. Clarification Questions

1. Is the intended total attempt count 2 or 3 (1 initial + 2 retries)? The source material contains conflicting values ("2," "3 total," and an ambiguous "3, 5, configurable" annotation) — please confirm the authoritative value with the story owner.
2. What retry interval or backoff strategy should be used between attempts (fixed delay, exponential backoff, immediate retry)?
3. Is there a distinct internal reason code associated with the referral, separate from the confirmed customer-facing message text?
4. Should the referral reason/messaging differ depending on which of the four listed failure points triggered it (eligibility questions, post-eligibility hard-stop, post-exposure/classification hard-stop, post-underwriting-questions referral/hard-stop) — or is one universal referral sufficient for all four?
5. What precisely qualifies as an "invalid response" from the external service (e.g., schema mismatch, business-rule-level invalidity, non-2xx status, empty payload)?
6. What qualifies as the service being "unavailable" (timeout threshold, connection failure, specific error codes)? Is there a defined timeout value per attempt?
7. Are "unavailable" and "invalid response" failure types counted/retried identically, or does either category have different retry behavior?
8. Is the failure flag/retry state scoped per quote, per integration call-site, or globally?
9. What is the expected system/UI behavior for the Agent immediately after a referral is generated (e.g., is the quote locked, is there an on-screen notification, can the Agent still view the quote)?
10. Are there authorization/visibility rules governing which Underwriters can see or action a given referral?

## 20. Testability

The story is partially testable as written:

- **Testable now:** AC1's core behavior — given the failure flag is `true`, a referral is generated with the exact confirmed message. This does not depend on the unresolved retry-count question, and the Testing Scope Notes explicitly confirm retry mechanics are out of test scope.
- **Not yet testable:** Anything depending on the exact number of retry attempts, backoff timing, differentiated referral reasons by failure point, or an internal reason code — because these are explicitly unconfirmed/contradictory in the source. Test cases for these aspects would require guessing at business intent, which this skill's rules prohibit.

## 21. Traceability

Proposed IDs (none were supplied in the source):

- Requirement ID: `REQ-RETRY-REFERRAL-01` (placeholder — confirm naming convention with team)
- Acceptance Criteria ID: `AC1` (as given in source)
- Suggested sub-IDs for the four failure-point scenarios, pending confirmation of whether they warrant separate ACs: `AC1a` (eligibility questions), `AC1b` (post-eligibility hard-stop), `AC1c` (post-exposure/classification hard-stop), `AC1d` (post-underwriting-questions referral/hard-stop).

## 22. Requirement Quality Score

**Score: 2.5 / 5**

Justification: The core referral behavior (AC1) is clearly and testably stated, and the story is commendably transparent about its own open questions. However, a material business rule (retry attempt count) is internally contradictory, two significant behavioral dimensions (backoff strategy, referral differentiation by failure point) are entirely undefined, and validation/error-condition definitions ("valid response," "unavailable") are left implicit. This keeps the story below "ready" quality despite AC1 itself being sound.

## 23. Refinement Status

**Needs Clarification**

Blocking items: attempt-count contradiction (§17), undefined backoff strategy, unconfirmed referral differentiation by failure point, unconfirmed internal reason code. AC1 itself (the referral-on-failure-flag behavior) is stable and can proceed to test design independently of these open items, per the Testing Scope Notes.

```
Requirement Quality Score: 2.5 / 5
Refinement Status: Needs Clarification
Open Clarification Questions: 10
```

**Note on scope:** No test scenarios or test cases were generated, per this skill's hard rule. Once the clarification questions above are resolved (particularly #1, #3, and #4), this refined requirement is ready to hand to the Test Case Authoring Agent — AC1's core referral behavior can already be handed off now, since it doesn't depend on the unresolved items.
