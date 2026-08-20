---
name: eshop-api-test-generator
description: Generate, audit, and extend API test cases for the HW06 EShop API Testing assignment, focused on the selected login, coupon, and admin order-status APIs.
metadata:
  short-description: HW06 EShop API test generator
---

# EShop API Test Generator

Use this skill when working on HW06 API Testing deliverables for the EShop SUT, especially when generating test cases, auditing AI output, extending coverage, preparing Postman/Newman artifacts, or drafting the test-generator demo.

This skill is scoped to the repository assignment and the selected APIs:

- `POST /api/login`
- `POST /api/apply-coupon`
- `PUT /api/admin/orders/:id/status`

Do not use this skill to fabricate execution evidence. Newman reports, screenshots, GitHub Issues, commit logs, and CI/CD run links must come from real execution.

## Required Context

Before generating final artifacts, read these files from the submission repository root:

- `README.md` for the selected API scope and deliverables.
- `docs/api_specification.md` for endpoint details.

Read `docs/2026.HW06.API Testing_En.md` when the task involves grading criteria, submission packaging, AI audit, AI critique, CI/CD evidence, or the Agent Skill section.

## Core Requirements

For each selected API:

- Generate at least 35 AI-generated test cases.
- Audit every generated test case as `VALID`, `INVALID`, or `INCOMPLETE`.
- Correct invalid or incomplete cases.
- Add at least 5 manual extension cases.
- Cover domain partitions, boundary values, missing/null/wrong-type fields, security, schema validation, and state behavior where applicable.
- Include `X-Student-Id: 23127364` in every executable request.

Across the full suite:

- Prefer Postman + Newman artifacts unless the user explicitly chooses another framework.
- Use variables for `baseUrl`, `studentId`, tokens, user IDs, coupon codes, order IDs, and reusable setup data.
- Keep generated, audited, and manually extended cases distinguishable.
- Be explicit when a case depends on seed data, admin credentials, or a previous request.

## Test Case Format

Use a table or spreadsheet-ready rows with these fields:

- `case_id`
- `api`
- `source`
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
- `audit_status`
- `audit_notes`

Use case ID prefixes:

- `LOGIN-GEN-001` for generated login cases.
- `LOGIN-EXT-001` for manual login extensions.
- `COUPON-GEN-001` for generated coupon cases.
- `COUPON-EXT-001` for manual coupon extensions.
- `ORDERSTATUS-GEN-001` for generated admin order-status cases.
- `ORDERSTATUS-EXT-001` for manual admin order-status extensions.

## Coverage Heuristics

### Login

Exercise:

- Valid email and password.
- Invalid email format, empty email, missing email, null email, wrong type email.
- Wrong password, empty password, missing password, null password, wrong type password.
- Non-existing account.
- SQL injection and script-like payloads.
- Case sensitivity and whitespace around credentials.
- Lockout or rate-limit behavior if implemented.
- JWT presence, user object shape, and sensitive field leakage.

### Apply Coupon

Exercise:

- Valid coupon and correct final amount calculation.
- Unknown coupon, expired coupon, lowercase coupon, whitespace, empty, missing, null, and wrong-type code.
- `total_amount` as zero, negative, below minimum, exactly minimum, above minimum, decimal, huge number, string, missing, and null.
- `user_id` valid, missing, null, wrong type, non-existing, and another user's ID.
- Max uses per user.
- SQL injection in coupon code.
- Response schema: `discount_amount` and `final_amount`.

### Admin Order Status

Exercise:

- Valid transitions: `pending -> confirmed -> shipping -> delivered`.
- Invalid reverse transitions.
- Invalid direct skips if business rules forbid them.
- Transition after `delivered`.
- Unknown status, empty status, missing status, null status, wrong type status.
- Unknown order ID, negative ID, zero ID, string ID, and injection-style ID.
- No token, malformed token, expired token, normal user token, and admin token.
- Persisted state verification after update.

## Audit Rules

Label a generated case:

- `VALID` if it is aligned with the spec, executable, and has clear assertions.
- `INVALID` if it contradicts the spec, requires unsupported fields, or expects impossible behavior.
- `INCOMPLETE` if it lacks setup data, request details, expected status, or assertions.

When fixing cases, keep the original test intent but update the request, preconditions, or expected assertions. If the spec is ambiguous, mark the assumption and design the case to reveal the implementation behavior.

## Postman/Newman Guidance

When producing Postman assets or instructions:

- Define `baseUrl` as `http://localhost:3000`.
- Define `studentId` as `23127364`.
- Add `X-Student-Id: {{studentId}}` to all requests.
- Store login tokens in collection or environment variables.
- Use setup requests for users, coupons, and orders when needed.
- Use test scripts for HTTP status, response schema, business calculations, and persisted-state checks.
- Use data-driven runs for boundary and negative partitions when practical.

Recommended Newman command:

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  -e postman/HW06_Local.postman_environment.json \
  -r cli,html \
  --reporter-html-export reports/newman/HW06_EShop_API_Tests.html
```

## Demo Generator Design

When drafting the Agent Skill or generator design section, describe a pipeline like this:

1. Parse API specification.
2. Select target APIs from the assignment scope.
3. Extract parameters, authentication requirements, response schemas, and business rules.
4. Generate candidate cases from domain, boundary, security, schema, and state-transition strategies.
5. Audit cases against the specification.
6. Add manual extension cases for risks the generator missed.
7. Export cases to Markdown, CSV/Excel, and Postman collection format.
8. Execute with Newman and collect real evidence.

Keep the diagram and pseudocode honest: they may describe the designed generator, but real reports and screenshots must be produced by running the tests.
