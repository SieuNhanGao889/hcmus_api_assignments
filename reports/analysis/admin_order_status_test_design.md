# PHASE_2_TEST_DESIGN - `PUT /api/admin/orders/:id/status`

## 1. Phạm vi phase

- `api`: `PUT /api/admin/orders/:id/status`
- `phase`: `PHASE_2_TEST_DESIGN`
- `artifact`: `reports/analysis/admin_order_status_test_design.md`
- `based_on`: `reports/analysis/admin_order_status_spec_analysis.md`
- `spec_analysis_status`: approved theo yêu cầu của người dùng
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `request_url`: `{{baseUrl}}/api/admin/orders/{{orderId}}/status`

Phase này chỉ thiết kế coverage. Không tạo `ORDERSTATUS-GEN-*`, không tạo `ORDERSTATUS-EXT-*`, không ghi execution result, không kết luận bug.

Nguồn áp dụng:

| source | Vai trò |
|---|---|
| `docs/api_specification.md` | Nguồn chính cho admin order status behavior |
| `reports/analysis/admin_order_status_spec_analysis.md` | Phân tích đặc tả đã được approved |
| `AGENT.md` | Quy trình phase, auth/security/state requirements |
| `agent-skill/SKILL.md` | Hướng dẫn parameter partitions, security/schema/state dimensions |

## 2. Endpoint baseline cho test design

| Thuộc tính | Giá trị thiết kế |
|---|---|
| `method` | `PUT` |
| `path` | `/api/admin/orders/:id/status` |
| `headers` | `Content-Type: application/json`, `Authorization: Bearer {{adminToken}}`, `X-Student-Id: {{studentId}}` |
| `Authorization` | Bắt buộc, token phải thuộc Admin |
| `path_params` | `id` |
| `query_params` | Không có |
| `body_fields` | `status` |
| `allowed_status_values` | `pending`, `confirmed`, `shipping`, `delivered`, `canceled` |
| `success_status` | `SPEC_UNDEFINED` |
| `success_schema` | `SPEC_UNDEFINED` |
| `error_status` | `SPEC_UNDEFINED` |
| `error_schema` | `SPEC_UNDEFINED` |

Nguyên tắc expected cho phase sau:

- Không tự gán `200`, `204`, `400`, `401`, `403`, `404`, `409`, `422`.
- Với admin token và allowed `status`, có thể thiết kế expected ở mức "request hợp lệ theo input/auth"; status vẫn `SPEC_UNDEFINED`.
- Với auth/input invalid, assertion chính là không được update trạng thái order và không lộ internal details.
- State transition rules ngoài allowed values phải được đánh dấu `exploratory` nếu spec không bổ sung transition matrix.

## 3. Parameter Inventory

### 3.1 Path parameter `id`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid existing order | `{{existingOrderId}}` | Cần order tồn tại, trạng thái hiện tại biết trước |
| non-existing order | `{{unknownOrderId}}` | Không được update order nào; exact status undefined |
| zero | `0` | ID min/format undefined |
| negative | `-1` | ID format undefined |
| decimal | `1.5` | ID format undefined |
| non-numeric string | `abc` | Parser/validation undefined |
| very large | ID rất lớn | Robustness/overflow |
| injection-style | SQL/path traversal/script-like segment | Không bypass, không leak internal details |
| missing/empty path segment | Không có `id` hoặc `//status` nếu router cho phép | Route behavior undefined |

### 3.2 Body field `status`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| allowed value | `pending`, `confirmed`, `shipping`, `delivered`, `canceled` | Input value hợp lệ theo enum, transition validity chưa rõ |
| unknown value | `returned`, `done`, `cancelled` | Không được update sang status ngoài allowed list |
| case variant | `Confirmed`, `CONFIRMED` | Case sensitivity undefined |
| leading/trailing whitespace | `" confirmed "` | Trim behavior undefined |
| empty | `""` | Không được update hợp lệ |
| whitespace-only | `"   "` | Không được update hợp lệ |
| missing | Body không có `status` | Không được update hợp lệ |
| null | `status: null` | Null handling undefined |
| wrong type | number, boolean, object, array | Type validation/type confusion |
| injection-style | SQL/script/operator-like string/object | Không bypass, không leak internal details |
| duplicate keys | Duplicate `status` trong raw JSON | Parser-dependent, exploratory |
| extra fields | `user_id`, `role`, `isAdmin`, `order_id` | Không được override auth/order identity |

### 3.3 Headers/auth inputs

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid admin token | `Authorization: Bearer {{adminToken}}` | Chỉ admin được gọi endpoint |
| missing token | Không có `Authorization` | Không được update order |
| malformed token | `Bearer abc.def`, no bearer, random string | Không được update order |
| expired/tampered token | Token hết hạn/sửa payload nếu có fixture | Không được update order |
| normal user token | `Bearer {{userToken}}` | Không đủ quyền Admin |
| wrong `Content-Type` | `text/plain` với JSON body | Parser behavior undefined |
| missing `X-Student-Id` | Không gửi header assignment | Assignment evidence requirement; SUT behavior undefined |

## 4. Coverage theo nhóm kiểm thử

| Coverage group | Mục tiêu | Traceability |
|---|---|---|
| Admin authorized update | Admin token + existing order + allowed `status` | Admin APIs 6, Admin orders 6.2 |
| Allowed status values | Bao phủ 5 status được liệt kê | Admin orders 6.2 |
| Status validation | Missing/null/empty/wrong type/unknown/case/whitespace | `AGENT.md`, `SKILL.md` |
| Order ID validation | Existing/non-existing/malformed/boundary/injection ID | Path param `id` |
| Authentication | Missing/malformed/expired/tampered token | Admin APIs 6 |
| Authorization | Normal user token không được update | Admin APIs 6, assignment security |
| State transition | Forward/reverse/skip/terminal/cancel exploratory | Assignment FR-10, spec allowed values |
| Persisted state | Verify order status sau update nếu endpoint đọc order có sẵn | State behavior |
| Information leakage | Error path không lộ internal details/order data nhạy cảm | Security expectation |

## 5. Security Coverage

| Risk | Applicability | Thiết kế coverage | Expected/ranh giới |
|---|---|---|---|
| Unauthenticated access | Có | Không gửi token | Không được update order; exact status undefined |
| Invalid authentication | Có | Malformed/expired/tampered token | Không được update order |
| Authorization bypass | Có | Normal user token gọi admin endpoint | Không được update order |
| Authorization bypass | Có | Normal user token gọi admin endpoint | Không được update order |
| Unexpected security-sensitive fields | Có | Gửi thêm `role`, `isAdmin` trong body | Các field ngoài contract không được dùng làm cơ sở nâng quyền; exact parser behavior là SPEC_UNDEFINED |
| IDOR/cross-resource | Có thể | User token với order của chính mình/khác user | Endpoint admin phải vẫn yêu cầu admin role |
| Token tampering | Có | Sửa token admin/user nếu setup cho phép | Không được update |
| Injection | Có | Injection-style `id` hoặc `status` | Không bypass, không leak stack trace/query |
| Information leakage | Có | Unknown ID, auth failure, invalid status | Không lộ internal path, stack trace, sensitive order/customer data |
| State-machine abuse | Có | Reverse/skip/terminal/cancel transitions | Mark exploratory nếu transition matrix chưa rõ |

## 6. Schema Coverage

### 6.1 Success response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Success response schema cụ thể | Spec không mô tả | `SPEC_UNDEFINED` |
| Nếu response có `status`, giá trị phải bằng allowed status requested | Endpoint cập nhật status | Reasonable, nhưng schema undefined |
| Nếu response có order object, `id` phải khớp path `id` | Consistency | Reasonable |
| Không lộ sensitive customer/payment/internal fields | Security expectation | Security assertion |

### 6.2 Error response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Authentication/authorization failure không được update persisted order state | Admin-only requirement | Strong behavior assertion |
| Input ngoài contract không được dẫn đến trạng thái nằm ngoài allowed status values | Allowed status contract | Strong behavior assertion |
| Parser/normalization behavior đối với case/whitespace/duplicate key | Spec không định nghĩa | Exploratory / `SPEC_UNDEFINED` |
| Error response không lộ stack trace/SQL/internal path | Security expectation | Security assertion |
| Error schema cụ thể | Không có trong spec | `SPEC_UNDEFINED` |

## 7. State Coverage

### 7.1 Allowed values versus transition matrix

Spec chỉ xác nhận allowed values, chưa xác nhận transition matrix. Vì vậy:

- Cases set `status` thành một trong 5 allowed values có thể là input-valid cases.
- Cases forward/reverse/skip/terminal nên đánh dấu exploratory nếu không có requirement bổ sung.
- Không tự kết luận skip/reverse là bug nếu spec chưa định nghĩa.

### 7.2 Transition dimensions nên bao phủ

| Dimension | Candidate state flow | Expected/ghi chú |
|---|---|---|
| Forward path | `pending -> confirmed -> shipping -> delivered` | Assignment nêu ví dụ FR-10; spec admin endpoint chưa xác nhận matrix |
| Skip transition | `pending -> delivered` | Exploratory nếu matrix thiếu |
| Reverse transition | `delivered -> pending`, `shipping -> confirmed` | Exploratory/negative tùy requirement bổ sung |
| Terminal delivered | Update từ `delivered` sang trạng thái khác | Terminal behavior chưa rõ |
| Terminal canceled | Update từ `canceled` sang trạng thái khác | Terminal behavior chưa rõ |
| Cancel from not delivered | `pending/confirmed -> canceled` | User cancel rule nói chỉ khi chưa giao, chưa rõ admin endpoint |
| Cancel after delivered | `delivered -> canceled` | Cần làm rõ rule có áp dụng admin không |
| Repeated same status | `confirmed -> confirmed` | Idempotency undefined |

### 7.3 Persisted-state verification needs

| Need | Endpoint/setup có thể dùng |
|---|---|
| Tạo order mới | `POST /api/checkout` nếu user/cart setup sẵn |
| Đọc order trước update | `GET /api/orders/:id` hoặc `GET /api/admin/orders` nếu token hợp lệ |
| Đọc order sau update | Same as above |
| Reset state | Tạo order mới cho từng transition hoặc dùng fixture reset |

Không ghi pass/fail nếu chưa có real execution evidence.

## 8. Dữ liệu và setup cần chuẩn bị

| Setup item | Trạng thái hiện tại | Cách xử lý đề xuất |
|---|---|---|
| `baseUrl` | Confirmed | `{{baseUrl}} = http://localhost:3000` |
| `studentId` | Confirmed | `{{studentId}} = 23127364` |
| Admin token | Missing | Seed admin credential hoặc setup login admin |
| Normal user token | Missing | User thường để test authorization |
| Existing order IDs | Missing | Seed orders hoặc tạo qua checkout |
| Orders by state | Missing | Cần order ở `pending`, `confirmed`, `shipping`, `delivered`, `canceled` nếu test transition |
| Unknown order ID | Missing | ID chắc chắn không tồn tại |
| State verification endpoint | Available in spec but auth details vary | `GET /api/admin/orders` hoặc `GET /api/orders/:id` |

## 9. Traceability map cho phase generation sau này

| Design area | Requirement/spec reference | Gợi ý `coverage_type` |
|---|---|---|
| Admin auth | `docs/api_specification.md` - API Dành cho Admin | `security`, `auth` |
| Path `id` | Admin orders 6.2 path | `domain`, `validation`, `security` |
| Allowed statuses | Admin orders 6.2 body `status` | `domain`, `state`, `validation` |
| State machine | Assignment FR-10; spec allowed values | `state`, `exploratory` |
| Unexpected security-sensitive fields | Admin role requirement + body chỉ mô tả `status` | `security`, `exploratory` |
| Missing/null/wrong type | `AGENT.md`, `SKILL.md` | `negative`, `validation` |
| Persisted verification | Endpoint updates order status | `state`, `persistence` |

## 10. Open Questions đưa sang human checkpoint/generation

1. Success status code là gì?
2. Success response body là gì?
3. Error status/error schema là gì?
4. Seed admin credential nằm ở đâu?
5. Có cách tạo/reset order theo từng trạng thái không?
6. Path `id` type chính thức là gì?
7. `status` case-sensitive/trim behavior ra sao?
8. Transition matrix chính thức là gì?
9. `delivered` và `canceled` có terminal không?
10. Admin endpoint có được set `canceled` không?
11. Rule cancel chỉ khi chưa giao có áp dụng cho admin endpoint không?
12. Endpoint nào dùng để verify persisted state sau update?
13. Exact expected cho normal user token là gì?

## 11. Generation guardrails cho `PHASE_3_AI_GENERATION`

Khi được phép tạo candidate cases:

- Tạo ít nhất 35 AI-generated candidate cases cho `PUT /api/admin/orders/:id/status`.
- Dùng prefix `ORDERSTATUS-GEN-001`.
- Mọi case có `source = AI_GENERATED` và `audit_status = PENDING_HUMAN_REVIEW`.
- Không dùng `ORDERSTATUS-EXT-*` trong AI generation.
- Dùng `SPEC_UNDEFINED` cho expected status khi spec chưa nêu.
- Phân biệt allowed status input coverage với transition validity; không tự suy diễn matrix.
- Mọi executable request plan phải có `X-Student-Id: {{studentId}}`.
- Không ghi pass/fail/bug nếu chưa có Postman/Newman evidence.

## 12. Kết luận phase

`PHASE_2_TEST_DESIGN` cho `PUT /api/admin/orders/:id/status` đã xác định coverage cho `id`, `status`, admin authentication/authorization, role escalation, injection, information leakage, allowed statuses, state transitions exploratory và persisted-state verification. Artifact này dừng ở test design và chưa tạo test case.
