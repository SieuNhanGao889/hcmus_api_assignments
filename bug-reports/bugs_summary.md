# Tổng hợp lỗi đã xác nhận

## 1. Phạm vi và trạng thái

Tài liệu này tổng hợp tám lỗi độc lập `PB-01` đến `PB-08` từ kết quả chạy Newman thật, phân tích nguyên nhân gốc và quyết định duyệt của người thực hiện. Nhiều assertion thất bại do cùng một hành vi chỉ được tính là một lỗi.

Tất cả tám lỗi có trạng thái:

`CONFIRMED_BY_HUMAN_REVIEW`

Các GitHub Issue `#1` đến `#8` đã được tạo trong repository `SieuNhanGao889/hcmus_api_assignments` và đang ở trạng thái `OPEN` tại thời điểm đối chiếu ngày 2026-08-21. Ảnh tổng thể danh sách issue: [githubissues.png](screenshots/githubissues.png).

## 2. Thống kê tổng quan

| Chỉ số | Giá trị |
|---|---:|
| Tổng số lỗi đã xác nhận | `8` |
| `CRITICAL` | `1` |
| `HIGH` | `5` |
| `MEDIUM` | `2` |
| GitHub Issue đã tạo | `8` |
| GitHub Issue đang mở | `8` |
| Ảnh bằng chứng riêng cho từng lỗi | `8` |

Phân bố theo API bị ảnh hưởng:

| API | Bug liên quan | Số lượng |
|---|---|---:|
| `POST /api/login` | `PB-01`, `PB-02`, `PB-03` | `3` |
| `POST /api/apply-coupon` | `PB-04`, `PB-05`, `PB-06` | `3` |
| `PUT /api/admin/orders/:id/status` | `PB-02`, `PB-07`, `PB-08` | `3` |

`PB-02` ảnh hưởng đồng thời Login và Admin Order Status nên tổng số ánh xạ API lớn hơn tổng số nguyên nhân gốc.

## 3. Danh sách lỗi đã xác nhận

| Bug ID | Tóm tắt | Mức độ | Endpoint | `case_id`/`scenario_id` chính | Yêu cầu/nguồn | GitHub Issue | Bằng chứng ảnh |
|---|---|---|---|---|---|---|---|
| [`PB-01`](PB-01.md) | Login làm lộ mật khẩu dạng rõ và các trường tài khoản nhạy cảm | `HIGH` | `POST /api/login` | `LOGIN-GEN-001/002/003/004/032/033/038`, `LOGIN-EXT-003` | API specification Authentication 1.2; yêu cầu không làm lộ dữ liệu nhạy cảm | [#1](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/1) | [PB-01.png](screenshots/PB-01.png) |
| [`PB-02`](PB-02.md) | Xử lý lỗi làm lộ stack trace, đường dẫn tuyệt đối và thông tin framework | `HIGH` | `POST /api/login`; `PUT /api/admin/orders/:id/status` | `LOGIN-GEN-036`, `LOGIN-EXT-005`, `ORDERSTATUS-GEN-033/035` | `NO_INTERNAL_LEAKAGE`; định nghĩa request body của hai API | [#2](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/2) | [PB-02.png](screenshots/PB-02.png) |
| [`PB-03`](PB-03.md) | Tài khoản bị khóa sau hai lần đăng nhập sai thay vì ba lần | `MEDIUM` | `POST /api/login` | `LOGIN-GEN-040` / `LOGIN-GEN-040-A` | `FR-02` | [#3](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/3) | [PB-03.png](screenshots/PB-03.png) |
| [`PB-04`](PB-04.md) | Coupon từ chối khi `total_amount = min_order_amount` | `MEDIUM` | `POST /api/apply-coupon` | `COUPON-GEN-006`, `COUPON-EXT-002-EQUAL` | `FR-09 C3` | [#4](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/4) | [PB-04.png](screenshots/PB-04.png) |
| [`PB-05`](PB-05.md) | API chấp nhận `user_id` dạng object và vẫn cấp giảm giá hợp lệ | `HIGH` | `POST /api/apply-coupon` | `COUPON-GEN-025` | API specification Coupons 5.1; kiểm tra type confusion | [#5](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/5) | [PB-05.png](screenshots/PB-05.png) |
| [`PB-06`](PB-06.md) | Coupon phần trăm tạo mức giảm âm và làm tăng `final_amount` | `HIGH` | `POST /api/apply-coupon` | `COUPON-EXT-001` | `FR-09` | [#6](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/6) | [PB-06.png](screenshots/PB-06.png) |
| [`PB-07`](PB-07.md) | JWT của user thường có thể cập nhật trạng thái qua endpoint admin | `CRITICAL` | `PUT /api/admin/orders/:id/status` | `ORDERSTATUS-GEN-010/011/031`, `ORDERSTATUS-EXT-001` | `FR-12`, `SEC-02`, `SEC-03` | [#7](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/7) | [PB-07.png](screenshots/PB-07.png) |
| [`PB-08`](PB-08.md) | Order đã `canceled` vẫn chuyển được sang `delivered` | `HIGH` | `PUT /api/admin/orders/:id/status` | `ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED` | `FR-10` | [#8](https://github.com/SieuNhanGao889/hcmus_api_assignments/issues/8) | [PB-08.png](screenshots/PB-08.png) |

## 4. Kết quả thực tế nổi bật

- `PB-01`: response login `200` chứa `user.password`, `reset_token`, `login_attempts` và `locked_until`.
- `PB-02`: JSON malformed trả HTML có `SyntaxError`, stack frame và đường dẫn tuyệt đối; wrong `Content-Type` trên Order Status trả `500` kèm `TypeError` và thông tin source/framework.
- `PB-03`: đăng nhập bằng mật khẩu đúng sau hai lần sai trả `403` với thông báo tài khoản đã bị khóa.
- `PB-04`: `total_amount=200000` bằng đúng `min_order_amount=200000` nhưng API trả `400` và từ chối coupon.
- `PB-05`: request có `"user_id":{"$gt":0}` vẫn trả `200`, `success=true` và mức giảm giá hợp lệ.
- `PB-06`: coupon `10%` trên tổng `500000` trả `discount_amount=-4500000` và `final_amount=5000000`.
- `PB-07`: normal-user JWT gọi endpoint admin nhận `200`; trạng thái order được lưu từ `pending` thành `confirmed`.
- `PB-08`: API chấp nhận chuyển trạng thái `canceled -> delivered` và lưu trạng thái `delivered`.

## 5. Bằng chứng thực thi

Bằng chứng chính:

- `reports/newman/login_run.json`
- `reports/newman/coupon_run.json`
- `reports/newman/order_status_run.json`
- `reports/newman/order_status_rerun_after_automation_fix.json`
- `reports/analysis/newman_execution_analysis.md`
- `reports/analysis/order_status_automation_fix_review.md`

Bằng chứng bổ sung:

- Báo cáo chi tiết: `bug-reports/PB-01.md` đến `bug-reports/PB-08.md`.
- Ảnh thực thi: `bug-reports/screenshots/PB-01.png` đến `bug-reports/screenshots/PB-08.png`.
- Ảnh tổng thể GitHub Issue: `bug-reports/screenshots/githubissues.png`.
- Bản nội dung issue đã soạn: `bug-reports/github-issues/PB-01_issue.md` đến `PB-08_issue.md`.

Hai lỗi automation `AF-01` và `AF-02` không được tính là bug của SUT. Sau khi sửa assertion mode và chạy lại Order Status, các false failure liên quan đã biến mất; bằng chứng cho `PB-02`, `PB-07` và `PB-08` vẫn tái hiện.

## 6. Bảo toàn lịch sử đánh giá

- Phân tích AI ban đầu chỉ phân loại các phát hiện là `POTENTIAL_SUT_BUG`; quyết định xác nhận cuối cùng thuộc về người duyệt.
- Riêng `PB-05`, phân loại lịch sử của AI vẫn là `POTENTIAL_SUT_BUG` với confidence `Medium`. Quyết định của người duyệt được lưu riêng và xác nhận bug cho mục đích báo cáo; lịch sử AI không bị viết lại.
- Các báo cáo Newman gốc và báo cáo chạy lại sau khi sửa automation đều được giữ nguyên.
- Nhiều assertion cùng phản ánh một nguyên nhân gốc không bị tính thành nhiều bug riêng.

## 7. Điểm cần rà soát trước khi nộp

1. Mã requirement trong tiêu đề các issue `#1`, `#2`, `#7` và `#8` nên được đối chiếu lại với báo cáo chi tiết: PB-01/PB-02 không chỉ được xác lập từ `FR-02`, PB-07 ánh xạ `FR-12` và `SEC-02/SEC-03`, PB-08 ánh xạ `FR-10`.
2. Đề bài không bắt buộc che password hoặc JWT trong ảnh bằng chứng. Nếu các giá trị này chỉ thuộc môi trường test cục bộ và dùng một lần thì có thể giữ nguyên; che dữ liệu chỉ là khuyến nghị bảo mật tùy chọn khi chia sẻ repository công khai.
