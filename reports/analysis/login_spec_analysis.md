# PHASE_1_SPEC_ANALYSIS - `POST /api/login`

## 1. Phạm vi và nguồn đặc tả

- `api`: `POST /api/login`
- `phase`: `PHASE_1_SPEC_ANALYSIS`
- `artifact`: `reports/analysis/login_spec_analysis.md`
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `selected_pool`: Pool A - Authentication

Nguồn đã sử dụng:

| source | Nội dung dùng để phân tích |
|---|---|
| `README.md` | Xác nhận API đã chọn, `baseUrl`, `student_id`, header bắt buộc khi thực thi |
| `docs/api_specification.md` | Đặc tả chính thức của `POST /api/login` |
| `docs/2026.HW06.API Testing_En.md` | Yêu cầu bài tập: AI-first workflow, human review, domain partitions, security, schema validation, Postman/Newman, `X-Student-Id` |
| `AGENT.md` | Quy trình phase, nguyên tắc không tự suy diễn behavior, yêu cầu artifact và ngôn ngữ |

Kết luận trạng thái: phân tích này là bản nháp AI cho `PHASE_1_SPEC_ANALYSIS`, chưa được đánh dấu là human-approved. Không có test case nào được tạo trong phase này.

## 2. Đặc tả endpoint

| Thuộc tính | Giá trị theo đặc tả |
|---|---|
| `method` | `POST` |
| `path` | `/api/login` |
| `baseUrl` | `http://localhost:3000` |
| `full_url` | `http://localhost:3000/api/login` |
| `Content-Type` | JSON body được mô tả trong spec; khi thực thi nên dùng `Content-Type: application/json` |
| `X-Student-Id` | Bắt buộc trong mọi executable request theo `README.md` và assignment: `X-Student-Id: 23127364` |
| `Authorization` | Không được yêu cầu cho login trong `docs/api_specification.md` |
| `path_params` | Không có |
| `query_params` | Không có |
| `body` | JSON gồm `email`, `password` |

Request body được đặc tả:

```json
{
  "email": "test@domain.com",
  "password": "Password123!"
}
```

Phản hồi thành công được đặc tả:

| Điều kiện | `expected_status` | `expected_response` |
|---|---:|---|
| Đăng nhập thành công với thông tin hợp lệ | `200 OK` | Trả về chuỗi JWT `token` và thông tin `user` |

## 3. Inventory trường request

| Field | Type theo ví dụ/spec | Required/Optional | Ràng buộc được nêu rõ | Phụ thuộc dữ liệu |
|---|---|---|---|---|
| `email` | string | Có trong body mẫu; spec không ghi riêng chữ "required" | Spec chỉ đưa ví dụ `test@domain.com`; không mô tả regex/format cụ thể | Cần tài khoản đã tồn tại, ví dụ có thể tạo bằng `POST /api/register` |
| `password` | string | Có trong body mẫu; spec không ghi riêng chữ "required" | Spec chỉ đưa ví dụ `Password123!`; không mô tả độ dài/độ phức tạp khi login | Cần mật khẩu khớp với tài khoản |

Ghi chú: Vì spec login không định nghĩa validation chi tiết cho `email` và `password`, các phân vùng như missing, empty, null, wrong type, invalid format, SQL injection, whitespace/case handling nên được thiết kế ở phase sau như test-design/exploratory candidates, nhưng không được gán expected status cụ thể nếu không có bằng chứng từ spec hoặc SUT.

## 4. Authentication và Authorization

- `POST /api/login` là endpoint xác thực, vì vậy không yêu cầu `Authorization: Bearer <token>` trong đặc tả.
- Các API khác trong tài liệu yêu cầu token sau khi login, nhưng login chính nó là điểm phát hành `token`.
- Không có vai trò `admin` hoặc authorization rule nào được mô tả cho endpoint này.
- Khi chuẩn bị execution sau này, vẫn phải thêm `X-Student-Id: 23127364` theo yêu cầu bài tập, dù đây không phải header nghiệp vụ của login.

## 5. Response schema đã biết

Schema thành công chỉ được mô tả ở mức khái quát:

| Field | Type/shape được mô tả | Bắt buộc theo spec | Ghi chú kiểm thử schema |
|---|---|---|---|
| `token` | JWT string | Có, vì spec nói trả về chuỗi JWT `token` | Có thể assert field tồn tại và là string non-empty; cấu trúc JWT có thể kiểm tra dạng 3 phần phân tách bởi `.` nếu SUT trả JWT thật |
| `user` | object/thông tin user | Có, vì spec nói trả về thông tin `user` | Spec không liệt kê các field con của `user`; không nên tự yêu cầu `id`, `name`, `email`, `role` nếu chưa xác minh |

Sensitive data rule:

- Spec không nói rõ các field nhạy cảm bị cấm trong `user`.
- Tuy nhiên, theo nguyên tắc security/schema thông thường, cần kiểm tra response không rò rỉ `password`, `password_hash`, reset token hoặc thông tin bí mật khác. Đây là assertion security hợp lý nhưng cần ghi rõ là kiểm tra rủi ro, không phải field-level rule đã được spec mô tả chi tiết.

## 6. Status code và error behavior

Status code được nêu rõ:

| Trường hợp | Status code |
|---|---:|
| Login thành công | `200 OK` |

Các status code chưa được spec định nghĩa:

- Sai mật khẩu.
- Email không tồn tại.
- Thiếu `email`.
- Thiếu `password`.
- `email` sai format.
- `email` hoặc `password` rỗng.
- `email` hoặc `password` là `null`.
- Sai type, ví dụ number/object/array/boolean.
- Body không phải JSON hợp lệ.
- Body thiếu hoàn toàn.
- Tài khoản bị khóa/account lockout.

Vì vậy, trong phase tạo test sau này, các case âm tính có thể có assertion dạng "SUT phải trả lỗi, không trả `200 OK` và không trả `token`", nhưng không nên tự gán `400`, `401`, `404`, hoặc `422` nếu chưa có nguồn đặc tả bổ sung hoặc bằng chứng execution.

## 7. Business rules liên quan

Được xác nhận từ spec:

- Login nhận `email` và `password`.
- Login thành công trả `token` dạng JWT và `user`.
- `token` được dùng cho các API yêu cầu `Authorization: Bearer <token>` sau đó.

Được nhắc trong assignment nhưng chưa được mô tả chi tiết trong `api_specification.md`:

- Assignment liệt kê `FR-02` là "Login and account lockout".
- `api_specification.md` không mô tả account lockout, số lần thử sai, thời gian khóa, response khi bị khóa, hoặc cách reset lockout.
- Vì thiếu chi tiết, account lockout chỉ nên được đưa vào `Open Questions` hoặc exploratory/security coverage, không được xem là requirement cụ thể.

Không thấy trong spec:

- Không có yêu cầu email case-insensitive.
- Không có yêu cầu trim whitespace.
- Không có yêu cầu password complexity khi login.
- Không có refresh token.
- Không có expiration behavior của token trong login response.
- Không có mô tả rate limit.

## 8. Security requirements áp dụng được

Assignment nói API testing phải bao phủ security như SQL injection, IDOR, role escalation và các yêu cầu `SEC-01` đến `SEC-07`; tuy nhiên file `api_specification.md` hiện có không liệt kê chi tiết `SEC-01` đến `SEC-07`. Với riêng `POST /api/login`, các rủi ro áp dụng hợp lý là:

| Risk | Áp dụng cho login? | Cơ sở/ghi chú |
|---|---|---|
| Account enumeration | Có | So sánh response giữa email không tồn tại và sai password; expected cụ thể chưa được spec hóa |
| Brute force/account lockout | Có thể | Assignment nhắc FR-02 account lockout, nhưng spec thiếu rule chi tiết |
| Injection | Có | `email` và `password` là input string; kiểm tra injection-style input là phù hợp với yêu cầu bài tập |
| Sensitive data leakage | Có | Response có `user`; cần kiểm tra không lộ password/hash/secret |
| Token tampering | Không trực tiếp cho login request | Login phát hành token; tampering phù hợp hơn với API dùng `Authorization` |
| Unauthenticated access | Không áp dụng như negative auth | Login không cần token theo spec |
| Authorization bypass/role escalation/IDOR | Không trực tiếp | Endpoint không có path resource ID và không nhận role trong body theo spec |

## 9. State và dependency

`POST /api/login` không được mô tả là thay đổi persistent state, ngoại trừ khả năng tạo/ghi nhận session/token ở phía server nếu SUT có cơ chế đó. Spec không nói rõ login attempt count, lockout state, last login, refresh token, hoặc session storage.

Dependency cần có để test hợp lệ:

- Một user đã tồn tại với `email` và `password` biết trước.
- Có thể chuẩn bị user bằng `POST /api/register` nếu môi trường test cho phép.
- Cần xác định dữ liệu fixture hoặc tài khoản seed của SUT trước khi execution.
- Không được tự bịa admin/user credentials nếu repo hoặc SUT chưa cung cấp.

## 10. Traceability sơ bộ

| Requirement/source | Nội dung liên quan | Ảnh hưởng đến phase sau |
|---|---|---|
| `docs/api_specification.md` - Authentication 1.2 | `POST /api/login`, body `email/password`, success `200 OK`, trả `token` và `user` | Là nguồn chính cho positive case và schema assertion |
| `README.md` | Header thực thi `X-Student-Id: 23127364` | Mọi request executable sau này phải có header này |
| Assignment section Requirements | Cần domain partitions, security, schema validation, human audit, execution evidence | Phase sau phải thiết kế coverage nhưng chưa tạo test case ở phase này |
| Assignment Pool A/FR-02 | Login/account lockout | Account lockout là câu hỏi mở vì API spec thiếu chi tiết |
| `AGENT.md` | Không suy diễn status/schema/rule; phải ghi ambiguous behavior | Các negative case sau này cần đánh dấu uncertainty nếu expected không được spec hóa |

## 11. Open questions

1. `email` và `password` có bắt buộc chính thức không, và lỗi khi thiếu field là status code nào?
2. Response lỗi chuẩn có shape nào, ví dụ `{ "message": "..." }` hay field khác?
3. Sai password và email không tồn tại có dùng cùng message/status để tránh account enumeration không?
4. Có account lockout không? Nếu có, số lần thử sai, thời gian khóa, trạng thái response và cách reset là gì?
5. `email` có được normalize bằng trim/lowercase không?
6. `password` có trim whitespace không, hay so khớp chính xác từng ký tự?
7. `user` trong response gồm những field nào, và có được phép chứa `role` không?
8. Token JWT có expiration claim (`exp`) bắt buộc không?
9. Có rate limiting cho login không?
10. SUT có cung cấp seeded user/admin credentials cho execution không, hay phải tạo bằng `POST /api/register`?

## 12. Kết luận phase

`POST /api/login` có đặc tả tối thiểu: nhận JSON body gồm `email` và `password`; thành công trả `200 OK` với JWT `token` và thông tin `user`. Đặc tả chưa nêu error status codes, error schema, validation constraints, account lockout rules hoặc cấu trúc chi tiết của `user`.

Phase tiếp theo, nếu được duyệt để thực hiện, nên là `PHASE_2_TEST_DESIGN`: lập phân vùng input, security coverage, schema coverage và danh sách ambiguity/exploratory areas. Không tạo test case trong phase hiện tại.
