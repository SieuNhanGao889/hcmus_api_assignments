# eshop-api-test-generator

## Skill này là gì

Một Agent Skill dùng để sinh **candidate test case** cho các API của EShop SUT trong HW06, dựa trên `docs/api_specification.md`. Skill chỉ làm đúng 1 việc:

```
Specification → Test Design → Candidate Test Cases
```

Không audit, không chạy Postman, không viết bug report.

## Áp dụng cho

- `POST /api/login`
- `POST /api/apply-coupon`
- `PUT /api/admin/orders/:id/status`

## Nó làm gì (7 bước)

1. **Parse spec** — bóc tách method, path, header, field, status code, schema. Nếu spec không định nghĩa → đánh dấu `unknown`, không đoán.
2. **Parameter partitions** — với mỗi input: valid, boundary, empty, missing, null, sai type, sai format...
3. **Security dimensions** — IDOR, role escalation, injection, token giả mạo... chỉ khi thực sự liên quan tới endpoint.
4. **Schema dimensions** — assertion về status/field/type, chỉ dùng field có trong spec.
5. **State dimensions** — transition hợp lệ/không hợp lệ, chỉ khi spec xác nhận; còn lại đánh dấu exploratory.
6. **Identify ambiguities** — liệt kê điều spec không nói rõ, không tự chế expected result.
7. **Generate cases** — ≥35 case/API, ưu tiên độ phủ hơn lặp biến thể.

## Output

Bảng test case gồm các field: `case_id`, `api`, `source`, `requirement_ref`, `security_ref`, `spec_reference`, `objective`, `preconditions`, `request_method/url/headers/params/body`, `expected_status`, `expected_response_or_assertion`, `coverage_type`, `assumption_or_open_question`, `audit_status`, `audit_notes`.

Mặc định:
- `source = AI_GENERATED`
- `audit_status = PENDING_HUMAN_REVIEW`
- `case_id` theo prefix: `LOGIN-GEN-001`, `COUPON-GEN-001`, `ORDERSTATUS-GEN-001`

## Nguyên tắc cốt lõi

- **Không bịa** status code, schema, response field không có trong spec.
- Khi spec không rõ status code → dùng `expected_status = SPEC_UNDEFINED` thay vì đoán, kèm giải thích ở `assumption_or_open_question`.
- **Không tự audit** — mọi case dừng ở `PENDING_HUMAN_REVIEW`, quyết định `VALID/INVALID/INCOMPLETE` thuộc về sinh viên.
- **Không nhận vơ** việc mình làm là đã thỏa mãn yêu cầu human-review của đề bài.
- Prefix `-EXT-` (manual extension) là của sinh viên tự viết — skill không được đụng vào hoặc gắn nhầm nhãn.

## Input cần có

- `README.md` — biết đang chọn API nào
- `docs/api_specification.md` — nguồn sự thật chính
- `docs/2026.HW06.API Testing_En.md` — chỉ đọc khi cần format/coverage riêng của đề

Skill không cache nội dung các file này — mỗi lần chạy phải đọc lại spec thật, không dùng trí nhớ từ lần trước.

## Cấu trúc thư mục

```text
agent-skill/
│
├── SKILL.md                    ← Định nghĩa Agent Skill tái sử dụng để sinh API test case
│
├── design/
│   ├── design.md               ← Giải thích thiết kế và luồng hoạt động của test generator
│   ├── pseudocode.md           ← Mã giả mô tả thuật toán sinh test case
│   └── diagram.png             ← Sơ đồ test generator do sinh viên tự vẽ
│
└── demo/
    ├── video_link.md          ← Link video demo Agent Skill
    └── prompt-demo.md          ← Prompt sử dụng để demo Agent Skill với một API
```

## Demo video
- Link: [https://youtu.be/4fAWYhzeHzQ](https://youtu.be/4fAWYhzeHzQ)
