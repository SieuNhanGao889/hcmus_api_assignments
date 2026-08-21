# Test Summary

## 1. Nguồn dữ liệu

- Designed/audited cases: `test-cases/*_test_cases.xlsx` và `test-cases/*_test_cases_audited.xlsx`
- Executable scenarios: `postman/data/login_data.json`, `coupon_data.json`, `order_status_data.json`
- Execution: `reports/newman/login_run.json`, `coupon_run.json`, `order_status_run.json`, `order_status_rerun_after_automation_fix.json`
- Failure analysis: `reports/analysis/newman_execution_analysis.md`, `order_status_automation_fix_review.md`
- Confirmed bugs: `bug-reports/bugs_summary.md`

Một designed `case_id` có thể mở rộng thành nhiều `scenario_id`, nên `Scenario rows executed` không bằng `Total designed`.

## 2. Design và human audit

| API | AI-generated | Manual extension | Total designed | `VALID` | `INCOMPLETE` | `INVALID` | `STUDENT_DESIGNED` |
|---|---:|---:|---:|---:|---:|---:|---:|
| `POST /api/login` | 40 | 5 | 45 | 38 | 2 | 0 | 5 |
| `POST /api/apply-coupon` | 40 | 5 | 45 | 36 | 4 | 0 | 5 |
| `PUT /api/admin/orders/:id/status` | 42 | 5 | 47 | 41 | 1 | 0 | 5 |
| **Tổng** | **122** | **15** | **137** | **115** | **7** | **0** | **15** |

Tất cả 122 AI-generated rows ban đầu có `PENDING_HUMAN_REVIEW`. Sinh viên audit từng row. Bảy row `INCOMPLETE` giữ reasoning và đều có `human_correction`; không có row `INVALID`.

## 3. Execution ban đầu

| API | Iterations | Requests | Assertions | Passed assertions | Failed assertions | Failed scenario rows |
|---|---:|---:|---:|---:|---:|---:|
| Login | 46 | 101 | 153 | 142 | 11 | 11 |
| Apply Coupon | 47 | 755 | 837 | 833 | 4 | 4 |
| Admin Order Status | 73 | 638 | 786 | 771 | 15 | 14 |
| **Tổng** | **166** | **1,494** | **1,776** | **1,746** | **30** | **29** |

Phân tích ban đầu xác định 22 failed assertions liên quan tám potential SUT root causes và tám failed assertions liên quan hai automation root causes AF-01/AF-02. Sau human review, PB-01 đến PB-08 được xác nhận; AF-01/AF-02 được phê duyệt correction.

## 4. Final effective execution sau correction

| API | Scenario rows executed | Passed rows | Failed rows | Assertions | Passed assertions | Failed assertions |
|---|---:|---:|---:|---:|---:|---:|
| Login | 46 | 35 | 11 | 153 | 142 | 11 |
| Apply Coupon | 47 | 43 | 4 | 837 | 833 | 4 |
| Admin Order Status rerun | 73 | 66 | 7 | 792 | 785 | 7 |
| **Tổng** | **166** | **144** | **22** | **1,782** | **1,760** | **22** |

Order Status rerun loại bỏ đúng tám false failures do AF-01/AF-02. Bảy assertion failures còn lại tiếp tục map tới PB-02, PB-07 và PB-08; confirmed-bug assertions không bị xóa hoặc làm yếu.

## 5. Confirmed bugs

| API | Bug liên quan | Số mapping |
|---|---|---:|
| Login | PB-01, PB-02, PB-03 | 3 |
| Apply Coupon | PB-04, PB-05, PB-06 | 3 |
| Admin Order Status | PB-02, PB-07, PB-08 | 3 |

Có **8 unique confirmed bugs**. PB-02 ảnh hưởng hai API nên tổng mapping theo API bằng 9.

## 6. Deferred coverage

- Login unlock sau 30 giây chưa được automation cover.
- Inactive coupon không có fixture hợp lệ từ API/seed nên không chỉnh database hoặc SUT để tạo setup.

Không tạo execution result giả cho các phần deferred này.
