# PHASE_2_TEST_DESIGN - `POST /api/apply-coupon`

## 1. Phạm vi phase

- `api`: `POST /api/apply-coupon`
- `phase`: `PHASE_2_TEST_DESIGN`
- `artifact`: `reports/analysis/coupon_test_design.md`
- `based_on`: `reports/analysis/coupon_spec_analysis.md`
- `spec_analysis_status`: approved theo yêu cầu của người dùng
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `request_url`: `{{baseUrl}}/api/apply-coupon`

Phase này chỉ thiết kế coverage. Không tạo `COUPON-GEN-*`, không tạo `COUPON-EXT-*`, không ghi execution result, không kết luận bug.

Nguồn áp dụng:

| source | Vai trò |
|---|---|
| `docs/api_specification.md` | Nguồn chính cho behavior của `POST /api/apply-coupon` |
| `reports/analysis/coupon_spec_analysis.md` | Phân tích đặc tả đã được approved |
| `AGENT.md` | Quy trình phase, ranh giới human audit, yêu cầu traceability |
| `agent-skill/SKILL.md` | Cách xây dựng parameter partitions, security/schema/state dimensions và ambiguity list |

## 2. Endpoint baseline cho test design

| Thuộc tính | Giá trị thiết kế |
|---|---|
| `method` | `POST` |
| `path` | `/api/apply-coupon` |
| `headers` | `Content-Type: application/json`, `X-Student-Id: {{studentId}}` |
| `Authorization` | Không yêu cầu trong spec của endpoint này |
| `path_params` | Không có |
| `query_params` | Không có |
| `body_fields` | `code`, `total_amount`, `user_id` |
| `success_status` | `SPEC_UNDEFINED` |
| `success_schema_known` | Có `discount_amount`, `final_amount` |
| `error_status` | `SPEC_UNDEFINED` |
| `error_schema` | `SPEC_UNDEFINED` |

Nguyên tắc expected cho phase sau:

- Không tự gán `200` cho success vì spec không nêu status code cho coupon.
- Success schema có thể assert `discount_amount` và `final_amount` tồn tại nếu response thành công.
- Negative/error cases dùng `expected_status = SPEC_UNDEFINED` nếu spec không bổ sung.
- Assertion hợp lý cho lỗi: không trả discount hợp lệ khi input không hợp lệ, không rò rỉ stack trace/SQL/internal details.

## 3. Parameter Inventory

### 3.1 `code`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid class | Coupon code tồn tại, ví dụ `{{validCouponCode}}` | Nếu success, response chứa `discount_amount` và `final_amount`; status vẫn `SPEC_UNDEFINED` |
| semantic invalid | Code đúng format nhưng không tồn tại | Không nên trả discount hợp lệ; exact error undefined |
| expired coupon | Code tồn tại nhưng `expired_at` đã qua | Spec admin coupon có `expired_at`, nhưng apply behavior chưa rõ |
| min-order not met | Code có `min_order_amount` lớn hơn `total_amount` | Behavior chưa được spec định nghĩa |
| usage limit reached | Code có `max_uses_per_user` đã hết cho `user_id` | Cần state/setup; behavior undefined |
| missing value | Body không có key `code` | Không được tính discount hợp lệ |
| empty value | `code: ""` | Không được tính discount hợp lệ |
| whitespace-only | `code: "   "` | Trim behavior chưa rõ |
| leading/trailing whitespace | `code: " SAVE10 "` | Whitespace normalization chưa rõ; exploratory |
| null | `code: null` | Null handling undefined |
| wrong type | number, boolean, object, array | Type validation undefined; kiểm tra type confusion |
| case behavior | `save10` so với `SAVE10` | Case sensitivity chưa rõ; exploratory |
| boundary values | Code rất ngắn, rất dài, ký tự đặc biệt | Spec không nêu min/max/regex |
| injection-style | SQL-like, script-like, operator-like string/object | Không bypass coupon lookup, không lộ lỗi nội bộ |
| unexpected extra fields | Body thêm `type`, `discount_value`, `isAdmin`, `role` | Không được override coupon rule từ client body |

### 3.2 `total_amount`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid nominal | Số tiền hợp lý lớn hơn hoặc bằng ngưỡng coupon nếu có | Nếu success, `final_amount` phải nhất quán với `discount_amount` |
| zero | `0` | Rule `> 0` chưa nêu; exploratory/negative |
| negative | `-1`, `-500000` | Không nên tạo final amount/discount bất thường |
| decimal | `99999.99` | Rounding/decimal support chưa rõ |
| very large | Số rất lớn | Kiểm tra overflow/robustness |
| below min_order | Nhỏ hơn `min_order_amount` nếu coupon có ngưỡng | Behavior undefined nhưng business-critical |
| equal min_order | Bằng `min_order_amount` | Boundary quan trọng nếu ngưỡng được enforce |
| above min_order | Lớn hơn `min_order_amount` | Boundary/control cho min-order |
| missing value | Body không có `total_amount` | Không được tính discount hợp lệ |
| null | `total_amount: null` | Null handling undefined |
| wrong type | string, boolean, object, array | Type validation/type coercion risk |
| numeric string | `"500000"` | Coercion behavior chưa rõ |
| special numbers | `NaN`, `Infinity` nếu tool gửi được raw JSON/non-standard | Parser-dependent, exploratory |

### 3.3 `user_id`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid existing user | `{{validUserId}}` tồn tại | Cần fixture/setup |
| non-existing user | `{{unknownUserId}}` | Behavior undefined |
| other user | `{{otherUserId}}` | Cross-user/business-rule abuse risk |
| missing value | Body không có `user_id` | Không rõ có bắt buộc không |
| null | `user_id: null` | Null handling undefined |
| wrong type | string, boolean, object, array | Type validation/type confusion |
| boundary values | `0`, negative, decimal, very large | ID format/min/max undefined |
| injection-style | object operator hoặc string injection nếu parser cho phép | Không được thao túng usage/cross-user logic |

### 3.4 Request body và HTTP-level inputs

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid JSON object | Có `code`, `total_amount`, `user_id` | Success schema nếu coupon/user/setup hợp lệ |
| empty object | `{}` | Không được tính discount hợp lệ |
| body missing | Không gửi body | Error behavior undefined |
| invalid JSON | Malformed JSON | Không crash, không lộ internal details |
| wrong top-level type | array, string, number, boolean | Không được tính discount hợp lệ |
| duplicate keys | Duplicate `code`, `total_amount`, hoặc `user_id` | Parser-dependent; exploratory |
| wrong/missing `Content-Type` | `text/plain` hoặc thiếu `Content-Type` | Parser behavior undefined |
| missing `X-Student-Id` | Không gửi header assignment | SUT behavior chưa được business spec mô tả |

## 4. Coverage theo nhóm kiểm thử

| Coverage group | Mục tiêu | Traceability |
|---|---|---|
| Positive coupon calculation | Xác minh coupon hợp lệ trả `discount_amount`, `final_amount` | `docs/api_specification.md` - Coupons 5.1 |
| Schema consistency | `discount_amount` và `final_amount` tồn tại, là number, và nhất quán với input | Coupons 5.1 |
| Coupon partitions | Valid, unknown, expired, min-order, usage limit, case/whitespace | Coupons 5.1 và Admin coupons 6.4 |
| Amount boundaries | Zero/negative/decimal/large/min-order boundaries | Assignment domain partitions |
| User identifier behavior | Existing/non-existing/other user/wrong type | Body field `user_id`; security/business-rule abuse |
| Missing/null/wrong type | Bao phủ từng field và top-level body | `AGENT.md`, `SKILL.md` |
| Injection robustness | `code`/`user_id`/body operator-style inputs | Assignment security coverage |
| Information leakage | Error path không lộ stack trace/query/internal details | Security expectation |
| State behavior | Usage count/max uses per user nếu SUT enforce | Admin coupon `max_uses_per_user`; exploratory |

## 5. Security Coverage

| Risk | Applicability | Thiết kế coverage | Expected/ranh giới |
|---|---|---|---|
| Unauthenticated access | Endpoint không yêu cầu auth theo spec | Không tạo case "missing token must fail" như expected cứng | Có thể exploratory nếu muốn quan sát auth behavior |
| Authorization bypass | Có thể nếu coupon usage cần user context | Dùng `user_id` của user khác hoặc extra fields | Không kết luận bug nếu spec chưa yêu cầu auth |
| IDOR/cross-user behavior | Có thể | Thử `user_id` khác, usage limit theo user | Không được thao túng discount/usage của user khác nếu business rule tồn tại |
| Unexpected security-sensitive/business-rule fields | Có thể | Body thêm `role`, `isAdmin`, `discount_value`, `type` ngoài các field được mô tả trong contract | Các extra fields không được dùng làm cơ sở nâng quyền hoặc override coupon rule; exact parser behavior là `SPEC_UNDEFINED` |
| Injection | Có | SQL/script/operator-style trong `code` và `user_id` | Không bypass, không leak stack trace/query |
| Information leakage | Có | Unknown/expired/invalid coupon error path | Không lộ internal details hoặc dữ liệu coupon nhạy cảm |
| Business-rule abuse | Có | `total_amount` âm/rất lớn, min-order bypass, usage-limit replay | Không tạo discount/final_amount vô lý |
| Token tampering | Không trực tiếp | Endpoint không dùng token theo spec | Không ưu tiên như group chính |

## 6. Schema Coverage

### 6.1 Success response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Response chứa `discount_amount` | Spec nói JSON chứa field này | Confirmed |
| Response chứa `final_amount` | Spec nói JSON chứa field này | Confirmed |
| `discount_amount` là number | Suy ra từ tính toán tiền | Reasonable schema assertion |
| `final_amount` là number | Suy ra từ tính toán tiền | Reasonable schema assertion |
| `final_amount = total_amount - discount_amount` | Logic tính toán giảm giá | Reasonable, nhưng công thức/rounding chưa chi tiết |
| `discount_amount >= 0` và `final_amount >= 0` | Business sanity | Reasonable, nhưng spec không nêu rõ |
| Không lộ stack trace/internal fields | Security expectation | Security assertion |

### 6.2 Error response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Không trả discount hợp lệ cho input invalid | Business expectation | Reasonable, exact status undefined |
| Không lộ stack trace/SQL/internal path | Security expectation | Security assertion |
| Error shape cụ thể | Không có trong spec | `SPEC_UNDEFINED` |

## 7. State Coverage

| State/Dependency | Vai trò | Thiết kế coverage |
|---|---|---|
| Coupon tồn tại | Precondition cho positive calculation | Cần seed hoặc tạo bằng admin API |
| Coupon expired | Business-rule state | Chỉ test nếu có fixture/cách tạo coupon expired |
| `min_order_amount` | Boundary state | Test below/equal/above nếu coupon có ngưỡng |
| `max_uses_per_user` | Usage state | Test first use/reuse nếu SUT ghi usage |
| Existing user | Precondition cho `user_id` | Cần user fixture |
| Other user | Cross-user behavior | Dùng để phát hiện usage/rule abuse |

Persisted-state verification chỉ nên thiết kế nếu xác minh SUT có usage tracking hoặc endpoint đọc coupon/usage. Không ghi nhận pass/fail khi chưa execution.

## 8. Dữ liệu và setup cần chuẩn bị

| Setup item | Trạng thái hiện tại | Cách xử lý đề xuất |
|---|---|---|
| `baseUrl` | Confirmed | `{{baseUrl}} = http://localhost:3000` |
| `studentId` | Confirmed | `{{studentId}} = 23127364` |
| Valid coupon | Missing | Dùng seed data hoặc tạo bằng `POST /api/admin/coupons` nếu có admin token |
| Expired coupon | Missing | Tạo fixture nếu admin setup cho phép |
| Min-order coupon | Missing | Tạo coupon có `min_order_amount` rõ ràng |
| Usage-limit coupon | Missing | Tạo coupon có `max_uses_per_user` nếu SUT enforce |
| Valid user id | Missing | Seed user hoặc tạo bằng `POST /api/register` |
| Admin token | Missing | Chỉ cần nếu tạo coupon setup qua admin API |

## 9. Traceability map cho phase generation sau này

| Design area | Requirement/spec reference | Gợi ý `coverage_type` |
|---|---|---|
| Coupon apply schema | `docs/api_specification.md` - Coupons 5.1 | `positive`, `schema`, `business` |
| `code` partitions | Body field `code` | `domain`, `validation`, `security` |
| `total_amount` boundaries | Body field `total_amount` | `boundary`, `business`, `validation` |
| `user_id` behavior | Body field `user_id` | `domain`, `security`, `state` |
| Expiration/min-order/usage | Admin coupon fields 6.4 | `business`, `state`, `exploratory` |
| Injection | Assignment security examples | `security`, `injection` |
| Missing/null/wrong type | `AGENT.md`, `SKILL.md` | `negative`, `validation` |

## 10. Open Questions đưa sang human checkpoint/generation

1. Success status code là gì?
2. Error status code và error schema là gì?
3. Coupon seed hợp lệ là gì?
4. Công thức tính discount cho `percent` là gì?
5. Có fixed-amount coupon không?
6. Rounding khi discount decimal ra sao?
7. `total_amount` có min/max không?
8. `code` có case-sensitive hoặc trim không?
9. `user_id` có bắt buộc là user tồn tại không?
10. Endpoint có thật sự không cần authentication không?
11. `max_uses_per_user` có được ghi nhận khi apply coupon không?
12. Có cần verify state sau apply không?

## 11. Generation guardrails cho `PHASE_3_AI_GENERATION`

Khi được phép tạo candidate cases:

- Tạo ít nhất 35 AI-generated candidate cases cho `POST /api/apply-coupon`.
- Dùng prefix `COUPON-GEN-001`.
- Mọi case có `source = AI_GENERATED` và `audit_status = PENDING_HUMAN_REVIEW`.
- Không dùng `COUPON-EXT-*` trong AI generation.
- Chỉ đặt expected status cụ thể nếu spec bổ sung; hiện tại ưu tiên `SPEC_UNDEFINED`.
- Mọi executable request plan phải có `X-Student-Id: {{studentId}}`.
- Không ghi pass/fail/bug nếu chưa có Postman/Newman evidence.

## 12. Kết luận phase

`PHASE_2_TEST_DESIGN` cho `POST /api/apply-coupon` đã xác định coverage cho `code`, `total_amount`, `user_id`, schema `discount_amount/final_amount`, business-rule state như expiration/min-order/usage-limit, và security risks như injection, cross-user behavior, unexpected security-sensitive/business-rule fields và information leakage. Artifact này dừng ở test design và chưa tạo test case.
