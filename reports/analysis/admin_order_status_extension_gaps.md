# PHASE_5_HUMAN_EXTENSION_PREPARATION - `PUT /api/admin/orders/:id/status`

## 1. Phạm vi

- `api`: `PUT /api/admin/orders/:id/status`
- `phase`: preparation for `PHASE_5_HUMAN_EXTENSION`
- `artifact`: `test-cases/admin_order_status_extension_gaps.md`
- `audited_suite`: `test-cases/admin_order_status_test_cases_audited.xlsx`
- `source_design`: `reports/analysis/admin_order_status_test_design.md`
- `student_id`: `23127364`

Mục tiêu của artifact này là phân tích bộ test đã được human audit và chỉ ra các khoảng trống còn lại để sinh viên có thể thiết kế manual extension cases ở bước sau.

Không tạo `ORDERSTATUS-EXT-*` trong artifact này. Không thay đổi audit decision. Không execute test. Không kết luận bug.

## 2. Tóm tắt bộ test đã audit

| Metric | Giá trị |
|---|---:|
| Tổng số AI-generated cases | `42` |
| `VALID` | `41` |
| `INCOMPLETE` | `1` |
| `INVALID` | `0` |
| `source` | `AI_GENERATED` |
| Manual extension đã có | `0` |

Case còn `INCOMPLETE` theo human audit:

| case_id | coverage_type | Lý do chính | Correction đã được ghi |
|---|---|---|---|
| `ORDERSTATUS-GEN-031` | `security,authorization,extra-field,negative` | Case trộn hai mục tiêu: normal user token bị từ chối và extra fields không nâng quyền | Tách thành hai ý khi triển khai sau: authorization bypass và unexpected security-sensitive fields |

## 3. Coverage hiện có

| Coverage group | Tình trạng |
|---|---|
| Admin authorized update | Có 5 allowed statuses |
| Authentication | Có missing, malformed, tampered token |
| Authorization | Có normal user token và own-order user token |
| Path `id` validation | Có unknown, zero, negative, decimal, string, very large, injection |
| `status` validation | Có missing/null/empty/whitespace/unknown/case/wrong type/injection |
| Extra fields | Có role/isAdmin và body `order_id` mismatch |
| HTTP/parser | Có malformed JSON, top-level array, wrong `Content-Type` |
| State transitions | Có forward, skip, reverse, cancel after delivered, repeated same status |
| Persistence | Có một case đọc order trước/sau update |

## 4. Remaining Coverage Gaps

### Gap 1 - Split authorization bypass and extra-field elevation

`ORDERSTATUS-GEN-031` bị audit `INCOMPLETE` vì trộn normal user token với extra `role/isAdmin`. Manual extension nên tách rõ hai case: normal user token bị từ chối dù body bình thường, và admin/normal user request có extra fields không thay đổi quyền.

Giá trị manual extension:

- Sửa điểm yếu đã được human audit chỉ ra.
- Traceability rõ hơn cho `authorization bypass` và `unexpected security-sensitive fields`.

Ranh giới:

- Không thay đổi case AI gốc.
- Nếu tạo manual extension sau này, cần ghi rõ vì sao tách.

### Gap 2 - Persisted-state verification after failed authorization

AI suite có auth negative cases nhưng chưa nhấn mạnh verify order state không đổi sau request bị từ chối.

Giá trị manual extension:

- Phát hiện bug nguy hiểm: response báo lỗi nhưng state vẫn bị update.
- Gắn security với persisted-state verification.

Ranh giới:

- Cần đọc order trước và sau bằng endpoint phù hợp.
- Không ghi pass/fail nếu chưa có execution.

### Gap 3 - Matrix-based transition coverage with controlled order fixtures

AI suite có một vài transition exploratory, nhưng chưa có matrix có kiểm soát cho từng trạng thái nguồn và trạng thái đích.

Giá trị manual extension:

- Bao phủ state machine có hệ thống.
- Phát hiện skip/reverse/terminal behavior bất thường.

Ranh giới:

- `api_specification.md` chưa định nghĩa transition matrix.
- Manual cases nên ghi exploratory hoặc dựa trên requirement bổ sung nếu có.

### Gap 4 - Terminal state immutability

AI suite có delivered -> pending và delivered -> canceled, nhưng chưa kiểm tra đầy đủ terminal behavior của cả `delivered` và `canceled` sang nhiều trạng thái khác.

Giá trị manual extension:

- Tăng coverage cho terminal-state risk.
- Bắt lỗi reopen/cancel sau terminal nếu requirement xác nhận.

Ranh giới:

- Terminal status chưa được spec xác nhận.
- Nếu thiếu requirement, chỉ ghi observation.

### Gap 5 - Cancellation rule consistency between user cancel and admin status endpoint

Spec user cancel nói chỉ hủy khi đơn chưa giao, nhưng không rõ admin endpoint có áp dụng rule đó. Manual extension có thể so sánh admin set `canceled` ở các trạng thái `pending`, `shipping`, `delivered`.

Giá trị manual extension:

- Làm rõ business-rule consistency giữa hai endpoint.
- Tập trung vào rule mà AI đã chỉ mới chạm một case.

Ranh giới:

- Không tự kết luận bug nếu admin endpoint có quyền override hợp lệ nhưng spec chưa nói.

### Gap 6 - Duplicate `status` keys in raw JSON

AI suite có wrong type và malformed JSON nhưng chưa có duplicate `status` keys.

Giá trị manual extension:

- Kiểm tra parser ambiguity cho field trạng thái.
- Phát hiện trường hợp first/last key dẫn đến update ngoài ý muốn.

Ranh giới:

- Parser-dependent.
- Assertion chính: không tạo state change ngoài intent được kiểm soát.

### Gap 7 - Query/body path mismatch with existing valid admin token

AI suite có body `order_id` mismatch, nhưng manual extension có thể mở rộng sang query params như `?order_id={{otherOrderId}}` hoặc body nested object để đảm bảo path param vẫn là source of truth.

Giá trị manual extension:

- Tăng coverage IDOR/path precedence.
- Phát hiện implementation dùng nhầm body/query thay vì path.

Ranh giới:

- Spec không có query params, nên query behavior là exploratory.

### Gap 8 - Audit/logging side effect expectations

Admin status update là thao tác nhạy cảm nhưng spec không nói audit log. Manual extension có thể chỉ ghi open question hoặc exploratory nếu SUT có dashboard/log.

Giá trị manual extension:

- Bổ sung góc nhìn operational/security.
- Hữu ích nếu assignment yêu cầu phân tích bug missed by AI.

Ranh giới:

- Không có spec audit log, nên không nên tạo expected cứng nếu không có nguồn bổ sung.

## 5. Ưu tiên đề xuất cho manual extension

| Priority | Gap | Lý do |
|---:|---|---|
| 1 | Split authorization bypass and extra-field elevation | Sửa trực tiếp `ORDERSTATUS-GEN-031` bị `INCOMPLETE` |
| 2 | Persisted-state verification after failed authorization | Security risk cao, dễ bị AI bỏ sót |
| 3 | Matrix-based transition coverage | Quan trọng cho `FR-10` state machine |
| 4 | Cancellation rule consistency | Liên quan trực tiếp đến rule cancel trong spec |
| 5 | Terminal state immutability | State-machine edge case quan trọng |
| 6 | Duplicate `status` keys | Parser/security edge case |
| 7 | Query/body path mismatch | IDOR/path precedence risk |
| 8 | Audit/logging side effect expectations | Chỉ nên exploratory nếu có nguồn bổ sung |

## 6. Vì sao AI có thể đã bỏ sót

- Spec admin endpoint chỉ liệt kê allowed statuses, không có transition matrix hoặc response schema.
- AI đã tạo nhiều single-request negative cases nhưng chưa đủ persisted-state verification sau auth failures.
- State-machine coverage cần dữ liệu order ở từng trạng thái, điều mà AI không thể invent.
- Một số security gaps cần tách objective rất rõ; `ORDERSTATUS-GEN-031` cho thấy AI đã gộp hai rủi ro vào một case.

## 7. Kết luận

Bộ audit hiện tại mạnh về auth, validation, allowed status và exploratory state transitions. Manual extension nên tập trung vào các điểm còn thiếu giá trị cao: tách authorization/extra-field elevation, verify state không đổi sau failed authorization, transition matrix có fixture, terminal/cancellation consistency, duplicate key parser behavior, và path precedence. Artifact này chỉ chuẩn bị cho `PHASE_5_HUMAN_EXTENSION`; chưa tạo manual extension cases.
