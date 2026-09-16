---
name: test-case-authoring
description: Generates test scenarios and detailed manual test cases (positive, negative, boundary, equivalence-partition, business-rule, validation, error/exception, authorization, integration, API, UI, database, end-to-end, regression, recovery/retry, concurrency) from refined requirements. Maintains requirement traceability. Does NOT refine or modify requirements.
---

# Test Case Authoring Skill

## Purpose

This skill generates test scenarios and detailed, executable manual test cases from a given requirement (ideally one already refined by the Requirements Refinement skill/agent, but it may also work directly from raw requirements if that's all that's available).

**This skill must never refine, rewrite, or silently modify the requirement it is given.** If the requirement is ambiguous, incomplete, or contradictory, the skill must flag the gap and either state an explicit assumption or produce a clarification question — it must not quietly fill in business intent and proceed as if it were confirmed.

## Inputs

- A requirement, user story, or refined requirement package (preferably with Requirement ID / Acceptance Criteria ID already assigned)
- Any known business rules, validation rules, actors, integrations, and non-functional constraints relevant to the requirement
- Existing test assets or naming conventions, if provided

If the input requirement lacks a Requirement ID or Acceptance Criteria ID, assign a placeholder ID (e.g., `REQ-TBD-01`) and flag it as a traceability gap rather than inventing a fictitious formal ID.

## Coverage Requirements

Generate test scenarios and test cases across all applicable categories below. If a category does not apply to the requirement, state "Not applicable" with a one-line reason rather than omitting it silently:

- Test scenarios (high-level, before detailed test cases)
- Manual test cases
- Positive tests
- Negative tests
- Boundary tests
- Equivalence partition tests
- Business-rule tests
- Validation tests
- Error and exception tests
- Authorization tests
- Integration tests
- API tests
- UI tests
- Database tests
- End-to-end tests
- Regression tests
- Recovery and retry tests
- Concurrency tests (where applicable to the requirement — e.g., shared resource access, race conditions, parallel submissions)

## Test Case Structure

Every generated test case must include exactly these fields, in this order:

1. **Test Case ID** — unique, sequential (e.g., `TC-<REQ-ID>-001`)
2. **Requirement ID** — traces to the source requirement
3. **Acceptance Criteria ID** — traces to the specific acceptance criterion covered
4. **Test Scenario** — the higher-level scenario this test case belongs to
5. **Test Case Title** — short, descriptive
6. **Test Objective** — what this test case verifies, in one sentence
7. **Test Type** — e.g., Positive, Negative, Boundary, Integration, API, UI, Database, E2E, Regression, Recovery, Concurrency
8. **Test Design Technique** — e.g., Equivalence Partitioning, Boundary Value Analysis, Decision Table, State Transition, Error Guessing
9. **Priority** — High / Medium / Low
10. **Risk** — High / Medium / Low, with brief rationale if High
11. **Preconditions** — system/data/session state required before execution
12. **Test Data** — concrete input values or data sets (not placeholders like "valid data" unless genuinely unconstrained)
13. **Execution Steps** — numbered, unambiguous, reproducible steps
14. **Expected Results** — precise, verifiable outcome per step or overall
15. **Postconditions** — resulting system/data state after execution
16. **Dependencies** — other test cases, environments, or data setup this depends on
17. **Automation Suitability** — Suitable / Not Suitable / Conditional, with a one-line reason
18. **Traceability** — explicit link back to Requirement ID + Acceptance Criteria ID (restated for standalone readability of the test case)

## Hard Rules

- **Never modify, reinterpret, or "fix" the requirement.** If the requirement is unclear, ambiguous, or missing information needed to write a specific test case, do one of:
  - Write the test case against the most literal reading of the requirement and add an explicit **Assumption** note, or
  - Skip the specific test case and list it under **Requirement Gaps Identified**, stating exactly what is missing.
- **Never silently invent business rules** to make a test case "complete." Gaps are surfaced, not filled in silently.
- Maintain strict requirement-to-test traceability at all times — every test case must be traceable to a Requirement ID and, where available, an Acceptance Criteria ID.
- Do not perform requirement analysis/refinement work (business objective extraction, requirement quality scoring, etc.) — that is out of scope for this skill. If asked, point to the Requirements Refinement skill/agent instead.

## Output Format

Structure output as:

1. **Traceability Header** — Requirement ID, Acceptance Criteria ID(s), source requirement summary (verbatim reference, not a rewrite)
2. **Test Scenarios** — bulleted list of high-level scenarios, each mapped to the categories above it covers
3. **Test Cases** — one table or structured block per test case, using the 18-field structure above
4. **Assumptions** — explicit list of assumptions made while authoring tests
5. **Requirement Gaps Identified** — anything that blocked full test case authoring, phrased as a gap, not a silent fix
6. **Coverage Summary** — a checklist of the coverage categories (positive, negative, boundary, etc.) marking Covered / Not Applicable / Blocked by Gap
