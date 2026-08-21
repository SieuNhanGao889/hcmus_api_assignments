# Bản nháp GitHub Issue: Coupon phần trăm trả mức giảm âm và làm tăng final amount


**Mã bug:** PB-06  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** High  
**Endpoint:** `POST /api/apply-coupon`

## Tóm tắt

Coupon `10%` được kiểm soát trên tổng `500000` trả mức giảm âm và làm final amount tăng gấp mười lần.

## Các bước tái hiện

1. Dùng percent coupon đủ điều kiện với `discount_value=10`.
2. Apply coupon cho `total_amount=500000` bằng authorization và user ID hợp lệ.

## Kết quả mong đợi

`discount_amount=50000`; `final_amount=450000`.

## Kết quả thực tế

`discount_amount=-4500000`; `final_amount=5000000`; phản hồi vẫn báo thành công.

## Test và bằng chứng liên quan

`COUPON-EXT-001`; `reports/newman/coupon_run.json`, vòng lặp `40`; assertion công thức phần trăm thất bại. Xem `bug-reports/PB-06.md`.

## Tác động

Giá order và dữ liệu kế toán downstream có thể bị sai lệch nghiêm trọng.
