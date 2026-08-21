# AI Critique

AI hữu ích nhất khi đọc nhanh đặc tả và mở rộng không gian kiểm thử. Với ba API đã chọn, AI giúp hệ thống hóa input validation, boundary, schema, authentication, authorization, business rule và các vùng chưa được mô tả rõ. Nhờ đó bộ candidate cases bao phủ rộng hơn cách chỉ viết happy path và một số negative cases thủ công.

Tuy nhiên, AI không phải test oracle đáng tin cậy. Khi specification thiếu error status, transition matrix, JWT behavior, coupon formula hoặc usage tracking, AI có xu hướng suy diễn expected behavior hoặc ghép nhiều mục tiêu vào một case. Human audit đã phát hiện bảy case `INCOMPLETE`, giữ lại assumption lịch sử và bổ sung correction có căn cứ. Manual extensions cũng bù các điểm AI dễ bỏ sót như duplicate JSON keys, cross-request token usability, controlled-fixture discount formula, persisted-state verification và systematic state transitions.

Execution cho thấy assertion failure cũng không tự động đồng nghĩa với SUT bug. AI phân tích lần chạy Newman ban đầu thành tám potential bug root causes và hai automation defects. Sinh viên đối chiếu requirement, xác nhận PB-01 đến PB-08, phê duyệt correction AF-01/AF-02 rồi tự rerun. Việc giữ cả report trước và sau correction giúp phân biệt lỗi sản phẩm với lỗi test automation.

Bài học quan trọng là cộng tác theo vòng lặp `generate -> audit -> execute -> analyze -> human decision -> correct`. AI tăng breadth, traceability và tốc độ tạo artifact; con người phải quyết định test oracle, requirement mapping, bug classification và chịu trách nhiệm với evidence cuối. Giá trị của human review không phải phê duyệt hình thức, mà là ngăn assumption của AI trở thành requirement giả.
