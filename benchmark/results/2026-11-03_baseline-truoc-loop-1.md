# Kết quả benchmark — 2026-11-03 — baseline — trước loop đầu tiên

- **Thời điểm đo:** 2026-11-03 (Asia/Saigon), khi làm rõ mục đích dự án và thiết kế loop
- **Ai đo:** agent phiên làm việc này, từ nội dung thật của repo `xuvi-harness` tại thời điểm đo
- **Đối tượng đo:** repo ở trạng thái specs-only — `harness/specs/` và
  `plugins/waldorf/specs/` đã có tài liệu; `benchmark/` rỗng; `cron.md` là bản
  nháp 11 dòng; **không có code sản phẩm nào trong repo này để chạy**
- **Baseline so sánh:** chưa có — đây là lần đo đầu tiên
- **Giả thuyết của loop này:** đây là baseline, chưa có giả thuyết thay đổi sản phẩm
- **Phán quyết:** GHI NHẬN — baseline này là điểm 0 để loop hackathon so sánh.
  Nó cố tình khắt khe: tài liệu "đã hoàn thành / đã kiểm chứng" trong
  `plugins/waldorf/specs/01,04,08,09` mô tả một đợt build ở repo khác
  (`xuvi-waldorf`), nên theo mục 2 của `benchmark/specs/01` chúng chỉ tính là
  `chưa kiểm chứng` cho repo này, tới khi được chạy lại tại đây.

## Cổng veto

| Cổng | Trạng thái | Bằng chứng |
|---|---|---|
| G1 Đạo & an toàn sự sống | CHƯA ĐO | Specs có các quy tắc đúng hướng (không điểm tổng cho trẻ, không dữ liệu học sinh thật trong seed), nhưng chưa có sản phẩm chạy trong repo này để kiểm hành vi thật |
| G2 Trung thực bằng chứng | CHƯA ĐO | Tài liệu phân biệt rõ live/mock/local (điểm tốt), nhưng các kiểm chứng được tuyên bố chưa chạy lại ở đây; một số tuyên bố "đã triển khai" dễ bị đọc nhầm thành trạng thái của repo này nếu không có baseline này |
| G3 Riêng tư & bí mật | CHƯA ĐO | `.env` bị ignore đúng; specs cấm đọc khoá/Keychain; chưa có sản phẩm chạy để kiểm rò rỉ thật |

## Điểm từng chiều

| Chiều | Điểm (0–4) | Mức bằng chứng cao nhất | Bằng chứng | Delta |
|---|---|---|---|---|
| B1 Giá trị & trọn vẹn sản phẩm (20%) | 1 | chưa kiểm chứng | Master prompt và kế hoạch rất đầy đủ (`00`, `01`), ma trận nghiệm thu có sẵn (`05`); không có sản phẩm chạy được trong repo này | — |
| B2 Giáo dục & văn hoá (15%) | 2 | thủ công | Agent đã đọc trực tiếp specs trong repo này: provenance MOET theo trang, 41 bản ghi chương trình, quy tắc không tự nhận chứng nhận; chưa lấy mẫu mở trang nguồn kiểm lại, nên chưa thể lên 3 | — |
| B3 Trải nghiệm người dùng (10%) | 1 | chưa kiểm chứng | Yêu cầu UX viết rõ trong brief; chưa có giao diện để đi thử một hành trình | — |
| B4 Kiến trúc harness (15%) | 2 | thủ công | Agent đã đọc trực tiếp 15 file core trong repo này: mạch lạc, có tiêu chí nghiệm thu kiến trúc; chưa có cài đặt nào để kiểm cắm/gỡ plugin thật, nên chưa thể lên 3 | — |
| B5 Tin cậy kỹ thuật (15%) | 1 | chưa kiểm chứng | `04_verification.md` ghi typecheck/test/build đạt ở đợt build trước; không tái dựng được từ repo này | — |
| B6 AI thật & chọn model (10%) | 1 | chưa kiểm chứng | `08_meta_live_check.md` ghi live Meta thật ngày 2026-11-03 (models/text/image/voice) do orchestrator chạy với `.env` của người dùng; chưa chạy lại qua sản phẩm của repo này nên bị trần 1 theo mục 2 của bộ tiêu chí | — |
| B7 Nghiên cứu & học hỏi (5%) | 1 | tự động | `research/specs/resources.md` chỉ có 3 link, không ngày kiểm tra, không quyết định adopt/watch/reject | — |
| B8 Sức khoẻ tiến hoá (10%) | 1 | tự động | `.gitignore` theo specs-only nhất quán và specs truy vết tốt; nhưng `cron.md` chưa có cổng đo/giữ/huỷ và benchmark chưa từng có kết quả lưu nào | — |
| **Tổng có trọng số (/100)** | **32.5** | — | 5.0 + 7.5 + 2.5 + 7.5 + 3.75 + 2.5 + 1.25 + 2.5 | — |

> Đọc số 32.5 này đúng cách: đây là điểm của **một repo specs chưa có sản phẩm
> chạy tại chỗ**, không phải điểm của ý tưởng hay của đợt build Waldorf trước.
> Nó thấp chủ yếu ở trần bằng chứng — và đó là chủ ý của benchmark: loop đầu
> tiên muốn nâng điểm phải *chạy lại và chứng minh tại đây*, không viết thêm
> tuyên bố.

## Research đã quét trong loop này

| Nguồn | Ngày kiểm tra | Điều mới | Quyết định |
|---|---|---|---|
| (chưa quét) | — | Loop này chỉ làm rõ mục đích và thiết kế cách đo, không kiểm tra nguồn ngoài | watch — việc quét thuộc pha 1 của loop hackathon |

## Điều đã sửa ở specs trong loop này

- `README.md` (mới), `cron.md` (viết lại), `AGENTS.md`, `.gitignore` — làm rõ
  mục đích, loop và luật commit (chi tiết trong git history của đợt này)
- `benchmark/specs/` (mới) — bộ tiêu chí tổng thể này và mẫu ghi kết quả
- `benchmark/results/` (mới) — cơ chế lưu kết quả, bắt đầu bằng file này
- `research/specs/`, `plugins/waldorf/specs/10_...`, `harness/specs/README.md`
  — nối các tầng lại cùng một bức tranh

Ghi chú về công bằng: B7/B8 được chấm theo trạng thái *trước* các sửa đổi trên.
Từ lần đo sau, hai chiều này phải tự chứng minh bằng cách vận hành thật (có quét
research có ghi quyết định; có loop qua đủ pha 8), không tự tăng vì đã có file mẫu.

## Bài học và điểm nghẽn kế tiếp

- Bài học lớn nhất của baseline: repo không thiếu ý tưởng hay tài liệu — nó
  thiếu **một sản phẩm chạy được tại đây kèm bằng chứng tại đây**. Mọi điểm yếu
  B1/B3/B5 đều hội tụ về điều này.
- Điểm nghẽn quan trọng nhất cho loop sau: **dựng lại/chạy được một hành trình
  Waldorf trọn vẹn trong repo này và đo lại** — xem
  `plugins/waldorf/specs/10_loop-dau-tien-hackathon.md`.

## Vệ sinh commit

- `git status` tại thời điểm kết thúc: chỉ gồm specs/root files thuộc diện commit (kiểm lại trước khi kết thúc đợt)
- ĐẠO.md có diff không: không (bắt buộc)
