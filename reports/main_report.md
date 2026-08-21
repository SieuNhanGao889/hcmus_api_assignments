# Báo cáo HW06 – API Testing

## 1. Thông tin bài làm

| Mục | Giá trị |
|---|---|
| MSSV | `23127364` |
| SUT | EShop API |
| Public repository | [SieuNhanGao889/hcmus_api_assignments](https://github.com/SieuNhanGao889/hcmus_api_assignments) |
| Base URL khi thực thi | `http://localhost:3000` |
| Header bắt buộc | `X-Student-Id: 23127364` |
| Công cụ | Postman, Newman, GitHub Actions, Codex/ChatGPT |

Bài làm áp dụng quy trình AI-first theo từng bước: phân tích đặc tả, thiết kế coverage, sinh candidate cases, human audit, manual extension, Postman implementation, execution, phân tích failure, human confirmation, automation correction, rerun và CI/CD. AI hỗ trợ phân tích và soạn artifact; sinh viên chịu trách nhiệm audit, execution, bug confirmation và bằng chứng cuối.

## 2. API đã chọn

| Pool | API | Lý do kiểm thử |
|---|---|---|
| A | `POST /api/login` | Authentication, credential handling, lockout, token và information leakage. |
| B | `POST /api/apply-coupon` | Boundary, discount formula, usage rule, type validation và business-rule integrity. |
| C | `PUT /api/admin/orders/:id/status` | Admin authorization, state transition và persisted-state integrity. |

`NEEDS_HUMAN_CONFIRMATION`: Việc bộ ba API không trùng với thành viên khác trong nhóm không thể xác minh chỉ từ repository.

## 3. AI-first test design

Mỗi API có artifact phân tích spec và test design riêng:

| API | Spec analysis | Test design |
|---|---|---|
| Login | [login_spec_analysis.md](analysis/login_spec_analysis.md) | [login_test_design.md](analysis/login_test_design.md) |
| Apply Coupon | [coupon_spec_analysis.md](analysis/coupon_spec_analysis.md) | [coupon_test_design.md](analysis/coupon_test_design.md) |
| Admin Order Status | [admin_order_status_spec_analysis.md](analysis/admin_order_status_spec_analysis.md) | [admin_order_status_test_design.md](analysis/admin_order_status_test_design.md) |

Coverage gồm domain partitions, missing/empty/null/wrong-type, boundary, schema, security, parser robustness, authorization và state behavior. Các điểm đặc tả chưa đủ rõ được giữ là `SPEC_UNDEFINED` hoặc exploratory thay vì tự tạo expected behavior.

## 4. Agent Skill và demo

Reusable Agent Skill tại [agent-skill/SKILL.md](../agent-skill/SKILL.md) nhận API specification và target endpoint, sau đó thực hiện:

`Specification Reader -> Rule Extractor -> Test Dimensions -> Candidate Generator -> Traceability Validator -> PENDING_HUMAN_REVIEW`

Skill không tự quyết định `VALID`, `INVALID`, `INCOMPLETE`, không tự tạo manual extension và không tạo execution evidence.

Artifact:

- Thiết kế: [design.md](../agent-skill/design/design.md)
- Pseudocode: [pseudocode.md](../agent-skill/design/pseudocode.md)
- Diagram: [diagram.png](../agent-skill/design/diagram.png)
- Demo prompt: [prompt-demo.md](../agent-skill/demo/prompt-demo.md)
- Video thật: [CSC13102_23KTPM1_23127364_HW6](https://youtu.be/4fAWYhzeHzQ)

`COMPLETE`: Diagram cuối do sinh viên tự thiết kế và tự vẽ sơ bộ; AI audit ghi rõ AI không thiết kế và tạo `design/diagram.png`.

## 5. Test-case generation, human audit và extension

### 5.1 Số lượng

| API | AI-generated | Manual extension | Tổng | `VALID` | `INCOMPLETE` | `INVALID` |
|---|---:|---:|---:|---:|---:|---:|
| Login | 40 | 5 | 45 | 38 | 2 | 0 |
| Apply Coupon | 40 | 5 | 45 | 36 | 4 | 0 |
| Admin Order Status | 42 | 5 | 47 | 41 | 1 | 0 |
| **Tổng** | **122** | **15** | **137** | **115** | **7** | **0** |

Nguồn: sáu workbook trong [test-cases/](../test-cases/). Workbook gốc giữ `PENDING_HUMAN_REVIEW`; workbook audited giữ quyết định của sinh viên. Bảy case `INCOMPLETE` đều có reasoning trong `audit_notes` và correction trong `human_correction`.

### 5.2 Manual extensions

Mỗi API có đúng năm extension với `source = MANUAL_EXTENSION`. Các gap nổi bật gồm token usability qua nhiều request, duplicate JSON keys, công thức coupon với controlled fixture, persisted-state verification sau authorization failure và ma trận state transition. Lý do AI bỏ sót/gợi ý coverage được ghi tại:

- [login_extension_gaps.md](analysis/login_extension_gaps.md)
- [coupon_extension_gaps.md](analysis/coupon_extension_gaps.md)
- [admin_order_status_extension_gaps.md](analysis/admin_order_status_extension_gaps.md)

## 6. Postman implementation

Artifact chính:

- Collection: [HW06_EShop_API_Tests.postman_collection.json](../postman/HW06_EShop_API_Tests.postman_collection.json)
- Environment: [HW06_Local.postman_environment.json](../postman/HW06_Local.postman_environment.json)
- External data: [postman/data/](../postman/data/)
- Implementation notes: [postman_implementation_notes.md](analysis/postman_implementation_notes.md)

Collection có bốn top-level folders, 43 requests, 44 pre-request event scripts (gồm collection-level script) và 43 post-response/test scripts. Mỗi iteration dựng disposable fixture, thực hiện CASE/SEQUENCE, trích xuất token/ID động, xác minh kết quả và cleanup khi áp dụng. Token, user ID, coupon ID và order ID không được hard-code trong repository.

Collection-level pre-request script upsert `X-Student-Id` từ environment cho mọi request. Mỗi request cũng khai báo header rõ ràng để tăng khả năng audit.

Postman Console evidence: [postman_23127364_id_console.png](screenshots/postman_23127364_id_console.png) hiển thị log `X-Student-Id attached: 23127364` trong workspace thật.

## 7. Postman features thực sự đã dùng

| Feature | Trạng thái | Evidence |
|---|---|---|
| Collection | `USED_AND_EVIDENCED` | Collection JSON hiện có. |
| Folders | `USED_AND_EVIDENCED` | Ba folder API và một folder deferred. |
| Requests | `USED_AND_EVIDENCED` | 43 request definitions. |
| Pre-request scripts | `USED_AND_EVIDENCED` | Header injection, runtime generation và flow guards. |
| Post-response/test scripts | `USED_AND_EVIDENCED` | Status, schema, business, security và persisted-state assertions. |
| Collection variables | `USED_AND_EVIDENCED` | Bốn collection variables. |
| Environment variables | `USED_AND_EVIDENCED` | 30 environment entries, gồm base URL, student ID và runtime placeholders. |
| External JSON data | `USED_AND_EVIDENCED` | Ba source dataset và ba CI smoke dataset. |
| Data-driven Newman runs | `USED_AND_EVIDENCED` | Newman JSON reports có 46, 47 và 73 iterations. |
| Dynamic variable extraction | `USED_AND_EVIDENCED` | Token và fixture IDs được lấy từ setup responses. |
| Assertions | `USED_AND_EVIDENCED` | Newman reports và collection test scripts. |
| Postman workspace | `USED_AND_EVIDENCED` | Screenshot Console hiển thị workspace, collection và environment thật. |
| GUI Collection Runner | `USED_BUT_DOCUMENTATION_MISSING` nếu đã dùng | Execution có bằng chứng Newman; không có bằng chứng riêng cho GUI Runner. |
| Monitor | `NOT_USED` | Không có monitor artifact/evidence. |
| Mock server | `NOT_USED` | Không có mock artifact/evidence. |

Monitor và mock server là ví dụ “as many as reasonably can”, không phải deliverable bắt buộc. Báo cáo không claim các feature này đã dùng.

## 8. Newman execution

### 8.1 Kết quả ban đầu

| API | Iterations | Requests | Assertions | Passed assertions | Failed assertions |
|---|---:|---:|---:|---:|---:|
| Login | 46 | 101 | 153 | 142 | 11 |
| Apply Coupon | 47 | 755 | 837 | 833 | 4 |
| Admin Order Status ban đầu | 73 | 638 | 786 | 771 | 15 |
| **Tổng ban đầu** | **166** | **1,494** | **1,776** | **1,746** | **30** |

Phân tích tại [newman_execution_analysis.md](analysis/newman_execution_analysis.md) phân loại 22 failed assertions thành tám potential SUT root causes và tám failed assertions thành hai automation root causes, không coi mọi assertion failure là bug riêng.

### 8.2 Human review, correction và rerun

Sinh viên xác nhận PB-01 đến PB-08 và phê duyệt correction cho AF-01/AF-02. Collection chỉ sửa assertion mode bị sai; các assertion chứng minh confirmed bug được giữ nguyên. Rerun Order Status có 792 assertions, 785 pass và 7 fail; bảy failure còn lại map tới PB-02, PB-07 và PB-08.

Final scenario-level view sau correction:

| API | Scenario rows | Passed rows | Failed rows |
|---|---:|---:|---:|
| Login | 46 | 35 | 11 |
| Apply Coupon | 47 | 43 | 4 |
| Admin Order Status rerun | 73 | 66 | 7 |
| **Tổng** | **166** | **144** | **22** |

Raw JSON và HTML evidence nằm tại [reports/newman/](newman/). Order Status rerun có JSON riêng; HTML hiện có là report của execution đã lưu trước đó.

## 9. Bug analysis và GitHub Issues

Có tám root causes được `CONFIRMED_BY_HUMAN_REVIEW`. Không nhân số bug theo số assertion failure.

| Bug | Severity | Endpoint | GitHub Issue |
|---|---|---|---|
| PB-01 | HIGH | Login | [#1](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/1) |
| PB-02 | HIGH | Login, Order Status | [#2](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/2) |
| PB-03 | MEDIUM | Login | [#3](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/3) |
| PB-04 | MEDIUM | Apply Coupon | [#4](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/4) |
| PB-05 | HIGH | Apply Coupon | [#5](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/5) |
| PB-06 | HIGH | Apply Coupon | [#6](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/6) |
| PB-07 | CRITICAL | Admin Order Status | [#7](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/7) |
| PB-08 | HIGH | Admin Order Status | [#8](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/8) |

Canonical details và ảnh: [bugs_summary.md](../bug-reports/bugs_summary.md), [PB-01.md](../bug-reports/PB-01.md) đến [PB-08.md](../bug-reports/PB-08.md), [bug-reports/screenshots/](../bug-reports/screenshots/). Live GitHub Issue bodies chứa image attachment syntax khi static audit đối chiếu.

## 10. CI/CD

Workflow: [api-tests.yml](../.github/workflows/api-tests.yml). Thiết kế chi tiết: [cicd_design_review.md](analysis/cicd_design_review.md). Báo cáo run: [cicd_report.md](cicd_report.md).

`smoke-gate` checkout repository và pinned SUT, setup Node.js, `npm ci`, cài Newman/HTMLExtra, start backend, poll readiness, rồi chạy một smoke row cho mỗi API với explicit `--folder`. JSON/HTML reports được upload bằng `always()`. Full regression vẫn có thể bật thủ công dưới dạng diagnostic/non-gating và không suppress confirmed-bug assertions.

- Successful run: [#3](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32447382046), commit `c6bb21d...`
- Intentional failed run: [#2](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445604904)
- Screenshots: [ci_success.png](screenshots/ci_success.png), [ci_intentional_failure.png](screenshots/ci_intentional_failure.png)

Run #2 xác nhận smoke Newman step thành công trước khi step `Intentional CI-only failure after successful smoke execution` fail. Passing run #3 dùng commit `c6bb21d...`; intentional failing run #2 dùng commit `a0adb735...`, nên đã có hai sample commits riêng.

## 11. AI usage và trách nhiệm con người

Canonical AI audit: [ai_audit_report.md](ai_audit_report.md). AI critique: [ai_critique.md](ai_critique.md).

AI hỗ trợ spec analysis, test design, candidate generation, correction theo quyết định human, Postman/CI implementation và đọc evidence. Sinh viên tự audit case, chọn extension, chạy Newman, xác nhận bug, tạo GitHub Issues, chạy CI và chụp evidence. AI audit không được viết lại lịch sử recommendation thành human decision.


## 12. Hướng dẫn tái hiện

Giữ backend EShop chạy liên tục tại port `3000`, sau đó chạy từng folder với đúng data file:

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "01 - POST Login - Data Run" \
  --iteration-data postman/data/login_data.json \
  --reporters cli,json,htmlextra

newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "02 - POST Apply Coupon - Data Run" \
  --iteration-data postman/data/coupon_data.json \
  --reporters cli,json,htmlextra

newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "03 - PUT Admin Order Status - Data Run" \
  --iteration-data postman/data/order_status_data.json \
  --reporters cli,json,htmlextra
```

CI có thể chạy bằng `push` hoặc `workflow_dispatch`. `force_failure=true` chỉ tạo failure sau khi smoke pass; `run_full_regression=true` bật diagnostic suite.

## 13. Artifact index

| Deliverable | Vị trí |
|---|---|
| Main report | `reports/main_report.md`, `reports/main_report.pdf` |
| Audited test cases | `test-cases/*_audited.xlsx` |
| Test summary | `test-cases/test_summary.md`, `README.md` |
| Collection/environment/data | `postman/` |
| Newman JSON/HTML | `reports/newman/` |
| Bug reports/issues/screenshots | `bug-reports/` |
| CI workflow/report/screenshots | `.github/workflows/api-tests.yml`, `reports/cicd_report.md`, `reports/screenshots/` |
| Agent Skill/design/demo | `agent-skill/` |
| AI audit/critique | `reports/ai_audit_report.md`, `reports/ai_critique.md` |
| Commit log | `git-log/commit_log.txt` sau khi sinh viên refresh ở commit cuối |

## 14. Known limitations và deferred items

- Test unlock tự động sau 30 giây của login không được automation cover.
- Inactive-coupon fixture không được tạo vì API/SUT không cung cấp cách hợp lệ để dựng `is_active=0`; không chỉnh database để manufacture setup.
- Postman workspace và Console đã có screenshot evidence; GUI Collection Runner chưa có execution evidence nên không claim đã dùng.
- Monitor và mock server không dùng.
- Full-regression diagnostic mode đã được thiết kế trong GitHub Actions nhưng hai CI evidence runs hiện chỉ chạy smoke gate; local full regression có Newman evidence riêng.
- X-Student-Id tồn tại trong collection, reports và Postman Console screenshot.
