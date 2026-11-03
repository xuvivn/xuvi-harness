# Nguồn nghiên cứu đã đăng ký

> Đây là **danh sách giác quan** của hệ: loop chỉ quét các nguồn đã đăng ký ở
> đây (pha 1 trong `cron.md`), và mỗi lần quét phải để lại ngày kiểm tra +
> quyết định trong file kết quả benchmark của loop đó. Thêm nguồn mới vào đây
> trước, rồi mới quét nó. Nguồn chưa đăng ký mà dùng trong một quyết định
> là lỗi quy trình — ghi nguồn vào đây kèm một dòng lý do.
>
> Quy trình quét và mẫu ghi: [01-quy-trinh-nghien-cuu.md](01-quy-trinh-nghien-cuu.md).

## 1. Năng lực agent, context engineering, và model platform đang dùng

| Nguồn | Theo dõi điều gì | Lần kiểm tra gần nhất |
|---|---|---|
| https://www.anthropic.com/research | Nghiên cứu về agent, độ tin cậy, context | chưa ghi — loop quét đầu tiên phải điền |
| https://www.anthropic.com/engineering | Thực hành xây agent harness, tools, multi-agent, context engineering (nền của `harness/specs/core/07,10`) | chưa ghi |
| https://dev.meta.ai/docs | Model và capability Meta chính thức: danh sách model, image generation, speech-to-text — nguồn duy nhất được bịa-không của endpoint/contract | 2026-11-03 (qua các brief trong `plugins/waldorf/specs/03,06,08`) |

## 2. Miền của plugin đầu tiên: giáo dục Waldorf-inspired tại Việt Nam

Các nguồn này hiện tồn tại dưới dạng tài liệu local đã dùng trong
`plugins/waldorf/specs/` (PDF GDPT 2018/TT32, các chương trình Waldorf lớp 1–12,
tài liệu Nhà Thuận Nhiên, truyện Việt). Khi research loop chạm vào miền này:

- Bổ sung vào bảng này **link chính thức** của nguồn (MOET, tổ chức Waldorf,
  nhà xuất bản) kèm ngày kiểm tra — không tự bịa URL, không lấy link từ trí nhớ
  của model mà không mở kiểm chứng.
- Phân biệt rõ ba loại căn cứ: *quy định chính thức* (văn bản Bộ), *tham khảo
  sư phạm* (chương trình Waldorf các nguồn), và *tư liệu văn hoá* (truyện).
  Không loại nào tự động là "đạt chuẩn" cho loại còn lại.

## 3. Watchlist model (kiểm ở pha 1, quyết ở pha 3)

Mỗi lần quét, trả lời ba câu hỏi cho các model đang dùng (hiện tại theo
`plugins/waldorf/specs/03_research_and_architecture.md`: Meta `muse-wonder-1.0`,
`muse-spark-1.3`, `muse-image-1.0`, `muse-voice-transcribe-1.0`):

1. Có model/capability mới nào làm một điểm nghẽn benchmark hiện tại tốt lên
   rõ rệt không — hay chỉ là "mới hơn"?
2. Có thay đổi contract, giá, giới hạn, hay deprecation nào chạm adapter hiện tại không?
3. Quyết định: **adopt** (đưa vào specs kèm cách đo lại) / **watch** (ghi điều
   kiện sẽ adopt) / **reject** (ghi lý do). Không có quyết định thì lần quét đó
   coi như chưa xảy ra.

## 4. Điều không làm

- Không quét tin tức AI tổng quát mỗi loop. Research phục vụ một điểm nghẽn
  benchmark cụ thể, hoặc một câu hỏi watchlist cụ thể — ngoài ra là nhiễu.
- Không sao chép nội dung nguồn vào specs. Specs ghi: nguồn, ngày, điều rút ra,
  và quyết định nó thay đổi. Nội dung gốc ở lại nguồn của nó, kèm provenance.
- Không dùng nguồn không mở lại được (link chết, ảnh chụp không nguồn) làm
  căn cứ cho một tuyên bố trong sản phẩm.
