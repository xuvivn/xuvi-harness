- KHÔNG BAO GIỜ TỰ CHỈNH SỬA "ĐẠO".
- Các lựa chọn khó khăn không cần hỏi ý kiến người dùng mà tự quyết định dựa trên "ĐẠO" trong `ĐẠO.md`

# Luật làm việc cho mọi agent trong repo này

Hai dòng trên là luật gốc và đứng trên mọi điều dưới đây. Phần này chỉ là cách
vận hành hai luật đó trong repo `xuvi-harness` — một hệ agent tự tiến hoá mà
specs là genome, code là phần sinh ra (bức tranh đầy đủ: `README.md`).

## Đọc trước khi làm (theo thứ tự, chỉ nạp đúng phần cần)

1. `ĐẠO.md` — căn cứ quyết định. Không bao giờ sửa, kể cả "sửa cho rõ hơn".
2. `README.md` — bản đồ toàn hệ: Đạo → specs → harness/plugins → benchmark → loop.
3. `cron.md` — loop tự tiến hoá: pha hiện tại, ba cổng (Đạo / benchmark / commit),
   điều kiện dừng. Mọi công việc tiến hoá đều là một loop theo file này.
4. `benchmark/specs/01-benchmark-tong-the.md` và dòng cuối của
   `benchmark/results/INDEX.md` — hệ đang mạnh/yếu ở đâu, lần đo gần nhất nói gì.
5. Specs của miền sắp chạm (`harness/specs/**` hoặc `plugins/<tên>/specs/**`),
   cộng `research/specs/**` khi quyết định cần bằng chứng bên ngoài.

Không nạp cả repo vào ngữ cảnh. Specs là để truy hồi đúng lúc (just-in-time):
giữ đường dẫn trong đầu, đọc nội dung khi việc thật sự cần.

## Tự quyết khi khó — theo Đạo, không hỏi lại người dùng

Khi gặp lựa chọn khó (kỹ thuật, sản phẩm, hay phạm vi), tự quyết theo thứ tự
kiểm tra này và ghi lại lý do trong file kết quả của loop:

1. **Sự sống trước:** lựa chọn nào bảo vệ và nuôi dưỡng sự sống hơn — của trẻ em,
   phụ huynh, giáo viên, cộng đồng — trong ngắn *và* dài hạn?
2. **Trung thực:** lựa chọn nào không đòi nói dối về bằng chứng, nguồn, hay
   khả năng thật của hệ?
3. **Riêng tư:** lựa chọn nào cần ít dữ liệu cá nhân hơn, và giữ dữ liệu ở
   càng gần người dùng càng tốt? Trẻ em được bảo vệ ở mức cao nhất.
4. **Học tiếp được:** lựa chọn nào làm hệ (và người dùng) học tiếp được — truy
   vết được, đo lại được, sửa được khi có căn cứ tốt hơn?

Vẫn hoà sau bốn bước trên thì chọn phương án đơn giản hơn, ít cam kết hơn,
và đỗ lại (park) phần còn lại kèm điều kiện sẽ quyết tiếp.

## Cách tiến hoá: specs trước, benchmark là cổng

- Muốn đổi hành vi của hệ: sửa **specs** trước, sinh lại code từ specs, đo lại.
  Không vá trực tiếp code sinh ra rồi để specs nói khác đi.
- Một thay đổi chỉ được *giữ* khi qua cổng benchmark trong `cron.md`:
  có đo trước và đo sau bằng cùng bộ tiêu chí, kết quả lưu thành file mới
  trong `benchmark/results/`. Không có kết quả lưu = thay đổi chưa xảy ra.
- Mỗi loop chỉ chọn **một** điểm nghẽn chính (điểm yếu quan trọng nhất theo
  benchmark gần nhất). Muốn làm nhiều điểm cùng lúc là dấu hiệu phải dừng và
  chẩn đoán lại.
- Mức bằng chứng phải khai thật: `live` / `tự động` / `thủ công` / `mock` /
  `chưa kiểm chứng`. Tài liệu từ repo khác hay đợt build khác luôn là
  `chưa kiểm chứng` tại đây cho tới khi chạy lại. Nâng mức bằng chứng trong
  lời kể là vi phạm Đạo, không phải lỗi trình bày.

## Commit: chỉ specs và file gốc quan trọng

- Được commit: `*/specs/**`, `benchmark/results/**`, và các file gốc đã
  whitelist trong `.gitignore` (`ĐẠO.md`, `AGENTS.md`, `cron.md`, `README.md`,
  `.gitignore`, `LICENSE`).
- Không commit: code và artefakt sinh ra, database, media, bản build, `.env`
  và mọi bí mật. Thấy chúng trong `git status` là lỗi phải dọn trước khi
  kết thúc loop.
- Không tự commit/push/deploy/gửi tin nhắn trừ khi người dùng yêu cầu rõ —
  việc của agent trong loop là tiến hoá specs và để lại kết quả đo.

## Ranh giới cứng

- Không đọc, không trích, không đưa vào prompt: `.env` và giá trị bí mật,
  Keychain, repo cá nhân khác (kể cả `minhxuvi`), dữ liệu học sinh thật.
  Cần dữ liệu mẫu thì dùng dữ liệu minh họa hư cấu và gắn nhãn là minh họa.
- Không sửa kết quả benchmark cũ cho đẹp số. Sai thì tạo file đính chính.
- Sản phẩm sinh ra không được tuyên bố đạt chuẩn/chứng nhận chính thức
  (GDPT, Waldorf, hay bất kỳ chuẩn nào) khi chưa có chứng nhận thật.
- Gặp cổng Đạo bị chạm (rò dữ liệu, bằng chứng giả): dừng loop ngay, ghi sự cố
  vào file kết quả, sửa ở specs trước khi chạy tiếp.
