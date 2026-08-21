# Bản nháp GitHub Issue: Coupon từ chối tổng tiền bằng min_order_amount


**Mã bug:** PB-04  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** Medium  
**Endpoint:** `POST /api/apply-coupon`

## Tóm tắt

FR-09 C3 quy định điều kiện `total_amount >= min_order_amount`, nhưng trường hợp bằng nhau lại bị từ chối.

## Các bước tái hiện

1. Dùng coupon đang hoạt động và đủ điều kiện với `min_order_amount=200000`.
2. Apply coupon bằng authorization hợp lệ, user ID dạng scalar và `total_amount=200000`.

## Kết quả mong đợi

Coupon được áp dụng và trả các trường giảm giá nhất quán.

## Kết quả thực tế

Phản hồi là `400` và cho rằng order chưa đạt mức tối thiểu `200000`.

## Test và bằng chứng liên quan

`COUPON-GEN-006`, `COUPON-EXT-002-EQUAL`; `reports/newman/coupon_run.json`, các vòng lặp `5,42`. Xem `bug-reports/PB-04.md`.

## Tác động

Khách hàng đủ điều kiện bị từ chối giảm giá đúng tại giá trị ngưỡng.
