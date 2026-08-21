# Phân tích kết quả Newman execution

## 1. Phạm vi và nguyên tắc

Phân tích này chỉ đọc ba execution report thực tế:

- `reports/newman/login_run.json`
- `reports/newman/coupon_run.json`
- `reports/newman/order_status_run.json`

Không sửa collection, environment, data files, SUT hoặc Newman reports. Không chạy lại API/Postman/Newman. Các mục `POTENTIAL_SUT_BUG` chỉ là ứng viên cần human review và reproduction; không mục nào được tự động coi là bug đã xác nhận.

## 2. Execution summary

| API | Iterations | Requests | Assertions | Passed assertions | Failed assertions | Unique failed scenarios | Scenario bị setup-blocked |
|---|---:|---:|---:|---:|---:|---:|---:|
| `POST /api/login` | 46 | 101 | 153 | 142 | 11 | 11 | 0 identifiable |
| `POST /api/apply-coupon` | 47 | 755 | 837 | 833 | 4 | 4 | 0 identifiable |
| `PUT /api/admin/orders/:id/status` | 73 | 638 | 786 | 771 | 15 | 14 | 0 identifiable |
| **Tổng** | **166** | **1,494** | **1,776** | **1,746** | **30** | **29** | **0 identifiable** |

`pending=0` và request-level `failed=0` trong cả ba report. Không có failure nào có assertion name prefix `SETUP`. Đối chiếu executions theo iteration cho thấy cả `166/166` data rows đều có selected CASE/sequence request; vì vậy không xác định được scenario nào bị setup gate block. Các request bị skip do branch/flow không áp dụng cho iteration không được tính là blocked scenario.

## 3. `POST /api/login`

### 3.1 Failed scenarios và observed behavior

| Iteration | `case_id` / `scenario_id` | Failed assertion | Actual observed behavior | Classification |
|---:|---|---|---|---|
| 0 | `LOGIN-GEN-001` | `[LOGIN-GEN-001] no sensitive user fields` | `200`; login schema/token hợp lệ, nhưng `user` chứa plaintext `password`, cùng các account fields như `reset_token` | `POTENTIAL_SUT_BUG` |
| 1 | `LOGIN-GEN-002` | `[LOGIN-GEN-002] no sensitive user fields` | `200`; cùng behavior trả plaintext `user.password` | `POTENTIAL_SUT_BUG` |
| 2 | `LOGIN-GEN-003` | `[LOGIN-GEN-003] no sensitive user fields` | `200`; cùng behavior trả plaintext `user.password` | `POTENTIAL_SUT_BUG` |
| 3 | `LOGIN-GEN-004` | `[LOGIN-GEN-004] no sensitive user fields` | First login setup và selected second login đều thành công; second response vẫn trả plaintext `user.password` | `POTENTIAL_SUT_BUG` |
| 31 | `LOGIN-GEN-032` | `[LOGIN-GEN-032] no sensitive user fields` | `200`; extra `role=admin` không nâng quyền (`role` vẫn `user`), nhưng response trả plaintext password | `POTENTIAL_SUT_BUG` |
| 32 | `LOGIN-GEN-033` | `[LOGIN-GEN-033] no sensitive user fields` | `200`; extra `isAdmin=true` không nâng quyền, nhưng response trả plaintext password | `POTENTIAL_SUT_BUG` |
| 35 | `LOGIN-GEN-036` | `[LOGIN-GEN-036] no internal details leak` | Malformed JSON trả `400` HTML chứa `SyntaxError`, stack frames, absolute Windows workspace paths và `node_modules/body-parser` paths; no-token assertion pass | `POTENTIAL_SUT_BUG` |
| 37 | `LOGIN-GEN-038` | `[LOGIN-GEN-038] no sensitive user fields` | Request thiếu `Content-Type` vẫn trả `200` login success và plaintext password trong `user` | `POTENTIAL_SUT_BUG` |
| 39 | `LOGIN-GEN-040` / `LOGIN-GEN-040-A` | `[LOGIN-GEN-040-A][FR-02] correct login succeeds after two failures` | Hai wrong attempts đều `401`; correct password ngay sau đó trả `403` với “Tài khoản đã bị khóa...” thay vì login success | `POTENTIAL_SUT_BUG` |
| 43 | `LOGIN-EXT-003` | `[LOGIN-EXT-003] no sensitive user fields` | Duplicate keys được parser xử lý theo effective last values và login trả `200`; response vẫn chứa plaintext password | `POTENTIAL_SUT_BUG` |
| 45 | `LOGIN-EXT-005` | `[LOGIN-EXT-005] no internal details leak` | Malformed JSON trả `400` HTML cùng stack trace/absolute path leakage; no-token assertion pass | `POTENTIAL_SUT_BUG` |

### 3.2 Root-cause grouping

1. **Login success serialization trả sensitive account fields**: `LOGIN-GEN-001/002/003/004/032/033/038`, `LOGIN-EXT-003` — 8 failed assertions, một behavior chung.
2. **Unhandled JSON parser error làm lộ internal details**: `LOGIN-GEN-036`, `LOGIN-EXT-005` — cùng behavior với malformed-input failures ở order-status và được nhóm cross-API tại mục potential bug PB-02.
3. **Lockout boundary xảy ra sau hai lần sai**: `LOGIN-GEN-040-A` — scenario B vẫn pass vì account đã locked ở attempt 3, nhưng A chứng minh boundary bị kích hoạt sớm hơn requirement.

Không có `SETUP_FAILURE`, `AUTOMATION_FAILURE`, `EXPECTED_EXPLORATORY_BEHAVIOR` hoặc `NEEDS_HUMAN_REVIEW` trong 11 login failures theo evidence hiện có.

## 4. `POST /api/apply-coupon`

### 4.1 Failed scenarios và observed behavior

| Iteration | `case_id` / `scenario_id` | Failed assertion | Actual observed behavior | Classification |
|---:|---|---|---|---|
| 5 | `COUPON-GEN-006` | `[COUPON-GEN-006] discount success schema and invariant` | Với `total_amount=200000` bằng `min_order_amount=200000`, SUT trả `400`: “Đơn hàng chưa đủ giá trị tối thiểu 200,000 ₫...” | `POTENTIAL_SUT_BUG` |
| 24 | `COUPON-GEN-025` | `[COUPON-GEN-025] no valid discount is returned` | `user_id={"$gt":0}` được accept; response `200`, `success=true`, `discount_amount=50000`, `final_amount=450000` | `POTENTIAL_SUT_BUG` |
| 40 | `COUPON-EXT-001` | `[COUPON-EXT-001][FR-09] percent formula` | Percent coupon `10%` trên `500000` trả `discount_amount=-4500000`, `final_amount=5000000`; schema invariant nội bộ vẫn pass nhưng công thức FR-09 fail | `POTENTIAL_SUT_BUG` |
| 42 | `COUPON-EXT-002` / `COUPON-EXT-002-EQUAL` | `[COUPON-EXT-002-EQUAL] discount success schema and invariant` | Cùng equality boundary `200000 == min_order_amount`; SUT trả `400` thay vì áp dụng coupon | `POTENTIAL_SUT_BUG` |

### 4.2 Root-cause grouping

1. **Minimum-order equality bị reject**: `COUPON-GEN-006` và `COUPON-EXT-002-EQUAL` là hai assertions của cùng boundary/root cause, không phải hai bug riêng.
2. **Thiếu type validation cho `user_id`**: `COUPON-GEN-025` nhận object operator nhưng vẫn cấp discount.
3. **Sai công thức percent coupon**: `COUPON-EXT-001` tạo negative discount và final amount lớn hơn total.

Không có setup failure hoặc blocked scenario. Không failure nào được phân loại là `EXPECTED_EXPLORATORY_BEHAVIOR`: equality và percent formula có source FR-09 rõ; object `user_id` đã tạo successful discount thay vì chỉ tạo một safe rejection/observation.

## 5. `PUT /api/admin/orders/:id/status`

### 5.1 Failed scenarios và observed behavior

| Iteration | `case_id` / `scenario_id` | Failed assertion | Actual observed behavior | Classification |
|---:|---|---|---|---|
| 9 | `ORDERSTATUS-GEN-010` | `[ORDERSTATUS-GEN-010] persisted state is unchanged` | Normal-user token nhận `200`, “Order status updated”; persisted state `pending -> confirmed` | `POTENTIAL_SUT_BUG` |
| 10 | `ORDERSTATUS-GEN-011` | `[ORDERSTATUS-GEN-011] persisted state is unchanged` | Normal user cập nhật chính order của mình qua admin endpoint; `200`, persisted `confirmed` | `POTENTIAL_SUT_BUG` |
| 11 | `ORDERSTATUS-GEN-012` | `[ORDERSTATUS-GEN-012] persisted state changed per FR-10` | Controlled absent ID trả `404 Order not found`; fixture path order vẫn `pending`. Failure do mode đòi fixture đổi sang `confirmed`, trái objective unknown-ID | `AUTOMATION_FAILURE` |
| 12 | `ORDERSTATUS-GEN-013` | `[ORDERSTATUS-GEN-013] persisted state changed per FR-10` | ID `0` trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 13 | `ORDERSTATUS-GEN-014` | `[ORDERSTATUS-GEN-014] persisted state changed per FR-10` | ID `-1` trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 14 | `ORDERSTATUS-GEN-015` | `[ORDERSTATUS-GEN-015] persisted state changed per FR-10` | ID `1.5` trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 15 | `ORDERSTATUS-GEN-016` | `[ORDERSTATUS-GEN-016] persisted state changed per FR-10` | ID `abc` trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 16 | `ORDERSTATUS-GEN-017` | `[ORDERSTATUS-GEN-017] persisted state changed per FR-10` | Very-large ID trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 17 | `ORDERSTATUS-GEN-018` | `[ORDERSTATUS-GEN-018] persisted state changed per FR-10` | Injection-style ID trả `404`; controlled order không đổi | `AUTOMATION_FAILURE` |
| 30 | `ORDERSTATUS-GEN-031` | `[ORDERSTATUS-GEN-031] persisted state is unchanged` | Normal-user token kèm body `role/isAdmin` nhận `200`; persisted `pending -> confirmed` | `POTENTIAL_SUT_BUG` |
| 32 | `ORDERSTATUS-GEN-033` | `[ORDERSTATUS-GEN-033] no internal details leakage` | Malformed JSON trả `400` HTML chứa parser `SyntaxError`, stack trace và absolute paths; order state vẫn unchanged | `POTENTIAL_SUT_BUG` |
| 34 | `ORDERSTATUS-GEN-035` | `[ORDERSTATUS-GEN-035] no internal details leakage` | `text/plain` body làm SUT trả `500` HTML, `TypeError`, `server.js:526`, framework paths | `POTENTIAL_SUT_BUG` |
| 34 | `ORDERSTATUS-GEN-035` | `[ORDERSTATUS-GEN-035] persisted state changed per FR-10` | Order vẫn `pending`; historical objective là exploratory wrong Content-Type, nên hard expectation chuyển `confirmed` không được justification | `AUTOMATION_FAILURE` |
| 42 | `ORDERSTATUS-EXT-001` | `[ORDERSTATUS-EXT-001] persisted state is unchanged` | Normal-user token nhận `200`; persisted `pending -> confirmed` | `POTENTIAL_SUT_BUG` |
| 67 | `ORDERSTATUS-EXT-003` / `ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED` | `[ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED] persisted state is unchanged` | CASE trả `200`; persisted state chuyển `canceled -> delivered` | `POTENTIAL_SUT_BUG` |

### 5.2 Root-cause grouping

1. **Admin role không được enforce**: `ORDERSTATUS-GEN-010/011/031`, `ORDERSTATUS-EXT-001` — bốn scenarios, một authorization root cause.
2. **Sai assertion mode cho invalid/unknown path IDs**: `ORDERSTATUS-GEN-012..018` — bảy automation failures; actual safe `404`/unchanged behavior không phải evidence của SUT bug.
3. **Wrong Content-Type bị hard-assert phải update**: state assertion của `ORDERSTATUS-GEN-035` là automation failure riêng; cùng execution vẫn cung cấp potential-bug evidence về `500`/internal leakage.
4. **Unhandled input errors làm lộ internal details**: `ORDERSTATUS-GEN-033/035`, cùng root-cause family với login malformed JSON.
5. **Terminal-state transition được chấp nhận**: `ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED` vi phạm final-state rule của FR-10.

## 6. Classification summary

| Classification | Failed assertions | Root-cause groups | Ghi chú |
|---|---:|---:|---|
| `SETUP_FAILURE` | 0 | 0 | Không có failed setup assertion; không có selected scenario bị block nhận diện được |
| `AUTOMATION_FAILURE` | 8 | 2 | `ORDERSTATUS-GEN-012..018` và state assertion của `ORDERSTATUS-GEN-035` |
| `EXPECTED_EXPLORATORY_BEHAVIOR` | 0 | 0 | Không có failed assertion nào chỉ phản ánh một outcome exploratory hợp lệ sau khi xem actual behavior |
| `POTENTIAL_SUT_BUG` | 22 | 8 | Chỉ là ứng viên; cần human review/reproduction |
| `NEEDS_HUMAN_REVIEW` | 0 | 0 | Các failure hiện có đủ evidence để xếp tạm vào automation hoặc potential bug; quyết định confirm bug vẫn thuộc human review |

`30` failed assertions không tương đương `30` bugs. Sau grouping, có `8` potential SUT root causes và `2` automation root causes.

## 7. Potential SUT bugs

### PB-01 — Login response exposes plaintext password and sensitive account fields

- **Affected cases:** `LOGIN-GEN-001/002/003/004/032/033/038`, `LOGIN-EXT-003`.
- **Requirement/source:** `docs/api_specification.md` Authentication 1.2 trả `token` và thông tin `user`; security expectation đã audit trong login test design là không trả password/reset secret.
- **Expected:** Success response có token/user cần thiết nhưng không chứa plaintext password, password hash, reset token hoặc secret tương tự.
- **Actual:** Tám success responses có `user.password` bằng chính credential vừa gửi; response còn expose fields như `reset_token`, `login_attempts`, `locked_until`.
- **Newman evidence:** `login_run.json`, iterations `0,1,2,3,31,32,37,43`; đều fail assertion `no sensitive user fields` với message `expected ... to not have property 'password'`. Status của CASE là `200`.
- **Proposed bug title:** `POST /api/login exposes plaintext password and sensitive account fields in success response`.
- **Confidence:** **High** về behavior vì lặp lại trên 8 executions; **medium-high** về requirement vì field-level response schema không liệt kê chi tiết nhưng plaintext credential exposure là rủi ro security rõ ràng. Cần human xác nhận severity và bug policy.

### PB-02 — Error handling exposes stack traces, absolute paths and framework internals

- **Affected cases:** `LOGIN-GEN-036`, `LOGIN-EXT-005`, `ORDERSTATUS-GEN-033`, `ORDERSTATUS-GEN-035`.
- **Requirement/source:** Audited no-internal-leakage assertions; assignment security coverage; endpoint specs không yêu cầu expose implementation details.
- **Expected:** Malformed/unsupported bodies bị reject an toàn, không trả stack trace, local path, framework/module details hoặc uncaught exception output.
- **Actual:** Login malformed JSON và order malformed JSON trả HTML `SyntaxError` pages với absolute Windows paths and `body-parser`/`raw-body` stack frames. Order wrong `Content-Type` trả `500` HTML với `TypeError`, `server.js:526` và framework paths.
- **Newman evidence:** `login_run.json` iterations `35,45`, status `400`; `order_status_run.json` iteration `32`, status `400`, và iteration `34`, status `500`; bốn `no internal details leak/leakage` assertions fail.
- **Proposed bug title:** `Unhandled request-body errors disclose server stack traces and absolute filesystem paths`.
- **Confidence:** **High** — response bodies trong reports chứa trực tiếp exception type, source line và local filesystem/module paths. Có thể là một shared Express error-handler root cause, nhưng cần reproduction để xác nhận phạm vi.

### PB-03 — Account locks after two failed login attempts instead of three

- **Affected cases:** `LOGIN-GEN-040` / `LOGIN-GEN-040-A`.
- **Requirement/source:** `EShop-source/README.md FR-02`: account chỉ tạm khóa khi có từ 3 consecutive failures trở lên.
- **Expected:** Sau hai wrong attempts, correct password vẫn login thành công; lần sai thứ ba mới tạo locked behavior.
- **Actual:** Hai wrong attempts trả `401`; correct password ngay sau attempt 2 trả `403` và “Tài khoản đã bị khóa...”. Scenario B cho thấy attempt 3 và correct-after-3 đều `403`, nhưng không loại bỏ evidence lock sớm từ scenario A.
- **Newman evidence:** `login_run.json`, iteration `39`: failed assertion `[LOGIN-GEN-040-A][FR-02] correct login succeeds after two failures`; actual token undefined. Iteration `40` lock-after-three assertions pass.
- **Proposed bug title:** `Login account lockout is triggered after two failed attempts instead of three`.
- **Confidence:** **High** — controlled disposable account và ordered three-request evidence trực tiếp khớp boundary FR-02. Unlock duration 30 giây không được test và không thuộc claim này.

### PB-04 — Coupon minimum-order equality is rejected

- **Affected cases:** `COUPON-GEN-006`, `COUPON-EXT-002` / `COUPON-EXT-002-EQUAL`.
- **Requirement/source:** `EShop-source/README.md FR-09 C3`: `total_amount >= min_order_amount` phải đủ điều kiện.
- **Expected:** Controlled coupon có `min_order_amount=200000` phải được áp dụng khi `total_amount=200000`, với success discount schema.
- **Actual:** Cả hai executions trả `400` và message nói đơn hàng chưa đủ minimum `200,000 ₫` dù total bằng đúng threshold.
- **Newman evidence:** `coupon_run.json`, iterations `5` và `42`; failed `discount success schema and invariant` vì response không có numeric discount fields.
- **Proposed bug title:** `Coupon rejects orders whose total equals min_order_amount`.
- **Confidence:** **High** — hai independent controlled fixtures reproduce cùng equality boundary và source dùng toán tử `>=` rõ ràng.

### PB-05 — Coupon accepts object-valued `user_id` and returns a valid discount

- **Affected cases:** `COUPON-GEN-025`.
- **Requirement/source:** `docs/api_specification.md` Coupons 5.1 documents scalar `user_id` example; audited security assertion requires object-operator input not manipulate usage/cross-user logic.
- **Expected:** Object-valued `user_id` không được tạo valid coupon application/usage result; safe rejection là acceptable.
- **Actual:** Body có `user_id={"$gt":0}` nhận `200`, `success=true`, fixed discount `50000`, final amount `450000`.
- **Newman evidence:** `coupon_run.json`, iteration `24`; assertion `[COUPON-GEN-025] no valid discount is returned` fails.
- **Proposed bug title:** `Apply-coupon accepts object-valued user_id and grants discount`.
- **Confidence:** **Medium** — actual acceptance rõ ràng, nhưng specification không ghi type table hoặc exact rejection behavior. Human review cần xác nhận intended validation and security impact trước khi tạo bug.

### PB-06 — Percent coupon formula produces a negative discount

- **Affected cases:** `COUPON-EXT-001`.
- **Requirement/source:** `EShop-source/README.md FR-09`: percent formula `discount_amount = total × discount_value / 100`; `final_amount = total - discount_amount`.
- **Expected:** `10%` của `500000` là `discount_amount=50000`, `final_amount=450000`.
- **Actual:** SUT trả `200`, `success=true`, `discount_amount=-4500000`, `final_amount=5000000`, message vẫn nói giảm `10%`.
- **Newman evidence:** `coupon_run.json`, iteration `40`; failed assertion expected `50000` but observed `-4500000`. Generic internal arithmetic invariant pass vì `500000 - (-4500000) = 5000000`, nhưng business formula fail.
- **Proposed bug title:** `Percent coupon calculation returns negative discount and inflates final amount`.
- **Confidence:** **High** — controlled percent fixture, exact inputs/outputs và FR-09 formula đều có trong evidence.

### PB-07 — Normal-user JWT can update order status through admin endpoint

- **Affected cases:** `ORDERSTATUS-GEN-010/011/031`, `ORDERSTATUS-EXT-001`.
- **Requirement/source:** `EShop-source/README.md FR-12`, `SEC-02`, `SEC-03`: `/api/admin/*` phải yêu cầu valid JWT và `role='admin'`.
- **Expected:** Normal-user token bị từ chối và path order giữ `pending`, bất kể order ownership hoặc extra body fields.
- **Actual:** Cả bốn CASE nhận `200`, “Order status updated”; read-after verification thấy path order chuyển `pending -> confirmed`.
- **Newman evidence:** `order_status_run.json`, iterations `9,10,30,42`; bốn failed assertions `persisted state is unchanged` với actual `confirmed`.
- **Proposed bug title:** `Admin order-status endpoint allows updates with normal-user JWT`.
- **Confidence:** **High** — explicit access-control requirement và persisted state evidence được reproduce trên bốn variants.

### PB-08 — Canceled order can transition to delivered

- **Affected cases:** `ORDERSTATUS-EXT-003` / `ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED`.
- **Requirement/source:** `EShop-source/README.md FR-10`: `canceled` và `delivered` là terminal states, không được chuyển sang trạng thái khác.
- **Expected:** `canceled -> delivered` bị reject; persisted order giữ `canceled`.
- **Actual:** CASE trả `200`, “Order status updated”; read-after verification thấy persisted status `delivered`.
- **Newman evidence:** `order_status_run.json`, iteration `67`; failed assertion expected `canceled` but observed `delivered`.
- **Proposed bug title:** `Order state machine allows terminal canceled order to transition to delivered`.
- **Confidence:** **High** — controlled source state, explicit terminal-state rule và persisted-state evidence đều có trong report.

## 8. Automation failures requiring test correction before bug analysis

### AF-01 — Invalid order IDs incorrectly expect the controlled order to change

- **Cases:** `ORDERSTATUS-GEN-012..018`.
- **Report behavior:** Mỗi CASE trả `404 {"error":"Order not found"}`; controlled order remains `pending`; no leakage assertion passes.
- **Why automation failure:** Objectives/original assertions yêu cầu unknown/invalid ID không update order, nhưng `assertion_mode=STATE_CHANGED_TO_TARGET` làm verification đòi unrelated controlled order thành `confirmed`.
- **Bug impact:** Bảy failed assertions này không phải seven SUT bugs và không cung cấp evidence SUT xử lý invalid IDs sai.

### AF-02 — Exploratory wrong-Content-Type case is forced to expect state change

- **Case:** `ORDERSTATUS-GEN-035`, assertion `persisted state changed per FR-10`.
- **Report behavior:** CASE trả `500`; order stays `pending`.
- **Why automation failure:** Historical objective nói quan sát parser behavior và không tự gán expected; hard state-change expectation không được requirement support.
- **Bug impact:** State assertion failure không phải bug evidence. Tuy nhiên response `500` kèm stack/path leakage vẫn là independent evidence trong PB-02.

## 9. Kết luận cho human review

- Có `30` failed assertions: `22` tạm phân loại `POTENTIAL_SUT_BUG`, `8` là `AUTOMATION_FAILURE`.
- Sau khi group theo behavior/root cause: `8` potential SUT bugs, không phải `22` bugs.
- Không có setup failure hoặc scenario-level block nhận diện được trong reports.
- Chưa có bug nào được xác nhận. Human cần review requirement mapping, ưu tiên reproduction PB-02/PB-03/PB-04/PB-06/PB-07/PB-08 và quyết định có tạo bug record hay không.

Phân tích dừng tại checkpoint này. Không artifact execution hoặc implementation nào được sửa.
