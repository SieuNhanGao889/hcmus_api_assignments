# Agent Skill Demo Script

## Target duration

3-5 phút.

## Demo goal

Trình bày rằng Agent Skill có thể đọc API specification và sinh structured candidate test cases cho một API, đồng thời vẫn giữ ranh giới human review.

API dùng trong demo:

`POST /api/login`

## 1. Introduction - 20-30 giây

Nói:

```text
Đây là Agent Skill em thiết kế cho HW06. Mục tiêu của skill là nhận API specification và một endpoint mục tiêu, sau đó phân tích specification và tạo candidate API test cases có traceability. Skill chỉ dừng ở PENDING_HUMAN_REVIEW, không tự thay thế phần human audit của sinh viên.
```

Show:

- `agent-skill/SKILL.md`
- `agent-skill/design/`

## 2. Show the input - 20-30 giây

Mở:

`docs/api_specification.md`

Đi tới:

`POST /api/login`

Nói:

```text
Trong demo này em dùng Login API. Input chính của generator là API specification và endpoint mục tiêu. Skill đọc method, request fields, authentication, response và các business/security rules được mô tả trong specification.
```

## 3. Show the generator design - 30-45 giây

Mở diagram tự vẽ.

Giải thích ngắn:

```text
Generator gồm các bước chính: đọc specification, extract rule, xây test dimensions, generate candidate cases và validate traceability. Các behavior không được specification định nghĩa sẽ được đánh dấu SPEC_UNDEFINED hoặc exploratory thay vì tự suy diễn expected result.
```

Chỉ vào các block:

- `Specification Reader`
- `Rule Extractor`
- `Test Dimension Builder`
- `Candidate Test Generator`
- `Traceability Validator`
- `PENDING_HUMAN_REVIEW`
- `Student Review`

## 4. Run the skill - 60-90 giây

Dùng prompt demo ngắn:

```text
Use agent-skill/SKILL.md.

Generate candidate API test cases for POST /api/login
from docs/api_specification.md.

Save the demo output separately so it does not overwrite
the audited assignment test cases.

Stop at PENDING_HUMAN_REVIEW.
```

Nếu cần lưu output demo riêng, dùng:

`agent-skill/demo/login_demo_test_cases.xlsx`

Nói trong lúc demo:

```text
Em dùng prompt ngắn vì các generation rules đã được định nghĩa trong SKILL.md. Điều này giúp skill reusable và tránh phải lặp lại một prompt dài cho mỗi lần sử dụng.
```

## 5. Show the output - 45-60 giây

Mở spreadsheet được sinh trong demo hoặc file đã có:

`test-cases/login_test_cases.xlsx`

Chỉ các cột:

- `case_id`
- `source = AI_GENERATED`
- `requirement_ref`
- `spec_reference`
- `coverage_type`
- request fields
- expected assertion
- `audit_status = PENDING_HUMAN_REVIEW`

Nói:

```text
Output không chỉ có test description mà còn giữ request, expected assertion, coverage type và traceability về specification. Quan trọng nhất, tất cả AI-generated cases vẫn ở PENDING_HUMAN_REVIEW.
```

## 6. Explain human-review boundary - 30-45 giây

Nói:

```text
Sau bước này generator dừng lại. Sinh viên mới là người review từng case và quyết định VALID, INVALID hoặc INCOMPLETE. Việc execution bằng Postman/Newman, bug classification và CI/CD cũng nằm ngoài core generator và phải dùng evidence thực tế.
```

Có thể mở thêm:

`test-cases/login_test_cases_audited.xlsx`

Chỉ nhanh:

- `38` `VALID`
- `2` `INCOMPLETE`
- `5` `MANUAL_EXTENSION`

Không cần review từng case trong video.

## 7. End - 15-20 giây

Nói:

```text
Như vậy Agent Skill tự động hóa phần specification-to-test-case generation nhưng vẫn giữ human accountability. Đây là phần em thiết kế để có thể tái sử dụng cho các API khác trong cùng SUT hoặc các bài API testing tương tự.
```

## Checklist trước khi quay

- `SKILL.md` có trong repo.
- Diagram đã được sinh viên tự vẽ.
- `pseudocode.md` có trong `agent-skill/design/`.
- Demo dùng một API: `POST /api/login`.
- Demo output không overwrite audited workbook.
- Generated rows dùng `AI_GENERATED`.
- Generated rows vẫn là `PENDING_HUMAN_REVIEW`.
- Không show Newman, screenshot, bug hoặc CI/CD evidence giả.
- Video link được thêm vào README/report sau khi upload.
