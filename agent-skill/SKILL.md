---
name: eshop-api-test-generator
description: Analyze an EShop API specification and generate traceable candidate API test cases for the selected HW06 endpoints.
metadata:
  short-description: Specification-to-test-case generator
---

# EShop API Test Generator

## Purpose

This skill is a reusable API test-case generator for the HW06 EShop SUT.

Given:

- the API specification
- one selected target endpoint
- optional assignment/security requirements

it analyzes the endpoint and produces structured candidate API test cases.

This skill focuses on:

`Specification -> Test Design -> Candidate Test Cases`

It does not replace the student's required human review.

It must not fabricate execution evidence, screenshots, Newman reports, GitHub Issues, commit logs, CI/CD results, or human audit decisions.

---

## Scope

The current HW06 repository uses this skill for:

- `POST /api/login`
- `POST /api/apply-coupon`
- `PUT /api/admin/orders/:id/status`

However, the generation method should remain reusable for similar API endpoints when a compatible specification is provided.

---

## Required Context

Before generating final candidate test cases, read:

- `README.md` for selected API scope when available
- `docs/api_specification.md` for endpoint details

Read `docs/2026.HW06.API Testing_En.md` only when assignment-specific coverage or formatting requirements are needed.

The API specification is the primary source of truth for endpoint behavior.

Do not treat examples in this skill as authoritative business rules.

---

## Core Behavior

For one target API:

1. Parse the endpoint definition.
2. Extract request structure.
3. Extract authentication and authorization rules.
4. Extract response and error schemas.
5. Extract business/state rules.
6. Build test-design dimensions.
7. Generate at least 35 candidate cases when used for HW06.
8. Attach traceability metadata to every case.
9. Set every generated case to `PENDING_HUMAN_REVIEW`.
10. Export structured rows suitable for Markdown, CSV, Excel, or later Postman conversion.

The skill must not finalize a generated case as `VALID`, `INVALID`, or `INCOMPLETE`.

Those decisions belong to the student's human-audit phase.

---

## Generation Pipeline

### Step 1 — Parse Specification

Extract:

- method
- path
- headers
- path parameters
- query parameters
- body fields
- field types
- required/optional status
- format rules
- boundary constraints
- authentication
- authorization
- success status codes
- error status codes
- success schema
- error schema
- business rules
- state rules
- relevant security requirements

If a value is not defined by the specification, mark it as unknown.

Do not guess.

---

### Step 2 — Build Parameter Partitions

For every input parameter, consider applicable classes:

- valid nominal value
- minimum boundary
- just below minimum
- maximum boundary
- just above maximum
- empty
- missing
- null
- wrong type
- malformed format
- semantically invalid value
- unexpected extra value

Only generate a class when it is meaningful for the parameter.

Do not mechanically create impossible cases.

---

### Step 3 — Build Security Dimensions

Identify applicable security cases from the endpoint behavior and assignment requirements.

Possible categories:

- missing authentication
- invalid authentication
- expired/malformed/tampered token
- authorization bypass
- role escalation
- IDOR
- cross-user access
- injection-style input
- account enumeration
- sensitive-data leakage
- business-rule abuse

Security cases must be tied to a plausible attack surface.

Do not add irrelevant security tests merely to increase case count.

---

### Step 4 — Build Schema Dimensions

Generate assertions for:

- expected HTTP status
- required response properties
- field types
- nested objects
- nullable behavior
- error shape
- absence of sensitive fields
- business-result consistency

Do not invent response fields that are not in the specification.

---

### Step 5 — Build State Dimensions

If the endpoint reads or modifies state, identify:

- prerequisite state
- valid transition
- invalid transition
- terminal-state behavior
- repeated operation
- replay behavior
- persisted-state verification
- cross-request dependency

Only treat a transition as valid/invalid when the specification supports that expectation.

Otherwise mark the case as exploratory.

---

### Step 6 — Identify Ambiguities

Before generating cases, list unresolved questions such as:

- unspecified case sensitivity
- unspecified whitespace behavior
- unspecified rate-limit threshold
- unspecified token expiry behavior
- unspecified exact error status
- unclear state transition

For ambiguous behavior:

- do not invent an expected result
- label the case as exploratory when useful
- state what observation would clarify the behavior

---

### Step 7 — Generate Candidate Cases

For HW06, generate at least 35 AI-generated candidate cases per selected API.

Prefer meaningful coverage breadth over repetitive variations.

Recommended distribution depends on the endpoint, but should usually include:

- positive behavior
- domain partitions
- boundaries
- missing/null/wrong-type inputs
- security
- schema
- state/business behavior

Every case must include a clear objective and executable request plan.

---

## Test Case Format

Use spreadsheet-ready fields:

- `case_id`
- `api`
- `source`
- `requirement_ref`
- `security_ref`
- `spec_reference`
- `objective`
- `preconditions`
- `request_method`
- `request_url`
- `headers`
- `path_params`
- `query_params`
- `body`
- `expected_status`
- `expected_response_or_assertion`
- `coverage_type`
- `assumption_or_open_question`
- `audit_status`
- `audit_notes`

Defaults:

- `source = AI_GENERATED`
- `audit_status = PENDING_HUMAN_REVIEW`

Do not populate `audit_status` with a human-review decision.

---

## Case ID Conventions

Use:

- `LOGIN-GEN-001`
- `COUPON-GEN-001`
- `ORDERSTATUS-GEN-001`

If the skill is reused for a different API, create a stable endpoint-specific prefix.

Manual extension IDs are reserved for the human workflow:

- `LOGIN-EXT-001`
- `COUPON-EXT-001`
- `ORDERSTATUS-EXT-001`

This skill must not falsely label AI-generated cases as manual extensions.

---

## Traceability Rules

Each case should reference the strongest available source:

- feature/requirement ID
- security requirement ID
- API specification section
- request field
- response field
- state/business rule

Prefer traceability like:

`Requirement -> Spec Rule -> Test Case`

This supports later mapping to:

`Test Case -> Postman Request -> Execution Result -> Bug`

Do not invent requirement identifiers.

---

## Coverage Heuristics

The following are candidate ideas only.

Always verify them against the API specification.

### Login

Possible dimensions:

- valid credentials
- malformed email
- missing/empty/null/wrong-type email
- wrong password
- missing/empty/null/wrong-type password
- non-existing account
- whitespace behavior
- case sensitivity
- injection-style input
- authentication response schema
- token field if specified
- sensitive-data leakage
- account enumeration
- lockout/rate limiting only if relevant

### Apply Coupon

Possible dimensions:

- valid coupon
- unknown coupon
- expiration if supported
- missing/empty/null/wrong-type coupon code
- whitespace
- case sensitivity as exploratory if unspecified
- amount below/equal/above thresholds when defined
- zero/negative/decimal/large/wrong-type amount
- user identifier validation if present
- per-user usage rules if defined
- discount calculation
- final amount calculation
- injection-style code
- response schema

### Admin Order Status

Possible dimensions:

- valid transitions defined by the specification
- invalid reverse transition
- invalid skip transition
- terminal-state behavior
- repeated transition
- unknown/missing/empty/null/wrong-type status
- invalid order ID
- unknown order ID
- unauthenticated request
- malformed/expired/tampered token
- normal-user authorization
- admin authorization
- role escalation attempt
- persisted-state verification

---

## Expected Status Rules

Only provide a concrete `expected_status` when:

- the API specification defines it, or
- the requirement unambiguously implies it.

If the expected code is not defined:

- do not guess
- use an exploratory expectation
- state the unresolved behavior in `assumption_or_open_question`

Example:

Instead of:

`expected_status = 400`

use:

`expected_status = SPEC_UNDEFINED`

with:

`assumption_or_open_question = Exact error status is not specified; observe implementation and compare with course/security expectations.`

---

## Output Validation

Before returning generated cases, verify:

- case IDs are unique
- the target API is correct
- requests use only documented fields unless explicitly exploratory
- security cases are relevant
- expected statuses are justified
- expected response fields exist in the spec
- preconditions are explicit
- state-dependent cases identify prerequisite states
- generated cases are not mislabeled as human extensions
- every generated case remains `PENDING_HUMAN_REVIEW`

---

## Export Guidance

The preferred output is a structured table suitable for conversion to:

- Markdown
- CSV
- Excel
- Postman request definitions

The skill may help create Postman-ready request data, but Postman execution is outside the core generator responsibility.

Real execution belongs to the broader HW06 workflow controlled by `AGENT.md`.

---

## Human Audit Boundary

After generation, stop at:

`PENDING_HUMAN_REVIEW`

The student must review each generated case and decide:

- `VALID`
- `INVALID`
- `INCOMPLETE`

The skill may later help explain or correct a case after the student's review, but it must not claim that its own judgment satisfies the assignment's human-review requirement.

---

## Demo Design

A clear demonstration should show:

1. Input API specification.
2. Select one target endpoint.
3. Extract endpoint rules.
4. Show partitions/security/schema/state dimensions.
5. Generate candidate cases.
6. Export structured test cases.
7. Show that all cases remain `PENDING_HUMAN_REVIEW`.

Suggested conceptual pipeline:

```text
API Specification
       |
       v
 Endpoint Parser
       |
       v
 Rule Extractor
       |
       +--> Parameter Partitions
       +--> Security Dimensions
       +--> Schema Dimensions
       +--> State Dimensions
       |
       v
 Candidate Case Generator
       |
       v
 Traceability Validator
       |
       v
 Structured Test Cases
       |
       v
 PENDING_HUMAN_REVIEW
```

The submitted diagram must comply with the assignment's requirement that the design diagram be self-drawn by the student.

---

## Output Quality Bar

A high-quality generated suite:

- follows the actual specification
- has clear traceability
- covers meaningful partitions and risks
- distinguishes requirements from assumptions
- avoids invented expected behavior
- contains executable request detail
- exposes setup dependencies
- does not fabricate execution results
- stops before human-audit decisions
