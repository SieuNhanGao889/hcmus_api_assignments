# Bản nháp GitHub Issue: Apply-coupon chấp nhận user_id dạng object và vẫn cấp giảm giá


**Mã bug:** PB-05  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** High  
**Endpoint:** `POST /api/apply-coupon`

## Tóm tắt

Endpoint chấp nhận `user_id` dạng object thay vì định danh scalar nhưng vẫn trả kết quả giảm giá thành công và hợp lệ.

## Các bước tái hiện

1. Xác thực một user dùng một lần.
2. Apply một fixed coupon đủ điều kiện.
3. Gửi `"user_id":{"$gt":0}` thay cho ID scalar.

## Kết quả mong đợi

Từ chối kiểu định danh không hợp lệ và không cấp giảm giá.

## Kết quả thực tế

Phản hồi là `200`, `success=true`, `discount_amount=50000`, `final_amount=450000`.

## Test và bằng chứng liên quan

`COUPON-GEN-025`; `reports/newman/coupon_run.json`, vòng lặp `24`. Xem `bug-reports/PB-05.md`.

## Tác động

Nhầm lẫn kiểu định danh có thể phá vỡ điều kiện coupon theo user và việc giới hạn số lần sử dụng.

## Ghi chú audit

AI ban đầu ghi độ tin cậy Medium; người duyệt xác nhận riêng hành vi quan sát được để lập báo cáo lỗi. Phân loại lịch sử của AI vẫn được giữ nguyên.
