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
| 6 | Hỗ trợ rà soát nội dung | AI có thể chỉ ra các điểm cần kiểm tra; sinh viên tự đối chiếu specification và quyết định có chỉnh sửa hay không | Sinh viên |

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

## 4. Nhật ký tương tác với AI

| ID | Ngày giờ | Công cụ AI | Model | Giai đoạn | Prompt / yêu cầu | Output của AI | Artifact | Quyết định / chỉnh sửa của sinh viên |
|---|---|---|---|---|---|---|---|---|
| AI-001 | 2026-08-20 22:19:11 +07:00 | Codex / ChatGPT | GPT-5 based coding agent | Chuẩn bị AI Audit | Yêu cầu AI tạo cấu trúc báo cáo AI Audit để ghi nhận việc sử dụng AI trong HW06. | AI tạo cấu trúc ban đầu cho báo cáo AI Audit, bao gồm phạm vi sử dụng, giới hạn trách nhiệm và bảng nhật ký tương tác. | `reports/ai_audit_report.md` | Sinh viên yêu cầu điều chỉnh báo cáo theo hướng phản ánh AI là công cụ hỗ trợ trong nhiều giai đoạn, không chỉ dùng để tóm tắt tài liệu. |
| AI-002 | Không ghi riêng thời điểm | Gemini Pro 3.1 | Không ghi nhận | Đọc hiểu tài liệu | Yêu cầu AI hỗ trợ đọc và phân tích nhanh đề bài HW06 và API specification. | AI hỗ trợ xác định yêu cầu chính của bài, phạm vi ba API được chọn và các deliverable cần chuẩn bị. | Ghi chú làm việc | Sinh viên đối chiếu lại với đề bài và API specification trước khi chốt phạm vi thực hiện. |
| AI-003 | 2026-08-20 | Codex / ChatGPT | GPT-5 based coding agent | PHASE_1_SPEC_ANALYSIS – Login | Yêu cầu phân tích `POST /api/login` theo `AGENT.md`, chưa tạo test case. | AI phân tích endpoint, request/response, requirements và các hành vi chưa được specification định nghĩa. | `reports/analysis/login_spec_analysis.md` | Sinh viên đã review và chấp nhận kết quả; không cần chỉnh sửa. |
| AI-004 | 2026-08-20 | Codex / ChatGPT | GPT-5 based coding agent | PHASE_2_TEST_DESIGN – Login | Yêu cầu thiết kế các vùng kiểm thử cho `POST /api/login` từ specification analysis đã được review, chưa generate test case. | AI đề xuất domain partitions, boundary/negative cases, security, schema và dependency coverage. | `reports/analysis/login_test_design.md` | Sinh viên review và chỉnh sửa 3 điểm trước khi approve: (1) chuyển password case-sensitivity thành exploratory vì spec không định nghĩa behavior; (2) đổi `role/isAdmin` từ “Role Escalation” thành “Unexpected security-sensitive fields”; (3) chuyển giả định `user` chắc chắn là object thành `SPEC_UNDEFINED` |
| AI-005 | 2026-08-20 | Codex / ChatGPT | GPT-5 based coding agent | `PHASE_3_AI_GENERATION` – Login | Yêu cầu AI generate candidate test cases cho `POST /api/login` theo test design đã được human-reviewed và `SKILL.md`. | AI tạo 40 candidate test cases; tất cả có `source = AI_GENERATED`, `audit_status = PENDING_HUMAN_REVIEW` và chưa có execution result. | `test-cases/login_test_cases.xlsx` | Sinh viên tự review toàn bộ 40 test case: **38 VALID, 2 INCOMPLETE, 0 INVALID**. `LOGIN-GEN-002` được đánh dấu `INCOMPLETE` vì assertion JWT phải có 3 phần vượt quá API contract; quyết định giữ kiểm tra token tồn tại/non-empty và hạ cấu trúc JWT thành soft-check. `LOGIN-GEN-040` được đánh dấu `INCOMPLETE` vì account lockout/rate-limit không có threshold, thời gian khóa hoặc response cụ thể trong `api_specification.md`; quyết định giữ dưới dạng exploratory/security test và không dùng để kết luận bug nếu thiếu requirement/evidence bổ sung. |
| AI-006 | 2026-08-20 | Codex / ChatGPT | GPT-5 based coding agent | `PHASE_4_HUMAN_AUDIT` – Login | Yêu cầu AI xử lý kết quả human audit đã hoàn thành trong `login_test_cases_audited.xlsx`, giữ nguyên quyết định audit của sinh viên và chỉ ghi correction cho các case cần sửa. | AI thêm cột `human_correction` và ghi correction cho 2 case `INCOMPLETE` là `LOGIN-GEN-002` và `LOGIN-GEN-040`; giữ nguyên 40 test case, nội dung AI-generated, `audit_status` và `audit_notes`. | `test-cases/login_test_cases_audited.xlsx` | Sinh viên giữ nguyên kết quả human audit: **38 VALID, 2 INCOMPLETE, 0 INVALID**. AI chỉ hỗ trợ ghi lại correction dựa trên quyết định audit đã có, không thay đổi quyết định của sinh viên. |

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
