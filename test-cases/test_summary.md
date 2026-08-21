# Test Summary

Nguồn tổng hợp:

- `test-cases/login_test_cases_audited.xlsx`
- `test-cases/coupon_test_cases_audited.xlsx`
- `test-cases/admin_order_status_test_cases_audited.xlsx`

File này tổng hợp bộ test case sau `PHASE_4_HUMAN_AUDIT` và `PHASE_5_HUMAN_EXTENSION`. Các số liệu `Executed`, `Passed`, `Failed` và `Confirmed bugs` vẫn là `0` vì chưa chạy Postman/Newman và chưa có execution evidence.

## 1. Overall summary

| API | AI-generated cases | Manual extension cases | Total designed | Executed | Passed | Failed | Confirmed bugs |
|---|---:|---:|---:|---:|---:|---:|---:|
| `POST /api/login` | `40` | `5` | `45` | `0` | `0` | `0` | `0` |
| `POST /api/apply-coupon` | `40` | `5` | `45` | `0` | `0` | `0` | `0` |
| `PUT /api/admin/orders/:id/status` | `42` | `5` | `47` | `0` | `0` | `0` | `0` |
| **Total** | **`122`** | **`15`** | **`137`** | **`0`** | **`0`** | **`0`** | **`0`** |

## 2. Audit status summary

| API | `VALID` | `INCOMPLETE` | `INVALID` | `STUDENT_DESIGNED` |
|---|---:|---:|---:|---:|
| `POST /api/login` | `38` | `2` | `0` | `5` |
| `POST /api/apply-coupon` | `36` | `4` | `0` | `5` |
| `PUT /api/admin/orders/:id/status` | `41` | `1` | `0` | `5` |
| **Total** | **`115`** | **`7`** | **`0`** | **`15`** |

## 3. Source summary

| API | `AI_GENERATED` | `MANUAL_EXTENSION` |
|---|---:|---:|
| `POST /api/login` | `40` | `5` |
| `POST /api/apply-coupon` | `40` | `5` |
| `PUT /api/admin/orders/:id/status` | `42` | `5` |
| **Total** | **`122`** | **`15`** |

## 4. API-level details

### 4.1 `POST /api/login`

| Metric | Value |
|---|---:|
| Total designed cases | `45` |
| AI-generated cases | `40` |
| Manual extension cases | `5` |
| `VALID` | `38` |
| `INCOMPLETE` | `2` |
| `INVALID` | `0` |
| `STUDENT_DESIGNED` | `5` |
| Executed | `0` |
| Confirmed bugs | `0` |

Incomplete AI-generated cases:

| Case ID | Audit status | Human correction summary |
|---|---|---|
| `LOGIN-GEN-002` | `INCOMPLETE` | Giữ assertion bắt buộc ở mức `token` tồn tại và là string non-empty; chuyển kiểm tra cấu trúc JWT ba phần thành soft-check. |
| `LOGIN-GEN-040` | `INCOMPLETE` | Giữ case ở dạng exploratory; không tự đặt threshold/rule cho account lockout hoặc rate limit. |

Manual extension cases:

| Case ID | Objective summary |
|---|---|
| `LOGIN-EXT-001` | Kiểm tra token nhận từ login có thể dùng cho API cần authentication. |
| `LOGIN-EXT-002` | Kiểm tra consistency giữa `user` trong login response và identity khi gọi `GET /api/users/me`. |
| `LOGIN-EXT-003` | Kiểm tra duplicate JSON keys không gây bypass authentication hoặc parser ambiguity nguy hiểm. |
| `LOGIN-EXT-004` | So sánh paired observation giữa unknown email và wrong password để đánh giá account enumeration. |
| `LOGIN-EXT-005` | Kiểm tra malformed HTTP/JSON request không làm rò rỉ internal details. |

Coverage nổi bật:

- Positive login và success schema.
- Missing, empty, null, wrong-type `email`/`password`.
- Malformed email, whitespace, case-sensitivity.
- Injection-style input.
- Sensitive-data leakage.
- Parser/header robustness.
- Cross-request token usability và token/user consistency qua manual extension.

### 4.2 `POST /api/apply-coupon`

| Metric | Value |
|---|---:|
| Total designed cases | `45` |
| AI-generated cases | `40` |
| Manual extension cases | `5` |
| `VALID` | `36` |
| `INCOMPLETE` | `4` |
| `INVALID` | `0` |
| `STUDENT_DESIGNED` | `5` |
| Executed | `0` |
| Confirmed bugs | `0` |

Incomplete AI-generated cases:

| Case ID | Audit status | Human correction summary |
|---|---|---|
| `COUPON-GEN-005` | `INCOMPLETE` | Giữ exploratory hoặc chỉ dùng khi có fixture xác nhận coupon có `min_order_amount` và SUT enforce rule này. |
| `COUPON-GEN-008` | `INCOMPLETE` | Giữ exploratory first-use observation; cần xác nhận apply-coupon có ghi nhận usage state. |
| `COUPON-GEN-009` | `INCOMPLETE` | Giữ exploratory reuse-after-limit; chỉ chốt assertion khi có persisted usage state và rule `max_uses_per_user`. |
| `COUPON-GEN-031` | `INCOMPLETE` | Giữ exploratory cross-user observation; cần làm rõ business rule/user context trước khi kết luận. |

Manual extension cases:

| Case ID | Objective summary |
|---|---|
| `COUPON-EXT-001` | Kiểm tra công thức discount với coupon fixture được kiểm soát. |
| `COUPON-EXT-002` | Kiểm tra minimum-order enforcement với fixture xác nhận `min_order_amount`. |
| `COUPON-EXT-003` | Kiểm tra usage-limit replay với persisted verification. |
| `COUPON-EXT-004` | Kiểm tra client không thể override coupon rule bằng extra fields. |
| `COUPON-EXT-005` | Kiểm tra duplicate `code` keys không gây parser ambiguity nguy hiểm. |

Coverage nổi bật:

- Valid coupon và response schema `discount_amount`/`final_amount`.
- Unknown, empty, whitespace, null, wrong-type coupon code.
- Boundary cho `total_amount`.
- Missing/null/wrong-type `user_id`.
- Business rules như expired coupon, min-order, usage-limit ở dạng exploratory khi spec chưa đủ rõ.
- Injection, extra fields, malformed JSON, wrong `Content-Type`.
- Discount formula, min-order fixture, replay và client-side override qua manual extension.

### 4.3 `PUT /api/admin/orders/:id/status`

| Metric | Value |
|---|---:|
| Total designed cases | `47` |
| AI-generated cases | `42` |
| Manual extension cases | `5` |
| `VALID` | `41` |
| `INCOMPLETE` | `1` |
| `INVALID` | `0` |
| `STUDENT_DESIGNED` | `5` |
| Executed | `0` |
| Confirmed bugs | `0` |

Incomplete AI-generated cases:

| Case ID | Audit status | Human correction summary |
|---|---|---|
| `ORDERSTATUS-GEN-031` | `INCOMPLETE` | Tách rõ thành hai ý khi triển khai sau: normal user token bị từ chối, và extra fields `role`/`isAdmin` không được nâng quyền. |

Manual extension cases:

| Case ID | Objective summary |
|---|---|
| `ORDERSTATUS-EXT-001` | Tách rõ authorization bypass khỏi extra-field elevation. |
| `ORDERSTATUS-EXT-002` | Xác minh order state không đổi sau failed authorization. |
| `ORDERSTATUS-EXT-003` | Kiểm tra transition matrix với controlled order fixtures. |
| `ORDERSTATUS-EXT-004` | Kiểm tra consistency của cancellation rule giữa user cancel endpoint và admin status endpoint. |
| `ORDERSTATUS-EXT-005` | Kiểm tra duplicate `status` keys không gây state update ngoài ý muốn. |

Coverage nổi bật:

- Admin authorized update với các status hợp lệ.
- Missing, malformed, tampered token.
- Normal-user authorization negative cases.
- Path `id` validation: unknown, zero, negative, decimal, string, very large, injection.
- `status` validation: missing, null, empty, whitespace, unknown, case variant, wrong type, injection.
- Extra fields và body/path mismatch.
- Malformed JSON, wrong `Content-Type`.
- State transition và persisted-state exploratory checks.
- Failed-authorization persistence, transition matrix, cancellation consistency và duplicate status keys qua manual extension.

## 5. Execution and bug status

Chưa thực hiện `PHASE_6_POSTMAN_IMPLEMENTATION`, `PHASE_7_EXECUTION`, `PHASE_8_BUG_ANALYSIS` hoặc `PHASE_9_CICD`.

Vì vậy:

- `Executed = 0`
- `Passed = 0`
- `Failed = 0`
- `Confirmed bugs = 0`
- Chưa có Newman HTML report.
- Chưa có screenshot execution.
- Chưa có GitHub Issue bug thật.
- Chưa có CI/CD passing/failing run evidence.

## 6. Conclusion

Sau human audit và manual extension, bộ test hiện có tổng cộng `137` designed test cases cho 3 API. Trong đó có `122` AI-generated cases và `15` student-designed manual extension cases. Human audit ghi nhận `115` AI-generated cases ở trạng thái `VALID`, `7` cases ở trạng thái `INCOMPLETE`, và `0` cases `INVALID`. Bộ test đã sẵn sàng làm đầu vào cho bước triển khai Postman/Newman, nhưng chưa được xem là execution result.
