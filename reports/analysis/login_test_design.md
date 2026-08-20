# PHASE_2_TEST_DESIGN - `POST /api/login`

## 1. Phạm vi phase

- `api`: `POST /api/login`
- `phase`: `PHASE_2_TEST_DESIGN`
- `artifact`: `reports/analysis/login_test_design.md`
- `based_on`: `reports/analysis/login_spec_analysis.md`
- `spec_analysis_status`: human-reviewed theo yêu cầu của người dùng
- `student_id`: `23127364`
- `base_url`: `http://localhost:3000`
- `request_url`: `{{baseUrl}}/api/login`

Phase này chỉ thiết kế phạm vi kiểm thử. Không tạo `LOGIN-GEN-*`, không tạo `LOGIN-EXT-*`, không ghi execution result, không kết luận bug.

Nguồn áp dụng:

| source | Vai trò |
|---|---|
| `docs/api_specification.md` | Nguồn chính cho behavior của `POST /api/login` |
| `reports/analysis/login_spec_analysis.md` | Phân tích đặc tả đã được human-reviewed |
| `AGENT.md` | Quy trình phase, ranh giới human audit, yêu cầu artifact và tiếng Việt |
| `agent-skill/SKILL.md` | Cách xây dựng parameter partitions, security dimensions, schema dimensions, state dimensions, ambiguity list |

## 2. Endpoint baseline cho test design

| Thuộc tính | Giá trị thiết kế |
|---|---|
| `method` | `POST` |
| `path` | `/api/login` |
| `headers` | `Content-Type: application/json`, `X-Student-Id: {{studentId}}` |
| `Authorization` | Không yêu cầu theo spec |
| `path_params` | Không có |
| `query_params` | Không có |
| `body_fields` | `email`, `password` |
| `success_status` | `200 OK` |
| `success_schema_known` | Có `token` và `user` |
| `error_status` | `SPEC_UNDEFINED` |
| `error_schema` | `SPEC_UNDEFINED` |

Nguyên tắc expected cho phase sau:

- Với happy path, có thể dùng `expected_status = 200` vì spec nêu rõ.
- Với negative/error behavior, không tự gán `400`, `401`, `404`, `422` nếu chưa có nguồn bổ sung.
- Với input không hợp lệ, assertion tối thiểu nên là không trả `200 OK`, không trả `token`, và response không rò rỉ thông tin nhạy cảm; status cụ thể để `SPEC_UNDEFINED` nếu spec không định nghĩa.

## 3. Parameter Inventory

### 3.1 `email`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid class | Email của user đã tồn tại, ví dụ user được tạo bằng `POST /api/register` | Với `password` đúng, kỳ vọng `200 OK`, có `token`, có `user` |
| semantic invalid | Email đúng format nhưng không tồn tại trong database | Error status/message chưa được spec định nghĩa; không được trả `token` |
| malformed format | Không có `@`, không có domain, domain thiếu dấu `.`, chứa khoảng trắng giữa chuỗi, chuỗi không giống email | Validation rule chưa được spec định nghĩa; dùng `SPEC_UNDEFINED` cho status |
| missing value | Body không có key `email` | Required status chưa ghi rõ nhưng body mẫu có `email`; không được login thành công |
| empty value | `email: ""` | Không được login thành công; exact error undefined |
| whitespace-only | `email: "   "` | Cần quan sát trim/validation; exact error undefined |
| leading/trailing whitespace | `email: " test@domain.com "` | Whitespace normalization chưa được spec định nghĩa; exploratory |
| null | `email: null` | Wrong/null handling chưa được spec định nghĩa; không được trả `token` |
| wrong type | number, boolean, object, array | Type validation chưa được spec định nghĩa; không được login thành công |
| boundary values | Email rất ngắn, email rất dài, local-part/domain dài, ký tự đặc biệt hợp lệ trong email | Spec không nêu min/max; dùng để phát hiện validation/server robustness |
| case behavior | `Test@Domain.com` so với `test@domain.com` | Case normalization chưa rõ; exploratory |
| injection-style | SQL-like string, NoSQL-like object nếu parser cho phép, script-like string | Không được bypass login hoặc gây rò rỉ lỗi nội bộ |
| unexpected extra value | Body có thêm field như `role`, `isAdmin`, `token` | Endpoint không mô tả extra fields; không được role escalation hoặc ảnh hưởng auth |

### 3.2 `password`

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid class | Password đúng của user đã tồn tại | Với `email` đúng, kỳ vọng `200 OK`, có `token`, có `user` |
| semantic invalid | Password sai cho email tồn tại | Error status/message chưa được spec định nghĩa; không được trả `token` |
| missing value | Body không có key `password` | Không được login thành công; exact error undefined |
| empty value | `password: ""` | Không được login thành công; exact error undefined |
| whitespace-only | `password: "   "` | Không được login thành công trừ khi đó là password thật; exact error undefined |
| leading/trailing whitespace | `password: " Password123! "` | Password trim behavior chưa rõ; exploratory |
| null | `password: null` | Null handling chưa được spec định nghĩa; không được trả `token` |
| wrong type | number, boolean, object, array | Type validation chưa được spec định nghĩa; không được login thành công |
| boundary values | Password rất ngắn, rất dài, ký tự Unicode/đặc biệt | Login spec không nêu min/max; dùng để kiểm tra robustness, không tự suy ra password policy |
| case behavior | Thay đổi chữ hoa/thường trong password | Password case behavior chưa được spec định nghĩa; exploratory,
không gán exact expected result. |
| injection-style | SQL-like string, command/script-like string | Không được bypass auth, không lộ stack trace/query |
| unexpected extra value | Body có thêm field `rememberMe`, `role`, `admin` | Spec không mô tả; không được ảnh hưởng kết quả auth ngoài `email/password` |

### 3.3 Request body và HTTP-level inputs

| Dimension | Classes/Values nên bao phủ | Expected/ghi chú |
|---|---|---|
| valid JSON object | Object có `email` và `password` string | Happy path hoặc credential error tùy dữ liệu |
| empty object | `{}` | Không được login thành công |
| body missing | Không gửi body | Error behavior undefined |
| invalid JSON | JSON malformed | Error behavior undefined; không được server crash |
| wrong top-level type | array, string, number, boolean | Error behavior undefined |
| `Content-Type` missing/wrong | Không có `Content-Type`, hoặc `text/plain` với body JSON | Parser behavior undefined; không được cấp token ngoài logic hợp lệ |
| `X-Student-Id` missing | Không gửi header bài tập | Đây là requirement execution của assignment; SUT behavior chưa được spec nghiệp vụ mô tả |
| duplicate keys | JSON có duplicate `email` hoặc `password` | Parser behavior phụ thuộc runtime; exploratory nếu công cụ cho phép gửi raw body |

## 4. Coverage theo nhóm kiểm thử

| Coverage group | Mục tiêu | Traceability |
|---|---|---|
| Positive authentication | Xác minh credential hợp lệ trả `200 OK`, `token`, `user` | `docs/api_specification.md` - Authentication 1.2 |
| Credential rejection | Xác minh wrong password và unknown email không được cấp `token` | Spec chỉ định success path; security expectation từ assignment |
| Required fields | Bao phủ missing/empty/null cho `email` và `password` | Body mẫu trong spec; `AGENT.md` yêu cầu missing/null/wrong-type |
| Type validation | Bao phủ wrong type cho từng field và top-level body | `AGENT.md` và `SKILL.md` yêu cầu wrong type |
| Format robustness | Bao phủ malformed email, whitespace, long strings | Assignment yêu cầu domain partitions/boundary |
| Injection robustness | Đảm bảo input dạng injection không bypass auth, không gây leakage | Assignment security coverage |
| Response schema | Xác minh success response có `token` và `user`, không có sensitive fields | Spec success response và security expectation |
| Account enumeration | So sánh observable response giữa wrong password và unknown email | Security risk áp dụng cho login; exact rule undefined |
| Account lockout/rate limit | Thăm dò nếu SUT có FR-02 lockout | Assignment nhắc FR-02, API spec thiếu chi tiết |

## 5. Security Coverage

| Risk | Applicability | Thiết kế coverage | Expected/ranh giới |
|---|---|---|---|
| Unauthenticated access | Không phải negative risk cho login | Không yêu cầu `Authorization`; login phải truy cập được khi chưa có token | Không tạo test "missing token must fail" cho login |
| Authorization bypass | Thấp/không trực tiếp | Gửi extra fields như `role`, `isAdmin` để kiểm tra không ảnh hưởng quyền | Không được cấp quyền từ body field không được spec mô tả |
| IDOR | Không áp dụng trực tiếp | Endpoint không có resource id/path param | Không đưa IDOR làm nhóm chính cho login |
| Role escalation | Áp dụng | Body thêm `role: "admin"` hoặc `isAdmin: true` ngoài hai field được mô tả là `email` và `password` | Extra fields không được spec định nghĩa; kiểm tra theo hướng exploratory và không giả định trước SUT sẽ ignore hay reject |
| Token tampering | Không áp dụng cho request login | Login không nhận token theo spec | Token quality kiểm ở schema, không phải tampered-token request |
| Injection | Áp dụng | SQL-like, NoSQL-like, script-like strings trong `email`/`password` | Không bypass auth, không rò rỉ stack trace/query/internal error |
| Information leakage | Áp dụng | Kiểm tra success/error response không lộ `password`, `password_hash`, reset token, stack trace | Sensitive-field list là security expectation, không phải schema chi tiết từ spec |
| Account enumeration | Áp dụng | So sánh status/message/timing ở unknown email và wrong password | Spec không nêu exact expected; ghi exploratory/observation |
| Brute force/account lockout | Có thể áp dụng | Lặp nhiều wrong password cho cùng email nếu môi trường cho phép | Assignment nhắc FR-02 nhưng spec thiếu threshold; không tự đặt số lần là requirement |
| Business-rule abuse | Áp dụng hạn chế | Extra fields, duplicate body keys, very large payload | Không được tạo token nếu `email/password` không hợp lệ |

## 6. Schema Coverage

### 6.1 Success response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Status là `200 OK` | Spec ghi rõ success response `200 OK` | Confirmed |
| Response có field `token` | Spec nói trả về chuỗi JWT `token` | Confirmed |
| `token` là string non-empty | JWT token theo spec | Confirmed ở mức type tổng quát |
| `token` có dạng JWT ba phần phân tách bằng `.` | Spec nói JWT; cấu trúc JWT chuẩn | Reasonable schema/security assertion |
| Response có field `user` | Spec nói trả về thông tin `user` | Confirmed |
| `user` là object | Spec chỉ nói response chứa thông tin `user`, nhưng không định nghĩa chính xác type/shape | `SPEC_UNDEFINED` - không dùng làm assertion bắt buộc cho đến khi có nguồn xác nhận |
| Response không có `password`, `password_hash`, `resetToken`, `newPassword` | Security leakage prevention | Security expectation |

### 6.2 Error response assertions

| Assertion | Cơ sở | Mức chắc chắn |
|---|---|---|
| Không trả `200 OK` cho credential/input không hợp lệ | Login success chỉ dành cho credential hợp lệ | Reasonable, nhưng exact status undefined |
| Không trả `token` trong error response | Không cấp token khi login không thành công | Security expectation |
| Error response không lộ stack trace, SQL query, internal path | Security expectation từ assignment | Security expectation |
| Error response shape cụ thể | Không có trong spec | `SPEC_UNDEFINED` |

### 6.3 Consistency checks

- Nếu `status = 200`, response phải có `token` và `user`.
- Nếu response có `token`, request phải là credential hợp lệ theo precondition.
- Nếu credential không hợp lệ, response không được chứa usable `token`.
- Nếu `user` xuất hiện, không được chứa secret fields.
- Nếu status/error shape khác nhau giữa unknown email và wrong password, cần ghi nhận rủi ro account enumeration thay vì vội kết luận bug nếu spec chưa định nghĩa.

## 7. State Coverage

Theo đặc tả hiện có, `POST /api/login` không mô tả state transition chính thức. Vì vậy không thiết kế state machine như order status.

State/dependency cần xem xét:

| State/Dependency | Vai trò | Thiết kế coverage |
|---|---|---|
| Existing registered user | Precondition cho happy path | Chuẩn bị bằng seeded account hoặc `POST /api/register` |
| Non-existing user | Negative semantic partition | Dùng email chắc chắn chưa tồn tại trong môi trường test |
| Password mismatch | Negative semantic partition | Dùng email tồn tại với password sai |
| Account lockout state | Open question | Chỉ exploratory nếu SUT có behavior; không tự định nghĩa threshold |
| Token validity after login | Cross-request dependency cho phase sau | Có thể verify token bằng API cần auth ở phase implementation nếu scope cho phép, nhưng không phải requirement trực tiếp của login spec |
| Repeated successful login | Replay/idempotence observation | Spec không nói session behavior; chỉ kiểm tra không crash và mỗi response hợp lệ |

Persisted-state verification:

- Không có persisted state chính thức cần verify cho login.
- Nếu account lockout được xác minh bằng tài liệu hoặc SUT behavior, cần thêm persisted-state verification ở phase sau.

## 8. Boundary và robustness strategy

| Area | Boundary/robustness idea | Ghi chú |
|---|---|---|
| `email` length | Rất ngắn, rất dài, domain/local-part dài | Spec không có min/max; không gán exact status |
| `password` length | Rất ngắn, rất dài | Không suy ra password policy khi login |
| Payload size | Extra-long strings hoặc nhiều extra fields | Dùng để phát hiện crash/leakage |
| Unicode | Vietnamese accents/Unicode trong `email` hoặc `password` | Email format support chưa rõ; password Unicode phụ thuộc đăng ký/user fixture |
| Whitespace | Leading/trailing/only whitespace | Normalization chưa rõ |
| Case sensitivity | Email case, password case | Email/password behavior chưa được spec mô tả chi tiết |
| Duplicate keys | Duplicate `email/password` trong raw JSON | Parser-dependent; exploratory |

## 9. Dữ liệu và setup cần chuẩn bị trước generation/execution

Không invent credentials. Trước khi tạo executable cases cần xác định:

| Setup item | Trạng thái hiện tại | Cách xử lý đề xuất |
|---|---|---|
| `baseUrl` | Confirmed: `http://localhost:3000` | Dùng variable `{{baseUrl}}` |
| `studentId` | Confirmed: `23127364` | Dùng variable `{{studentId}}` và header `X-Student-Id` |
| Valid user email/password | Missing | Tạo user bằng `POST /api/register` hoặc dùng seeded credential nếu repo/SUT cung cấp |
| Unknown email | Missing | Tạo giá trị unique không tồn tại trong test data |
| Locked account | Unknown | Chỉ dùng nếu xác minh SUT có lockout |
| Error schema | Unknown | Quan sát khi execution, không tự đặt schema bắt buộc |

## 10. Traceability map cho phase generation sau này

| Design area | Requirement/spec reference | Gợi ý coverage_type cho phase sau |
|---|---|---|
| Happy path login | `docs/api_specification.md` - `POST /api/login` success `200 OK` | `positive`, `schema` |
| `email` partitions | Request body field `email`; assignment domain partitions | `domain`, `validation`, `boundary` |
| `password` partitions | Request body field `password`; assignment domain partitions | `domain`, `validation`, `boundary` |
| Missing/null/wrong type | `AGENT.md`, `SKILL.md`, assignment coverage | `negative`, `schema`, `validation` |
| Injection-style inputs | Assignment security examples | `security`, `injection` |
| Sensitive data leakage | Response includes `user`; assignment security/schema | `security`, `schema` |
| Account enumeration | Login security risk; FR-02 related area | `security`, `exploratory` |
| Account lockout/rate limit | Assignment mentions FR-02 but spec lacks details | `security`, `state`, `exploratory` |
| Extra fields/security-sensitive fields | Endpoint chỉ mô tả `email/password` | `security`, `negative`, `exploratory` |

## 11. Open Questions đưa sang human checkpoint/generation

1. Error status code chuẩn cho invalid credentials là gì?
2. Error status code chuẩn cho validation errors là gì?
3. Error response có bắt buộc field `message` không?
4. SUT có intentionally dùng cùng response cho unknown email và wrong password để giảm account enumeration không?
5. Account lockout có được implement không? Nếu có, threshold và thời gian khóa là gì?
6. `email` có trim và lowercase trước khi so khớp không?
7. `password` có trim không hay exact match?
8. `user` response gồm field nào, và field nào bị cấm?
9. Token JWT có bắt buộc chứa `exp`, `iat`, `id`, `role` hoặc claim nào khác không?
10. Có seeded credential chính thức trong SUT không?
11. `X-Student-Id` thiếu/sai có ảnh hưởng SUT không, hay chỉ là yêu cầu evidence của assignment?
12. Body có extra fields thì SUT ignore hay reject?

## 12. Generation guardrails cho `PHASE_3_AI_GENERATION`

Khi được phép chuyển sang phase tạo candidate cases:

- Tạo ít nhất 35 AI-generated candidate cases cho `POST /api/login`.
- Dùng prefix `LOGIN-GEN-001`.
- Mọi case phải có `source = AI_GENERATED`.
- Mọi case phải có `audit_status = PENDING_HUMAN_REVIEW`.
- Không dùng `LOGIN-EXT-*` trong AI generation.
- Với success path, có thể đặt `expected_status = 200`.
- Với behavior spec chưa định nghĩa, dùng `expected_status = SPEC_UNDEFINED` hoặc assertion không cấp `token`, kèm `assumption_or_open_question`.
- Mọi executable request plan phải có `X-Student-Id: {{studentId}}`.
- Không ghi pass/fail/bug nếu chưa có Postman/Newman evidence.

## 13. Kết luận phase

`PHASE_2_TEST_DESIGN` đã xác định các dimension chính cho `POST /api/login`: parameter partitions cho `email`, `password` và request body; security coverage cho injection, information leakage, account enumeration và lockout exploratory; schema coverage cho `token`/`user`; cùng các open questions do spec chưa định nghĩa error behavior.

Đúng theo yêu cầu, artifact này dừng ở test design và chưa tạo bất kỳ test case nào.
