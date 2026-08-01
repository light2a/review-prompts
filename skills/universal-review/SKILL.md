---
name: universal-review
description: Phân tích phản biện (critical review) một dự án theo artifact thực tế — tự thích nghi với repo chỉ có tài liệu (BRD/PRD), chỉ có code, hoặc cả hai; xuất bộ báo cáo Markdown chuẩn hóa vào analysis/. Dùng khi cần review BRD/PRD, review codebase, đối chiếu implementation với requirements, hoặc đánh giá agent harness/kit đã cài trong dự án.
---

# Universal Harness-Based Project Review

Thực hiện phân tích phản biện theo đúng quy trình dưới đây. Nếu người dùng truyền tham số (`$ARGUMENTS`), coi đó là chỉ định phạm vi hoặc đường dẫn artifact bổ sung.

**Ngôn ngữ output:** theo ngôn ngữ của tài liệu chính trong repo (hoặc theo chỉ định của người dùng). Thuật ngữ kỹ thuật giữ tiếng Anh.

## Bước 0 — Tiền đề & phạm vi (BẮT BUỘC, làm trước mọi thứ)

1. `ls` thư mục làm việc, ghi nhận toàn bộ nội dung cấp cao.
2. **Halt rule:** thư mục rỗng hoặc chỉ có file cấu hình/tooling (`.git`, `.claude/`, `.gitignore`…) mà không có tài liệu lẫn code → **DỪNG**, báo người dùng và hỏi artifact nằm ở đâu. Không tạo file phân tích nào.
3. **Tài liệu ngoài repo:** có dấu hiệu tài liệu nguồn nằm ngoài thư mục (nhắc trong chat, trong README, hoặc chỉ có bản trích xuất thứ cấp) → hỏi người dùng đường dẫn **trước khi** kết luận "không có tài liệu".
4. **Thư mục output:** mặc định `analysis/`. Nếu đã tồn tại → không đè: ghi vào `analysis-<YYYY-MM-DD>/` và **đọc review cũ trước** để tham chiếu chéo thay vì lặp lại finding.

## Nguyên tắc

- **Reality First:** chỉ phân tích cái tồn tại thật. Không giả định ngôn ngữ, framework, kiến trúc, database, cloud, domain. Artifact không tồn tại → ghi: "**Artifact này chưa tồn tại trong repository hiện tại nên không thể xác minh.**"
- **Evidence First (2 lớp):** code = path + class/function + snippet ≤20 dòng; tài liệu = tên file + section + requirement ID. Thiếu bằng chứng → "**Chưa xác minh được từ artifact hiện có.**" Suy luận → dán nhãn "**Suy luận từ artifact hiện có, chưa có bằng chứng trực tiếp.**"
- **Adversarial:** không mô tả suông — với mỗi thành phần hỏi: hỏng gì trong production? mâu thuẫn tài liệu? khó mở rộng? dễ gây bug/nợ? dựa assumption nào chưa kiểm chứng?
- **Chống hallucination:** tuyệt đối không bịa file path, class/function, requirement ID, công nghệ, cloud, CI/CD.

## Bước 1 — Inventory

Bảng `| Artifact | Tồn tại | Mục đích |` rà tối thiểu: BRD/PRD · source code · API spec · DB schema/migration · Docker/Compose · CI/CD · ADR/architecture docs · README · test suite · **kết quả review trước đó** · **agent kit/tooling đã cài** (`.claude/`, `.cursor/`, …).

## Bước 2 — Chọn mode

| Mode | Khi nào | Nhóm findings | Thang readiness (0–10) |
|---|---|---|---|
| **A — Docs only** | Chưa có code | Requirements · Nhất quán nội bộ · NFR đo được · Delivery · Governance | **Next-phase readiness**: open questions đã chốt? traceability? tiêu chí chấp nhận đo được? approval? |
| **B — Code + Docs** | Có cả hai | 7 nhóm: Requirements · Architecture · Security · Data consistency · Operations · Maintainability · Delivery | **Production readiness**: test luồng chính, security cơ bản, vận hành được, docs khớp code |
| **C — Code only** | Không có docs | Như B, Requirements → "Ý định suy đoán từ code" | Như B |

## Bước 3 — Nếu có agent harness/kit

Nhận diện bằng **dấu hiệu**, hai dạng: (1) clone/nguồn (kiểu `.harness-skills/` có README, CONCEPTS, skills/, agents/, hooks/); (2) **đã cài** (`.claude/skills|agents|hooks` + config hiệu lực tại root như `.hs.json` + hooks trong `.claude/settings.json`). Quy tắc: cả hai cùng tồn tại → **review bản đã cài là chính**, diff với bản nguồn tìm drift; đọc config hiệu lực chứ không phải settings mẫu; đánh giá guard nào thực thi bằng code vs chỉ là lời nhắc. Không có kit → bỏ qua.

## Bước 4 — Output

**Luôn tạo 5 file cốt lõi** trong `<output-dir>/`: `00-executive-summary.md`, `01-artifact-inventory.md`, `02-methodology-and-workflow.md`, `03-findings.md`, `04-recommendations.md`, + thư mục `evidence/` (trích xuất tài liệu, log lệnh; được phép rỗng).

**Tạo có điều kiện — không tạo file rỗng:** `05-architecture-review.md` (có code) · `06-security-review.md` (có auth/API/infra) · `07-database-review.md` (có schema/migration) · `08-ci-cd-review.md` (có CI/CD) · `09-harness-fit-review.md` (có kit) · `10-requirements-traceability.md` (có docs + code).

`03-findings.md` >15 findings → tách `03-findings/<nhóm>.md` + file gốc làm mục lục. Nhóm không có artifact → một dòng "chưa tồn tại", không ép finding.

**Nội dung bắt buộc:**
- `00`: loại dự án, mode, mức độ hiểu, 5 điểm mạnh, 5 rủi ro, readiness theo thang của mode kèm tiêu chí, khuyến nghị 30 ngày, bảng severity. *Không bắt buộc Finding.*
- `01`: bảng `| Artifact | Trạng thái | Đã phân tích | Ghi chú |`. *Không bắt buộc Finding.*
- `02`: mode + lý do, quy trình, giới hạn, assumption. **≥1 finding về giới hạn dữ liệu.**
- `04`: P0/P1/P2, mỗi mục có Impact, Effort (S/M/L), Owner, Dependency.
- Mỗi file phân tích (02, 03, 05–10) có **≥1 `## Finding`** đúng format dưới.

## Finding format (bắt buộc mọi finding)

```md
## Finding: [Tên ngắn gọn]
**Severity:** Critical | High | Medium | Low
### Evidence
- File / Section / Requirement ID + snippet/trích dẫn ≤20 dòng
### Analysis
Vì sao đây là vấn đề (hoặc điểm mạnh đáng giữ).
### Counter-argument
Trường hợp thiết kế hiện tại là lựa chọn có chủ đích hợp lý.
### Verdict
✅ Acceptable | ⚠️ Risk accepted | ❌ Should be changed
### Recommended Fix
Hành động cụ thể, khả thi.
```

## Bước 5 — Tự kiểm chứng (BẮT BUỘC, sau khi viết xong file cuối)

1. Đếm severity bằng lệnh (`grep -c "Severity:"`) — phải khớp bảng trong `00`; lệch thì sửa bảng.
2. Spot-check mọi file path trong Evidence bằng `ls` — path chết thì sửa/gỡ finding.
3. Đủ 5 file cốt lõi, không file rỗng, file điều kiện đúng điều kiện, mỗi file phân tích ≥1 Finding.

## Output cuối

Chỉ trả về:
```text
Analysis completed.
Mode: A|B|C
Output: <output-dir>/
Generated files:
- <danh sách file>
Critical findings: X
High findings: Y
Medium findings: Z
Low findings: W
```
Không in nội dung file trừ khi được yêu cầu.
