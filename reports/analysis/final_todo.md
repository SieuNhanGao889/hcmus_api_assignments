# Final TODO Before Submission

## Blocking

- [ ] Chụp screenshot Postman Console thật cho thấy pre-request script gắn `X-Student-Id: 23127364`; lưu dưới `reports/screenshots/` hoặc evidence folder được chọn và link từ main report.
- [ ] Giải quyết yêu cầu “two sample commits”: tạo/ghi nhận một commit cho passing run và một commit cho intentional failing run, hoặc lưu xác nhận của giảng viên rằng hai run modes trên cùng commit `a0adb735...` được chấp nhận.
- [ ] Human-correct `reports/ai_audit_report.md`: bổ sung exact data còn thiếu cho AI-002 nếu có, sửa broken paths AI-016/017/020, ghi nhận CI execution/finalization interactions bằng timestamp thật nếu áp dụng, và làm rõ provenance self-drawn của diagram.
- [ ] Regenerate `reports/ai_audit_report.pdf` sau khi AI audit Markdown được duyệt.
- [ ] Human review `reports/main_report.md`, xác nhận cách kèm AI audit appendix, rồi export `reports/main_report.pdf`.
- [ ] Regenerate `reports/ai_critique.pdf` từ `reports/ai_critique.md`; PDF hiện tại vẫn chứa bản critique dài trước finalization.
- [ ] Commit/push final documentation, rồi regenerate `git-log/commit_log.txt` để chứa final commit.
- [ ] Tạo ZIP `23127364_HW06_AI_API_100.zip`, kiểm tra required contents và upload Moodle trước deadline.

## Needs human confirmation

- [ ] Xác nhận bộ ba API không trùng bộ ba của thành viên khác trong nhóm.
- [ ] Xác nhận `agent-skill/design/diagram.png` thực sự self-drawn và không AI-generated; sửa tài liệu nếu provenance hiện tại chưa chính xác.
- [ ] Xác nhận granularity của 18 commits hiện có đáp ứng yêu cầu “new commit for each step”; không rewrite history nếu chưa được yêu cầu.
- [ ] Xác nhận đã dùng Postman workspace hoặc GUI Collection Runner hay chưa; chỉ claim nếu có evidence thật.

## Optional

- [ ] Regenerate `bug-reports/bugs_summary.pdf` để đồng bộ với Markdown.
- [ ] Cân nhắc redact JWT/password test trong screenshot public sau khi giữ bản evidence được duyệt.
- [ ] Xóa `.github/workflows/.gitkeep` sau human approval.
- [ ] Tùy chọn chạy CI full-regression diagnostic; không claim run nếu chưa thực hiện.
