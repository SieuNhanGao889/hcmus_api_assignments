# Static review Postman artifacts sau PHASE_6_POSTMAN_FIX

## 1. Phạm vi và giới hạn

Đã review tĩnh các file:

- `postman/HW06_EShop_API_Tests.postman_collection.json`
- `postman/HW06_Local.postman_environment.json`
- `postman/data/login_data.json`
- `postman/data/coupon_data.json`
- `postman/data/order_status_data.json`
- `reports/analysis/postman_implementation_notes.md`

Review này chỉ dùng parse JSON, compile JavaScript bằng `new Function`, đối chiếu placeholder/variable, kiểm tra request tree, data mapping, assertion mode, readiness/skip gate và audit fields. Không khởi động backend, không gửi API request, không chạy Postman/Newman và không tạo execution evidence.

## 2. Kết quả tổng hợp

| Hạng mục | Kết quả | Bằng chứng tĩnh |
|---|---|---|
| JSON parsing | **PASS** | `5/5` Postman JSON artifacts parse thành công |
| JavaScript syntax | **PASS** | `86/86` pre-request/post-response scripts compile; `0` syntax error |
| Executable placeholders | **PASS** | `33` placeholder names trong executable URL/body/path đều có nguồn; `0` unresolved executable placeholder |
| Variable initialization/reset | **PASS** | Cả `3` folder reset token, ID, readiness, CASE state và observation variables ở request đầu iteration |
| Setup readiness gating | **PASS** | `loginSetupReady`, `couponSetupReady`, `orderSetupReady` chỉ được nâng sau setup validation; CASE có guard và `setupBlockedReason` |
| CASE/verification skip | **PASS** | `5/5` selected CASE/sequence pre-scripts có readiness gate; `5/5` verification/continuation pre-scripts được kiểm tra có skip gate theo readiness/`caseExecuted` |
| Mapping | **PASS** | `166` data rows, `137` unique `case_id`, `166` unique `scenario_id`, không duplicate `scenario_id` |
| `X-Student-Id` | **PASS** | `43/43` request objects có `X-Student-Id: {{studentId}}` |
| Assertion modes | **PASS** | Các mode mới có implementation tương ứng; `SPEC_UNDEFINED` không bị thêm exact status assertion |
| Historical audit | **PASS** | Không row nào thiếu nhóm `original_*`; hai scenario `LOGIN-GEN-040` có original history giống nhau; `ORDERSTATUS-GEN-001` vẫn giữ original `VALID` và correction riêng |

Các placeholder chỉ xuất hiện trong `original_body`/narrative history như `{{targetStatus}}` hoặc `{{amountAroundMinOrder}}` không được tính là executable placeholder và không bị sửa, nhằm giữ nguyên lịch sử audit.

## 3. Validation theo approved findings

### F-01 — PASS

`validCouponCodeLowercase` được tạo theo iteration bằng `validCouponCode.toLowerCase()`. Login/coupon/order CASE đều có unresolved-placeholder guard trước khi request được gửi.

### F-02 — PASS

`otherUserId` không còn lấy stale `adminUserId` ở đầu iteration. Sau khi admin identity và disposable user identity đều có numeric ID, setup mới set `otherUserId`; assertion yêu cầu ID tồn tại, numeric và khác `validUserId`. `COUPON-GEN-031` có guard lặp lại ngay trước CASE.

### F-03 — PASS

Request đầu của từng data-run folder unset runtime token, ID, readiness flag và observation variable thuộc folder, sau đó set readiness/`caseExecuted` về `false`. Không còn dựa vào giá trị runtime của iteration trước.

### F-04 — PASS

Setup assertions dùng prefix `SETUP` và ghi `setupBlockedReason`. Final setup readiness được xác nhận độc lập trước selected CASE. CASE bị `skipRequest()` và return khi readiness false; verification/continuation bị skip nếu CASE chưa chạy hoặc prerequisite step chưa ready. Vì vậy setup failure được phân biệt với selected-API assertion failure.

### F-05 — PASS

`LOGIN-GEN-004` dùng `REPEATED_LOGIN_SEQUENCE`: request setup thực hiện first successful login và chỉ khi `firstLoginReady=true` thì generic CASE mới thực hiện selected second login bằng cùng credential.

### F-06 — PASS

`LOGIN-GEN-040` giữ cùng `case_id` và được tách thành:

- `LOGIN-GEN-040-A`: wrong attempt 1 + wrong attempt 2 + correct login phải success, chứng minh hai lần sai chưa lock;
- `LOGIN-GEN-040-B`: wrong attempt 1 + attempt 2 + attempt 3 + correct login không có token, có error message và không lộ internal details.

`original_expected_status` vẫn là `SPEC_UNDEFINED`. Kiểm tra unlock sau 30 giây được ghi rõ `NOT_AUTOMATED`; không có assertion hoặc claim duration coverage.

### F-07 — PASS

`LOGIN-GEN-032/033` dùng `ROLE_NOT_ELEVATED`; nếu response success thì role/isAdmin không được nâng và returned user ID phải khớp fixture. `LOGIN-GEN-036` và `LOGIN-EXT-005` dùng `NO_TOKEN_AND_NO_INTERNAL_LEAKAGE`. Không thêm exact status cho các row `SPEC_UNDEFINED`.

### F-08 — PASS

`COUPON-GEN-028/029/035` dùng `IF_SUCCESS_VALIDATE_SCHEMA`. Chỉ response có success schema mới bị validate calculation; safe rejection được log như exploratory observation và không tự bị coi là failure.

### F-09 — PASS

`COUPON-GEN-034` và `COUPON-EXT-004` dùng `IF_SUCCESS_FIXED_RULE`. Nếu accepted, `discount_amount` phải đúng fixed rule `50000` và `final_amount = total_amount - 50000`; nếu safely rejected, outcome được chấp nhận vì requirement không bắt buộc extra fields phải được accept.

### F-10 — PASS

`ORDERSTATUS-GEN-032` dùng `PATH_TARGET_ONLY`: verification assert path-target order đạt target và secondary/body-referenced order vẫn `pending`.

### F-11 — PASS

`ORDERSTATUS-EXT-005` dùng `DUPLICATE_STATUS_OBSERVATION`, không chọn trước first-key/last-key semantics. Persisted path order chỉ được là `confirmed` hoặc `pending`; snapshot setup được dùng để fail nếu order không liên quan bị mutate. Internal leakage check vẫn áp dụng.

### F-12 — PASS

JWT được split thành đúng ba segment và thay một ký tự ở giữa signature segment. Pre-request guard xác nhận token tampered khác token gốc và vẫn có đúng ba segment.

### F-13 — PASS

Unknown user/order IDs được derive từ `max(observed IDs) + 1` và assert absent trong discovery response. Unknown coupon code được assert không xuất hiện trong `/api/coupons`. Final readiness không thể thành `true` nếu absence không được thiết lập, nên CASE tương ứng bị block.

## 4. Preservation checks

- `original_*` audit/history fields không bị thay bằng implementation decisions.
- `case_id` traceability giữ nguyên `137` unique cases; scenario expansion chỉ thêm `LOGIN-GEN-040-A/B` dưới cùng `case_id`.
- `ORDERSTATUS-GEN-001` vẫn có `original_audit_status = VALID`, `implementation_status = CORRECTED_AFTER_HUMAN_REVIEW`, mode `STATE_UNCHANGED_INVALID_TRANSITION`.
- Coupon selected requests tiếp tục có Authorization theo correction đã duyệt; `original_headers` được giữ.
- Inactive coupon vẫn là `BLOCKED_NOT_AUTOMATED` trong folder deferred.
- Mọi request object tiếp tục có `X-Student-Id: {{studentId}}`.

## 5. Kết luận

Static review sau fix: **PASS — READY_FOR_FINAL_HUMAN_APPROVAL**.

Kết luận này chỉ xác nhận tính nhất quán tĩnh của Postman artifacts và việc áp dụng F-01 đến F-13. Nó không phải execution result, không xác nhận SUT pass/fail và không tạo bug evidence. Dừng tại checkpoint này theo yêu cầu; chưa chuyển sang PHASE_7_EXECUTION.
