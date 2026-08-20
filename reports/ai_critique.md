# AI Critique

AI hữu ích nhất ở giai đoạn đọc nhanh đặc tả và mở rộng không gian kiểm thử. Với ba API đã chọn, AI giúp tách tham số, schema, authentication, authorization, business rule và các vùng mơ hồ trong spec. Nhờ đó bộ test ban đầu có độ phủ rộng hơn so với việc chỉ viết happy path và vài negative cases thủ công.

Tuy nhiên, AI cũng có giới hạn rõ. Khi đặc tả không nêu status code lỗi, transition matrix, JWT claims, discount rounding hoặc usage tracking, AI có xu hướng tạo case exploratory hoặc đôi lúc ghép nhiều mục tiêu vào một case. Ví dụ các case `INCOMPLETE` ở Login, Coupon và Admin Order Status đều liên quan đến rule chưa đủ rõ hoặc thiếu fixture/state để kiểm chứng. Vì vậy human audit là bắt buộc để không biến giả định của AI thành expected result chính thức.

Manual extension của sinh viên đã bù các điểm AI dễ bỏ sót: cross-request token usability, duplicate JSON keys, discount formula với fixture kiểm soát, persisted-state verification sau failed authorization và transition coverage có dữ liệu chuẩn bị trước. Kết luận của em là AI phù hợp để tăng breadth và tạo bản nháp có cấu trúc, nhưng phần quyết định cuối, thiết kế case nghiệp vụ sâu và phân loại bug vẫn phải do sinh viên thực hiện dựa trên spec và evidence thật.
