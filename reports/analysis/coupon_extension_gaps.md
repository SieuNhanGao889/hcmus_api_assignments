# PHASE_5_HUMAN_EXTENSION_PREPARATION - `POST /api/apply-coupon`

## 1. Phạm vi

- `api`: `POST /api/apply-coupon`
- `phase`: preparation for `PHASE_5_HUMAN_EXTENSION`
- `artifact`: `reports/analysis/coupon_extension_gaps.md`
- `audited_suite`: `test-cases/coupon_test_cases_audited.xlsx`
- `source_design`: `reports/analysis/coupon_test_design.md`
- `student_id`: `23127364`

Mục tiêu của artifact này là phân tích bộ test đã được human audit và chỉ ra các khoảng trống còn lại để sinh viên có thể thiết kế manual extension cases ở bước sau.

Không tạo `COUPON-EXT-*` trong artifact này. Không thay đổi audit decision. Không execute test. Không kết luận bug.

## 2. Tóm tắt bộ test đã audit

| Metric | Giá trị |
|---|---:|
| Tổng số AI-generated cases | `40` |
| `VALID` | `36` |
| `INCOMPLETE` | `4` |
| `INVALID` | `0` |
| `source` | `AI_GENERATED` |
| Manual extension đã có | `0` |

Các case còn `INCOMPLETE` theo human audit:

| case_id | coverage_type | Lý do chính | Correction đã được ghi |
|---|---|---|---|
| `COUPON-GEN-005` | `boundary,business,state` | Expected phụ thuộc việc `min_order_amount` có được enforce hay không | Giữ exploratory hoặc chỉ dùng khi có fixture xác nhận coupon có `min_order_amount` và SUT enforce rule |
| `COUPON-GEN-008` | `state,business,exploratory` | Phụ thuộc cơ chế usage tracking chưa được spec xác nhận | Cần xác nhận apply-coupon có ghi nhận usage state và setup coupon `max_uses_per_user` |
| `COUPON-GEN-009` | `state,business,negative,exploratory` | Reuse-after-limit phụ thuộc persisted usage state | Chỉ chốt assertion khi có bằng chứng SUT enforce `max_uses_per_user` |
| `COUPON-GEN-031` | `security,cross-user,exploratory` | Cross-user expected phụ thuộc user context/business rule chưa rõ | Giữ exploratory và cần làm rõ rule/user context |

## 3. Coverage hiện có

| Coverage group | Tình trạng |
|---|---|
| Positive coupon schema | Có kiểm tra coupon hợp lệ và response chứa `discount_amount`, `final_amount` |
| Coupon code partitions | Có valid, unknown, empty, whitespace, lowercase, null, wrong type, injection |
| Amount boundaries | Có below/equal/above min-order, zero, negative, decimal, very large |
| User identifier | Có missing/null/wrong type/non-existing/other user |
| Business rules | Có expired coupon, min-order, usage-limit exploratory |
| Security | Có injection, type confusion, extra fields, cross-user, information leakage |
| HTTP/parser | Có missing body, malformed JSON, top-level array, wrong `Content-Type` |

## 4. Remaining Coverage Gaps

### Gap 1 - Discount formula with controlled coupon fixture

AI suite kiểm tra schema `discount_amount` và `final_amount`, nhưng chưa có manual case dùng coupon fixture có `type`, `discount_value`, và `total_amount` được kiểm soát để tính expected numeric result rõ ràng.

Giá trị manual extension:

- Biến schema check thành business calculation check.
- Phát hiện lỗi tính sai `discount_amount` hoặc `final_amount`.
- Tạo bằng chứng rõ hơn cho `FR-09 Discount coupons`.

Ranh giới:

- Cần coupon fixture rõ ràng, ví dụ percent coupon được tạo qua admin API.
- Vì `api_specification.md` chưa mô tả công thức/rounding chi tiết, phải ghi rõ assumption hoặc chỉ dùng khi requirement/setup được xác nhận.

### Gap 2 - Rounding and decimal amount behavior

AI suite có decimal amount nhưng chưa tách riêng rounding rule cho percent coupon tạo ra discount lẻ.

Giá trị manual extension:

- Kiểm tra rounding/truncation khi `discount_amount` không nguyên.
- Phát hiện sai lệch tiền tệ ở boundary thực tế.

Ranh giới:

- Spec chưa nêu rounding rule.
- Nếu không có rule chính thức, case nên là observation/exploratory thay vì pass/fail cứng.

### Gap 3 - Minimum-order enforcement with verified fixture

`COUPON-GEN-005` bị audit `INCOMPLETE` vì thiếu fixture xác nhận `min_order_amount`. Manual extension nên thiết kế lại với setup rõ: tạo coupon có `min_order_amount`, rồi test below/equal/above threshold trong cùng fixture.

Giá trị manual extension:

- Sửa điểm yếu của AI case bằng setup cụ thể.
- Bao phủ boundary quan trọng của coupon.

Ranh giới:

- Cần admin token hoặc seed coupon.
- Không được tự kết luận behavior nếu SUT không expose setup/fixture.

### Gap 4 - Usage-limit replay with persisted verification

AI suite có first-use và reuse-after-limit nhưng human audit đánh dấu `INCOMPLETE`. Manual extension nên thiết kế một flow rõ: apply lần 1, apply lần 2 cùng `user_id`, và nếu có endpoint/state để kiểm chứng thì ghi nhận persisted usage.

Giá trị manual extension:

- Bao phủ `max_uses_per_user`, một rule quan trọng trong admin coupon model.
- Phát hiện replay/reuse bug.

Ranh giới:

- Spec chưa xác nhận apply-coupon có ghi usage state.
- Nếu không có requirement bổ sung, case chỉ nên exploratory.

### Gap 5 - Cross-user usage isolation

AI suite có `other user_id` nhưng expected còn phụ thuộc rule. Manual extension có thể tập trung vào isolation: cùng coupon, user A đã dùng, user B chưa dùng, so sánh behavior nếu `max_uses_per_user` tồn tại.

Giá trị manual extension:

- Phân biệt per-user limit với global limit.
- Tăng coverage cross-user/business-rule abuse.

Ranh giới:

- Cần ít nhất hai user fixture.
- Chỉ áp dụng nếu SUT enforce usage theo user.

### Gap 6 - Client-side override of coupon rule

AI suite có extra fields `discount_value` và `type`, nhưng chưa có manual flow so sánh output với coupon rule thật trên server để chứng minh extra fields bị ignore/reject.

Giá trị manual extension:

- Kiểm tra mass-assignment/over-posting style risk.
- Phát hiện client override discount.

Ranh giới:

- Cần biết coupon rule thật của fixture.
- Không bắt buộc SUT reject extra fields; assertion chính là extra fields không được thay đổi discount.

### Gap 7 - Authentication ambiguity for `user_id`

Spec không yêu cầu auth cho `POST /api/apply-coupon` nhưng body có `user_id`. Manual extension có thể quan sát cùng request có/không có `Authorization` để làm rõ SUT có dùng token context hay chỉ dùng `user_id`.

Giá trị manual extension:

- Làm rõ security model của endpoint.
- Chuẩn bị cho phân tích cross-user ở Phase 8 nếu execution cho thấy rủi ro.

Ranh giới:

- Không kết luận thiếu auth là bug vì spec chưa yêu cầu auth.
- Chỉ ghi observation nếu behavior khác nhau.

### Gap 8 - Coupon code duplicate keys

AI suite có wrong type và malformed JSON nhưng chưa có raw JSON duplicate keys cho `code`, `total_amount`, hoặc `user_id`.

Giá trị manual extension:

- Kiểm tra parser ambiguity.
- Phát hiện bypass nếu parser dùng first/last key bất ngờ.

Ranh giới:

- Parser-dependent, nên expected status để `SPEC_UNDEFINED`.

## 5. Ưu tiên đề xuất cho manual extension

| Priority | Gap | Lý do |
|---:|---|---|
| 1 | Discount formula with controlled coupon fixture | Giá trị nghiệp vụ cao nhất cho coupon |
| 2 | Minimum-order enforcement with verified fixture | Sửa trực tiếp một AI case `INCOMPLETE` |
| 3 | Usage-limit replay with persisted verification | Bao phủ state/replay mà AI chưa chốt được |
| 4 | Client-side override of coupon rule | Security/business-rule abuse rõ |
| 5 | Cross-user usage isolation | Bổ sung per-user behavior còn mơ hồ |
| 6 | Rounding and decimal amount behavior | Quan trọng cho tiền tệ nhưng cần rule |
| 7 | Authentication ambiguity for `user_id` | Observation hữu ích, không nên bug hóa vội |
| 8 | Coupon code duplicate keys | Parser edge case tốt cho manual exploratory |

## 6. Vì sao AI có thể đã bỏ sót

- Spec coupon rất ngắn và không nêu success status, công thức discount, rounding, hoặc error schema.
- AI đã tạo coverage rộng theo partitions nhưng nhiều business-rule cases thiếu fixture/state cụ thể.
- Các gap mạnh nhất cần multi-step setup với admin coupon creation, user fixtures, hoặc persisted usage verification.
- AI thường tránh expected quá cụ thể khi spec thiếu rule, dẫn đến nhiều case business chỉ ở mức exploratory.

## 7. Kết luận

Bộ audit hiện tại đủ rộng cho baseline AI-generated suite, nhưng manual extension nên tập trung vào business-rule precision: discount formula, min-order fixture, usage-limit replay, server-side coupon rule integrity, cross-user isolation, và parser/security edge cases. Artifact này chỉ chuẩn bị cho `PHASE_5_HUMAN_EXTENSION`; chưa tạo manual extension cases.
