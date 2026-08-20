# PHASE_1_SPEC_ANALYSIS - `PUT /api/admin/orders/:id/status`

## 1. Phạm vi và nguồn đặc tả

- `api`: `PUT /api/admin/orders/:id/status`
- `phase`: `PHASE_1_SPEC_ANALYSIS`
- `artifact`: `reports/analysis/admin_order_status_spec_analysis.md`
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `selected_pool`: Pool C - Admin order status / cập nhật trạng thái đơn hàng

Nguồn đã sử dụng:

| source | Nội dung dùng để phân tích |
|---|---|
| `README.md` | Xác nhận API đã chọn, `baseUrl`, `student_id`, header bắt buộc khi thực thi |
| `docs/api_specification.md` | Đặc tả chính thức của `PUT /api/admin/orders/:id/status` |
| `docs/2026.HW06.API Testing_En.md` | Yêu cầu bài tập: state transitions, security, schema validation, human review, Postman/Newman |
| `AGENT.md` | Quy trình phase, nguyên tắc không tự suy diễn behavior, yêu cầu artifact và ngôn ngữ |

Kết luận trạng thái: đây là artifact Phase 1 cho phân tích đặc tả. Chưa tạo test case, chưa thiết kế test chi tiết, chưa execute, chưa kết luận bug.

## 2. Đặc tả endpoint

| Thuộc tính | Giá trị theo đặc tả |
|---|---|
| `method` | `PUT` |
| `path` | `/api/admin/orders/:id/status` |
| `baseUrl` | `http://localhost:3000` |
| `full_url` | `http://localhost:3000/api/admin/orders/:id/status` |
| `Content-Type` | JSON body được mô tả trong spec; khi thực thi nên dùng `Content-Type: application/json` |
| `X-Student-Id` | Bắt buộc trong mọi executable request theo `README.md` và assignment: `X-Student-Id: 23127364` |
| `Authorization` | Bắt buộc: `Authorization: Bearer <token>` |
| `authorization_rule` | Tài khoản phải có quyền Admin |
| `path_params` | `id` |
| `query_params` | Không có |
| `body` | JSON gồm `status` |

Request body được đặc tả:

```json
{
  "status": "confirmed"
}
```

Các trạng thái được liệt kê:

```text
pending, confirmed, shipping, delivered, canceled
```

## 3. Inventory input

### 3.1 Path parameter `id`

| Field | Type theo ngữ cảnh | Required/Optional | Ràng buộc được nêu rõ | Phụ thuộc dữ liệu |
|---|---|---|---|---|
| `id` | order identifier, có khả năng number/string path segment | Bắt buộc vì nằm trong path | Spec không nêu type, min/max, format, hoặc behavior khi không tồn tại | Cần order tồn tại trong database để update hợp lệ |

Spec không mô tả:

- `id` có phải integer dương không.
- Behavior với `id` không tồn tại.
- Behavior với `id` sai format như string, decimal, negative, zero.
- Authorization theo ownership không áp dụng cho admin, nhưng vẫn cần admin role.

### 3.2 Body field `status`

| Field | Type theo ví dụ/spec | Required/Optional | Ràng buộc được nêu rõ | Phụ thuộc dữ liệu |
|---|---|---|---|---|
| `status` | string | Có trong body mẫu; spec không ghi riêng chữ "required" | Giá trị thuộc `pending`, `confirmed`, `shipping`, `delivered`, `canceled` | Phụ thuộc order hiện có và rule chuyển trạng thái nếu SUT enforce state machine |

Các status hợp lệ được spec liệt kê:

- `pending`
- `confirmed`
- `shipping`
- `delivered`
- `canceled`

Spec không nêu:

- Chuyển từ trạng thái nào sang trạng thái nào được phép.
- Có được chuyển ngược hay skip trạng thái không.
- `delivered` hoặc `canceled` có phải terminal state không.
- Case sensitivity hoặc whitespace handling của `status`.

## 4. Authentication và Authorization

Mục "API Dành cho Admin" nêu:

- Tất cả API dưới mục này yêu cầu `Authorization: Bearer <token>`.
- Tài khoản phải có quyền Admin.

Vì vậy, với `PUT /api/admin/orders/:id/status`:

| Scenario | Theo spec |
|---|---|
| Không có token | Không được phép truy cập; exact status chưa được nêu |
| Token malformed/expired/tampered | Không được phép truy cập; exact status chưa được nêu |
| Token user thường | Không đủ quyền Admin; exact status chưa được nêu |
| Token admin | Có quyền gọi endpoint nếu request hợp lệ |

Khi thực thi sau này, vẫn phải thêm `X-Student-Id: 23127364` theo yêu cầu assignment.

Dependency cần có:

- Admin credential/token hợp lệ.
- Normal user token để kiểm tra authorization negative.
- Order ID tồn tại trong trạng thái xác định.

Không được tự bịa credentials nếu repo/SUT chưa cung cấp.

## 5. Response schema đã biết

Spec không mô tả response body hoặc success status code cho `PUT /api/admin/orders/:id/status`.

Được biết:

- Endpoint dùng để cập nhật trạng thái đơn hàng.
- Body nhận `status`.
- Các giá trị status được liệt kê.

Chưa biết:

- Success status code (`200`, `204`, hoặc khác).
- Success response có trả order object, message, hay status mới không.
- Error response shape.
- Có trả `order.id`, `status`, `updated_at`, hoặc fields khác không.

Trong phase sau, không nên yêu cầu schema chi tiết nếu spec chưa bổ sung. Có thể thiết kế assertion tối thiểu kiểu "nếu success, order status phải phản ánh status được yêu cầu" nhưng persisted verification cần endpoint đọc order và dữ liệu thật.

## 6. Status code và error behavior

Status code chưa được spec định nghĩa:

| Trường hợp | Status code |
|---|---|
| Admin cập nhật status hợp lệ | `SPEC_UNDEFINED` |
| Thiếu token | `SPEC_UNDEFINED` |
| Token malformed/expired | `SPEC_UNDEFINED` |
| User thường gọi admin endpoint | `SPEC_UNDEFINED` |
| Order ID không tồn tại | `SPEC_UNDEFINED` |
| `status` không thuộc allowed list | `SPEC_UNDEFINED` |
| Missing/null/wrong type `status` | `SPEC_UNDEFINED` |
| Invalid state transition | `SPEC_UNDEFINED` |

Vì vậy, Phase 2/3 không được tự gán `200`, `400`, `401`, `403`, `404`, `409`, hoặc `422` nếu chỉ dựa trên suy đoán. Có thể dùng `SPEC_UNDEFINED` và assertion bảo thủ như "không được update trạng thái" cho input/authorization không hợp lệ.

## 7. Business rules và state

Được xác nhận từ spec:

- Endpoint cập nhật trạng thái đơn hàng.
- Chỉ Admin được gọi endpoint.
- Allowed status values: `pending`, `confirmed`, `shipping`, `delivered`, `canceled`.

Liên quan từ assignment:

- Assignment nhắc `FR-10 Order state machine`, ví dụ `pending -> confirmed -> shipping -> delivered`, cùng cancellation rules.
- Assignment cũng nhắc Pool C `FR-18 Order management (admin)`.

Nhưng trong `api_specification.md`, mục admin order status chỉ liệt kê allowed statuses, chưa định nghĩa transition matrix.

State rules chưa được spec xác nhận:

- Có cho phép `pending -> confirmed` không? Có khả năng nhưng chưa mô tả trực tiếp.
- Có cho phép `confirmed -> shipping` không?
- Có cho phép `shipping -> delivered` không?
- Có cho phép skip `pending -> delivered` không?
- Có cho phép reverse `delivered -> pending` không?
- `canceled` và `delivered` có terminal không?
- API này có được cancel order hay cancellation chỉ nên qua `PUT /api/orders/:id/cancel`?

Đặc biệt, mục user order cancel ghi: `PUT /api/orders/:id/cancel` chuyển status sang `canceled` và chỉ khi đơn hàng chưa giao. Nhưng spec không nói rule đó có áp dụng cho admin status endpoint hay không.

## 8. State và dependency cần có

Để test hợp lệ ở phase sau, cần:

- Ít nhất một order tồn tại.
- Biết trạng thái hiện tại của order trước khi update.
- Admin token hợp lệ.
- Normal user token để kiểm tra authorization.
- Endpoint đọc order để verify persisted state, ví dụ `GET /api/orders/:id` hoặc `GET /api/admin/orders`, nếu được phép dùng trong setup/verification.
- Cách reset hoặc tạo order mới để tránh test order bị ảnh hưởng bởi state trước đó.

Không được ghi nhận state transition pass/fail nếu không có execution evidence.

## 9. Security requirements áp dụng được

Assignment yêu cầu security coverage như SQL injection, IDOR, role escalation và `SEC-01` đến `SEC-07`, nhưng `api_specification.md` hiện không liệt kê chi tiết `SEC-01` đến `SEC-07`. Với admin order status endpoint, các rủi ro áp dụng rõ là:

| Risk | Áp dụng? | Cơ sở/ghi chú |
|---|---|---|
| Unauthenticated access | Có | Endpoint admin yêu cầu `Authorization: Bearer <token>` |
| Authorization bypass | Có | Tài khoản phải có quyền Admin |
| Role escalation | Có | Normal user không được update admin order status |
| Token tampering | Có | Header token là bề mặt bảo mật |
| IDOR/cross-resource access | Có thể | `id` trong path chọn order; admin có thể quản lý toàn hệ thống, nhưng user thường không được gọi endpoint |
| Injection | Có | `id` path param và `status` string có thể nhận injection-style input |
| Information leakage | Có | Error path không nên lộ stack trace/order details nhạy cảm |
| State-machine abuse | Có | Update status có thể vi phạm rule nghiệp vụ nếu transition không được enforce |

## 10. Traceability sơ bộ

| Requirement/source | Nội dung liên quan | Ảnh hưởng đến phase sau |
|---|---|---|
| `docs/api_specification.md` - Admin APIs 6 | Admin APIs yêu cầu `Authorization: Bearer <token>` và quyền Admin | Nguồn chính cho auth/security cases |
| `docs/api_specification.md` - Admin orders 6.2 | `PUT /api/admin/orders/:id/status`, body `status`, allowed statuses | Nguồn chính cho input/state inventory |
| `docs/api_specification.md` - Order cancel 4.6 | User cancel chuyển status sang `canceled`, chỉ khi đơn chưa giao | Gợi ý cancellation rule nhưng chưa rõ áp dụng cho admin endpoint |
| Assignment Pool C/FR-18 | Order management admin | Gắn API với Pool C |
| Assignment FR-10 | Order state machine | Gợi ý state-transition coverage, nhưng cần phân biệt với spec thiếu transition matrix |
| Assignment Requirements | Security, state transitions, schema validation | Phase sau phải bao phủ nhưng không sinh test ở Phase 1 |
| `README.md` | Header `X-Student-Id: 23127364` | Mọi executable request plan sau này phải có header này |
| `AGENT.md` | Không suy diễn status/schema/state transition | Các expected chưa rõ cần `SPEC_UNDEFINED` |

## 11. Open questions

1. Success status code của admin status update là gì?
2. Success response body là gì?
3. Error response shape chuẩn là gì?
4. Path param `id` có bắt buộc là integer dương không?
5. Behavior khi order ID không tồn tại là gì?
6. `status` có bắt buộc không?
7. `status` có case-sensitive không?
8. Có trim whitespace cho `status` không?
9. Có cho phép update sang mọi allowed status từ mọi trạng thái hiện tại không?
10. Transition matrix chính thức là gì?
11. `delivered` và `canceled` có terminal không?
12. Admin endpoint có được set `canceled` không, hay cancellation phải qua `/api/orders/:id/cancel`?
13. Rule "chỉ hủy khi đơn chưa giao" có áp dụng cho admin status endpoint không?
14. Có persisted-state verification endpoint chính thức nào nên dùng sau update?
15. Seed admin credential/order data nằm ở đâu?
16. Exact status cho thiếu token, token sai, user thường không đủ quyền là gì?

## 12. Kết luận phase

`PUT /api/admin/orders/:id/status` có đặc tả tối thiểu: admin API yêu cầu `Authorization: Bearer <token>` và quyền Admin; nhận path param `id`; nhận JSON body `status`; allowed values gồm `pending`, `confirmed`, `shipping`, `delivered`, `canceled`. Spec chưa nêu success/error status code, response schema, validation constraints cho `id/status`, transition matrix, terminal states, cancellation rules cho admin endpoint hoặc persisted verification method.

Phase này dừng ở `PHASE_1_SPEC_ANALYSIS`. Không tạo test case.
