# Báo Cáo AI Audit

## 1. Khai báo sử dụng AI

Trong bài tập này, em sử dụng AI như một công cụ hỗ trợ trong quá trình làm
bài. AI có thể được dùng để đọc nhanh tài liệu, tóm tắt ý chính, phân tích yêu
cầu, gợi ý hướng kiểm thử, hỗ trợ soạn nháp nội dung báo cáo và giúp rà soát
tính rõ ràng của tài liệu.

AI chỉ đóng vai trò hỗ trợ. AI không tự động phê duyệt hoặc hoàn tất test case
thay em. Mọi test case cuối cùng, nhãn audit, request Postman, kết quả Newman,
bug report, ảnh minh chứng, bằng chứng CI/CD và quyết định nộp bài đều do em
kiểm tra và chịu trách nhiệm.

## 2. Phạm vi hỗ trợ của AI

AI được sử dụng cho các công việc hỗ trợ sau:

| STT | Công việc hỗ trợ | Mục đích | Người quyết định cuối cùng |
|---|---|---|---|
| 1 | Đọc đề bài HW06 | Hiểu các yêu cầu cần nộp, tiêu chí chấm điểm và yêu cầu về AI audit | Sinh viên |
| 2 | Đọc tài liệu đặc tả API | Xác định nhanh endpoint đã chọn, request body, yêu cầu xác thực và phản hồi mong đợi | Sinh viên |
| 3 | Tóm tắt các API đã chọn | Nắm ý chính cho `POST /api/login`, `POST /api/apply-coupon` và `PUT /api/admin/orders/:id/status` | Sinh viên |
| 4 | Phân tích và gợi ý vùng bao phủ kiểm thử | Hỗ trợ suy nghĩ về domain, boundary, schema, security và state-based testing | Sinh viên |
| 5 | Hỗ trợ soạn nháp tài liệu | Gợi ý cách diễn đạt cho báo cáo, audit log, mô tả quy trình và phần giải thích | Sinh viên |
| 6 | Rà soát nội dung ở mức hỗ trợ | Phát hiện chỗ diễn đạt chưa rõ, thiếu liên kết với đặc tả hoặc cần kiểm tra lại | Sinh viên |

## 3. Giới hạn quan trọng

AI không được xem là nguồn quyết định cuối cùng. Cụ thể:

- AI không trực tiếp quyết định nội dung nộp cuối cùng nếu chưa có kiểm tra của
  sinh viên.
- AI không thay sinh viên gán nhãn test case là `VALID`, `INVALID` hoặc
  `INCOMPLETE`.
- AI không tạo bằng chứng thực thi thật.
- AI không tự tạo kết quả Newman, ảnh chụp màn hình, CI/CD run, GitHub Issue,
  timestamp hoặc xác nhận bug.
- Các kết quả do AI gợi ý chỉ được xem là tài liệu hỗ trợ. Tài liệu đề bài, tài
  liệu đặc tả API gốc và kết quả thực thi thật vẫn là nguồn tham chiếu chính
  thức.

## 4. Nhật ký tương tác với AI

| ID | Ngày giờ | Công cụ AI | Model | Giai đoạn | Tóm tắt prompt / yêu cầu | Tóm tắt output của AI | Tài liệu được tạo hoặc chỉnh sửa | Quyết định / chỉnh sửa của sinh viên |
|---|---|---|---|---|---|---|---|---|
| AI-001 | 2026-08-20 22:19:11 +07:00 | Codex / ChatGPT | GPT-5 based coding agent | Báo cáo / AI audit | Yêu cầu AI viết báo cáo audit theo hướng AI chỉ hỗ trợ, không tự điều chỉnh hoặc quyết định thay sinh viên. | AI soạn báo cáo AI Audit, mô tả phạm vi hỗ trợ và các giới hạn trách nhiệm của AI. | `reports/ai_audit_report.md` | Sinh viên yêu cầu viết lại phần audit theo hướng chung hơn để phù hợp nếu AI được dùng cho nhiều tác vụ hỗ trợ khác ngoài tóm tắt. |
| AI-002 | Không ghi riêng thời điểm | Gemini Pro 3.1 | Không ghi nhận | Đọc hiểu / phân tích tài liệu | Yêu cầu AI đọc, tóm tắt hoặc phân tích nhanh tài liệu bài tập/API trước khi làm chi tiết. | AI hỗ trợ xác định các yêu cầu chính của bài tập, phạm vi API đã chọn và các điểm cần kiểm tra lại. | Ghi chú đọc hiểu / hiểu biết làm việc | Sinh viên cần kiểm tra lại với tài liệu gốc trước khi nộp cuối cùng. |

## 5. Tóm tắt hiểu biết về các API đã chọn từ hỗ trợ của AI

| API | Mục đích chính được xác định trong quá trình hỗ trợ | Ghi chú cần sinh viên kiểm tra lại |
|---|---|---|
| `POST /api/login` | Xác thực người dùng bằng `email` và `password`; phản hồi thành công trả về JWT token và thông tin user. | Cần kiểm tra các trường hợp credential hợp lệ/không hợp lệ, response schema và rò rỉ thông tin nhạy cảm. |
| `POST /api/apply-coupon` | Tính toán giảm giá dựa trên `code`, `total_amount` và `user_id`; phản hồi gồm `discount_amount` và `final_amount`. | Cần kiểm tra dữ liệu coupon, boundary của số tiền, assertion tính toán và hành vi với input không hợp lệ. |
| `PUT /api/admin/orders/:id/status` | Cho phép admin cập nhật trạng thái đơn hàng bằng trường `status`; các trạng thái gồm `pending`, `confirmed`, `shipping`, `delivered` và `canceled`. | Cần kiểm tra quyền admin, truy cập bằng role không hợp lệ, order ID không hợp lệ và các giả định về chuyển trạng thái. |

## 6. Tuyên bố trách nhiệm

AI được sử dụng để hỗ trợ quá trình đọc hiểu, phân tích, gợi ý và soạn thảo.
AI không thay thế việc kiểm tra của sinh viên. Em chịu trách nhiệm kiểm tra tài
liệu gốc, xác thực các ý tưởng được gợi ý, tự đưa ra quyết định cuối cùng, thực
thi test và chỉ báo cáo các bằng chứng có thật.
