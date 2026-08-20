# 23127364_HW06_AI_API_100

## Project Overview

This repository contains the deliverables for **HW06 - API Testing** using the EShop SUT.

- Student ID: `23127364`
- Base URL: `http://localhost:3000`
- API specification: `docs/api_specification.md`
- Testing tool: Postman + Newman
- Required execution header: `X-Student-Id: 23127364`

## Selected APIs

| Pool | Feature | Selected API |
|---|---|---|
| Pool A | Login | `POST /api/login` |
| Pool B | Discount coupon | `POST /api/apply-coupon` |
| Pool C | Admin order status | `PUT /api/admin/orders/:id/status` |

The selected three-API set should not be duplicated by another group member.

## Folder Structure

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

## Run Newman

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  -e postman/HW06_Local.postman_environment.json \
  -r cli,html \
  --reporter-html-export reports/newman/HW06_EShop_API_Tests.html
```

## Submission Checklist

- Main report: Markdown and PDF
- AI audit report: Markdown and PDF
- AI critique: 200-300 words
- Excel test cases and test summary
- Postman collection, environment, and data files
- Newman HTML report
- Bug reports and screenshots
- CI/CD report and evidence for one passing run and one failing run
- AI-driven Agent Skill, generator script, design, diagram, pseudocode, and demo notes
- Git commit log

## Self-Assessment

| No. | Criteria | Grade | Self-Assessed Grade |
|---|---:|---:|---:|
| 1 | API1 - full pipeline: generate, audit, extend, execute, bugs | 30 | |
| 2 | API2 - full pipeline: generate, audit, extend, execute, bugs | 30 | |
| 3 | API3 - full pipeline: generate, audit, extend, execute, bugs | 30 | |
| 4 | Agent Skill - AI-driven test generator | 10 | |
| | Total | 100 | |
