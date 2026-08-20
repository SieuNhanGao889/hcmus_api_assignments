# Thiết kế AI-driven API Test Generator

## 1. Mục tiêu

Generator được thiết kế để nhận đặc tả API của EShop SUT và sinh candidate test cases có traceability cho một endpoint được chọn. Công cụ không thay thế human review: mọi case do AI sinh phải dừng ở `PENDING_HUMAN_REVIEW`.

Pipeline chính:

`API Specification -> Test Design -> Candidate Test Cases -> PENDING_HUMAN_REVIEW`

Sau điểm này, sinh viên tự review và quyết định `VALID`, `INVALID` hoặc `INCOMPLETE`.

## 2. Input

| Input | Vai trò |
|---|---|
| `docs/api_specification.md` | Nguồn sự thật chính về endpoint, request, response và business rule |
| `README.md` | Xác định API được chọn, `studentId`, base URL và cấu trúc nộp bài |
| `docs/2026.HW06.API Testing_En.md` | Yêu cầu bài tập, evidence, Postman/Newman, CI/CD và Agent Skill |
| Target endpoint | Ví dụ `POST /api/login` |

## 3. Output

Output là bảng test case có các cột chính:

`case_id`, `api`, `source`, `requirement_ref`, `security_ref`, `spec_reference`, `objective`, `preconditions`, `request_method`, `request_url`, `headers`, `path_params`, `query_params`, `body`, `expected_status`, `expected_response_or_assertion`, `coverage_type`, `assumption_or_open_question`, `audit_status`, `audit_notes`.

Mặc định:

- `source = AI_GENERATED`
- `audit_status = PENDING_HUMAN_REVIEW`
- `expected_status = SPEC_UNDEFINED` nếu đặc tả không xác định rõ status code.

## 4. Pipeline xử lý

```text
API Specification
       ↓
Specification Reader
       ↓
Rule Extractor
       ↓
Test Dimension Builder
  ├─ Domain / Boundary
  ├─ Security
  ├─ Schema
  ├─ State
  └─ Robustness / Parser
       ↓
Candidate Test Generator
       ↓
Traceability Validator
       ↓
Structured Test Cases
PENDING_HUMAN_REVIEW
       ║
       ║ Human boundary
       ↓
Student Review
├─ VALID
├─ INVALID
└─ INCOMPLETE
```

Diagram nộp cuối cần được sinh viên tự vẽ lại bằng công cụ vẽ bất kỳ. Sơ đồ text ở trên chỉ là mô tả thiết kế. Khi vẽ, nên dùng đường phân cách nét đứt trước `Student Review` để thể hiện phần bên trái là `AI Test Generator`, còn phần bên phải là `Human Responsibility`.

## 5. Thành phần chính

### 5.1 Specification Reader

Đọc đúng endpoint cần sinh test và bỏ qua các API khác trong spec. Ví dụ:

- `POST /api/login`
- `POST /api/apply-coupon`
- `PUT /api/admin/orders/:id/status`

Reader trích xuất:

- Method / Path
- Parameters
- Auth
- Response
- Business Rules

### 5.2 Rule Extractor

Chuyển thông tin từ specification thành tập rule có cấu trúc. Nếu thông tin không có trong spec, rule được phân loại là `SPEC_UNDEFINED` thay vì đoán.

Rule extractor phân biệt:

- `SPEC_DEFINED`: behavior được đặc tả trực tiếp.
- `ASSIGNMENT_REQUIRED`: behavior đến từ yêu cầu bài tập, ví dụ header `X-Student-Id`.
- `SPEC_UNDEFINED`: behavior chưa được đặc tả.
- `EXPLORATORY`: case quan sát hữu ích nhưng không có expected result cứng.

### 5.3 Test Dimension Builder

Tạo các vùng kiểm thử chính:

- Domain / Boundary
- Security
- Schema
- State
- Robustness / Parser

Không tạo dimension không phù hợp chỉ để tăng số lượng test case.

### 5.4 Candidate Test Generator

Kết hợp endpoint rules và test dimensions để sinh candidate cases. Mỗi case cần:

- Objective rõ ràng.
- Request cụ thể.
- Preconditions.
- Expected assertion có căn cứ.
- Traceability.
- Uncertainty note khi spec không đủ.

### 5.5 Traceability Validator

Kiểm tra:

- ID duy nhất.
- Case đúng endpoint.
- Không invent field ngoài spec trừ exploratory/security case.
- Expected status có căn cứ.
- Security test phù hợp attack surface.
- State case có precondition.
- Mọi AI case vẫn là `PENDING_HUMAN_REVIEW`.

### 5.6 Export Layer

Xuất test cases sang format dễ dùng tiếp:

- Markdown table
- CSV / Excel
- Postman-ready structured data

Execution bằng Postman/Newman nằm ngoài core generator và thuộc workflow của `AGENT.md`.

## 6. Human Review Boundary

Skill phải dừng tại:

`PENDING_HUMAN_REVIEW`

Sau đó sinh viên tự audit:

- `VALID`
- `INVALID`
- `INCOMPLETE`

Skill chỉ được hỗ trợ correction sau khi sinh viên đã đưa ra quyết định.

## 7. Error / Ambiguity Handling

Nếu specification không định nghĩa:

- exact error status
- error schema
- transition matrix
- normalization rule
- rate-limit threshold
- rounding rule

thì skill:

1. Không tự bịa expected behavior.
2. Dùng `SPEC_UNDEFINED`.
3. Ghi rõ assumption/open question.
4. Có thể tạo exploratory case nếu có giá trị kiểm thử.

## 8. Cách áp dụng trong bài làm

Trong bài này, skill đã được dùng theo từng phase:

1. `PHASE_1_SPEC_ANALYSIS`: tạo phân tích spec.
2. `PHASE_2_TEST_DESIGN`: tạo test-design dimensions.
3. `PHASE_3_AI_GENERATION`: sinh candidate cases.
4. `PHASE_4_HUMAN_AUDIT`: ghi lại audit/correction do sinh viên cung cấp.
5. `PHASE_5_HUMAN_EXTENSION`: thêm manual extension cases do sinh viên chọn.

Kết quả hiện tại:

- Login: `40` AI-generated + `5` manual extension.
- Coupon: `40` AI-generated + `5` manual extension.
- Admin Order Status: `42` AI-generated + `5` manual extension.

## 9. Giới hạn

Skill không:

- Tự human-audit.
- Tự tạo manual extension cases.
- Chạy Postman/Newman.
- Tạo screenshot.
- Tạo GitHub Issue.
- Tạo CI/CD evidence.
- Kết luận bug khi chưa có execution evidence.
