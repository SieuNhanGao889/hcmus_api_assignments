# HW06 - API Testing Report

- MSSV: `23127364`
- SUT: EShop API
- Base URL dự kiến: `http://localhost:3000`
- Header bắt buộc khi thực thi: `X-Student-Id: 23127364`
- API đã chọn: `POST /api/login`, `POST /api/apply-coupon`, `PUT /api/admin/orders/:id/status`

## 1. Introduction

Bài làm này áp dụng quy trình kiểm thử API theo hướng AI-assisted nhưng vẫn giữ vai trò quyết định của sinh viên. AI được dùng để đọc đặc tả, phân tích tham số, thiết kế hướng kiểm thử và sinh candidate test cases. Sau đó sinh viên thực hiện human audit, chọn các khoảng trống coverage còn lại và bổ sung manual extension cases.

Phạm vi hiện tại đã hoàn thành đến `PHASE_5_HUMAN_EXTENSION` cho cả 3 API. Chưa có thực thi Postman/Newman, chưa có evidence chạy test, chưa có bug được xác nhận và chưa có CI/CD run thật.

## 2. Test Environment and Tools

| Thành phần | Giá trị |
|---|---|
| API specification | `docs/api_specification.md` |
| Assignment description | `docs/2026.HW06.API Testing_En.md` |
| Test case format | Excel workbook `.xlsx` |
| AI workflow artifact | `reports/ai_audit_report.md` |
| Agent Skill | `agent-skill/SKILL.md` |
| Planned API tool | Postman + Newman |
| Execution status | Chưa thực hiện |

Các request plan trong test cases được thiết kế để sau này chuyển sang Postman/Newman và luôn cần header `X-Student-Id: 23127364`.

## 3. API Selection

| Pool | API | Lý do chọn |
|---|---|---|
| Pool A | `POST /api/login` | API xác thực quan trọng, có rủi ro credential handling, token, schema và account enumeration. |
| Pool B | `POST /api/apply-coupon` | API nghiệp vụ tính tiền, có rủi ro boundary, discount formula, usage limit và client-side override. |
| Pool C | `PUT /api/admin/orders/:id/status` | API admin nhạy cảm, có rủi ro authorization, role escalation, state transition và persisted-state integrity. |

## 4. API 1 - Login

### 4.1 API Analysis

Artifact phân tích: `reports/analysis/login_spec_analysis.md`.

`POST /api/login` nhận body JSON gồm `email` và `password`. Theo đặc tả, response thành công `200 OK` trả về `token` dạng JWT và thông tin `user`. Đặc tả không nêu rõ error schema, status code cho credential sai, token claims, lockout threshold hoặc behavior với whitespace/case sensitivity.

### 4.2 AI-Assisted Test Generation

Artifact thiết kế: `reports/analysis/login_test_design.md`.

File AI-generated: `test-cases/login_test_cases.xlsx`.

AI đã sinh `40` candidate test cases với `source = AI_GENERATED` và `audit_status = PENDING_HUMAN_REVIEW`. Coverage chính gồm happy path, invalid credential, missing/empty/null/wrong-type fields, malformed email, whitespace/case behavior, injection-style inputs, schema checks, sensitive-data leakage, malformed JSON và exploratory lockout/rate-limit.

### 4.3 Human Audit

File audited: `test-cases/login_test_cases_audited.xlsx`.

Kết quả human audit:

| Audit status | Số lượng |
|---|---:|
| `VALID` | `38` |
| `INCOMPLETE` | `2` |
| `INVALID` | `0` |

Hai case `LOGIN-GEN-002` và `LOGIN-GEN-040` giữ nguyên nội dung AI gốc, đồng thời ghi correction được duyệt trong `human_correction`.

### 4.4 Human Extension

Sinh viên chọn và bổ sung đúng `5` manual extension cases:

| Case ID | Ý tưởng |
|---|---|
| `LOGIN-EXT-001` | Cross-request token usability |
| `LOGIN-EXT-002` | Token/user consistency |
| `LOGIN-EXT-003` | Duplicate JSON keys |
| `LOGIN-EXT-004` | Paired account enumeration observation |
| `LOGIN-EXT-005` | HTTP parser error leakage |

Các case này có `source = MANUAL_EXTENSION` và được tách khỏi AI-generated baseline.

### 4.5 Test Execution

Chưa thực hiện. Chưa có Postman collection, Newman HTML report hoặc execution evidence thật cho Login trong repo tại thời điểm viết báo cáo này.

### 4.6 Findings and Bugs

Chưa ghi nhận bug confirmed vì chưa chạy test và chưa reproduce behavior thực tế so với đặc tả.

## 5. API 2 - Coupon

### 5.1 API Analysis

Artifact phân tích: `reports/analysis/coupon_spec_analysis.md`.

`POST /api/apply-coupon` nhận body JSON gồm `code`, `total_amount`, `user_id`. Đặc tả mô tả API tính tổng tiền sau giảm và trả JSON có `discount_amount`, `final_amount`. Đặc tả chưa nói rõ status code lỗi, công thức rounding, authentication requirement cho `user_id`, hoặc việc usage limit có được persist khi apply coupon hay không.

### 5.2 AI-Assisted Test Generation

Artifact thiết kế: `reports/analysis/coupon_test_design.md`.

File AI-generated: `test-cases/coupon_test_cases.xlsx`.

AI đã sinh `40` candidate test cases với coverage về valid coupon, unknown/empty/null/wrong-type code, amount boundaries, user identifier, expired/min-order/usage-limit exploratory behavior, injection-style input, extra fields, malformed JSON và response schema.

### 5.3 Human Audit

File audited: `test-cases/coupon_test_cases_audited.xlsx`.

Kết quả human audit:

| Audit status | Số lượng |
|---|---:|
| `VALID` | `36` |
| `INCOMPLETE` | `4` |
| `INVALID` | `0` |

Các case `INCOMPLETE` chủ yếu phụ thuộc fixture hoặc state chưa được đặc tả rõ như `min_order_amount`, `max_uses_per_user` và cross-user behavior.

### 5.4 Human Extension

Sinh viên chọn và bổ sung đúng `5` manual extension cases:

| Case ID | Ý tưởng |
|---|---|
| `COUPON-EXT-001` | Discount formula with controlled coupon fixture |
| `COUPON-EXT-002` | Minimum-order enforcement with verified fixture |
| `COUPON-EXT-003` | Usage-limit replay with persisted verification |
| `COUPON-EXT-004` | Client-side override of coupon rule |
| `COUPON-EXT-005` | Coupon code duplicate keys |

Các case này tập trung vào business-rule precision và parser/security edge cases mà AI baseline chưa bao phủ đủ sâu.

### 5.5 Test Execution

Chưa thực hiện. Chưa có Postman/Newman output hoặc screenshot execution cho Coupon.

### 5.6 Findings and Bugs

Chưa có bug confirmed cho Coupon vì chưa có bằng chứng chạy test.

## 6. API 3 - Admin Order Status

### 6.1 API Analysis

Artifact phân tích: `reports/analysis/admin_order_status_spec_analysis.md`.

`PUT /api/admin/orders/:id/status` yêu cầu `Authorization: Bearer <token>` và tài khoản có quyền Admin. Body có `status` với các giá trị `pending`, `confirmed`, `shipping`, `delivered`, `canceled`. Đặc tả chưa nêu response schema, status code lỗi, transition matrix đầy đủ hoặc quyền override của admin với trạng thái terminal.

### 6.2 AI-Assisted Test Generation

Artifact thiết kế: `reports/analysis/admin_order_status_test_design.md`.

File AI-generated: `test-cases/admin_order_status_test_cases.xlsx`.

AI đã sinh `42` candidate test cases với coverage về admin authorized update, authentication/authorization negative cases, path `id` validation, `status` validation, extra fields, malformed JSON, content type, state transitions và persisted-state exploratory checks.

### 6.3 Human Audit

File audited: `test-cases/admin_order_status_test_cases_audited.xlsx`.

Kết quả human audit:

| Audit status | Số lượng |
|---|---:|
| `VALID` | `41` |
| `INCOMPLETE` | `1` |
| `INVALID` | `0` |

Case `ORDERSTATUS-GEN-031` được giữ nguyên nội dung AI gốc và ghi correction để tách rõ authorization bypass với extra-field elevation khi triển khai sau.

### 6.4 Human Extension

Sinh viên chọn và bổ sung đúng `5` manual extension cases:

| Case ID | Ý tưởng |
|---|---|
| `ORDERSTATUS-EXT-001` | Split authorization bypass and extra-field elevation |
| `ORDERSTATUS-EXT-002` | Persisted-state verification after failed authorization |
| `ORDERSTATUS-EXT-003` | Matrix-based transition coverage with controlled fixtures |
| `ORDERSTATUS-EXT-004` | Cancellation rule consistency |
| `ORDERSTATUS-EXT-005` | Duplicate status keys |

Các case này tập trung vào authorization, persisted state và state-machine behavior.

### 6.5 Test Execution

Chưa thực hiện. Chưa có Postman/Newman output hoặc evidence chạy API admin order status.

### 6.6 Findings and Bugs

Chưa có bug confirmed cho Admin Order Status vì chưa có execution evidence.

## 7. Postman Features Used

Hiện tại chưa triển khai Postman collection nên chưa thể claim feature đã dùng. Khi chuyển sang `PHASE_6_POSTMAN_IMPLEMENTATION`, các feature dự kiến phù hợp gồm:

- Collection variables: `baseUrl`, `studentId`, token, user/order/coupon IDs.
- Environment variables cho local SUT.
- Pre-request scripts để inject `X-Student-Id: {{studentId}}` và chuẩn bị token khi cần.
- Data-driven runs cho partitions có thể lặp lại.
- Test scripts để assert status, schema, security leakage và state consistency.
- Newman HTML reporter để tạo evidence thực thi.

## 8. CI/CD Integration

Chưa thực hiện CI/CD. Repo chưa có workflow thật trong `.github/workflows` và chưa có passing/failing pipeline evidence. Khi làm Phase 9, pipeline nên cài Newman, chạy collection với environment, xuất HTML report và upload artifact. Hai run bắt buộc của đề là một run pass và một run fail có chủ đích; các run này chưa tồn tại nên không được ghi là hoàn thành.

## 9. AI-Driven API Test Generator

Agent Skill được định nghĩa tại `agent-skill/SKILL.md`. Thiết kế của skill đi theo pipeline:

`API Specification -> Endpoint Parser -> Rule Extractor -> Coverage Dimensions -> Candidate Case Generator -> Traceability Validator -> PENDING_HUMAN_REVIEW`.

Các artifact hỗ trợ:

- `agent-skill/README.md`: mô tả cách dùng skill.
- `agent-skill/design/design.md`: thiết kế generator.
- `agent-skill/design/pseudocode.md`: pseudocode.
- `agent-skill/design/diagram_drawing_guide.md`: hướng dẫn bố cục để sinh viên tự vẽ diagram.
- `agent-skill/scripts/generate_api_tests.py`: prototype script minh họa luồng tạo candidate rows.
- `agent-skill/demo/prompt-demo.md`: prompt demo đã dùng.
- `agent-skill/demo/demo_notes.md`: ghi chú demo.
- `agent-skill/demo/demo_script.md`: script nói khi quay demo.

Lưu ý: diagram nộp cuối phải do sinh viên tự vẽ theo yêu cầu đề bài. Nội dung trong repo chỉ mô tả design để hỗ trợ sinh viên vẽ lại.

## 10. Test Summary

| API | AI-generated | Manual extension | Total designed | `VALID` | `INCOMPLETE` | `INVALID` | Executed | Passed | Failed | Confirmed bugs |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `POST /api/login` | `40` | `5` | `45` | `38` | `2` | `0` | `0` | `0` | `0` | `0` |
| `POST /api/apply-coupon` | `40` | `5` | `45` | `36` | `4` | `0` | `0` | `0` | `0` | `0` |
| `PUT /api/admin/orders/:id/status` | `42` | `5` | `47` | `41` | `1` | `0` | `0` | `0` | `0` | `0` |
| Total | `122` | `15` | `137` | `115` | `7` | `0` | `0` | `0` | `0` | `0` |

## 11. Bug Summary

Chưa có bug được xác nhận. Các file trong `bug-reports/` hiện chưa có report thật vì chưa thực thi test. Nếu sau này có lỗi, mỗi bug cần có request, expected result, actual result, evidence screenshot/Newman output và GitHub Issue link thật.

## 12. AI Usage, Audit, and Limitations

Toàn bộ quá trình dùng AI được ghi tại `reports/ai_audit_report.md`. Vai trò của AI là hỗ trợ phân tích, sinh candidate cases, ghi correction theo human audit và soạn báo cáo. AI không tự phê duyệt case thay sinh viên, không tự kết luận bug và không tạo bằng chứng thực thi giả.

Hạn chế chính ở thời điểm này là pipeline mới dừng ở thiết kế test. Các phần còn thiếu để hoàn tất bài nộp đầy đủ gồm Postman implementation, execution evidence, Newman report, bug analysis nếu có lỗi thật, CI/CD runs và PDF export của báo cáo.

## 13. Conclusion

Bài làm đã hoàn thành phần AI-assisted specification analysis, test design, candidate generation, human audit và manual extension cho cả ba API đã chọn. Tổng cộng có `137` test cases được thiết kế, trong đó `122` AI-generated cases và `15` manual extension cases. Các artifact hiện tại đủ để tiếp tục sang Postman/Newman implementation, nhưng chưa đủ để claim kết quả pass/fail hoặc bug confirmed.
