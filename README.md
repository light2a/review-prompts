# review-prompts

Bộ prompt chuẩn hóa để **phân tích phản biện (critical review)** dự án phần mềm bằng AI coding assistant — tự thích nghi với repo chỉ có tài liệu (BRD/PRD), chỉ có code, hoặc cả hai. Đã được kiểm chứng bằng chạy thử thật (xem Changelog).

## Cấu trúc

```text
prompts/
└── universal-harness-review.md   # prompt thuần — dán vào Claude, ChatGPT, hoặc AI assistant bất kỳ
skills/
└── universal-review/
    └── SKILL.md                  # bản đóng gói Claude Code skill — gọi /universal-review ở mọi dự án
```

## Usage

### Với Claude Code (khuyến nghị — cài 1 lần, dùng mọi dự án)

```bash
mkdir -p ~/.claude/skills/universal-review && cp skills/universal-review/SKILL.md ~/.claude/skills/universal-review/SKILL.md
```

Mở phiên Claude Code mới (skill listing nạp lúc khởi động), rồi ở bất kỳ dự án nào:

```text
/universal-review
```

### Với AI assistant khác

Dán toàn bộ nội dung `prompts/universal-harness-review.md` vào đầu hội thoại, trong thư mục dự án cần review.

### Kết quả

Bộ báo cáo Markdown trong `analysis/` (hoặc `analysis-<ngày>/` nếu đã có review cũ): executive summary, artifact inventory, methodology, findings (format chuẩn 6 phần có Counter-argument), recommendations P0/P1/P2, cùng các file điều kiện (architecture/security/database/CI-CD/harness-fit/traceability) chỉ khi có artifact tương ứng.

## Cập nhật & push lên GitHub

Repo này nằm ở `~/Downloads/review-prompts`, remote `origin` trỏ tới `https://github.com/light2a/review-prompts.git`, nhánh `main` đã set upstream.

Quy trình cập nhật sau khi sửa `prompts/` hoặc `skills/`:

```bash
cd ~/Downloads/review-prompts
git status                     # xem file đã đổi
git add -A
git commit -m "mô tả thay đổi"
git push                       # đã set-upstream, không cần -u origin main nữa
```

Sau khi sửa `skills/universal-review/SKILL.md`, đồng bộ lại bản đã cài (bắt buộc — Claude Code đọc từ `~/.claude/skills/`, không đọc trực tiếp từ repo này):

```bash
cp skills/universal-review/SKILL.md ~/.claude/skills/universal-review/SKILL.md
```

Mở phiên Claude Code mới để skill listing nạp lại bản vừa cập nhật.

Lần đầu clone repo này trên máy khác:

```bash
git clone https://github.com/light2a/review-prompts.git
cd review-prompts
mkdir -p ~/.claude/skills/universal-review
cp skills/universal-review/SKILL.md ~/.claude/skills/universal-review/SKILL.md
```

## Thiết kế — vì sao prompt có hình dạng này

Ba nguyên tắc lõi, mỗi cái sinh ra từ một lỗi thật của các phiên bản trước:

1. **Reality First + Bước 0.** Phiên bản đầu (v1) yêu cầu review code trên một repo trống và viện dẫn thư mục không tồn tại — agent làm theo sẽ bịa ra kiến trúc để có cái viết. v3 bắt kiểm tra tiền đề trước, có halt rule, và có quy tắc hỏi người dùng khi tài liệu nằm ngoài repo.
2. **Evidence First + Counter-argument.** Mọi finding phải dẫn chứng được (path/section/ID) và phải tự phản biện ("thiết kế này có thể có chủ đích không?") trước khi phán ❌.
3. **Tự kiểm chứng bằng lệnh.** Trong lần chạy thử v2, chính bảng tổng hợp severity bị đếm sai và chỉ được phát hiện nhờ `grep -c` — nên v3 đưa bước này thành bắt buộc.

## Changelog

- **v1** (dùng 1 lần, đã bỏ): hardcode domain dự án khác (payment/interview/AI, C#/.NET), yêu cầu đọc thư mục không tồn tại, ép 16 file cố định, không có tiền đề → bị phản biện toàn diện, kết quả tại review dự án gốc.
- **v2** (bản tổng quát hóa đầu tiên): thêm Reality First, 3 mode A/B/C, file điều kiện, chống hallucination. Được **chạy thử thật** trên một repo Mode A có harness kit → lộ 9 friction: đè `analysis/` có sẵn; không nhận diện kit đã cài vào `.claude/` (chỉ biết bản clone); inventory thiếu 2 loại artifact; "production readiness" vô nghĩa ở mode docs-only; ép Finding vào file tổng hợp; 7 nhóm findings cứng không khớp Mode A; thiếu chỉ định ngôn ngữ; thiếu tự kiểm chứng số liệu; thiếu halt rule cho repo trống.
- **v3** (hiện tại): sửa toàn bộ 9 friction + 4 lỗ hổng suy luận từ walkthrough Mode B/C; giữ nguyên phần đã chứng minh hoạt động tốt (finding format, mode, file điều kiện, chống hallucination, output cuối).

## Ghi chú

- "Harness" trong tên chỉ **phong cách review có kỷ luật bằng chứng** lấy cảm hứng từ [harness-skills](https://github.com/Unibean9/harness-skills); prompt không phụ thuộc repo đó — mục harness-fit chỉ chạy khi dự án thực sự có kit.
- Prompt viết tiếng Việt; output tự khớp theo ngôn ngữ tài liệu của dự án được review.
