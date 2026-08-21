# Final Rubric Audit – HW06 API Testing

## 1. Phạm vi audit

Audit này đối chiếu repository với `docs/2026.HW06.API Testing_En.md`, `AGENT.md` và evidence thật hiện có. Không chạy Postman/Newman, không trigger GitHub Actions, không sửa SUT/assertion và không tạo evidence giả.

Allowed status:

- `COMPLETE`: có evidence trực tiếp.
- `TODO`: thiếu artifact/action bắt buộc.
- `NEEDS_HUMAN_CONFIRMATION`: có dấu hiệu/evidence nhưng không thể kết luận an toàn bằng static audit.
- `N/A`: đề ghi rõ optional hoặc không áp dụng; có giải thích.

## 2. Requirement-to-evidence matrix

| Requirement / rubric item | Status | Evidence | Remaining action |
|---|---|---|---|
| Thông tin sinh viên và assignment | COMPLETE | `README.md`, `reports/main_report.md`, MSSV `23127364` | None. |
| Chọn đúng ba API từ Pool A/B/C | COMPLETE | README và main report map Login/Coupon/Admin Order Status | None. |
| Bộ ba API không trùng thành viên khác | NEEDS_HUMAN_CONFIRMATION | Repository chỉ ghi tuyên bố, không có danh sách lựa chọn của nhóm | Sinh viên xác nhận trước khi nộp. |
| Quy trình AI-first theo từng bước | COMPLETE | `reports/analysis/*_spec_analysis.md`, `*_test_design.md`, workbooks, AI audit | None. |
| Ít nhất 35 AI-generated cases/API | COMPLETE | Workbook gốc: Login 40, Coupon 40, Order Status 42 | None. |
| Domain partitions/boundaries mọi tham số phù hợp | COMPLETE | Test design và audited workbooks | None. |
| Security coverage | COMPLETE | Injection, leakage, auth, role bypass, type confusion, ID/path risks trong designs/data | None. |
| Schema validation | COMPLETE | Collection test scripts và test design | None. |
| Human audit `VALID/INVALID/INCOMPLETE` có reasoning | COMPLETE | Audited workbooks: 115 VALID, 7 INCOMPLETE, 0 INVALID | None. |
| Correct invalid/incomplete cases | COMPLETE | Cả 7 INCOMPLETE rows có `audit_notes` và `human_correction` | None. |
| Ít nhất 5 manual extension/API | COMPLETE | 5 Login, 5 Coupon, 5 Order Status | None. |
| Giải thích AI bỏ sót extension | COMPLETE | `reports/analysis/*_extension_gaps.md`, audited workbook metadata | None. |
| Postman collection/environment/external data | COMPLETE | `postman/` artifacts parse và có traceability | None. |
| `X-Student-Id` được inject trong request | COMPLETE | Collection-level pre-request upsert và environment `studentId=23127364` | None. |
| Console screenshot chứng minh `X-Student-Id` | TODO | Không có screenshot Postman Console phù hợp trong repository | Chụp console thật khi pre-request script log header và thêm vào report/evidence. |
| Thực thi cả ba API bằng Postman/Newman | COMPLETE | Login/Coupon/Order Status Newman JSON reports | None. |
| Newman JSON/HTML reports | COMPLETE | `reports/newman/*.json`, `reports/newman/html/*.html` | None. |
| Newman hostname là localhost/127.0.0.1 | COMPLETE | Raw reports chứa request URL `http://localhost:3000` | None. |
| Liệt kê Postman features thực dùng | COMPLETE | `reports/main_report.md` mục 7 | None. |
| Monitor/mock server | N/A | Đề nêu như ví dụ “as many as reasonably can”, không bắt buộc; không có evidence sử dụng | Không claim đã dùng. |
| Bug report Markdown cho genuine bugs | COMPLETE | `bug-reports/PB-01.md` đến `PB-08.md` | None. |
| Mỗi bug có severity/endpoint/reproduction/expected/actual/evidence/human status | COMPLETE | Tám canonical bug reports | None. |
| GitHub Issues thật | COMPLETE | Live Issues #1–#8 và links trong `bugs_summary.md` | None. |
| Screenshot đính kèm mỗi Issue | COMPLETE | GitHub Issue API bodies #1–#8 chứa image attachment syntax; local PB screenshots tồn tại | None. |
| CI workflow thực thi Newman | COMPLETE | `.github/workflows/api-tests.yml` | None. |
| CI smoke cover ba API với explicit folder isolation | COMPLETE | Sáu `--folder` mappings; ba smoke datasets | None. |
| CI upload JSON/HTML artifacts kể cả failure | COMPLETE | Hai `upload-artifact@v4` steps dùng `always()` | None. |
| Real successful CI run | COMPLETE | Run [#1](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445200879), screenshot `ci_success.png` | None. |
| Real intentional failed CI run | COMPLETE | Run [#2](https://github.com/SieuNhanGao889/hcmus_api_assignments/actions/runs/32445604904); smoke pass trước CI-only failure | None. |
| CI report có configuration, links và screenshots | COMPLETE | `reports/cicd_report.md`, `reports/screenshots/` | None. |
| Hai sample commits: một pass, một fail | TODO | Hai run thật cùng dùng commit `a0adb735...` | Tạo/ghi nhận hai commit riêng hoặc lấy xác nhận giảng viên rằng dispatch trên cùng commit được chấp nhận. |
| AI-driven API test generator design | COMPLETE | `agent-skill/SKILL.md`, `design/design.md` | None. |
| Reusable Agent Skill implementation | COMPLETE | `agent-skill/SKILL.md` và README | None. |
| Pseudocode | COMPLETE | `agent-skill/design/pseudocode.md` | None. |
| Diagram artifact tồn tại | COMPLETE | `agent-skill/design/diagram.png` | None. |
| Diagram self-drawn, không AI-generated | NEEDS_HUMAN_CONFIRMATION | Demo notes đánh dấu self-drawn; AI audit AI-020 lại nói AI hỗ trợ “diagram” | Sinh viên xác nhận provenance và sửa câu chữ AI audit nếu cần. |
| Demo video YouTube | COMPLETE | [Video thật](https://youtu.be/4fAWYhzeHzQ), `video_link.md` | None; video là encouraged, không bắt buộc. |
| Khai báo công cụ AI | COMPLETE | `reports/ai_audit_report.md` | None. |
| AI audit có tool/date-time/prompt/output cho mọi interaction | TODO | AI-002 thiếu thời điểm cụ thể/model; audit dừng ở AI-032 và chưa ghi finalization interaction | Sinh viên bổ sung dữ liệu thật, không đoán timestamp. |
| AI/human responsibility separation | COMPLETE | AI audit và execution analysis phân biệt recommendation, human decision và execution | None. |
| AI audit artifact paths chính xác | TODO | AI-016/017 trỏ nhầm `test-cases/*_extension_gaps.md`; AI-020 nhắc các demo filenames không tồn tại | Sinh viên sửa path thành artifact thật và review toàn bộ links. |
| AI audit Markdown/PDF đồng bộ | TODO | Markdown đang cần correction/final interaction; PDF không thể coi là final sau correction | Cập nhật Markdown rồi regenerate `reports/ai_audit_report.pdf`. |
| AI Critique 200–300 words | COMPLETE | `reports/ai_critique.md` có 286 whitespace-delimited words | None. |
| AI Critique PDF đồng bộ Markdown | TODO | `reports/ai_critique.pdf` hiện chứa bản critique dài trước finalization | Regenerate PDF từ Markdown 286 từ sau human review. |
| Có lịch sử Git commits theo quy trình | COMPLETE | Git history có 18 commits từ setup đến CI | None. |
| Một commit mới cho mỗi step/procedure | NEEDS_HUMAN_CONFIRMATION | Có commit theo nhiều phase, nhưng một số phase/API được gộp | Sinh viên đối chiếu kỳ vọng giảng viên; không rewrite history nếu chưa được yêu cầu. |
| Git commit log text artifact | TODO | `git-log/commit_log.txt` là snapshot thật nhưng chưa chứa final documentation commit | Sau final commit, regenerate file từ `git log`. |
| Main report Markdown | COMPLETE | `reports/main_report.md` đã cập nhật từ evidence thật | None. |
| Main report PDF | TODO | `reports/main_report.pdf` chưa tồn tại | Human review Markdown rồi export PDF cuối. |
| Main report bao gồm/đính kèm AI audit appendix | NEEDS_HUMAN_CONFIRMATION | Main report link AI audit; AI audit có MD/PDF riêng | Khi xuất PDF/ZIP, xác nhận appendix được ghép hoặc đính kèm theo cách giảng viên chấp nhận. |
| Public GitHub repository link | COMPLETE | README/main report có link repository thật | None. |
| Excel test cases và test summary | COMPLETE | Sáu `.xlsx`, `test-cases/test_summary.md`, README summary | None. |
| Bug report và screenshots trong package | COMPLETE | `bug-reports/` có Markdown/PDF và 8 PB images | None. |
| README có test summary | COMPLETE | README có generated/added/executed/passed/failed/bugs | None. |
| README có self-assessment table đã điền | COMPLETE | Sinh viên đã điền 30/30/30/10, tổng 100 | None. |
| ZIP đúng `<StudentID>_HW06_AI_API_<SelfAssessedGrade>.zip` | TODO | Self-assessed grade đã chốt `100`, nhưng chưa có final ZIP | Tạo `23127364_HW06_AI_API_100.zip` sau final review. |
| Optional OpenAPI conversion | N/A | Đề ghi optional; repository không có conversion | Không ảnh hưởng blocking rubric. |
| Oral defense preparation | N/A | Chỉ áp dụng nếu sinh viên thuộc nhóm 30% được chọn sau deadline | Chuẩn bị giải thích artifact nếu được gọi. |

## 3. Postman feature audit

| Feature | Classification | Evidence / action |
|---|---|---|
| Workspace | USED_BUT_DOCUMENTATION_MISSING nếu đã dùng | Không có export/screenshot; sinh viên xác nhận, nếu không thì `NOT_USED`. |
| Collection | USED_AND_EVIDENCED | Collection JSON. |
| Folders | USED_AND_EVIDENCED | Ba API folders + deferred folder. |
| Requests | USED_AND_EVIDENCED | 43 request definitions. |
| Pre-request scripts | USED_AND_EVIDENCED | 44 event definitions gồm collection-level script. |
| Post-response/test scripts | USED_AND_EVIDENCED | 43 request-level test event definitions. |
| Collection variables | USED_AND_EVIDENCED | 4 variables. |
| Environment | USED_AND_EVIDENCED | Local environment với 30 entries. |
| External data files | USED_AND_EVIDENCED | 3 source JSON + 3 CI smoke JSON. |
| Data-driven Newman | USED_AND_EVIDENCED | 46/47/73 iteration reports. |
| GUI Collection Runner | USED_BUT_DOCUMENTATION_MISSING nếu đã dùng | Chỉ Newman execution được chứng minh. |
| Dynamic extraction | USED_AND_EVIDENCED | Setup scripts lấy token/user/coupon/order IDs. |
| Assertions | USED_AND_EVIDENCED | JSON/HTML reports và collection tests. |
| Monitor | NOT_USED | Không có artifact. |
| Mock server | NOT_USED | Không có artifact. |
| TODO_IF_NEEDED_FOR_RUBRIC | Không có | Monitor/mock/workspace không phải hard requirement; không tạo chỉ để trang trí. |

## 4. CI/CD evidence audit

- Workflow tồn tại và YAML parse được.
- Smoke uses `LOGIN-GEN-005`, `COUPON-GEN-003`, `ORDERSTATUS-GEN-002` với đúng folder.
- Artifact upload dùng `always()`.
- `force_failure` chỉ chạy sau smoke success.
- Full regression không dùng `--suppress-exit-code`; PB assertions vẫn còn.
- Hai screenshots tồn tại và hiển thị run #1 success, run #2 failure.
- GitHub Actions API xác nhận run #2 có smoke step `success`, intentional step `failure`, upload step `success`.
- Hai run có URL thật nhưng cùng head SHA; “two sample commits” chưa hoàn thành literal.

## 5. Bug/GitHub Issue audit

PB-01 đến PB-08 đều có canonical report, severity, endpoint, reproduction, expected/actual, Newman reference và `CONFIRMED_BY_HUMAN_REVIEW`. Live Issue #1–#8 tồn tại, đang open khi audit, và mỗi body chứa image attachment syntax. Không tạo hoặc sửa Issue trong phase này.

## 6. Video demo audit

URL `https://youtu.be/4fAWYhzeHzQ` có trong `agent-skill/demo/video_link.md`, `agent-skill/README.md` và demo notes. URL resolve tới YouTube video `CSC13102_23KTPM1_23127364_HW6`. Main report và README đã tham chiếu link thật này.

## 7. AI audit integrity

| Check | Result |
|---|---|
| Chronology | AI-001 đến AI-032 theo thứ tự; AI-002 không có exact time. |
| AI vs human | Phân tách rõ AI analysis với human audit/execution/bug confirmation. |
| Execution claims | Audit ghi execution do sinh viên thực hiện; không claim AI tự chạy Newman. |
| Newman/correction history | AI-023 đến AI-027 có initial analysis, human decision, correction và rerun. |
| CI design/folder correction | AI-031 và AI-032 có design và human correction. |
| Real CI execution acknowledgment | Chưa có entry sau AI-032. |
| Artifact paths | Có broken/stale paths tại AI-016/017/020. |
| Diagram provenance | Câu chữ AI-020 cần human confirmation so với self-drawn constraint. |

Không tự sửa lịch sử/timestamp trong AI audit. Các điểm thiếu được đưa vào blocking TODO.

## 8. Cleanup recommendations

Không file nào bị xóa trong phase này.

| Classification | Artifact | Lý do |
|---|---|---|
| KEEP | Original và audited `.xlsx` | Cần trace `AI output -> human audit -> extension`. |
| KEEP | `reports/newman/order_status_run.json` và rerun JSON | Evidence lịch sử trước/sau automation correction; không phải duplicate thừa. |
| KEEP | Toàn bộ `reports/analysis/` | Audit trail theo phase và requirement traceability. |
| KEEP | `bug-reports/github-issues/*.md` | Draft/history hỗ trợ đối chiếu live Issues. |
| KEEP | Bug/CI screenshots | Evidence bắt buộc. |
| KEEP | Assignment MD/PDF và API specification | Source-of-truth. |
| KEEP | Agent Skill design/demo artifacts | Deliverable G9.5 và video support. |
| SAFE_TO_REMOVE | `.github/workflows/.gitkeep` | Placeholder không còn cần vì workflow thật đã tồn tại. Chỉ xóa sau human approval. |
| REVIEW_BEFORE_REMOVE | `bug-reports/bugs_summary.pdf` | Extra PDF hữu ích nhưng có thể lệch Markdown sau correction nhỏ; regenerate hoặc giữ có chú thích. |
| REVIEW_BEFORE_REMOVE | `reports/ai_audit_report.pdf` | Required artifact, không xóa; phải regenerate sau khi human sửa Markdown. |
| REVIEW_BEFORE_REMOVE | Any runtime CI artifact downloaded later | Kiểm tra token/runtime data trước khi commit; GitHub artifact không cần copy toàn bộ vào repo nếu screenshots/links đủ. |

## 9. Blocking TODOs before submission

1. Chụp và thêm screenshot Postman Console thật chứng minh pre-request script gắn `X-Student-Id: 23127364`.
2. Giải quyết yêu cầu hai sample commits riêng cho CI pass/fail, hoặc có xác nhận giảng viên chấp nhận hai modes trên cùng commit.
3. Human-correct AI audit: bổ sung dữ liệu thật còn thiếu, sửa broken artifact paths, ghi interaction CI/finalization nếu áp dụng, xác nhận diagram provenance; sau đó regenerate AI audit PDF.
4. Human review main report, quyết định cách đính kèm AI audit appendix, rồi export `reports/main_report.pdf`.
5. Regenerate `reports/ai_critique.pdf` từ bản Markdown 286 từ hiện tại.
6. Xác nhận API selection không trùng nhóm.
7. Commit/push final documentation; regenerate `git-log/commit_log.txt` để chứa final commit.
8. Tạo ZIP đúng tên `23127364_HW06_AI_API_100.zip` và kiểm tra toàn bộ required contents trước upload Moodle.

## 10. Optional improvements

- Xác nhận/ghi bằng chứng Postman workspace hoặc GUI Collection Runner nếu thực sự đã dùng; nếu không, để `NOT_USED`.
- Cân nhắc redact JWT/password dùng một lần trong screenshot khi public repository, không xóa evidence gốc khi chưa có bản thay thế được duyệt.
- Xóa `.github/workflows/.gitkeep` sau human approval.
- Regenerate `bug-reports/bugs_summary.pdf` để đồng bộ hoàn toàn với Markdown.
- Tùy chọn chạy CI full-regression diagnostic để minh họa non-gating behavior; không bắt buộc vì local Newman full-suite evidence đã tồn tại.
