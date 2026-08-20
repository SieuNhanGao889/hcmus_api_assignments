# PHASE_5_HUMAN_EXTENSION_PREPARATION - `POST /api/login`

## 1. Phạm vi

- `api`: `POST /api/login`
- `phase`: preparation for `PHASE_5_HUMAN_EXTENSION`
- `artifact`: `reports/analysis/login_extension_gaps.md`
- `audited_suite`: `test-cases/login_test_cases_audited.xlsx`
- `source_design`: `reports/analysis/login_test_design.md`
- `student_id`: `23127364`

Mục tiêu của artifact này là phân tích bộ test đã được human audit và chỉ ra các khoảng trống còn lại để sinh viên có thể thiết kế manual extension cases ở bước sau.

Không tạo `LOGIN-EXT-*` trong artifact này. Không thay đổi audit decision. Không execute test. Không kết luận bug.

## 2. Tóm tắt bộ test đã audit

| Metric | Giá trị |
|---|---:|
| Tổng số AI-generated cases | `40` |
| `VALID` | `38` |
| `INCOMPLETE` | `2` |
| `INVALID` | `0` |
| `source` | `AI_GENERATED` |
| Manual extension đã có | `0` |

Hai case còn `INCOMPLETE` theo human audit:

| case_id | coverage_type | Lý do chính | Correction đã được ghi |
|---|---|---|---|
| `LOGIN-GEN-002` | `schema,security` | Assertion về JWT ba phần vượt quá mức spec mô tả trực tiếp | Giữ assertion bắt buộc ở mức `token` tồn tại và là string non-empty; chuyển JWT structure thành soft-check |
| `LOGIN-GEN-040` | `security,state,exploratory` | Account lockout/rate limit chưa có threshold/rule trong `api_specification.md` | Giữ exploratory; không tự đặt threshold/rule; chỉ ghi nhận quan sát |

## 3. Coverage hiện có

Bộ AI-generated đã bao phủ khá rộng các nhóm sau:

| Coverage group | Tình trạng |
|---|---|
| Happy path | Có case đăng nhập thành công với credential hợp lệ |
| Success schema | Có kiểm tra `token`, `user`, sensitive-field leakage |
| Invalid credentials | Có unknown email, wrong password, unknown email + wrong password |
| Missing/empty/null | Có cho `email`, `password`, empty object, missing body |
| Wrong type | Có number/object/array và top-level array |
| Email format | Có malformed email không có `@`, domain thiếu hợp lệ |
| Whitespace/case | Có leading/trailing whitespace, whitespace-only, case variant |
| Boundary/robustness | Có very long email/password |
| Injection | Có SQL-like, NoSQL/operator-style, script-like input |
| Extra fields/role escalation | Có `role` và `isAdmin` extra fields |
| HTTP parser/header | Có missing/wrong `Content-Type`, malformed JSON |
| Account enumeration | Có semantic comparison areas qua unknown email/wrong password |
| Account lockout/rate limit | Có một exploratory case nhưng bị audit là `INCOMPLETE` vì thiếu rule trong spec |

## 4. Remaining Coverage Gaps

### Gap 1 - Cross-request token usability

AI suite kiểm tra `token` tồn tại trong login response, nhưng chưa thiết kế coverage rõ ràng để dùng token đó gọi một API yêu cầu `Authorization: Bearer <token>` như `GET /api/users/me`.

Giá trị manual extension:

- Kiểm tra token không chỉ tồn tại mà còn usable cho authenticated API.
- Tạo traceability từ login sang các API yêu cầu token trong `docs/api_specification.md`.
- Phát hiện lỗi dạng login trả token malformed hoặc token không khớp user.

Ranh giới:

- Đây là cross-request dependency, không phải schema đơn thuần của login.
- Cần setup user hợp lệ và cần endpoint xác thực phụ trợ.
- Không được claim pass/fail nếu chưa execution.

### Gap 2 - Token/user consistency

AI suite có kiểm tra `token` và `user`, nhưng chưa có manual-level check về consistency giữa thông tin `user` trong response và identity bên trong token hoặc API `GET /api/users/me`.

Giá trị manual extension:

- Phát hiện response trả `user` của người khác trong khi token thuộc user hiện tại.
- Phát hiện token hợp lệ nhưng user object sai hoặc bị stale.
- Tăng coverage cho information leakage/cross-user behavior.

Ranh giới:

- `api_specification.md` chưa mô tả claim cụ thể trong JWT, nên không tự yêu cầu claim `id`, `email`, hoặc `role`.
- Nếu không decode token, có thể kiểm tra consistency gián tiếp bằng API `GET /api/users/me`.

### Gap 3 - Duplicate JSON keys

Phase 2 đã nêu duplicate keys là exploratory, nhưng AI suite chưa có case cụ thể gửi raw JSON có duplicate `email` hoặc duplicate `password`.

Giá trị manual extension:

- Kiểm tra parser behavior khi request có cùng key lặp lại.
- Phát hiện bypass hoặc ambiguity nếu parser lấy first value/last value không như mong đợi.
- Hữu ích với security/type confusion, đặc biệt khi một key hợp lệ và một key malicious cùng tồn tại.

Ranh giới:

- JSON duplicate key behavior phụ thuộc runtime/parser.
- Expected status nên để `SPEC_UNDEFINED`; assertion chính là không bypass auth và không rò rỉ lỗi nội bộ.

### Gap 4 - `X-Student-Id` execution evidence behavior

AI suite có mọi request plan với `X-Student-Id`, nhưng chưa có negative/exploratory coverage cho thiếu hoặc sai `X-Student-Id`.

Giá trị manual extension:

- Giúp làm rõ header này chỉ là assignment evidence requirement hay SUT có kiểm tra thật.
- Hữu ích cho Postman pre-request script/evidence sau này.

Ranh giới:

- `X-Student-Id` là yêu cầu assignment, không phải business rule của login trong `api_specification.md`.
- Nếu tạo manual case, cần đánh dấu exploratory và không tự kết luận bug nếu SUT vẫn xử lý request khi header thiếu/sai.

### Gap 5 - Account enumeration comparison as paired observation

AI suite có separate cases cho unknown email và wrong password, nhưng chưa có paired assertion rõ ràng để so sánh response observable giữa hai request trong cùng execution flow.

Giá trị manual extension:

- Kiểm tra khác biệt về `status`, `message`, response length, hoặc timing ở mức quan sát.
- Tập trung vào rủi ro account enumeration thay vì chỉ kiểm tra từng request âm tính riêng lẻ.

Ranh giới:

- Spec chưa định nghĩa phải trả cùng message/status.
- Nếu phát hiện khác biệt, chỉ ghi nhận risk/observation; cần human analysis trước khi bug classification.

### Gap 6 - Rate limiting/account lockout setup safety

`LOGIN-GEN-040` đã có exploratory idea cho nhiều lần login sai nhưng bị human audit là `INCOMPLETE` vì thiếu threshold/rule. Manual extension có thể thiết kế lại theo hướng an toàn hơn: chỉ thăm dò có kiểm soát, không tự đặt pass/fail cứng.

Giá trị manual extension:

- Bao phủ requirement area `FR-02 Login and account lockout` được assignment nhắc.
- Giúp ghi nhận implementation behavior mà AI case ban đầu chưa đủ setup guardrail.
- Có thể yêu cầu dùng tài khoản test riêng để tránh khóa tài khoản chính.

Ranh giới:

- Không tự đặt threshold như 3/5/10 lần nếu spec không bổ sung.
- Không chạy trong môi trường dùng chung nếu có nguy cơ khóa account.
- Không kết luận bug nếu thiếu requirement rõ ràng.

### Gap 7 - Error response leakage for malformed HTTP-level requests

AI suite có malformed JSON và wrong `Content-Type`, nhưng chưa nhấn mạnh kiểm tra leakage chi tiết cho HTTP parser errors, ví dụ stack trace, file path, framework error, SQL details.

Giá trị manual extension:

- Tập trung vào security hardening cho error path.
- Có thể phát hiện lỗi implementation bị bỏ sót vì AI chủ yếu tập trung vào auth logic.

Ranh giới:

- Error schema/status vẫn `SPEC_UNDEFINED`.
- Assertion nên là không lộ internal details, không phải yêu cầu message cụ thể.

### Gap 8 - Password exact-match với ký tự đặc biệt/Unicode đã được register

AI suite có password very long và whitespace/case variants, nhưng chưa có coverage rõ ràng cho một user được đăng ký với password chứa ký tự đặc biệt/Unicode rồi login đúng credential đó.

Giá trị manual extension:

- Phân biệt validation lúc register với exact-match lúc login.
- Kiểm tra encoding/normalization thật của password.
- Hữu ích trong môi trường tiếng Việt vì người dùng có thể dùng ký tự Unicode.

Ranh giới:

- Cần setup account bằng `POST /api/register`.
- Không suy ra password policy nếu register không chấp nhận password đó; lúc đó case phải được chỉnh theo setup thực tế.

### Gap 9 - Body top-level primitive values

AI suite đã có top-level array, empty object, missing body, malformed JSON, nhưng chưa có top-level string/number/boolean.

Giá trị manual extension:

- Hoàn thiện wrong top-level type coverage đã nêu trong Phase 2.
- Phát hiện parser/type handling khác nhau giữa array và primitive.

Ranh giới:

- Error status/schema chưa được spec định nghĩa.
- Assertion chính: không cấp `token`, không crash, không lộ lỗi nội bộ.

### Gap 10 - Request with valid credential plus irrelevant security-sensitive extra fields beyond `role/isAdmin`

AI suite đã có `role` và `isAdmin`, nhưng chưa có các extra fields khác như `user_id`, `id`, `email_verified`, `permissions`, hoặc nested `user`.

Giá trị manual extension:

- Tăng coverage cho mass assignment/over-posting style risk.
- Kiểm tra login chỉ dựa trên `email/password`, không nhận identity/permission từ request body.

Ranh giới:

- Spec không định nghĩa behavior với extra fields, nên không tự bắt SUT phải reject.
- Assertion phù hợp: extra fields không được thay đổi identity/quyền trong response hoặc token.

## 5. Ưu tiên đề xuất cho manual extension

Nên ưu tiên các gap có giá trị cao và không trùng lặp quá nhiều với AI suite:

| Priority | Gap | Lý do |
|---:|---|---|
| 1 | Cross-request token usability | Biến token từ schema check thành behavior check có ý nghĩa |
| 2 | Token/user consistency | Bao phủ cross-user/security risk mà AI chưa đi sâu |
| 3 | Paired account enumeration observation | Tốt hơn các negative cases rời rạc hiện có |
| 4 | Duplicate JSON keys | Parser/security edge case chưa được AI sinh thành row |
| 5 | Controlled lockout/rate-limit exploratory | Liên quan `FR-02`, nhưng cần guardrail do spec thiếu rule |
| 6 | `X-Student-Id` exploratory | Hữu ích cho assignment evidence, nhưng không phải business requirement |
| 7 | HTTP parser error leakage | Tăng security quality cho malformed paths |
| 8 | Unicode/special password setup | Hữu ích nhưng phụ thuộc register/setup |
| 9 | Top-level primitive body | Hoàn thiện validation matrix |
| 10 | Extra identity/permission fields | Mở rộng mass-assignment angle ngoài `role/isAdmin` |

## 6. Vì sao AI có thể đã bỏ sót

- AI đã ưu tiên breadth để đạt yêu cầu `>=35` cases nên nhiều coverage dừng ở single-request checks.
- Spec của login rất ngắn, không mô tả error schema, JWT claims, lockout threshold hoặc normalization rules, khiến AI tránh expected quá cụ thể.
- Một số gap cần cross-request setup với `POST /api/register` hoặc `GET /api/users/me`, vượt khỏi endpoint đơn lẻ `POST /api/login`.
- Các edge cases như duplicate JSON keys, top-level primitive body, mass-assignment fields thường cần kinh nghiệm security/manual exploratory hơn là sinh phân vùng cơ bản.

## 7. Kết luận

Bộ audit hiện tại đủ rộng cho AI-generated baseline, nhưng Phase 5 manual extension nên tập trung vào các điểm AI chưa bao phủ sâu: token usable sau login, consistency giữa `token` và `user`, account enumeration dạng paired observation, duplicate JSON keys, lockout/rate-limit exploratory có guardrail, và các parser/security edge cases.

Artifact này chỉ chuẩn bị cho `PHASE_5_HUMAN_EXTENSION`; chưa tạo manual extension cases.
