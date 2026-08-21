# Rà soát thiết kế CI/CD cho bộ kiểm thử API

## 1. Phạm vi và trạng thái bằng chứng

Artifact này mô tả thiết kế tĩnh cho `PHASE_10_CICD_DESIGN_AND_IMPLEMENTATION`. Phase này không chạy Newman, không gửi request tới SUT, không push repository và không kích hoạt GitHub Actions. Vì vậy tài liệu không tuyên bố workflow đã `PASS` hoặc `FAIL`, không ghi run URL/run ID/timestamp và không tạo screenshot.

Workflow dùng collection và environment hiện có, không sửa SUT và không thay đổi assertion của PB-01 đến PB-08. SUT được checkout riêng tại commit `85af3ba875c88283615e22cb108f13e2fccaf0e9`, là commit đã được ghi nhận trong `reports/analysis/postman_setup_analysis.md` và hiện là commit của nhánh `main` khi thực hiện static review.

## 2. Workflow triggers

File `.github/workflows/api-tests.yml` có ba trigger:

- `push`: luôn chạy smoke gate.
- `pull_request`: luôn chạy smoke gate.
- `workflow_dispatch`: luôn chạy smoke gate và cung cấp hai boolean input mặc định `false`:
  - `force_failure`: sau khi smoke thực sự chạy thành công, chạy một step CI-only có nhãn rõ ràng và `exit 1`.
  - `run_full_regression`: bật job full regression diagnostic, không gating.

Không có trigger nào tự động chạy full regression trên mọi commit. Điều này giữ smoke gate nhỏ và ổn định, trong khi toàn bộ suite vẫn có thể gọi thủ công.

## 3. Cấu trúc job GitHub Actions

| Job | Vai trò | Gating | Điều kiện chạy |
|---|---|---|---|
| `smoke-gate` | Chạy ba smoke dataset, mỗi dataset đại diện một API | Có | Mọi trigger |
| `full-regression-diagnostics` | Chạy ba source dataset đầy đủ và giữ nguyên failure thật | Không | Chỉ `workflow_dispatch` với `run_full_regression=true` |

Hai job dùng runner riêng, cùng checkout đúng test repository và đúng SUT commit. Step Newman của full regression có `continue-on-error: true`; `steps.regression.outcome` vẫn ghi `failure` nếu assertion thật thất bại, báo cáo vẫn được upload, nhưng job diagnostic không chặn smoke gate.

## 4. Cài đặt và khởi động backend

Mỗi job thực hiện:

1. `actions/checkout@v4` cho repository bài làm.
2. `actions/checkout@v4` lần thứ hai cho `ttbhanh/eshop-sut`, pin ref `85af3ba875c88283615e22cb108f13e2fccaf0e9`, đặt tại `EShop-source`.
3. `actions/setup-node@v4` với Node.js `20` và npm cache dựa trên `EShop-source/backend/package-lock.json`.
4. `npm ci` trong `EShop-source/backend`.
5. Chạy `node server.js` bằng `nohup` ở background, ghi PID vào `GITHUB_ENV` và log vào thư mục report runtime.
6. Chỉ dừng backend ở step `always()` cuối job, sau khi toàn bộ Newman execution và upload artifact đã hoàn tất.

SUT upstream không có script `start` trong `package.json`; entry point thực tế là `backend/server.js`, lắng nghe port `3000`. Workflow không sửa file SUT.

## 5. Chiến lược readiness

Workflow poll public endpoint `GET http://localhost:3000/api/products` tối đa 30 lần, mỗi lần cách 2 giây. Readiness chỉ thành công khi `curl --fail` nhận response 2xx. Trong lúc chờ, workflow kiểm tra PID; nếu backend chết sớm hoặc timeout thì in `backend.log` và fail ngay, không chạy Newman trên một SUT chưa sẵn sàng.

## 6. Cài Newman và reporter

Cả hai job chạy:

```bash
npm install --global newman newman-reporter-htmlextra
```

Mỗi Newman run dùng đồng thời reporter `cli,json,htmlextra`, tạo cả JSON và HTML. Không dùng reporter hoặc option làm im lặng assertion failure.

## 7. Lựa chọn smoke scenario

| `case_id` | `scenario_id` | API | Lý do an toàn cho gating | Bằng chứng Newman trước đó |
|---|---|---|---|---|
| `LOGIN-GEN-005` | `LOGIN-GEN-005` | `POST /api/login` | Email đúng format nhưng không tồn tại; assertion yêu cầu không cấp token. Không thuộc PB-01, PB-02 hoặc PB-03. | `reports/newman/login_run.json`, iteration `4`: `0` failed assertions, CASE status `401`, tổng `3` assertions pass trong iteration. |
| `COUPON-GEN-003` | `COUPON-GEN-003` | `POST /api/apply-coupon` | Coupon code không tồn tại; assertion yêu cầu không trả discount hợp lệ và không lộ internal detail. Không thuộc PB-04, PB-05 hoặc PB-06. | `reports/newman/coupon_run.json`, iteration `2`: `0` failed assertions, CASE status `404`, tổng `18` assertions pass trong iteration. |
| `ORDERSTATUS-GEN-002` | `ORDERSTATUS-GEN-002` | `PUT /api/admin/orders/:id/status` | Admin thực hiện transition hợp lệ `pending -> confirmed`, có read-after kiểm tra persisted state. Không thuộc PB-02, PB-07 hoặc PB-08. | `reports/newman/order_status_rerun_after_automation_fix.json`, iteration `1`: `0` failed assertions, CASE status `200`, tổng `10` assertions pass trong iteration. |

Ba row trong `postman/data/ci/` là bản sao nguyên vẹn của đúng object tương ứng trong source data; giữ nguyên `case_id`, `scenario_id`, body, mode, requirement source và traceability. Không có expected behavior mới được tạo.

## 8. Chiến lược data-driven

Smoke gate gọi cùng collection và environment hiện có ba lần. Mỗi lần vừa chọn rõ folder tương ứng bằng `--folder`, vừa truyền một external JSON file qua `--iteration-data`. Cặp folder/data được cố định như sau để Newman không chạy các folder API không liên quan:

| Folder được chọn | External data file |
|---|---|
| `01 - POST Login - Data Run` | `postman/data/ci/login_smoke.json` |
| `02 - POST Apply Coupon - Data Run` | `postman/data/ci/coupon_smoke.json` |
| `03 - PUT Admin Order Status - Data Run` | `postman/data/ci/order_status_smoke.json` |

Field `dataset` trong từng row vẫn giữ nguyên để collection flow và setup script hiện có hoạt động đúng. Setup tiếp tục tạo user, coupon, order, token và runtime ID theo cách động như lần chạy trước.

Các file smoke:

- `postman/data/ci/login_smoke.json`
- `postman/data/ci/coupon_smoke.json`
- `postman/data/ci/order_status_smoke.json`

Collection-level pre-request script vẫn chạy cho mọi request và gọi:

```javascript
pm.request.headers.upsert({ key: "X-Student-Id", value: studentId });
```

`studentId=23127364` tiếp tục lấy từ `postman/HW06_Local.postman_environment.json`; workflow không thay thế hoặc bỏ qua cơ chế này.

## 9. Chiến lược full regression diagnostic

Khi `run_full_regression=true`, job diagnostic chạy nguyên ba file:

- `postman/data/login_data.json`
- `postman/data/coupon_data.json`
- `postman/data/order_status_data.json`

Mỗi source data file cũng được ghép với đúng folder tương ứng: Login dùng `01 - POST Login - Data Run`, Coupon dùng `02 - POST Apply Coupon - Data Run`, và Order Status dùng `03 - PUT Admin Order Status - Data Run`.

Ba Newman process đều được chạy kể cả khi process trước trả non-zero; biến `regression_status` tổng hợp outcome và cuối step trả non-zero nếu có bất kỳ failure. `continue-on-error: true` chỉ làm job này không gating; nó không thay đổi assertion, không chuyển failure thành pass trong Newman report và không dùng `--suppress-exit-code`. Step summary ghi trực tiếp `steps.regression.outcome`, còn JSON/HTML giữ chi tiết failure PB-01 đến PB-08.

## 10. Chiến lược tạo run thành công

Sau human approval, chạy `workflow_dispatch` với `force_failure=false` và `run_full_regression=false`, hoặc dùng một `push`/`pull_request`. Chỉ ba scenario đã có execution pass trước đó được dùng làm gate. Đây mới là chiến lược dự kiến; chỉ được ghi nhận là successful CI run sau khi GitHub Actions thật sự hoàn tất thành công.

## 11. Chiến lược intentional failure

Chạy `workflow_dispatch` với `force_failure=true`. Smoke Newman phải chạy và thành công trước. Chỉ khi toàn bộ smoke step thành công, expression sau mới cho phép step CI-only chạy:

```yaml
if: ${{ success() && github.event_name == 'workflow_dispatch' && inputs.force_failure == true }}
```

Step đó chỉ in nhãn giải thích rồi `exit 1`. Nó không sửa data, credential, SUT, collection hoặc assertion và không giả làm một API failure. Nếu smoke thật sự fail, `success()` ngăn step intentional failure chạy để không che khuất nguyên nhân thật.

## 12. Đường dẫn JSON/HTML report

Smoke runtime reports:

- `reports/ci/smoke/login_smoke.json`
- `reports/ci/smoke/login_smoke.html`
- `reports/ci/smoke/coupon_smoke.json`
- `reports/ci/smoke/coupon_smoke.html`
- `reports/ci/smoke/order_status_smoke.json`
- `reports/ci/smoke/order_status_smoke.html`
- `reports/ci/smoke/backend.log`

Full-regression runtime reports:

- `reports/ci/full-regression/login_regression.json`
- `reports/ci/full-regression/login_regression.html`
- `reports/ci/full-regression/coupon_regression.json`
- `reports/ci/full-regression/coupon_regression.html`
- `reports/ci/full-regression/order_status_regression.json`
- `reports/ci/full-regression/order_status_regression.html`
- `reports/ci/full-regression/backend.log`

Các path này chỉ được tạo trên GitHub runner khi workflow thực sự chạy; phase hiện tại không tạo execution report.

## 13. Chiến lược upload artifact

`actions/upload-artifact@v4` upload toàn bộ thư mục report tương ứng với `if: always()`:

- Smoke: artifact `newman-smoke-reports-${{ github.run_id }}`.
- Full regression: artifact `newman-full-regression-reports-${{ github.run_id }}`.

Vì upload dùng `always()`, report đã tạo vẫn được thu thập nếu Newman fail, nếu intentional CI-only step fail hoặc nếu job diagnostic ghi nhận failure. `if-no-files-found: warn` giúp readiness/setup failure không tạo thêm một lỗi upload gây nhiễu nguyên nhân gốc. Retention là 14 ngày.

## 14. PB-01 đến PB-08 không bị suppress

Không sửa `postman/HW06_EShop_API_Tests.postman_collection.json`, ba source data file hoặc bug report. Full regression vẫn dùng toàn bộ source row, gồm mọi scenario liên quan PB-01 đến PB-08. Không có `--suppress-exit-code`, assertion filter, data rewrite hay conditional skip mới.

Smoke chỉ chọn case khác để tạo gate nhỏ; việc không đưa confirmed-bug scenario vào smoke không xóa chúng khỏi full regression. Hash collection và source data được so sánh trong static validation để bảo đảm assertion/data lịch sử không thay đổi.

## 15. Secrets và runtime variables

- Environment chỉ chứa hai credential seed công khai đã được xác nhận từ SUT: `admin@eshop.com` / `Admin123!` và `test@eshop.com` / `Test1234!`.
- Các field token (`adminToken`, `userToken`, `loginToken`, `tamperedAdminToken`) và runtime ID trong environment đều để rỗng trong repository.
- Collection setup/extraction tạo token, user ID, coupon ID và order ID trong runtime; smoke data không hard-code các giá trị đó.
- Workflow không export environment sau execution, nên không ghi JWT/runtime ID ngược vào repository.
- JSON/HTML runtime reports có thể chứa dữ liệu execution và chỉ được giữ dưới dạng GitHub artifact; chúng không được commit trong phase này.

## 16. Exact shell commands trong workflow

### Cài đặt

```bash
npm ci
npm install --global newman newman-reporter-htmlextra
```

### Khởi động và readiness

```bash
mkdir -p "$GITHUB_WORKSPACE/reports/ci/smoke"
nohup node server.js > "$GITHUB_WORKSPACE/reports/ci/smoke/backend.log" 2>&1 &
echo "BACKEND_PID=$!" >> "$GITHUB_ENV"
```

Job diagnostic dùng cùng ba lệnh, thay `reports/ci/smoke` bằng `reports/ci/full-regression`.

```bash
for attempt in {1..30}; do
  if curl --silent --show-error --fail http://localhost:3000/api/products > /dev/null; then
    echo "EShop backend is ready."
    exit 0
  fi
  if ! kill -0 "$BACKEND_PID" 2>/dev/null; then
    echo "EShop backend stopped before becoming ready."
    cat reports/ci/smoke/backend.log
    exit 1
  fi
  sleep 2
done
echo "Timed out waiting for EShop backend readiness."
cat reports/ci/smoke/backend.log
exit 1
```

### Smoke Newman

```bash
newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "01 - POST Login - Data Run" \
  --iteration-data postman/data/ci/login_smoke.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/smoke/login_smoke.json \
  --reporter-htmlextra-export reports/ci/smoke/login_smoke.html

newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "02 - POST Apply Coupon - Data Run" \
  --iteration-data postman/data/ci/coupon_smoke.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/smoke/coupon_smoke.json \
  --reporter-htmlextra-export reports/ci/smoke/coupon_smoke.html

newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "03 - PUT Admin Order Status - Data Run" \
  --iteration-data postman/data/ci/order_status_smoke.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/smoke/order_status_smoke.json \
  --reporter-htmlextra-export reports/ci/smoke/order_status_smoke.html
```

### Full-regression Newman

```bash
newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "01 - POST Login - Data Run" \
  --iteration-data postman/data/login_data.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/full-regression/login_regression.json \
  --reporter-htmlextra-export reports/ci/full-regression/login_regression.html

newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "02 - POST Apply Coupon - Data Run" \
  --iteration-data postman/data/coupon_data.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/full-regression/coupon_regression.json \
  --reporter-htmlextra-export reports/ci/full-regression/coupon_regression.html

newman run "$COLLECTION" \
  --environment "$ENVIRONMENT" \
  --folder "03 - PUT Admin Order Status - Data Run" \
  --iteration-data postman/data/order_status_data.json \
  --reporters cli,json,htmlextra \
  --reporter-json-export reports/ci/full-regression/order_status_regression.json \
  --reporter-htmlextra-export reports/ci/full-regression/order_status_regression.html
```

### Intentional failure và cleanup

```bash
echo "Intentional CI-only failure requested with force_failure=true."
exit 1
```

```bash
if [ -n "${BACKEND_PID:-}" ] && kill -0 "$BACKEND_PID" 2>/dev/null; then
  kill "$BACKEND_PID"
fi
```

## 17. Static validation

Đã thực hiện các kiểm tra read-only sau; không chạy Newman và không gửi API request:

| Kiểm tra | Kết quả | Cách đối chiếu |
|---|---|---|
| YAML syntax/structure | **PASS** | PyYAML parse thành công; có đúng ba trigger, hai job và 11 step/job. |
| Smoke JSON syntax | **PASS** | Cả ba file parse bằng `JSON.parse`. |
| Repository input paths | **PASS** | Collection, environment, ba source data và ba smoke data đều tồn tại. Các report path là runtime output nên được tạo bởi `mkdir -p` trên runner, không tạo trong phase này. |
| Newman folder/data mapping | **PASS** | Cả sáu command có đúng một `--folder`; ba folder trong collection tồn tại và mỗi folder được ghép đúng smoke/source data của API tương ứng. |
| SUT ref và backend paths | **PASS** | `git ls-remote` xác nhận `main` trỏ tới commit pin `85af3ba...`; raw `backend/package-lock.json` và `backend/server.js` tại ref này trả HTTP `200`. |
| Smoke row identity | **PASS** | Mỗi smoke array có đúng một row và `deepStrictEqual` với object có cùng `case_id` + `scenario_id` trong source data. |
| Previous pass evidence | **PASS** | Đọc trực tiếp executions theo iteration: Login `4` có `3/3` assertions pass; Coupon `2` có `18/18`; Order Status rerun `1` có `10/10`. |
| PB mapping exclusion | **PASS** | Không smoke `scenario_id` nào nằm trong mapping PB-01 đến PB-08 từ tám bug report. |
| Confirmed-bug assertion/data preservation | **PASS** | SHA-256 của collection, environment và ba source data khớp snapshot trước phase; không file nào bị sửa. |
| `X-Student-Id` | **PASS** | Collection vẫn có `pm.request.headers.upsert(...)`; environment vẫn có `studentId=23127364`. |
| Runtime secret/ID state | **PASS** | Các token và runtime user/coupon/order ID được kiểm tra vẫn là chuỗi rỗng trong environment; smoke row chỉ dùng template động. Không có JWT literal được thêm. |
| Report upload on failure | **PASS** | Cả hai `actions/upload-artifact@v4` step dùng `if: always()`; Newman tạo JSON + HTML trước upload. |
| Full-regression failure integrity | **PASS** | Không có `--suppress-exit-code`; aggregate status trả non-zero, step outcome được ghi, và `continue-on-error` chỉ làm job diagnostic không gating. |
| Intentional failure ordering | **PASS** | Step nằm sau smoke Newman và có guard `success()`, `workflow_dispatch`, `force_failure == true`. |
| Evidence integrity | **PASS** | Không tạo report runtime, screenshot, run URL, run ID hoặc timestamp; tài liệu chỉ mô tả chiến lược dự kiến. |

Các SHA-256 được dùng để chứng minh source artifact không đổi:

- Collection: `44F146FB5E2B60C050C9C84E11E5B871FA51350F558BC9AACE795F78CFB8E425`
- Environment: `C89C5CE4268911786889908DE9298C09A2C8D397A037F4D052B3C8866D89DCC4`
- Login source data: `699FE2505856EB51ADBF26150C85D9201B4ABA5EF5B54B925F00D673B6B6B6AA`
- Coupon source data: `B22C917AB6160766B7697973DA4ECE2C6360A7628297C64A8C3FB308D1FCA3C8`
- Order Status source data: `D184BB7282078D22E521FBF4F465FA58B9E1FA33222F76E85C45D9E8B75B10C6`

## 18. Các bước thủ công sau human approval

1. Review bốn nhóm artifact được tạo trong phase này và xác nhận ba smoke scenario phù hợp.
2. Commit `.github/workflows/api-tests.yml`, ba file `postman/data/ci/*.json` và `reports/analysis/cicd_design_review.md` bằng commit message phù hợp.
3. Push commit lên public GitHub repository của sinh viên.
4. Mở GitHub Actions, chạy workflow với `force_failure=false`, `run_full_regression=false`.
5. Chỉ sau khi run kết thúc, lưu link/run ID thật và chụp screenshot passing run theo yêu cầu bài tập.
6. Chạy lại bằng `workflow_dispatch` với `force_failure=true`, giữ `run_full_regression=false` để tạo intentional failed run sau smoke.
7. Chỉ sau khi run kết thúc, lưu link/run ID thật và chụp screenshot failed run.
8. Tùy chọn chạy `run_full_regression=true` để thu report diagnostic PB-01 đến PB-08; không diễn giải failure đã biết thành lỗi pipeline setup.
9. Cập nhật `reports/cicd_report.md` và báo cáo cuối bằng URL, run ID, screenshot và commit thật. Không điền dữ liệu giả nếu run chưa xảy ra.

Stop tại human-approval checkpoint. Không có execution evidence nào được tạo trong phase này.
