## Prompt 1 — Login analysis
```
Start PHASE_1_SPEC_ANALYSIS for POST /api/login.


Follow AGENT.md and use the repository sources defined there.


Save the result to:
reports/analysis/login_spec_analysis.md


Do not generate test cases yet.
Stop after PHASE_1_SPEC_ANALYSIS.
```
## Prompt 2 — Login test design

Sau khi bạn xem analysis:
```
Proceed to PHASE_2_TEST_DESIGN for POST /api/login.


Use the approved specification analysis.


Save the result to:
reports/analysis/login_test_design.md


Do not generate test cases yet.
Stop after PHASE_2_TEST_DESIGN.
```
Bạn xem file này. Nếu ổn → generate.

## Prompt 3 — Generate
```
The Login test design has been reviewed and approved.


Proceed to PHASE_3_AI_GENERATION for POST /api/login.


Generate the required candidate test cases according to SKILL.md.


Save them to:
test-cases/login_test_cases.xlsx


Stop after PHASE_3_AI_GENERATION.
```

## Prompt 4: Sau khi audit xong
```
Proceed to PHASE_4_HUMAN_AUDIT for POST /api/login.

Use my completed human audit in:
test-cases/login_test_cases_audited.xlsx

Follow AGENT.md.

Preserve all original AI-generated content, audit_status, and audit_notes.

For INVALID or INCOMPLETE cases, record the approved correction in:
human_correction

Do not overwrite the original AI-generated content.
Do not change my audit decisions.
Do not add extension cases or execute tests.

Stop after PHASE_4_HUMAN_AUDIT.
```
## Prompt 5 — tìm gap
```
Proceed to preparation for PHASE_5_HUMAN_EXTENSION for POST /api/login.


Analyze the audited test suite and identify remaining coverage gaps.


Save the analysis to:
reports/analysis/login_extension_gaps.md


Do not generate manual extension cases.
```
Sau đó bạn tự chọn ≥5 case.

Rồi:
```
Proceed to PHASE_5_HUMAN_EXTENSION.


These are my student-designed extension cases:


1. ...
2. ...
3. ...
4. ...
5. ...


Add them to:
test-cases/login_test_cases.xlsx


Follow AGENT.md for formatting and traceability.
Stop after PHASE_5_HUMAN_EXTENSION.
```
## Vậy toàn bộ Login thực chất chỉ là:
```
Prompt 1
"Analyze Login"
      ↓
Prompt 2
"Design Login tests"
      ↓
    YOU REVIEW
      ↓
Prompt 3
"Generate Login tests"
      ↓
    YOU AUDIT
      ↓
Prompt 4
"Process my audit"
      ↓
Prompt 5
"Find remaining gaps"
      ↓
 YOU DESIGN ≥5
      ↓
Prompt 6
"Add my extension cases"
```