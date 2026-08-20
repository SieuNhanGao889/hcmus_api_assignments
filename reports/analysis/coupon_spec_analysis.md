# PHASE_1_SPEC_ANALYSIS - `POST /api/apply-coupon`

## 1. Phạm vi và nguồn đặc tả

- `api`: `POST /api/apply-coupon`
- `phase`: `PHASE_1_SPEC_ANALYSIS`
- `artifact`: `reports/analysis/coupon_spec_analysis.md`
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `selected_pool`: Pool B - Discount coupon / mã giảm giá

Nguồn đã sử dụng:

| source | Nội dung dùng để phân tích |
|---|---|
| `README.md` | Xác nhận API đã chọn, `baseUrl`, `student_id`, header bắt buộc khi thực thi |
| `docs/api_specification.md` | Đặc tả chính thức của `POST /api/apply-coupon` |
| `docs/2026.HW06.API Testing_En.md` | Yêu cầu bài tập: domain partitions, security, schema validation, AI-first workflow, Postman/Newman |
| `AGENT.md` | Quy trình phase, nguyên tắc không tự suy diễn behavior, yêu cầu artifact và ngôn ngữ |

Kết luận trạng thái: đây là artifact Phase 1 cho phân tích đặc tả. Chưa tạo test case, chưa thiết kế test chi tiết, chưa execute, chưa kết luận bug.

## 2. Đặc tả endpoint

| Thuộc tính | Giá trị theo đặc tả |
|---|---|
| `method` | `POST` |
| `path` | `/api/apply-coupon` |
| `baseUrl` | `http://localhost:3000` |
| `full_url` | `http://localhost:3000/api/apply-coupon` |
| `Content-Type` | JSON body được mô tả trong spec; khi thực thi nên dùng `Content-Type: application/json` |
| `X-Student-Id` | Bắt buộc trong mọi executable request theo `README.md` và assignment: `X-Student-Id: 23127364` |
| `Authorization` | Không được yêu cầu trong mục `POST /api/apply-coupon` của `docs/api_specification.md` |
| `path_params` | Không có |
| `query_params` | Không có |
| `body` | JSON gồm `code`, `total_amount`, `user_id` |

Request body được đặc tả:

```json
{
  "code": "SAVE10",
  "total_amount": 500000,
  "user_id": 1
}
```

Mô tả behavior theo spec:

- Tính toán tổng tiền sau khi giảm.
- Trả về cấu trúc JSON chứa `discount_amount` và `final_amount`.

## 3. Inventory trường request

| Field | Type theo ví dụ/spec | Required/Optional | Ràng buộc được nêu rõ | Phụ thuộc dữ liệu |
|---|---|---|---|---|
| `code` | string | Có trong body mẫu; spec không ghi riêng chữ "required" | Ví dụ `SAVE10`; không nêu case sensitivity, trim, regex, độ dài, hoặc danh sách code hợp lệ | Cần coupon tồn tại trong database, ví dụ có thể được tạo qua admin coupon API nếu có quyền |
| `total_amount` | number | Có trong body mẫu; spec không ghi riêng chữ "required" | Ví dụ `500000`; không nêu min/max, integer/decimal, đơn vị tiền tệ, hoặc rule `> 0` | Cần tương ứng với đơn hàng/cart giả lập hoặc input tính toán |
| `user_id` | number | Có trong body mẫu; spec không ghi riêng chữ "required" | Ví dụ `1`; không nêu có bắt buộc là existing user hay không | Cần user tồn tại nếu coupon có rule theo user |

Ghi chú: Vì spec không định nghĩa validation chi tiết, các phân vùng missing, empty, null, wrong type, boundary, malformed, injection hoặc semantic invalid nên được đưa sang Phase 2 như test-design coverage. Không được tự gán error status cụ thể nếu spec không nêu.

## 4. Authentication và Authorization

- `POST /api/apply-coupon` không nằm trong nhóm API được mô tả là yêu cầu `Authorization: Bearer <token>`.
- `GET /api/coupons` là admin API và có header `Authorization`, nhưng đây là endpoint khác.
- Spec có trường `user_id` trong body, nhưng không mô tả cơ chế xác thực người dùng hoặc kiểm tra quyền sở hữu user.
- Khi thực thi sau này, mọi request vẫn phải có `X-Student-Id: 23127364` theo yêu cầu assignment.

Security implication:

- Vì `user_id` được gửi trực tiếp trong body và không có auth rule rõ ràng, cần xem đây là vùng rủi ro về business-rule abuse/cross-user behavior trong phase thiết kế.
- Không được tự kết luận rằng thiếu auth là bug, vì spec không yêu cầu auth cho endpoint này.

## 5. Response schema đã biết

Spec mô tả success response ở mức khái quát:

| Field | Type/shape được mô tả | Bắt buộc theo spec | Ghi chú kiểm thử schema |
|---|---|---|---|
| `discount_amount` | number, suy ra từ "tổng tiền sau khi giảm" | Có, vì spec nói response chứa field này | Type chính xác, integer/decimal và rounding chưa được mô tả |
| `final_amount` | number, suy ra từ "tổng tiền sau khi giảm" | Có, vì spec nói response chứa field này | Công thức dự kiến là `total_amount - discount_amount`, nhưng spec không ghi công thức chi tiết |

Các field chưa được spec mô tả:

- `message`
- `coupon`
- `code`
- `type`
- `discount_value`
- `min_order_amount`
- `expired_at`
- `max_uses_per_user`
- error object/schema

Không nên bắt buộc các field trên xuất hiện hoặc không xuất hiện nếu chưa có nguồn bổ sung. Tuy nhiên, không rò rỉ stack trace, SQL query hoặc internal details là assertion security hợp lý ở phase sau.

## 6. Status code và error behavior

Status code được nêu rõ:

| Trường hợp | Status code |
|---|---|
| Không có status success cụ thể được nêu cho `POST /api/apply-coupon` | `SPEC_UNDEFINED` |

Khác với `POST /api/login` và `POST /api/register`, mục coupon không nêu rõ `200 OK` trong dòng response. Vì vậy:

- Không tự gán `200` cho success nếu chỉ dựa vào suy đoán.
- Có thể ghi `expected_status = SPEC_UNDEFINED` cho đến khi có nguồn spec bổ sung hoặc execution evidence.
- Không tự gán `400`, `401`, `403`, `404`, `409`, `422` cho lỗi coupon nếu spec không định nghĩa.

Các trường hợp error chưa được spec định nghĩa:

- Unknown coupon code.
- Coupon expired.
- Coupon chưa đạt `min_order_amount`.
- Coupon vượt `max_uses_per_user`.
- `total_amount` bằng `0`, âm, decimal, quá lớn.
- `user_id` không tồn tại.
- Thiếu/null/wrong type cho `code`, `total_amount`, `user_id`.
- Body không phải JSON hợp lệ.

## 7. Business rules liên quan

Được xác nhận từ spec:

- Endpoint dùng để áp dụng coupon.
- Input gồm `code`, `total_amount`, `user_id`.
- Output chứa `discount_amount` và `final_amount`.
- Có API admin để tạo coupon với các field `code`, `type`, `discount_value`, `min_order_amount`, `expired_at`, `max_uses_per_user`.

Business rules có thể liên quan nhưng chưa được định nghĩa đầy đủ:

- Coupon có loại `percent` trong ví dụ admin coupon, nhưng spec không nêu các loại hợp lệ khác.
- Spec không mô tả công thức tính `discount_amount` cho `percent` hoặc fixed amount.
- Spec không mô tả rounding khi discount tạo ra số lẻ.
- Spec không mô tả behavior khi `total_amount < min_order_amount`.
- Spec không mô tả behavior khi coupon hết hạn.
- Spec không mô tả usage tracking theo `user_id`.
- Spec không mô tả coupon case sensitivity hoặc trim whitespace.

Những nội dung này nên chuyển sang `Open Questions` và Phase 2 test-design, không được biến thành expected behavior cứng trong Phase 1.

## 8. State và dependency

`POST /api/apply-coupon` không được mô tả là tạo đơn hàng hoặc thay đổi trạng thái đơn hàng. Tuy nhiên, do coupon admin model có `max_uses_per_user`, endpoint này có thể liên quan đến state usage nếu SUT implement rule đó.

Dependency cần có để test hợp lệ:

- Coupon hợp lệ tồn tại trong database, ví dụ `SAVE10` hoặc coupon tạo bằng `POST /api/admin/coupons`.
- Nếu cần tạo coupon, cần admin token cho `POST /api/admin/coupons`.
- Cần `user_id` hợp lệ nếu usage rule hoặc user-specific rule được áp dụng.
- Cần dữ liệu coupon expired/min-order/usage-limit nếu muốn kiểm tra các business rule đó.
- Không được tự bịa admin credentials, user id hoặc coupon data nếu repo/SUT chưa cung cấp.

Persisted-state questions:

- Áp dụng coupon có tăng usage count không?
- Áp dụng coupon có phụ thuộc order/checkout thực tế không?
- Có cần verify persisted usage sau apply không?

Spec hiện tại không trả lời các câu hỏi này.

## 9. Security requirements áp dụng được

Assignment yêu cầu bao phủ security như SQL injection, IDOR, role escalation và các requirement `SEC-01` đến `SEC-07`, nhưng `api_specification.md` hiện không liệt kê chi tiết `SEC-01` đến `SEC-07`. Với riêng `POST /api/apply-coupon`, các rủi ro áp dụng hợp lý là:

| Risk | Áp dụng? | Cơ sở/ghi chú |
|---|---|---|
| Injection | Có | `code` là input string, có thể gửi SQL/script/operator-like values |
| Business-rule abuse | Có | `total_amount`, `user_id`, coupon rule có thể bị thao túng |
| Cross-user behavior / IDOR-like risk | Có thể | Body nhận `user_id`; nếu không auth, cần kiểm tra không lạm dụng usage của user khác |
| Information leakage | Có | Error path có thể rò rỉ coupon existence, SQL query, stack trace |
| Authorization bypass | Có thể | Nếu coupon usage cần user context nhưng endpoint không yêu cầu auth trong spec |
| Token tampering | Không trực tiếp | Endpoint không yêu cầu `Authorization` theo spec |
| Role escalation | Không trực tiếp | Endpoint không nhận role, nhưng có thể test extra fields ở phase sau |

## 10. Traceability sơ bộ

| Requirement/source | Nội dung liên quan | Ảnh hưởng đến phase sau |
|---|---|---|
| `docs/api_specification.md` - Coupons 5.1 | `POST /api/apply-coupon`, body `code`, `total_amount`, `user_id`, response `discount_amount`, `final_amount` | Nguồn chính cho schema và input inventory |
| `docs/api_specification.md` - Admin coupons 6.4 | Coupon có `type`, `discount_value`, `min_order_amount`, `expired_at`, `max_uses_per_user` | Gợi ý dependency/business-rule coverage, nhưng không đủ để tự suy ra expected |
| Assignment Pool B/FR-09 | Discount coupons | Gắn API với feature pool đã chọn |
| Assignment Requirements | Domain partitions, security, schema validation | Phase sau phải bao phủ validation/security nhưng không sinh test ở Phase 1 |
| `README.md` | Header `X-Student-Id: 23127364` | Mọi executable request plan sau này phải có header này |
| `AGENT.md` | Không suy diễn status/schema/rule | Các negative cases sau này cần `SPEC_UNDEFINED` nếu spec thiếu expected |

## 11. Open questions

1. Success status code của `POST /api/apply-coupon` là gì?
2. Error response shape chuẩn là gì?
3. `code`, `total_amount`, `user_id` có bắt buộc chính thức không?
4. `code` có case-sensitive không?
5. `code` có được trim whitespace không?
6. Coupon hợp lệ mặc định trong seed data là gì?
7. `total_amount` có phải `> 0` không? Có cho phép decimal không?
8. Có giới hạn max/min cho `total_amount` không?
9. `user_id` có bắt buộc tồn tại không?
10. Endpoint có cần authentication không, hay chỉ dùng `user_id` trong body?
11. Công thức tính `discount_amount` và `final_amount` cho `percent` là gì?
12. Có coupon fixed-amount không, ngoài `percent` trong ví dụ admin?
13. `min_order_amount`, `expired_at`, `max_uses_per_user` có được enforce tại `POST /api/apply-coupon` không?
14. Áp dụng coupon có thay đổi usage count/persistent state không?
15. Response có được phép chứa thông tin chi tiết coupon không?

## 12. Kết luận phase

`POST /api/apply-coupon` có đặc tả tối thiểu: nhận JSON body gồm `code`, `total_amount`, `user_id`; tính toán giảm giá; trả JSON chứa `discount_amount` và `final_amount`. Spec chưa nêu success status code, error status code, error schema, validation constraints, công thức tính discount, rounding, coupon expiration, usage-limit behavior hoặc authentication rule.

Phase này dừng ở `PHASE_1_SPEC_ANALYSIS`. Không tạo test case.
