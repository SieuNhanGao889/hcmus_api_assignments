# Static review Postman artifacts trước execution

## 1. Phạm vi và phương pháp

Đã review tĩnh các file:

- `postman/HW06_EShop_API_Tests.postman_collection.json`
- `postman/HW06_Local.postman_environment.json`
- `postman/data/login_data.json`
- `postman/data/coupon_data.json`
- `postman/data/order_status_data.json`
- `reports/analysis/postman_implementation_notes.md`

Không khởi động backend, không gửi API request và không chạy Postman/Newman. Các kiểm tra chỉ gồm parse JSON, compile syntax JavaScript bằng `new Function`, đối chiếu placeholder/variable, kiểm tra thứ tự request, mapping data row, audit history và assertion source.

Kết quả kiểm tra nền:

- Cả năm JSON artifact parse thành công.
- `79` pre-request/post-response scripts hợp lệ về JavaScript syntax.
- `137/137` original `case_id` có mặt trong data files.
- `39/39` executable request objects khai báo rõ `X-Student-Id: {{studentId}}`.
- Không có runtime token hoặc runtime user/coupon/order ID thật trong artifact.

## 2. Tổng hợp kết quả theo 12 mục review

| # | Review item | Kết quả | Finding liên quan |
|---:|---|---|---|
| 1 | Variable scope hoặc unresolved variable | **FAIL** | F-01, F-02, F-03 |
| 2 | Setup/CASE/verification ordering | **FAIL** | F-02, F-04, F-05, F-06 |
| 3 | `X-Student-Id` | **PASS** | Không thiếu header trong bất kỳ request object nào; collection-level script còn upsert lại header |
| 4 | URL, method, body, auth | **FAIL** | F-02, F-12, F-13 |
| 5 | Assertion so với audited cases/approved source | **FAIL** | F-06, F-07, F-08, F-09, F-10, F-11 |
| 6 | `SPEC_UNDEFINED` | **FAIL** | F-08; exact HTTP status nhìn chung được xử lý đúng, nhưng một số case vẫn bị ép thành success |
| 7 | Correction `ORDERSTATUS-GEN-001` | **PASS** | Lịch sử audit được giữ; correction và source FR-10 đúng |
| 8 | Coupon Authorization | **PASS** | Mọi apply-coupon CASE/SEQUENCE đều dùng `Bearer {{userToken}}`; original headers vẫn được lưu riêng |
| 9 | Data row mapping | **FAIL** | F-01, F-02, F-05, F-11 |
| 10 | Postman Runner/Newman API compatibility | **PASS** | `pm.execution.skipRequest()` chỉ được gọi từ pre-request scripts và được Postman hỗ trợ trong Runner/Newman |
| 11 | Setup failure bị báo như selected-API failure | **FAIL** | F-03, F-04 |
| 12 | Fabricated ID/token/credential/result | **PASS** | Seed credentials có source; disposable values là setup input; runtime values rỗng/dynamic; không có result/evidence |

## 3. Findings chi tiết

### F-01 — Unresolved coupon variable

- **Severity:** HIGH
- **Affected file/request/case:** `coupon_data.json`, `COUPON-GEN-016`; request `CASE - Data-driven POST /api/apply-coupon`
- **Problem:** Body dùng `{{validCouponCodeLowercase}}`, nhưng không environment/collection variable hay pre-request script nào tạo biến này.
- **Why it matters:** Request có thể gửi literal `{{validCouponCodeLowercase}}` thay vì lowercase variant của controlled coupon. Case không kiểm tra case-sensitivity đúng mục tiêu và kết quả dễ bị hiểu sai thành unknown-coupon behavior.
- **Recommended correction:** Sau khi tạo `validCouponCode`, đặt `validCouponCodeLowercase = validCouponCode.toLowerCase()` trong coupon iteration initialization, hoặc tạo local variable tương đương trước CASE. Thêm guard fail-fast nếu còn unresolved placeholder.

### F-02 — `otherUserId` được gán trước khi có `adminUserId`

- **Severity:** HIGH
- **Affected file/request/case:** Collection, pre-request của `SETUP - Login confirmed seed admin` trong coupon folder; `COUPON-GEN-031`
- **Problem:** Coupon initialization chạy trước admin-login response và thực hiện `otherUserId = adminUserId`. Trong iteration đầu, `adminUserId` chưa tồn tại; trong iteration sau, nó có thể là stale value của iteration trước. Admin login post-response chỉ cập nhật `adminUserId`, không cập nhật lại `otherUserId`.
- **Why it matters:** `COUPON-GEN-031` có thể tạo invalid JSON (`"user_id":` không có giá trị), dùng stale identity, hoặc không còn đại diện cho một “other valid user”.
- **Recommended correction:** Gán `otherUserId` trong admin-login post-response ngay sau khi extract `j.user.id`, hoặc tạo một disposable second user riêng. Trước CASE, assert `otherUserId` tồn tại, là numeric ID và khác `validUserId`.

### F-03 — Runtime variables không được reset theo iteration

- **Severity:** HIGH
- **Affected file/request/case:** Collection và environment; đầu cả ba data-run folders
- **Problem:** Các init scripts ghi đè một số email/code nhưng không unset token/ID như `adminToken`, `userToken`, `loginToken`, coupon IDs, `orderUnderTestId`, `otherOrderId` và observation variables. Environment mutations tồn tại trong cùng run qua các iteration.
- **Why it matters:** Nếu setup ở iteration N thất bại, CASE của iteration N có thể tái sử dụng token/ID từ iteration N-1. Đây vừa là scope defect vừa có thể tạo false pass, mutate nhầm fixture, hoặc gán setup failure cho selected API.
- **Recommended correction:** Ở request đầu mỗi folder iteration, unset toàn bộ runtime variables của folder và đặt `setupReady=false`. Chỉ set từng readiness flag sau khi response setup đã được kiểm tra thành công.

### F-04 — Setup failure không chặn CASE và verification

- **Severity:** CRITICAL
- **Affected file/request/case:** Tất cả ba data-run folders; đặc biệt coupon/order setup chains
- **Problem:** Setup requests chỉ có `pm.test(...)`. Assertion setup fail không dừng các request kế tiếp. CASE và verification không kiểm tra `setupReady`. Order setup transitions còn set `beforeOrderStatus` ngay sau khi parse JSON mà không xác minh persisted transition thành công.
- **Why it matters:** Một register/login/create/checkout/transition failure vẫn cho CASE chạy với empty hoặc stale variables. Selected-API assertion sau đó có thể fail vì setup, hoặc thậm chí pass sai trên fixture cũ. Điều này vi phạm yêu cầu tách setup failure khỏi selected-API result.
- **Recommended correction:** Dùng explicit readiness state, ví dụ `userSetupReady`, `couponSetupReady`, `orderSetupReady`, `sourceStateReady`. Post-response setup phải set flag chỉ khi status/schema/ID/state hợp lệ; CASE pre-request phải `skipRequest()` nếu flag false và log một setup-blocked record. Verification cũng phải skip khi CASE không thực sự được gửi. Có thể dùng fail-fast workflow/`--bail`, nhưng không nên chỉ dựa vào Newman option.

### F-05 — `LOGIN-GEN-004` không thực hiện “login lại”

- **Severity:** MEDIUM
- **Affected file/request/case:** `login_data.json`, `LOGIN-GEN-004`; login folder ordering
- **Problem:** Case objective yêu cầu login lại bằng cùng credential sau một login thành công. Iteration hiện chỉ register user rồi gửi một selected `POST /api/login`; register không phải lần login thứ nhất.
- **Why it matters:** Case đang chạy trùng nominal login behavior và không kiểm tra repeated-login/session behavior đã audit.
- **Recommended correction:** Đặt `LOGIN-GEN-004` vào dedicated flow gồm first login setup/observation rồi second login là CASE, hoặc gửi hai selected calls trong cùng scenario và ghi rõ request nào là setup/reference, request nào được tính cho case.

### F-06 — Lockout flow không kiểm tra đúng boundary/rule đã trace

- **Severity:** HIGH
- **Affected file/request/case:** Collection requests `CASE - Data-driven POST /api/login`, `LOCKOUT ... attempt 2`, `attempt 3`, `Correct password...`; `LOGIN-GEN-040`
- **Problem:** Flow chỉ assert cả ba wrong attempts không trả token và correct login sau ba attempts không trả token. Nó không chứng minh account chưa bị khóa sau hai lần sai, không kiểm tra counter tăng đúng một, và không kiểm tra unlock sau 30 giây dù metadata nói dùng FR-02.
- **Why it matters:** Implementation khóa account sớm hơn threshold vẫn có thể làm toàn bộ assertion hiện tại pass. Assertion vì vậy không phát hiện chính lỗi boundary quan trọng trong FR-02. Duration cũng chỉ được mô tả, chưa được automated.
- **Recommended correction:** Tách thành ít nhất hai disposable-user scenarios: (A) hai lần sai rồi correct login phải còn được phép; (B) ba lần sai rồi correct login phải bị từ chối. Nếu kiểm duration, thêm scenario có controlled delay/readiness riêng; nếu chưa muốn chờ, đánh dấu duration `NOT_AUTOMATED` thay vì mô tả như đã implement. Có thể dùng admin discovery để kiểm counter chỉ khi request đó dùng admin hợp lệ và được ghi là verification.

### F-07 — Một số login assertions không thực hiện objective đã audit

- **Severity:** MEDIUM
- **Affected file/request/case:** `LOGIN-GEN-032`, `LOGIN-GEN-033`, `LOGIN-GEN-036`, `LOGIN-EXT-005`; generic login post-response script
- **Problem:** `LOGIN-GEN-032/033` không assert extra `role`/`isAdmin` không nâng quyền; generic script chỉ kiểm secret fields nếu có `user`. Malformed JSON cases `LOGIN-GEN-036` và `LOGIN-EXT-005` không kiểm stack trace, file path, SQL/framework/internal leakage theo expected assertion.
- **Why it matters:** Case có thể pass/không có assertion liên quan trực tiếp ngay cả khi extra fields nâng quyền hoặc malformed request làm lộ internal details.
- **Recommended correction:** Thêm assertion mode riêng: `ROLE_NOT_ELEVATED` so sánh response role/identity với account fixture; `NO_INTERNAL_LEAKAGE` kiểm response text không chứa stack frame, local path, SQL query hoặc framework exception. Không ép exact status cho các historical `SPEC_UNDEFINED` rows.

### F-08 — Một số `SPEC_UNDEFINED`/exploratory coupon cases bị ép phải success

- **Severity:** HIGH
- **Affected file/request/case:** `COUPON-GEN-028`, `COUPON-GEN-029`, `COUPON-GEN-034`, `COUPON-GEN-035`, `COUPON-EXT-004`; `assertion_mode=SUCCESS_SCHEMA`
- **Problem:** Original rows dùng conditional wording như “nếu success” hoặc chỉ yêu cầu robustness/observation. Generic `SUCCESS_SCHEMA` bắt buộc response có numeric `discount_amount` và `final_amount`, tức là quyết định request phải được chấp nhận.
- **Why it matters:** Đây là hard expected behavior mới không được original audit hoặc approved additional source xác định. Một implementation reject an toàn có thể bị báo fail sai.
- **Recommended correction:** Dùng mode `IF_SUCCESS_VALIDATE_SCHEMA`: không assert exact status/success; nếu response thực sự là success thì mới kiểm schema/calculation, còn response reject được ghi observation và vẫn kiểm no leakage/no invalid discount. Chỉ giữ mandatory success cho case có FR-09 conditions rõ ràng và controlled fixture thỏa mọi điều kiện.

### F-09 — Coupon override cases không kiểm server-side rule

- **Severity:** HIGH
- **Affected file/request/case:** `COUPON-GEN-034`, `COUPON-EXT-004`; fixed coupon fixture và coupon generic post-response
- **Problem:** Hai case gửi `discount_value=99` và `type=percent`, nhưng assertion chỉ kiểm `final_amount = total - discount_amount`. Nó không assert discount vẫn là fixed `50000` từ server fixture.
- **Why it matters:** Nếu SUT dùng chính extra fields từ client để tính 99%, invariant hiện tại vẫn đúng và case pass, trong khi objective mass-assignment/override đã bị vi phạm.
- **Recommended correction:** Cho hai case mode riêng. Nếu response success, assert `discount_amount === 50000`, `final_amount === total_amount - 50000`, và rule tương ứng với controlled fixed coupon. Nếu SUT reject extra fields, ghi nhận như một behavior hợp lệ nếu requirement không bắt buộc accept.

### F-10 — Body/path mismatch không verify secondary order

- **Severity:** MEDIUM
- **Affected file/request/case:** `ORDERSTATUS-GEN-032`; `SETUP - Checkout secondary order...`; generic order verification
- **Problem:** Case tạo `otherOrderId` và gửi nó trong body, nhưng verification chỉ đọc `orderUnderTestId`. Không assert secondary order giữ `pending`.
- **Why it matters:** Nếu buggy SUT update cả path order lẫn body order, primary assertion vẫn pass và side effect ngoài target không bị phát hiện.
- **Recommended correction:** Với `ORDERSTATUS-GEN-032`, verification phải assert path order chuyển đúng target và secondary/body order không đổi. Giữ path ID là authority theo audited assertion.

### F-11 — Duplicate `status` case giả định parser luôn dùng last key

- **Severity:** HIGH
- **Affected file/request/case:** `ORDERSTATUS-EXT-005`; `order_status_data.json`
- **Problem:** Data generation parse duplicate JSON bằng Python và đặt `target_status=delivered`, sau đó assert state phải không đổi từ `pending`. Original case là exploratory: phải quan sát parser dùng first hay last key. Nếu parser dùng first key `confirmed`, state `confirmed` là kết quả phù hợp với parser đó.
- **Why it matters:** Assertion hiện tại biến một parser-dependent exploratory case thành hard last-key assumption, trái với original audit intent.
- **Recommended correction:** Đổi sang observation mode. Sau request, chấp nhận/ghi nhận hai outcome có giải thích: `pending` nếu effective value là invalid `delivered`, hoặc `confirmed` nếu effective value là first `confirmed`; không tự chọn parser semantics trước execution. Vẫn fail nếu state chuyển sang giá trị ngoài hai outcome hợp lý hoặc có leakage.

### F-12 — Tampered JWT thay ký tự cuối có thể không đổi payload bytes hiệu dụng

- **Severity:** MEDIUM
- **Affected file/request/case:** `ORDERSTATUS-GEN-009`; order CASE pre-request script
- **Problem:** Script thay đúng ký tự cuối của toàn JWT. Với base64url không padding, một số thay đổi ở ký tự cuối có thể chỉ đổi unused padding bits và decode thành cùng byte sequence/signature.
- **Why it matters:** “Tampered” token có khả năng vẫn tương đương token gốc, làm case không ổn định hoặc không thực sự kiểm signature tampering.
- **Recommended correction:** Split JWT thành ba segment và thay một ký tự ở giữa signature segment (hoặc payload segment) trong khi giữ format ba phần. Assert generated token khác original và vẫn có ba segment; không tự tạo token/claim mới.

### F-13 — Unknown IDs/code chưa được xác minh là absent

- **Severity:** LOW
- **Affected file/request/case:** `unknownUserId`, `unknownOrderId`, `unknownCouponCode`; `COUPON-GEN-003/030`, `ORDERSTATUS-GEN-012`
- **Problem:** Unknown user/order IDs được tính bằng current ID `+1000000`; unknown coupon code dùng unique-looking `runId`, nhưng discovery scripts không assert các giá trị này thật sự absent trước CASE.
- **Why it matters:** Trong một backend session rất dài hoặc dữ liệu collision, negative case có thể vô tình trỏ đến entity thật. Đây không phải hard-coded runtime ID nhưng chưa đạt controlled-absence guarantee đã phê duyệt trong setup analysis.
- **Recommended correction:** Sau discovery, tính từ maximum observed ID và assert không có match; với coupon, assert `unknownCouponCode` không xuất hiện trong `/api/coupons`. Nếu không chứng minh được absence, block CASE thay vì chạy.

## 4. PASS details

### Item 3 — `X-Student-Id`: PASS

Mọi request object có header `X-Student-Id: {{studentId}}`. Collection-level pre-request script upsert cùng header. Không tìm thấy request thiếu header.

### Item 7 — `ORDERSTATUS-GEN-001`: PASS

Data row giữ:

- `original_audit_status = VALID`;
- original positive objective/body/history;
- `implementation_status = CORRECTED_AFTER_HUMAN_REVIEW`.

Implementation dùng source `confirmed`, target `pending`, mode `STATE_UNCHANGED_INVALID_TRANSITION` và trace `EShop-source/README.md FR-10`. Cách này đúng quyết định human-review và không overwrite audit history.

### Item 8 — Coupon Authorization: PASS

Generic coupon CASE và hai selected calls của replay sequence đều có `Authorization: Bearer {{userToken}}`. Token được extract từ disposable-user login. `/api/coupon-usage` cũng dùng cùng user token, còn admin coupon setup dùng admin seed token. Việc bổ sung Authorization được mô tả và trace tới FR-09 C4; `original_headers` không bị sửa.

### Item 10 — `pm.execution.skipRequest()`: PASS

Collection chỉ gọi `pm.execution.skipRequest()` trong pre-request scripts. Tài liệu Postman chính thức xác nhận method này bỏ qua current request, post-response scripts và chuyển sang request tiếp theo trong Collection Runner; cùng behavior áp dụng cho Newman và Postman CLI: [Postman Sandbox — `pm.execution.skipRequest`](https://learning.postman.com/docs/tests-and-scripts/write-scripts/postman-sandbox-reference/pm-execution/#pmexecutionskiprequest).

Không dùng `pm.execution.runRequest()`, nên không gặp giới hạn Newman của API đó. Cần dùng phiên bản Postman/Newman hiện hành hỗ trợ `pm.execution.skipRequest()`; collection hiện không pin tool version, nhưng bản thân API usage là đúng.

### Item 12 — Không fabricate runtime/evidence: PASS

- Admin/user seed credentials trùng source/database đã xác nhận.
- Disposable credentials/code là input để tạo fixture, không được trình bày như entity tồn tại trước.
- Runtime token/ID trong environment để rỗng và được extract/set trong scripts.
- Các UUID metadata của collection/environment không phải SUT entity ID.
- Không có actual status, pass/fail count, token, Newman report hoặc execution evidence.

## 5. Kết luận review

Artifact **chưa nên chuyển sang execution**. Blocker chính là F-04 (setup failure không chặn CASE), cùng F-01/F-02/F-03 khiến request có thể dùng unresolved hoặc stale data. Sau khi sửa các HIGH/CRITICAL findings, cần static-review lại JSON, scripts, placeholder resolution, per-case flow và setup gating trước khi import/run.

Tài liệu dừng tại static review. Không file implementation nào được sửa và không test nào được thực thi.
