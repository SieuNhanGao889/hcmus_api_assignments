# CI/CD Report

## 1. Workflow

Workflow thật: [`.github/workflows/api-tests.yml`](../.github/workflows/api-tests.yml). Thiết kế tĩnh và command đầy đủ: [`reports/analysis/cicd_design_review.md`](analysis/cicd_design_review.md).

Workflow có hai job:

- `smoke-gate`: gating job chạy một scenario đã pass trước đó cho mỗi API.
- `full-regression-diagnostics`: mode thủ công, non-gating, giữ nguyên toàn bộ failure của confirmed bugs.

Mỗi job checkout test repository và EShop SUT tại pinned commit, setup Node.js, chạy `npm ci`, cài Newman + HTMLExtra, khởi động backend ở background, poll `GET /api/products`, chạy Newman và upload JSON/HTML reports bằng `if: always()`.

## 2. Folder isolation và smoke data

| API | Folder | Smoke data | Case |
|---|---|---|---|
| Login | `01 - POST Login - Data Run` | `postman/data/ci/login_smoke.json` | `LOGIN-GEN-005` |
| Apply Coupon | `02 - POST Apply Coupon - Data Run` | `postman/data/ci/coupon_smoke.json` | `COUPON-GEN-003` |
| Admin Order Status | `03 - PUT Admin Order Status - Data Run` | `postman/data/ci/order_status_smoke.json` | `ORDERSTATUS-GEN-002` |

Mọi Newman command dùng explicit `--folder`, nên data của một API không chạy folder khác. Ba smoke rows là bản sao nguyên vẹn của source rows đã pass trong execution thật và không map tới PB-01 đến PB-08.

## 3. Successful CI run

| Field | Evidence |
|---|---|
| Run | [EShop API tests #1](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445200879) |
| Event | `push` |
| Conclusion | `success` |
| Commit | `a0adb735f70f48788cd7edefd7d11b32d7b4d74a` |
| Smoke Newman step | `success` |
| Artifact upload | `success` |
| Screenshot | [`reports/screenshots/ci_success.png`](screenshots/ci_success.png) |
| Link | https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445200879 |

Các field trên được đối chiếu từ GitHub Actions run/job API và screenshot thật trong repository.

## 4. Intentional failed CI run

| Field | Evidence |
|---|---|
| Run | [EShop API tests #2](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445604904) |
| Event | `workflow_dispatch` |
| Conclusion | `failure` |
| Commit | `a0adb735f70f48788cd7edefd7d11b32d7b4d74a` |
| Smoke Newman step | `success` |
| CI-only failure step | `failure` |
| Artifact upload | `success` |
| Screenshot | [`reports/screenshots/ci_intentional_failure.png`](screenshots/ci_intentional_failure.png) |
| Link | https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445604904   |

Failure xảy ra tại step `Intentional CI-only failure after successful smoke execution`, sau khi smoke tests chạy bình thường và pass. Workflow không sửa SUT, data, credential hoặc assertion để tạo API failure giả.

## 5. Full-regression diagnostic

`run_full_regression=true` chạy nguyên ba source data files với đúng folder và giữ failure thật. Newman aggregate status vẫn non-zero khi có assertion failure; `continue-on-error` chỉ làm job diagnostic không chặn smoke gate. Không dùng `--suppress-exit-code` và PB-01 đến PB-08 không bị suppress.

Hai CI evidence runs hiện tại đều để diagnostic job ở trạng thái `skipped`; không claim đã có CI full-regression execution. Local full-regression evidence nằm trong `reports/newman/`.

## 6. Remaining assignment action

`TODO`: Đề bài yêu cầu “two sample commits”, nhưng hai run thật hiện cùng trỏ tới commit `a0adb735...`. Sinh viên cần tạo/ghi nhận hai commit riêng theo đúng wording của đề, hoặc có xác nhận của giảng viên rằng một commit với hai workflow modes được chấp nhận. Không thay URL/commit bằng dữ liệu giả.
