# Bản nháp GitHub Issue: Tài khoản bị khóa sau hai lần đăng nhập sai thay vì ba lần


**Mã bug:** PB-03  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** Medium  
**Endpoint:** `POST /api/login`

## Tóm tắt

FR-02 yêu cầu khóa sau ba lần sai liên tiếp, nhưng mật khẩu đúng bị từ chối vì tài khoản đã khóa ngay sau hai lần sai.

## Các bước tái hiện

1. Tạo user mới dùng một lần.
2. Đăng nhập sai mật khẩu hai lần.
3. Đăng nhập ngay bằng mật khẩu đúng.

## Kết quả mong đợi

Đăng nhập đúng phải thành công sau hai lần sai; lockout chỉ bắt đầu ở lần sai thứ ba.

## Kết quả thực tế

Request thứ ba dùng mật khẩu đúng trả `403`, thông báo tài khoản bị khóa và không có token.

## Test và bằng chứng liên quan

`LOGIN-GEN-040-A`; `reports/newman/login_run.json`, vòng lặp `39`; assertion thất bại: `[LOGIN-GEN-040-A][FR-02] correct login succeeds after two failures`. Xem `bug-reports/PB-03.md`.

## Tác động

User hợp lệ có thể bị khóa sớm, làm tăng rủi ro gián đoạn tài khoản và denial-of-service.
