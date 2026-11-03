# 14 — Lộ trình xây dựng và kiểm thử

> Biến specs thành code theo từng giai đoạn. Mỗi giai đoạn cho ra một thứ **chạy được và kiểm chứng được**, xây trên giai đoạn trước. Đừng nhảy giai đoạn: kiến trúc này trả công cho thứ tự đúng và phạt nặng thứ tự sai (đặc biệt là làm UI trước khi có log).

Trước khi viết dòng code đầu tiên: điền [technology/quyet-dinh-hien-tai.md](../technology/quyet-dinh-hien-tai.md).

---

## 1. Các giai đoạn

### Giai đoạn 0 — Khung module tối thiểu

**Đọc:** [02](02-module-va-plugin.md) · **Công nghệ:** runtime & ngôn ngữ.

Xây: context/kho dịch vụ, inject, effect/disposer, dispatch sự kiện 5 chế độ, scope theo agent.

Xong khi: một plugin "hello" đăng ký được một dịch vụ và một listener; gỡ plugin thì cả hai biến mất. (Đây là toàn bộ giai đoạn. Đừng xây thêm.)

### Giai đoạn 1 — Session log trong bộ nhớ

**Đọc:** [04](04-session-log.md).

Xây: `SessionEvent`, `Session.append` (validate JSON + hợp đồng bề mặt), `deriveMessages()`, header yêu cầu trong bộ nhớ.

Xong khi: các tiêu chí nghiệm thu của file 04 pass bằng test thuần, chưa cần lưu đĩa, chưa cần model.

### Giai đoạn 2 — Model gateway với adapter giả

**Đọc:** [05](05-model-gateway.md).

Xây: từ vựng Message/ContentBlock/StreamChunk, assembler dùng chung, registry adapter. Adapter đầu tiên là **adapter phát lại** (replay): đọc một luồng chunk đã ghi sẵn từ fixture.

Xong khi: tiêu chí của file 05 pass mà chưa gọi bất kỳ API thật nào. Adapter thật đầu tiên của bạn cắm vào sau, ở giai đoạn 6, không sửa gì ở đây.

### Giai đoạn 3 — Agent loop + tool registry (bản tối thiểu)

**Đọc:** [03](03-agent-loop.md), [06](06-tools-va-thuc-thi.md) (chỉ mục 1–4, bỏ qua executor thay thế và spill), [07](07-prompt-va-ngu-canh.md) (mục 1–3).

Xây: inbox hai hàng, vòng turn/step, ghi log đầy đủ, pipeline thẳng (chưa guard/approval — nhưng để sẵn điểm waterfall), một tool mẫu (ví dụ `echo` hoặc đọc file đơn giản qua một bản cài fs tối thiểu), và bộ đo token của lần ghép (file 07, mục 6.1) ở mức báo cáo — chưa cần cơ chế nén.

Xong khi: dùng adapter phát lại, chạy được kịch bản "người hỏi → model gọi tool → model trả lời", log tái dựng đúng mọi yêu cầu, và đọc được số token theo loại của từng step. **Từ đây harness đã "sống".**

### Giai đoạn 4 — Mặt tiếp xúc tối thiểu (headless trước)

**Đọc:** [09](09-tuong-tac-nguoi.md) (mục 3), [12](12-composition-va-cau-hinh.md) (mục 1–3).

Xây: một profile headless (đọc một việc từ dòng lệnh, chạy một turn, in kết quả, thoát) + cơ chế profile/bundle/lớp vá tối thiểu + dump bản phối.

Xong khi: đổi một tool bằng lớp vá (không sửa code) và thấy hiệu quả ở lần chạy sau.

> Làm headless trước UI không phải vì UI kém quan trọng, mà vì headless bắt mọi tầng dưới phải đứng vững không cần giao diện chống lưng.

### Giai đoạn 5 — Persistence và khôi phục

**Đọc:** [11](11-luu-tru-va-khoi-phuc.md), [13](13-bao-mat-loi-va-quan-sat.md).

Xây: backend lưu trữ đầu tiên qua seam persistence, checkpoint flush, khôi phục turn bị crash, phiên bản định dạng v1 + khung migration.

Xong khi: các tiêu chí của file 11 pass, gồm cả test kill -9.

### Giai đoạn 6 — Adapter model thật + credentials

**Đọc:** [05](05-model-gateway.md) (mục 3–5), [08](08-capability-seams.md) (mục 2.11–2.12) · **Công nghệ:** ai-models, ai-model-runtime.

Xây: adapter cho provider đã chọn, seam credentials, phân giải metadata đúng-tuyến, chính sách retry.

Xong khi: chuyển provider/model bằng cấu hình; xoay bí mật có hiệu lực ngay yêu cầu kế tiếp.

### Giai đoạn 7 — Tools thật và an toàn thực thi

**Đọc:** [06](06-tools-va-thuc-thi.md) (toàn bộ), [08](08-capability-seams.md) (fs/shell/subprocess/sandbox), [09](09-tuong-tac-nguoi.md) (mục 1), [13](13-bao-mat-loi-va-quan-sat.md).

Xây: các seam thực thi + tool fs/shell thật, approval đầy đủ (hỏi/never), sandbox theo nền tảng đã chọn, giới hạn kết quả + spill.

Xong khi: tiêu chí của file 06, 09 (mục 1) và 13 pass — đặc biệt các test phủ định (đường vòng cũng bị chặn).

### Giai đoạn 8 — Mặt tiếp xúc chính

**Đọc:** [09](09-tuong-tac-nguoi.md) · **Công nghệ:** file giao diện tương ứng (web/macOS/chat…).

Xây: mặt chat (hoặc mặt đã chọn): hiển thị từ feed sự kiện của session + khung stream sống, thẻ tool từ presenter, hỏi approval/câu hỏi, lệnh của người.

Nguyên tắc hiển thị: **host presenter thuần** (dữ liệu → mô hình hiển thị, không I/O); thẻ trên UI suy ra từ sự kiện thô + metadata kết quả đã lưu, nên replay vẽ lại y hệt lúc sống.

Xong khi: mở lại một session cũ trên UI thấy đúng transcript và đúng thẻ tool như lúc chạy.

### Giai đoạn 9 — Phối hợp và mở rộng (theo nhu cầu, từng mục độc lập)

**Đọc:** [10](10-phoi-hop-va-mo-rong.md).

Thứ tự gợi ý: jobs → subagent → compaction (khi bắt đầu đau vì ngữ cảnh dài) → skills → goal/schedule → workflow. Mỗi mục là một thay đổi độc lập, không mục nào là tiền đề của mục nào (trừ workflow dùng subagent). Riêng bộ ba ngữ cảnh dài hạn (file 10, mục 6) thì làm theo thứ tự rẻ trước: cắt kết quả tool và ghi chú có cấu trúc không cần provider mới, compaction bằng model làm sau cùng khi hai kỹ thuật kia không đủ.

### Giai đoạn 10 — Cứng hoá

Tìm kiếm toàn văn trên session, xuất/nhập, telemetry qua seam, quản lý plugin lúc chạy, kiểm tra tương thích phiên bản plugin, nạp nóng. Đọc lại [12](12-composition-va-cau-hinh.md) và [13](13-bao-mat-loi-va-quan-sat.md) và đóng mọi tiêu chí còn mở.

---

## 2. Chiến lược kiểm thử (áp cho mọi giai đoạn)

1. **Replay là xương sống kiểm thử.** Mọi kịch bản hội thoại đều kiểm thử được không cần API thật: ghi một phiên với model thật một lần, sau đó phát lại qua adapter replay trong test tự động. Test hành vi không được phụ thuộc mạng/khoá API.
2. **Ba tầng test:**
   - *Unit*: hàm thuần — gấp bề mặt, ghép stream, phân giải chính sách, projection.
   - *Composition thật*: boot bản phối thu nhỏ qua đúng cơ chế nạp profile (không dựng tay từng plugin trong test) và khẳng định đầu ra model thấy được / bền vững / người dùng thấy được. Chỉ mock dịch vụ ngoài và đầu vào không xác định.
   - *Đầu-cuối bằng model thật*: ít, có khoá mới chạy, không có khoá thì tự bỏ qua — không bao giờ là cổng duy nhất.
3. **Snapshot đầu ra**: thay đổi nào chạm vào thứ model hoặc người dùng thấy được đều cập nhật một snapshot phiên phát lại tương ứng. Test mô tả **hành vi**, không mô tả "tính đúng" chung chung.
4. **Test huỷ và teardown**: mọi đăng ký phải có test "huỷ fiber/effect → đóng góp biến mất"; mọi test chiếm tài nguyên (cổng mạng, thư mục tạm, tiến trình con) tự dọn trong teardown và chạy được song song với test khác.
5. **Test phủ định cho ranh giới an toàn** (file 13, mục 6) là bắt buộc, không phải tuỳ chọn.
6. **Đánh giá thay đổi ngữ cảnh như đánh giá hành vi, không bằng cảm tính.** Mọi thay đổi chạm prompt, mô tả/tham số tool, chiến lược truy hồi hoặc prompt tóm tắt nén đều chạy qua một bộ việc thật nhỏ trước khi nhận:
   - Bắt đầu sớm với khoảng hai chục việc đại diện cho cách dùng thật (việc phải đủ khó, cần nhiều lần gọi tool); bộ nhỏ vẫn thấy được thay đổi lớn, đừng chờ có hàng trăm việc mới bắt đầu đo. Giữ một tập giữ riêng (held-out) không dùng khi tinh chỉnh để phát hiện học vẹt theo tập đánh giá.
   - Đo cùng lúc: tỉ lệ hoàn thành việc, số token của step đầu và tổng token, số lần gọi tool (gồm lần gọi thừa), tỉ lệ lỗi tool/tham số, và — với nén — tỉ lệ dữ kiện quan trọng còn giữ lại sau nén (recall trước, độ gọn sau; file 10, mục 6.1).
   - Đọc transcript của các lần hỏng để tìm kiểu thất bại cụ thể, rồi sửa đúng chỗ sở hữu nó: chọn sai tool thì sửa định nghĩa tool (file 06, mục 2), thiếu dữ kiện thì sửa chiến lược truy hồi (file 07, mục 5), chỉ thị mơ hồ thì sửa mảnh prompt (file 07, mục 2.1). Không sửa một chỗ bằng cách nhồi thêm chữ vào chỗ khác.
   - Việc có nhiều đường đi đúng thì chấm theo kết quả cuối và tính hợp lý của quá trình, không ép đúng một dãy step đã định trước.

---

## 3. Bảng tra cứu: tính năng → file spec

| Muốn làm | Đọc core | Điền technology |
|---|---|---|
| Đổi ngôn ngữ/framework | 02 | runtime-va-ngon-ngu |
| Thêm nhà cung cấp model | 05 | ai-models, ai-model-runtime |
| Thêm một tool | 06, 08 | chat-va-tools-integration |
| Nối MCP/server ngoài | 08 (2.9), 06 | chat-va-tools-integration |
| Làm app macOS native | 09, 12 | ung-dung-macos-native, nen-tang-luu-tru-va-dong-goi |
| Lưu session bằng DB khác | 11, 04 | nen-tang-luu-tru-va-dong-goi |
| Chạy model local | 05 | ai-models, ai-model-runtime |
| Nhiều agent làm việc cùng nhau | 10 | ai-models (model cho agent con) |
| Tối ưu prompt / tool / ngữ cảnh (token cao, chọn sai tool, việc dài mất mạch) | 07 (mục 1, 2, 5, 6), 06 (mục 2), 10 (mục 6) | ai-model-runtime (bộ đếm token, model tóm tắt), quyet-dinh-hien-tai (chiến lược ngữ cảnh) |
| Tự động chạy theo lịch/mục tiêu | 10 (mục 4–5), 09 | quyet-dinh-hien-tai |

---

## 4. Dấu hiệu đi sai đường (dừng lại và đọc lại spec)

- Bạn đang sửa agent loop để thêm một tính năng mới → thiếu một điểm cắm (file 01, mục 3.1; file 10).
- Bạn lưu lịch sử hội thoại ở hai nơi và đang viết code đồng bộ chúng → vi phạm file 04.
- Một tool kiểm tra quyền bằng cách tự hỏi UI → quyền phải qua approval/pipeline (file 06, 09).
- Bạn thêm `if (provider == "…")` trong vòng lặp → rò rỉ công nghệ vào core (file 05).
- Test của bạn cần API thật và khoá thật mới chạy → thiếu adapter replay (giai đoạn 2).
