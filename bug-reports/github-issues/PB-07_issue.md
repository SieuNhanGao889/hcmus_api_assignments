# Bản nháp GitHub Issue: JWT của user thường có thể cập nhật trạng thái qua endpoint admin order


**Mã bug:** PB-07  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** Critical  
**Endpoint:** `PUT /api/admin/orders/:id/status`

## Tóm tắt

Endpoint admin kiểm tra sự tồn tại của token nhưng không enforce admin role; user thường có thể thay đổi và lưu trạng thái order.

## Các bước tái hiện

1. Xác thực bằng user thường.
2. Tạo order mới ở trạng thái `pending`.
3. Gọi endpoint admin bằng token user thường và body `{"status":"confirmed"}`.
4. Đọc lại trạng thái order.

## Kết quả mong đợi

Từ chối caller không phải admin; order giữ `pending`.

## Kết quả thực tế

Phản hồi là `200`, “Order status updated”; persisted status chuyển thành `confirmed`.

## Test và bằng chứng liên quan

`ORDERSTATUS-GEN-010/011/031`, `ORDERSTATUS-EXT-001`; báo cáo Order Status ban đầu và lần chạy lại sau khi sửa tại các vòng lặp `9,10,30,42`. Xem `bug-reports/PB-07.md`.

## Tác động

Bất kỳ user đã xác thực nào cũng có thể thực hiện thay đổi workflow dành riêng cho admin, làm mất tính toàn vẹn của authorization, fulfillment và audit.
