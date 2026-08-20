# CI/CD Report

## Trạng thái hiện tại

Chưa thực hiện `PHASE_9_CICD`. Repo chưa có GitHub Actions workflow thật để chạy Newman và chưa có bằng chứng pipeline pass/fail.

Vì vậy, báo cáo này không claim:

- Pipeline đã chạy thành công.
- Pipeline đã chạy fail có chủ đích.
- Có screenshot hoặc link GitHub Actions thật.
- Có commit minh chứng cho passing/failing run.

## Thiết kế CI/CD dự kiến

Khi chuyển sang Phase 9, workflow nên thực hiện:

1. Checkout repository.
2. Setup runtime cần thiết cho SUT.
3. Khởi động hoặc kết nối tới SUT tại `http://localhost:3000`.
4. Chờ health check hoặc endpoint sẵn sàng.
5. Cài Newman.
6. Chạy Postman collection với environment local.
7. Xuất CLI/HTML report.
8. Upload Newman report như artifact.
9. Fail pipeline nếu assertion fail.

Lệnh Newman dự kiến:

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  -e postman/HW06_Local.postman_environment.json \
  -r cli,html \
  --reporter-html-export reports/newman/HW06_EShop_API_Tests.html
```

## Evidence còn thiếu

| Evidence | Trạng thái |
|---|---|
| `.github/workflows/api-tests.yml` | Chưa có |
| Passing pipeline run | Chưa có |
| Intentional failing pipeline run | Chưa có |
| Screenshot passing run | Chưa có |
| Screenshot failing run | Chưa có |
| Commit link cho passing/failing run | Chưa có |

Phần này cần được cập nhật sau khi Postman/Newman test suite được triển khai và chạy thật.
