# Bản nháp GitHub Issue: `POST /api/login` làm lộ mật khẩu dạng rõ và các trường tài khoản nhạy cảm

> Chỉ là bản nháp — chưa tạo GitHub Issue.

**Mã bug:** PB-01  
**Trạng thái:** CONFIRMED_BY_HUMAN_REVIEW  
**Mức độ nghiêm trọng:** High  
**Endpoint:** `POST /api/login`

## Tóm tắt

Phản hồi đăng nhập thành công chứa mật khẩu dạng rõ đã gửi trong `user.password` và làm lộ các trường trạng thái xác thực không cần thiết.

## Các bước tái hiện

1. Đăng ký một user thường dùng một lần.
2. Đăng nhập bằng credential hợp lệ.
3. Kiểm tra object `user` trong phản hồi `200`.

## Kết quả mong đợi

Chỉ trả token và các trường hồ sơ user cần thiết; không trả password, reset token hoặc secret xác thực.

## Kết quả thực tế

`user.password` bằng mật khẩu dạng rõ đã gửi; các trường `reset_token`, `login_attempts` và `locked_until` cũng được trả về.

## Test và bằng chứng liên quan

`LOGIN-GEN-001/002/003/004/032/033/038`, `LOGIN-EXT-003`; `reports/newman/login_run.json`, các vòng lặp `0,1,2,3,31,32,37,43`. Xem `bug-reports/PB-01.md`.

## Tác động

Credential và trạng thái xác thực nội bộ có thể bị lộ qua client, log, proxy và hệ thống giám sát.
