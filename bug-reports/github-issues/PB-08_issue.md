# Bản nháp GitHub Issue: Order đã canceled vẫn có thể chuyển sang delivered


**Mã bug:** PB-08  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** High  
**Endpoint:** `PUT /api/admin/orders/:id/status`

## Tóm tắt

FR-10 quy định `canceled` là trạng thái kết thúc, nhưng API chấp nhận `canceled -> delivered` và lưu thay đổi.

## Các bước tái hiện

1. Tạo order và chuyển sang `canceled`.
2. Xác minh source state đã được lưu.
3. Dùng admin token hợp lệ gửi body `{"status":"delivered"}`.
4. Đọc lại order.

## Kết quả mong đợi

Từ chối transition; trạng thái giữ `canceled`.

## Kết quả thực tế

Phản hồi là `200`, “Order status updated”; trạng thái được lưu thành `delivered`.

## Test và bằng chứng liên quan

`ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED`; báo cáo Order Status ban đầu và lần chạy lại sau khi sửa tại vòng lặp `67`. Xem `bug-reports/PB-08.md`.

## Tác động

Tính toàn vẹn của terminal state bị phá vỡ, làm sai lịch sử fulfillment, báo cáo và audit record.
