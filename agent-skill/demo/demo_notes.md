# Demo notes - AI-driven API Test Generator

## 1. Mục tiêu demo

Demo trình bày cách Agent Skill `eshop-api-test-generator` hỗ trợ sinh candidate API test cases từ đặc tả EShop. Demo nên nhấn mạnh rằng AI chỉ sinh bản nháp có traceability; sinh viên vẫn phải review, audit và bổ sung manual extension cases.

Endpoint nên dùng để demo: `POST /api/login`, vì workflow đã có đủ artifact từ phân tích spec đến manual extension.

## 2. File nên mở trong demo

| File | Nội dung cần chỉ |
|---|---|
| `docs/api_specification.md` | Phần `POST /api/login`, body `email/password`, response có `token` và `user` |
| `agent-skill/SKILL.md` | Quy tắc generator, case format, `PENDING_HUMAN_REVIEW` |
| `reports/analysis/login_spec_analysis.md` | Kết quả `PHASE_1_SPEC_ANALYSIS` |
| `reports/analysis/login_test_design.md` | Coverage dimensions ở `PHASE_2_TEST_DESIGN` |
| `test-cases/login_test_cases.xlsx` | `40` AI-generated candidate cases |
| `test-cases/login_test_cases_audited.xlsx` | Human audit và `5` manual extension cases |
| `reports/ai_audit_report.md` | Log quá trình dùng AI |

## 3. Kịch bản demo ngắn

### Bước 1 - Giới thiệu input

Mở `docs/api_specification.md` và chỉ endpoint `POST /api/login`. Nói rõ spec chỉ định body gồm `email`, `password`, response thành công có `token` và `user`, nhưng không nêu rõ nhiều error cases.

### Bước 2 - Giới thiệu Agent Skill

Mở `agent-skill/SKILL.md` và chỉ các điểm:

- Generator đọc spec trước.
- Không đoán status/schema khi spec không rõ.
- Sinh ít nhất `35` candidate cases cho mỗi API trong HW06.
- Mọi AI-generated case phải có `audit_status = PENDING_HUMAN_REVIEW`.
- Manual extension cases không được gán nhãn nhầm thành AI-generated.

### Bước 3 - Chạy theo phase

Trình bày các prompt đã dùng trong `agent-skill/demo/prompt-demo.md`:

1. `PHASE_1_SPEC_ANALYSIS`: phân tích endpoint.
2. `PHASE_2_TEST_DESIGN`: thiết kế partitions/security/schema/state coverage.
3. `PHASE_3_AI_GENERATION`: sinh candidate test cases.
4. `PHASE_4_HUMAN_AUDIT`: ghi audit status do sinh viên review.
5. `PHASE_5_HUMAN_EXTENSION`: thêm cases do sinh viên tự chọn.

### Bước 4 - Show output

Mở `test-cases/login_test_cases.xlsx` và chỉ:

- Có `40` rows AI-generated.
- `source = AI_GENERATED`.
- `audit_status = PENDING_HUMAN_REVIEW`.
- Có traceability qua `requirement_ref`, `security_ref`, `spec_reference`.

Sau đó mở `test-cases/login_test_cases_audited.xlsx` và chỉ:

- Human audit có `38` `VALID`, `2` `INCOMPLETE`, `0` `INVALID`.
- Các correction nằm trong `human_correction`, không overwrite nội dung AI gốc.
- Có `5` rows `LOGIN-EXT-001` đến `LOGIN-EXT-005` với `source = MANUAL_EXTENSION`.

### Bước 5 - Nói rõ giới hạn

Kết luận demo bằng việc nói rõ:

- Demo này chứng minh generator hỗ trợ thiết kế test cases.
- Demo chưa chứng minh test pass/fail vì chưa chạy Postman/Newman.
- Chưa có bug confirmed hoặc CI/CD evidence.
- Các phần execute, Newman report, bug analysis và CI/CD là các phase sau.

## 4. Câu nói gợi ý khi quay demo

```text
Trong phần demo này, em dùng Agent Skill để biến đặc tả API thành bộ candidate test cases có traceability. Với Login, AI không tự quyết định case nào đúng hay sai mà chỉ dừng ở PENDING_HUMAN_REVIEW. Sau đó em thực hiện human audit, ghi correction cho các case incomplete và tự thiết kế thêm 5 manual extension cases để lấp các gap còn lại.
```

```text
Điểm quan trọng là workflow tách rõ AI-generated content, human audit và manual extension. Vì em chưa chạy Postman/Newman trong phần này, báo cáo không ghi pass/fail hoặc bug confirmed.
```

## 5. Checklist demo

- Mở đúng endpoint trong spec.
- Mở `SKILL.md` để chứng minh generator có quy tắc.
- Mở diagram tự vẽ và chỉ rõ ranh giới `AI Test Generator` / `Human Responsibility`.
- Mở analysis và test design để chứng minh không sinh test một bước.
- Mở Excel AI-generated để chỉ `PENDING_HUMAN_REVIEW`.
- Mở Excel audited để chỉ human audit và manual extension.
- Không nói rằng test đã pass/fail nếu chưa có Newman evidence.
- Không nói diagram đã hoàn tất nếu chưa tự vẽ file ảnh theo yêu cầu đề.
