# Bản nháp GitHub Issue: Lỗi request body làm lộ stack trace và đường dẫn tuyệt đối trên server


**Mã bug:** PB-02  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** High  
**Endpoint:** `POST /api/login`, `PUT /api/admin/orders/:id/status`

## Tóm tắt

JSON malformed và Content-Type không được hỗ trợ làm server trả trang HTML exception chứa stack trace, đường dẫn tuyệt đối, dòng source và thông tin nội bộ framework.

## Các bước tái hiện

1. Gửi JSON bị cắt dở tới Login.
2. Gửi JSON bị cắt dở tới Admin Order Status bằng admin token hợp lệ.
3. Gửi text có hình thức JSON tới Order Status với `Content-Type: text/plain`.
4. Kiểm tra response body.

## Kết quả mong đợi

Trả lỗi client được kiểm soát, không chứa chi tiết triển khai nội bộ.

## Kết quả thực tế

Phản hồi chứa `SyntaxError`/`TypeError`, đường dẫn workspace tuyệt đối, `server.js:526` và stack frame của dependency; Content-Type sai trả `500`.

## Test và bằng chứng liên quan

`LOGIN-GEN-036`, `LOGIN-EXT-005`, `ORDERSTATUS-GEN-033/035`; các báo cáo Newman gốc và `reports/newman/order_status_rerun_after_automation_fix.json`, các vòng lặp `32,34`. Xem `bug-reports/PB-02.md`.

## Tác động

Việc lộ thông tin giúp fingerprint server/framework và cho thấy xử lý lỗi không an toàn.
