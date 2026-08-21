# Agent Skill Demo Notes

## Demo video

Video link: [https://youtu.be/4fAWYhzeHzQ](https://youtu.be/4fAWYhzeHzQ)

## Demo scope

- Agent Skill: `agent-skill/SKILL.md`
- Demo API: `POST /api/login`
- Input specification: `docs/api_specification.md`
- Design artifact: `agent-skill/design/design.md`
- Diagram artifact: `agent-skill/design/diagram.png`
- Pseudocode artifact: `agent-skill/design/pseudocode.md`

Demo này tập trung chứng minh Agent Skill có thể hỗ trợ workflow:

```text
API Specification
  -> Specification Reader
  -> Rule Extractor
  -> Test Dimension Builder
  -> Candidate Test Generator
  -> Traceability Validator
  -> Structured Test Cases
  -> PENDING_HUMAN_REVIEW
```

Sau `PENDING_HUMAN_REVIEW`, sinh viên tự review và phân loại test cases thành `VALID`, `INVALID` hoặc `INCOMPLETE`.

## What the demo shows

1. Mở `agent-skill/SKILL.md` để giới thiệu quy tắc generator.
2. Mở `docs/api_specification.md` và chọn endpoint `POST /api/login`.
3. Mở `agent-skill/design/diagram.png` hoặc `design.md` để giải thích pipeline.
4. Trình bày cách AI sinh candidate test cases có traceability.
5. Mở `test-cases/login_test_cases.xlsx` để chỉ các cột chính:
   - `case_id`
   - `source`
   - `spec_reference`
   - `coverage_type`
   - `expected_status`
   - `expected_response_or_assertion`
   - `audit_status`
6. Chỉ rõ các AI-generated rows có `source = AI_GENERATED` và `audit_status = PENDING_HUMAN_REVIEW`.
7. Mở `test-cases/login_test_cases_audited.xlsx` để minh họa human audit là bước riêng biệt.

## Human-review boundary

Agent Skill không tự phê duyệt test cases. Output của AI chỉ là candidate test cases. Sinh viên chịu trách nhiệm:

- Human audit.
- Manual extension cases.
- Postman/Newman execution.
- Bug classification.
- CI/CD evidence.

## Evidence boundary

Demo này không claim:

- Test cases đã pass.
- Test cases đã fail.
- Bug đã được xác nhận.
- Newman report đã được tạo.
- CI/CD pipeline đã chạy.

Các kết quả pass/fail, bug report, screenshot và CI/CD link chỉ được ghi nhận sau khi có evidence thực tế.

## Demo checklist

| Item | Status |
|---|---|
| `SKILL.md` is included | Done |
| `design.md` is included | Done |
| `diagram.png` self-drawn diagram is included | Done |
| `pseudocode.md` is included | Done |
| Demo uses `POST /api/login` | Done |
| AI-generated rows remain `PENDING_HUMAN_REVIEW` | Done |
| Video link added | [https://youtu.be/4fAWYhzeHzQ](https://youtu.be/4fAWYhzeHzQ) |
