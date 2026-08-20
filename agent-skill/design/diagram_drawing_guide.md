# Self-Drawn Diagram Guide

> Lưu ý: đề bài yêu cầu diagram của test generator phải do sinh viên tự vẽ. File này chỉ là hướng dẫn bố cục để vẽ lại bằng Canva, draw.io, PowerPoint, Figma hoặc công cụ tương tự.

## Bố cục khuyến nghị

Nên vẽ từ trên xuống dưới để giống workflow đã dùng trong bài.

```text
API Specification
       ↓
Specification Reader
       ↓
Rule Extractor
       ↓
Test Dimension Builder
  ├─ Domain / Boundary
  ├─ Security
  ├─ Schema
  ├─ State
  └─ Robustness / Parser
       ↓
Candidate Test Generator
       ↓
Traceability Validator
       ↓
Structured Test Cases
PENDING_HUMAN_REVIEW
       ║
       ║ Human boundary
       ↓
Student Review
├─ VALID
├─ INVALID
└─ INCOMPLETE
```

## Nội dung từng block

### Block 1 - API Specification

Subtext nên ghi: `api_specification.md + target endpoint`.

### Block 2 - Specification Reader

Sub-items:

- Method / Path
- Parameters
- Auth
- Response
- Business Rules

### Block 3 - Rule Extractor

Sub-items:

- `SPEC_DEFINED`
- `ASSIGNMENT_REQUIRED`
- `SPEC_UNDEFINED`
- `EXPLORATORY`

### Block 4 - Test Dimension Builder

Vẽ năm nhánh từ block này:

- Domain / Boundary
- Security
- Schema
- State
- Robustness / Parser

Năm nhánh nên quay lại một merge point trước khi đi xuống `Candidate Test Generator`.

### Block 5 - Candidate Test Generator

Subtext nên ghi: `Create >=35 AI-generated candidate cases`.

### Block 6 - Traceability Validator

Checklist nên ghi:

- Unique IDs
- Requirement refs
- Spec refs
- No invented expectations
- Preconditions

### Block 7 - Structured Test Cases

Subtext nên ghi: `Excel / CSV / Markdown`.

Gắn nhãn lớn: `PENDING_HUMAN_REVIEW`.

### Block 8 - Student Review

Vẽ block này bằng màu hoặc border khác để thể hiện nó nằm ngoài generator.

Branches:

- `VALID`
- `INVALID`
- `INCOMPLETE`

Subtext: `Student decision`.

## Nhấn mạnh trực quan

Nên đặt một đường phân cách nét đứt trước `Student Review`.

Bên trái hoặc phía trên đường phân cách ghi: `AI Test Generator`.

Bên phải hoặc phía dưới đường phân cách ghi: `Human Responsibility`.

Cách vẽ này giúp phần oral defense/demo thể hiện rõ rằng AI chỉ hỗ trợ sinh candidate cases, còn quyết định audit thuộc về sinh viên.
