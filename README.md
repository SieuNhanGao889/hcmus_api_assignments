# 23127364_HW06_AI_API_100

## Tổng quan dự án

Repository này chứa các tài liệu nộp bài cho **HW06 - API Testing** trên hệ
thống EShop SUT.

- MSSV: `23127364`
- Base URL: `http://localhost:3000`
- Tài liệu đặc tả API: `docs/api_specification.md`
- Công cụ kiểm thử: Postman + Newman
- Header bắt buộc khi thực thi: `X-Student-Id: 23127364`

## API đã chọn

| Pool | Chức năng | API đã chọn |
|---|---|---|
| Pool A | Login | `POST /api/login` |
| Pool B | Discount coupon / mã giảm giá | `POST /api/apply-coupon` |
| Pool C | Admin order status / cập nhật trạng thái đơn hàng | `PUT /api/admin/orders/:id/status` |

Bộ ba API này được chọn để tránh trùng với thành viên khác trong nhóm.

## Khai báo sử dụng AI

Trong bài tập này, AI được sử dụng như công cụ hỗ trợ trong quá trình làm bài.
AI có thể hỗ trợ đọc nhanh tài liệu, tóm tắt ý chính, phân tích yêu cầu, gợi ý
hướng kiểm thử, hỗ trợ soạn nháp báo cáo và rà soát nội dung ở mức tham khảo.

AI không tự động phê duyệt hoặc quyết định nội dung nộp cuối cùng. Mọi test
case, kết quả thực thi, bug report, screenshot và bằng chứng CI/CD đều do sinh
viên tự kiểm tra và chịu trách nhiệm. Chi tiết được ghi trong
`reports/ai_audit_report.md`.

## Cấu trúc thư mục

```text
23127364_HW06_AI_API_100/
|-- README.md
|-- docs/
|   |-- 2026.HW06.API Testing_En.md
|   |-- 2026.HW06.API Testing_En.pdf
|   `-- api_specification.md
|-- test-cases/
|   |-- login_test_cases.xlsx
|   |-- coupon_test_cases.xlsx
|   |-- admin_order_status_test_cases.xlsx
|   `-- test_summary.md
|-- postman/
|   |-- HW06_EShop_API_Tests.postman_collection.json
|   |-- HW06_Local.postman_environment.json
|   `-- data/
|       |-- login_data.json
|       |-- coupon_data.json
|       `-- order_status_data.json
|-- reports/
|   |-- main_report.md
|   |-- main_report.pdf
|   |-- ai_audit_report.md
|   |-- ai_audit_report.pdf
|   |-- ai_critique.md
|   |-- cicd_report.md
|   `-- newman/
|       `-- HW06_EShop_API_Tests.html
|-- bug-reports/
|   |-- bugs_summary.md
|   `-- screenshots/
|-- agent-skill/
|   |-- SKILL.md
|   |-- design/
|   |   |-- design.md
|   |   |-- pseudocode.md
|   |   `-- diagram.png
|   |-- scripts/
|   |   `-- generate_api_tests.py
|   `-- demo/
|       `-- demo_notes.md
|-- evidence/
|   |-- postman/
|   |-- newman/
|   |-- bugs/
|   |-- cicd/
|   |   |-- passing-run/
|   |   `-- failing-run/
|   `-- ai/
|-- .github/
|   `-- workflows/
|       `-- api-tests.yml
`-- git-log/
    `-- commit_log.txt
```

## Chạy Newman

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  -e postman/HW06_Local.postman_environment.json \
  -r cli,html \
  --reporter-html-export reports/newman/HW06_EShop_API_Tests.html
```

## Danh sách tài liệu nộp

- Báo cáo chính: Markdown và PDF
- Báo cáo AI audit: Markdown và PDF
- AI critique: 200-300 từ
- File Excel test case và test summary
- Postman collection, environment và data files
- Newman HTML report
- Bug report và screenshot minh chứng
- Báo cáo CI/CD và bằng chứng cho một lần chạy pass, một lần chạy fail
- AI-driven Agent Skill, generator script, design, diagram, pseudocode và demo notes
- Git commit log

## Tự đánh giá

| STT | Tiêu chí | Điểm | Điểm tự đánh giá |
|---|---:|---:|---:|
| 1 | API1 - đầy đủ pipeline: generate, audit, extend, execute, bugs | 30 | |
| 2 | API2 - đầy đủ pipeline: generate, audit, extend, execute, bugs | 30 | |
| 3 | API3 - đầy đủ pipeline: generate, audit, extend, execute, bugs | 30 | |
| 4 | Agent Skill - AI-driven test generator | 10 | |
| | Tổng | 100 | |
