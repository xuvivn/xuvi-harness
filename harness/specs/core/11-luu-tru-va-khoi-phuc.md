# 11 — Lưu trữ bền vững và khôi phục

> File 04 định nghĩa log *trong bộ nhớ*. File này định nghĩa cách log sống sót qua tắt máy, crash và nâng cấp — qua một seam persistence duy nhất. Định dạng vật lý cụ thể (JSONL, SQLite…) là quyết định công nghệ: xem [technology/nen-tang-luu-tru-va-dong-goi.md](../technology/nen-tang-luu-tru-va-dong-goi.md).

---

## 1. Seam lưu session

```
Service Definition: SessionPersistence
   create(...) / open(id, 'read'|'write') / stat(id) / list()
        │
        ▼ trả về
   SessionHandle (một kênh mở duy nhất tới một session đã lưu):
   read(offset, length) / append(events) / flush() / close()
Provider: một backend cụ thể (mỗi session một artifact, một DB…)
Consumer: agent loop, truy vấn session, xuất bản…
```

Nguyên tắc:

1. **Không có kiểu sự kiện lưu trữ song song.** Backend lưu đúng `SessionEvent` của file 04. Thêm một biểu diễn lưu trữ thứ hai là tạo ra hai nguồn sự thật.
2. **Mọi đọc/ghi đều qua handle**, không qua phương thức đánh địa chỉ bằng id trên dịch vụ. Handle là cánh cửa duy nhất mà quyền sở hữu ghi độc quyền đi qua.
3. **Một chủ ghi duy nhất trong tiến trình**: mở `write` lần hai trên cùng session khi chủ cũ còn sống → lỗi `AlreadyOwned`. Đọc (`read`) thì tự do.
4. **Đọc không bao giờ thấy đuôi rách**: phần vật lý ghi dở của một lần append chưa hoàn tất không bao giờ xuất hiện trong kết quả đọc; đọc lặp trên cùng handle không bao giờ thấy trạng thái cũ hơn lần đọc trước.
5. **`flush()` là hàng rào bền vững duy nhất**: `append` resolve chỉ hứa "đã nhận, đã xếp thứ tự, đọc trong cùng backend thấy được"; chỉ `flush` resolve mới hứa sống sót qua crash.
6. Session mới tạo có thể chưa vật chất hoá trên đĩa cho tới lần append/flush đầu — một session "chưa từng vật chất hoá" khi crash thì coi như chưa từng tồn tại.

---

## 2. Đánh dấu kiểm soát ghi (checkpoint) lúc chạy

- Sự kiện được ghi vào log trong bộ nhớ **đồng bộ**; backend nghe feed sự kiện và gom vào cửa sổ ghi nền có giới hạn thời gian, không chặn người ghi.
- `flush(session)` huỷ chờ, xả hết qua trạng thái lắng, và là điểm consumer quan sát lỗi ghi. Vòng lặp dùng nó làm mốc thứ tự trước khi nhận turn thường tiếp theo.
- Một lần ghi nền thất bại: giữ nguyên các sự kiện theo thứ tự, tạm dừng đường tự động, báo log rõ ràng; lần `flush` tường minh kế tiếp thử lại và **báo lỗi lớn tới người gọi**.
- Đóng session (dispose) xả lần cuối rồi mới đóng handle — teardown không được làm mất sự kiện đã ghi.

---

## 3. Khôi phục sau crash

Một log chết giữa turn sẽ kết thúc bằng `turn/start` không có `turn/end`. Quy tắc:

1. **Persistence không tự sửa log.** Một turn có thể rất lớn; các sự kiện của nó đã ghi bền trước khi crash và phải giữ nguyên. Persistence chỉ trả về tiền tố vật lý hợp lệ liên tục (vứt đúng mảnh rách cuối cùng chưa từng hoàn tất; nếu mảnh rách vẫn giải mã được một phần bản ghi hoàn chỉnh thì đường ghi viết lại chúng bền vững trước lần append mới đầu tiên).
2. **Sửa là việc của reader có quyền ghi.** Khi resume, vòng lặp đọc log qua handle ghi, tính các sự kiện đóng còn thiếu — kết quả tool lỗi cho các call mồ côi, `step/end` nếu thiếu, và `turn/end {interrupted}` — rồi append chúng như một batch thường **trước khi** công bố session.
3. Reader chỉ-đọc (trình duyệt/tìm kiếm session) áp dụng đúng các sự kiện đóng đó **trong bộ nhớ**, không ghi ngược lại đĩa.
4. Mọi chốt mở do plugin tự sở hữu (ví dụ khoá compaction) không được vòng lặp đóng hộ: chúng được phân biệt "của lifecycle đã chết" nhờ mốc `session/end-seed` (file 04, mục 6), và chỉ chủ sở hữu của chúng xử lý.

---

## 4. Header và phiên bản định dạng

Mỗi session lưu kèm một **header** ngoài log (metadata lưu trữ: phiên bản định dạng, thư mục làm việc gốc, có phải fork không…) — header không phải trạng thái hội thoại và không replay.

Chính sách phiên bản (bắt buộc phải có từ ngày đầu, đừng để sau):

1. Mỗi định dạng đã **phát hành** có một phiên bản; backend từ chối đọc phiên bản *tương lai* (mới hơn bản đang chạy hiểu).
2. Đọc một session cũ: chọn thế hệ lưu trữ mới nhất, dịch qua một **chuỗi migration kề nhau** (`vN → vN+1`, mỗi bước một module sở hữu duy nhất) trong bộ nhớ. Đọc-để-đọc không ghi lại; lần mở-ghi đầu tiên mới ghi ra thế hệ kế tiếp **bên cạnh** bản cũ — không bao giờ di chuyển/ghi đè/xoá thế hệ đã cam kết.
3. Tăng phiên bản chỉ khi các discriminator hiện có không ngăn được cách hiểu sai nguy hiểm; thay đổi cùng phiên bản phải an toàn với reader cũ.
4. `stat`/`list` phải làm được chỉ từ header (không nạp toàn bộ sự kiện).

---

## 5. Truy vấn và xuất (đọc lạnh)

- Một tầng truy vấn trên seam này phục vụ: liệt kê/đọc bounded một session không cần agent sống, lần theo lineage fork, tìm kiếm toàn văn trên corpus session.
- Đọc lạnh dùng cache theo *token thay đổi* của `stat` (revision): token không đổi → không đọc lại đĩa.
- Xuất (export) một session đi qua cùng handle đọc — định dạng xuất là tầng trình bày, không phải định dạng lưu trữ thứ hai.

---

## 6. Các kho phụ trợ

| Kho | Hợp đồng |
|---|---|
| **Attachments** | Nội dung nhị phân (ảnh, file) lưu theo định danh nội dung (content-addressed), kiểm chứng khi nhận; session chỉ mang tham chiếu. Đường đọc của adapter model phân giải tham chiếu thành nội dung native của provider tại lúc gọi. |
| **Spill** | Kết quả tool quá lớn: lưu toàn văn ra kho phụ, log chỉ giữ tham chiếu + phần đầu. Tham chiếu và quyết định tràn đều nằm trong log để replay giống hệt. |
| **Storage hub** | Dữ liệu phụ không-phải-session (file 08, mục 2.10). |
| **Projection cache** | Cache trạng thái đã gấp (file 10, mục 10) để mở session dài không phải gấp lại từ đầu; luôn xoá được và dựng lại được từ log, keyed theo revision của log. |

---

## 7. Tiêu chí nghiệm thu

- [ ] Giết tiến trình giữa một turn dài (kill -9): mở lại, log giữ nguyên mọi sự kiện đã flush; resume bổ sung đúng các sự kiện đóng còn thiếu và chạy tiếp được.
- [ ] Mở một session đang có agent sống bằng handle ghi thứ hai → lỗi `AlreadyOwned` rõ ràng; mở đọc vẫn được.
- [ ] Viết một fixture log ở định dạng phiên bản cũ: reader hiện tại đọc được; sau một lần ghi, thế hệ mới xuất hiện bên cạnh và bản cũ còn nguyên.
- [ ] Một kết quả tool khổng lồ: log nhẹ, toàn văn nằm ở kho spill, replay cho model thấy đúng bản đã thay thế.
- [ ] Xoá projection cache: mọi trạng thái suy ra dựng lại giống hệt.
