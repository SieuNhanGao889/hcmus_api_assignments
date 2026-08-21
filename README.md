# 23127364_HW06_AI_API_100

Repository bài làm **HW06 – API Testing** cho EShop SUT.

- MSSV: `23127364`
- Public repository: [SieuNhanGao889/hcmus_api_assignments](https://github.com/SieuNhanGao889/hcmus_api_assignments)
- Base URL khi kiểm thử: `http://localhost:3000`
- Công cụ: Postman, Newman, GitHub Actions và Codex/ChatGPT
- Báo cáo chính: [reports/main_report.md](reports/main_report.md)
- Demo video: [https://youtu.be/4fAWYhzeHzQ](https://youtu.be/4fAWYhzeHzQ)

## API đã chọn

| Pool | Chức năng | API |
|---|---|---|
| A | Login / FR-02 | `POST /api/login` |
| B | Apply Coupon / FR-09 | `POST /api/apply-coupon` |
| C | Admin Order Status / FR-18 và FR-10 | `PUT /api/admin/orders/:id/status` |

`NEEDS_HUMAN_CONFIRMATION`: Sinh viên cần xác nhận ba API này không trùng đúng bộ ba API của thành viên khác trong nhóm.

## Test summary

### Thiết kế và human audit

| API | AI-generated | Manual extension | Tổng case thiết kế | `VALID` | `INCOMPLETE` đã có correction | `INVALID` |
|---|---:|---:|---:|---:|---:|---:|
| Login | 40 | 5 | 45 | 38 | 2 | 0 |
| Apply Coupon | 40 | 5 | 45 | 36 | 4 | 0 |
| Admin Order Status | 42 | 5 | 47 | 41 | 1 | 0 |
| **Tổng** | **122** | **15** | **137** | **115** | **7** | **0** |

Nguồn: ba workbook gốc và ba workbook audited trong [test-cases/](test-cases/). Tất cả bảy case `INCOMPLETE` đều có `audit_notes` và `human_correction`; quyết định audit thuộc về sinh viên.

### Execution cuối sau automation correction

Một designed case có thể mở rộng thành nhiều `scenario_id`, nên số scenario execution lớn hơn số case thiết kế.

| API | Scenario rows executed | Passed | Failed | Assertions passed | Assertions failed | Confirmed bugs liên quan |
|---|---:|---:|---:|---:|---:|---:|
| Login | 46 | 35 | 11 | 142 | 11 | PB-01, PB-02, PB-03 |
| Apply Coupon | 47 | 43 | 4 | 833 | 4 | PB-04, PB-05, PB-06 |
| Admin Order Status (rerun) | 73 | 66 | 7 | 785 | 7 | PB-02, PB-07, PB-08 |
| **Tổng** | **166** | **144** | **22** | **1,760** | **22** | **8 unique bugs** |

Nguồn execution: [reports/newman/](reports/newman/) và [reports/analysis/newman_execution_analysis.md](reports/analysis/newman_execution_analysis.md). `PB-02` ảnh hưởng hai API nên tổng số ánh xạ theo API là 9 nhưng chỉ có 8 nguyên nhân lỗi độc lập.

## Điều hướng artifact

| Nhóm | Artifact chính |
|---|---|
| Assignment và API spec | [docs/2026.HW06.API Testing_En.md](docs/2026.HW06.API%20Testing_En.md), [docs/api_specification.md](docs/api_specification.md) |
| Test cases | [test-cases/](test-cases/), [test-cases/test_summary.md](test-cases/test_summary.md) |
| Postman | [postman/HW06_EShop_API_Tests.postman_collection.json](postman/HW06_EShop_API_Tests.postman_collection.json), [postman/HW06_Local.postman_environment.json](postman/HW06_Local.postman_environment.json), [postman/data/](postman/data/) |
| Newman evidence | [reports/newman/](reports/newman/) |
| Execution analysis | [reports/analysis/newman_execution_analysis.md](reports/analysis/newman_execution_analysis.md), [reports/analysis/order_status_automation_fix_review.md](reports/analysis/order_status_automation_fix_review.md) |
| Bug reports | [bug-reports/bugs_summary.md](bug-reports/bugs_summary.md), [bug-reports/PB-01.md](bug-reports/PB-01.md) đến [bug-reports/PB-08.md](bug-reports/PB-08.md) |
| CI/CD | [.github/workflows/api-tests.yml](.github/workflows/api-tests.yml), [reports/cicd_report.md](reports/cicd_report.md), [reports/screenshots/](reports/screenshots/) |
| Agent Skill | [agent-skill/SKILL.md](agent-skill/SKILL.md), [agent-skill/design/](agent-skill/design/), [demo video](https://youtu.be/4fAWYhzeHzQ) |
| AI documentation | [reports/ai_audit_report.md](reports/ai_audit_report.md), [reports/ai_critique.md](reports/ai_critique.md) |

## CI/CD evidence

- Successful run: [EShop API tests #1](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32447382046)
- Intentional failed run: [EShop API tests #2](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445604904)
- Screenshots: [ci_success.png](reports/screenshots/ci_success.png), [ci_intentional_failure.png](reports/screenshots/ci_intentional_failure.png)


## Chạy Newman theo từng API

Backend EShop phải chạy liên tục tại `http://localhost:3000` trong toàn bộ session.

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "01 - POST Login - Data Run" \
  --iteration-data postman/data/login_data.json

newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "02 - POST Apply Coupon - Data Run" \
  --iteration-data postman/data/coupon_data.json

newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  --environment postman/HW06_Local.postman_environment.json \
  --folder "03 - PUT Admin Order Status - Data Run" \
  --iteration-data postman/data/order_status_data.json
```

Các command trên là hướng dẫn tái hiện khớp collection hiện tại; kết quả đã nộp nằm trong `reports/newman/`.

## Tự đánh giá

| STT | Tiêu chí | Điểm tối đa | Điểm tự đánh giá |
|---|---|---:|---:|
| 1 | API 1 – full pipeline | 30 | `30` |
| 2 | API 2 – full pipeline | 30 | `30` |
| 3 | API 3 – full pipeline | 30 | `30` |
| 4 | Agent Skill – AI-driven test generator | 10 | `10` |
| | **Tổng** | **100** | **`100`** |
