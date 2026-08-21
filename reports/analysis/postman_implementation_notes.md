# Ghi chú triển khai Postman — PHASE_6

## 1. Trạng thái

Đã tạo collection, local environment và ba data file cho 137 test case gốc. Chưa khởi động backend, chưa gửi API request và chưa chạy Postman/Newman. Các artifact không chứa execution result, token hay runtime ID thật.

Collection dùng mô hình **một data row = một iteration cô lập**. Mỗi folder tự tạo prerequisite bằng request có prefix `SETUP`, chạy request selected API có prefix `CASE`/`SEQUENCE`, rồi verification/cleanup. Setup request không được tính là execution của test case selected API.

## 2. Traceability và audit history

Mỗi data row giữ nguyên các field lịch sử:

- `original_audit_status`
- `original_audit_notes`
- `original_human_correction`
- `original_expected_status`
- `original_expected_assertion`
- `original_assumption_or_open_question`
- `original_headers` và `original_body`

Các quyết định implementation nằm ở field riêng như `implementation_status`, `assertion_mode`, `assertion_requirement_source`, `request_body` và `automation_note`. Vì vậy assertion mới từ `EShop-source/README.md` không âm thầm ghi đè `SPEC_UNDEFINED` cũ.

`ORDERSTATUS-GEN-001` giữ nguyên lịch sử `VALID`/positive trong các field `original_*`, nhưng được đánh dấu `CORRECTED_AFTER_HUMAN_REVIEW`. Scenario thực thi là `confirmed -> pending`, với expected persisted state không đổi theo `EShop-source/README.md FR-10` vì không có transition hợp lệ nào có target `pending`.

`COUPON-EXT-002` được mở rộng thành ba scenario dưới/bằng/trên ngưỡng. `ORDERSTATUS-EXT-003` được mở rộng thành ma trận 25 cặp source/target; `ORDERSTATUS-EXT-004` thành ba source-state scenario. Tất cả vẫn dùng original `case_id` và thêm `scenario_id`.

## 3. Variable strategy

- Environment: `baseUrl`, `studentId`, credential seed được source xác nhận, token/ID runtime.
- Collection: domain status và dataset identifiers ổn định.
- Data files: test-case metadata, original audit fields, body/content type, flow, source state, assertion mode.
- Dynamic extraction: user/coupon/order IDs và token được lấy từ response setup. Unknown user/order IDs được suy ra từ `max(observed IDs) + 1` sau request discovery và được assert không có trong tập quan sát; unknown coupon code được assert không có trong `/api/coupons`. Nếu không chứng minh được absence, CASE bị block.
- Disposable email/password/coupon code được sinh trong pre-request script và chỉ trở thành credential/fixture thật sau response setup thành công.
- Đầu mỗi iteration reset toàn bộ token, ID, readiness flag và observation variable của folder. `validCouponCodeLowercase` được tạo động từ `validCouponCode.toLowerCase()`. CASE có guard unresolved placeholder trước khi gửi.
- Các cờ `loginSetupReady`, `couponSetupReady`, `orderSetupReady` chỉ được set `true` sau khi setup cần thiết được kiểm tra. CASE và verification kiểm tra readiness/`caseExecuted`; setup failure được ghi bằng assertion có prefix `SETUP` và `setupBlockedReason`, không bị trình bày như selected-API failure.

Collection-level pre-request script upsert `X-Student-Id` và mọi request object cũng khai báo rõ `X-Student-Id: {{studentId}}`. Script chỉ log student ID, không log password/token.

## 4. Data-driven execution design (chưa chạy)

Mỗi folder phải được chọn riêng trong Collection Runner/Newman với đúng data file:

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json   -e postman/HW06_Local.postman_environment.json   --folder "01 - POST Login - Data Run"   -d postman/data/login_data.json

newman run postman/HW06_EShop_API_Tests.postman_collection.json   -e postman/HW06_Local.postman_environment.json   --folder "02 - POST Apply Coupon - Data Run"   -d postman/data/coupon_data.json

newman run postman/HW06_EShop_API_Tests.postman_collection.json   -e postman/HW06_Local.postman_environment.json   --folder "03 - PUT Admin Order Status - Data Run"   -d postman/data/order_status_data.json
```

Các lệnh trên chỉ là kế hoạch; chưa được thực thi. Backend phải được giữ trong một session trong suốt setup + execution vì startup load `database.js` và reset database.

## 5. Assertion policy

- Nếu workbook có expected status số, collection assert status đó.
- Nếu historical value là `SPEC_UNDEFINED`, collection không tự gán exact HTTP status.
- Invariant như “không có token”, “không có valid discount”, schema success, identity consistency và persisted-state check được dùng khi traceable.
- Rule lockout, coupon C1-C5/formula và transition matrix có `assertion_requirement_source` trỏ tới `EShop-source/README.md` theo phê duyệt human review.
- Setup failure được report là setup failure; không tự phân loại thành SUT bug.

## 6. Explicit setup flows

### Login

Mỗi iteration tạo disposable normal user. `LOGIN-GEN-004` có first successful login là setup/reference rồi CASE thực hiện second login cùng credential. `LOGIN-GEN-040` được tách thành scenario A (hai lần sai rồi correct login phải thành công) và B (ba lần sai rồi correct login không được có token). Kiểm tra unlock sau 30 giây giữ `NOT_AUTOMATED`; collection không claim duration coverage. `LOGIN-EXT-001/002` có login + `/api/users/me`; `LOGIN-EXT-004` ghi paired observation vào console mà không tự kết luận account-enumeration bug.

`LOGIN-GEN-032/033` dùng mode `ROLE_NOT_ELEVATED`; nếu request success thì user trả về không được thành admin và identity phải khớp disposable account. `LOGIN-GEN-036` và `LOGIN-EXT-005` dùng `NO_TOKEN_AND_NO_INTERNAL_LEAKAGE`. Các row `SPEC_UNDEFINED` vẫn không có exact status assertion.

### Coupon

Mỗi iteration login admin seed, tạo normal user và năm controlled coupons. `COUPON-GEN-009` ghi prior usage bằng `/api/coupon-usage`. `COUPON-EXT-003` thực hiện apply lần đầu, request setup ghi usage, rồi replay apply. `/api/coupon-usage` được ghi rõ là setup và không tính là selected API execution.

Authorization user được thêm cho apply-coupon theo `EShop-source/README.md FR-09 C4`; original workbook headers vẫn được giữ trong data file.

Các case exploratory `COUPON-GEN-028/029/035` dùng `IF_SUCCESS_VALIDATE_SCHEMA`: safe rejection được ghi observation, chỉ response success mới validate schema/calculation. `COUPON-GEN-034` và `COUPON-EXT-004` dùng `IF_SUCCESS_FIXED_RULE`; nếu success phải giữ fixed discount `50000`, còn safe rejection là outcome chấp nhận được vì requirement không bắt buộc accept extra fields.

### Order status

Mỗi scenario tạo order `pending` mới và secondary order. Các setup transition dựng `source_state` bằng admin seed token. Request CASE chọn auth profile và path/body từ data row; verification đọc `/api/admin/orders` bằng admin token để kiểm tra persisted state.

Normal-user/tampered/missing-token scenario dùng dedicated order vì implementation lỗi có thể mutate state. Không dùng lỗ hổng thiếu role check làm setup.

`ORDERSTATUS-GEN-032` verify cả path-target order chuyển sang target và secondary/body-referenced order vẫn `pending`. `ORDERSTATUS-EXT-005` là observation mode, chỉ chấp nhận `pending` (effective last key `delivered` bị reject) hoặc `confirmed` (effective first key), đồng thời so snapshot để phát hiện mutation ở order không liên quan. JWT tampering thay ký tự ở giữa signature segment, giữ đúng ba segment và assert token mới khác token gốc.

## 7. Deferred limitation

Inactive-coupon scenario là `BLOCKED_NOT_AUTOMATED`. `POST /api/admin/coupons` không nhận `is_active`, seed không có inactive coupon, và `DELETE` chỉ tạo trạng thái nonexistent. Không chỉnh database hoặc SUT để manufacture fixture. Limitation được ghi ở folder `99 - Deferred / Not Automated` và tại đây; không tạo execution result giả.

## 8. Human-review checklist trước execution

1. Review các field `implementation_status` và `assertion_requirement_source` trong ba data file.
2. Xác nhận correction của `ORDERSTATUS-GEN-001` và expansion của các matrix/boundary case.
3. Review việc thêm Authorization cho coupon request theo FR-09 C4.
4. Kiểm tra Postman sandbox hỗ trợ `pm.execution.skipRequest()` ở phiên bản đang dùng.
5. Chỉ sau approval mới import/run collection; giữ lại console/Newman evidence thật.

## 9. Corrections sau static review được phê duyệt

Đã áp dụng đúng phạm vi F-01 đến F-13 từ `reports/analysis/postman_static_review.md`. Không redesign kiến trúc data-run tổng thể, không sửa `original_*`, không thay `case_id`, và chỉ thêm `scenario_id` thứ hai cho boundary lockout của cùng `LOGIN-GEN-040`. Correction `ORDERSTATUS-GEN-001`, coupon Authorization, inactive coupon `BLOCKED_NOT_AUTOMATED` và `X-Student-Id` tiếp tục được giữ.

Static validation sau fix được ghi riêng tại `reports/analysis/postman_static_review_after_fix.md`. Không backend/API/Postman/Newman nào được chạy và không có execution evidence được tạo trong phase này.
