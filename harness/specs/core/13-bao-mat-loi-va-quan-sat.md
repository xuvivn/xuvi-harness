# 13 — Bảo mật, lỗi, giới hạn và quan sát

> File này là các quy tắc cắt ngang mọi file khác. Nó không thêm tính năng; nó định nghĩa cách mọi tính năng phải *thất bại an toàn*.

---

## 1. Ranh giới tin cậy: validate ở đâu

| Dữ liệu | Tin cậy? | Việc phải làm |
|---|---|---|
| Tham số tool do model sinh (JSON) | Không | Parse + validate theo schema tại pipeline; không bao giờ đưa thẳng vào shell/path |
| Nội dung file đọc từ đĩa, output của tiến trình con, phản hồi mạng/MCP | Không | Coi là dữ liệu, không phải chỉ thị; giới hạn kích thước |
| Log đã lưu đọc lại từ đĩa | Không hoàn toàn | Validate envelope, tính liên tục của seq, hợp đồng bề mặt trước khi dựng session |
| Config của người dùng trên máy | Một phần | Validate theo schema lúc nạp, fail loud |
| Giá trị cùng tiến trình đã qua hệ thống kiểu | Có | **Không** validate thừa, không viết test "đầu vào thù địch" cho thứ kiểu đã cấm |

Nguyên tắc: tin vào kiểu ở ranh giới cùng-tiến-trình đã có kiểu; kiểm chứng ở mọi ranh giới parser/config/hàng đợi/model/file/wire/tiến trình con.

---

## 2. An ninh thực thi

1. **Sandbox mặc định cho thực thi của model.** Lệnh/tool chạy dưới chính sách sandbox đã phân giải (file 08, mục 2.5); chế độ không-giam (`danger-full-access`) chỉ đến từ quyết định tường minh của người dùng ở tầng composition, không phải mặc định lén trong code.
2. **Approval thực thi tại nơi ra quyết định.** Giấu tool khỏi prompt, lọc schema, hay bọc wrapper đều *không phải* thực thi chính sách — nơi gọi vòng được hết. Kiểm soát phải nằm trong chính thao tác/thực thi (pipeline tool, thao tác ghi file), và test phải chứng minh được đường gọi trực tiếp cũng bị chặn.
3. **Fail-closed khắp nơi**: thiếu người trả lời approval → từ chối; thiếu provider cho một thao tác → lỗi; chính sách không phân giải được → từ chối, không phải cho phép.
4. **Bí mật**: chỉ đi qua seam credentials (file 08, mục 2.11); không ghi bí mật vào log, config, telemetry hay thông báo lỗi. ID mờ (opaque id) không phải bí mật — an ninh dựa trên kiểm soát quyền của chủ sở hữu, không dựa trên việc giấu id.
5. **Đường dẫn**: thao tác file phải giữ trong gốc cho phép (kiểm tra chứa đựng sau khi phân giải symlink, kiểm tra lại metadata tại thời điểm thao tác) — đặc biệt mọi đường dẫn do model hoặc người dùng từ xa đưa vào.

---

## 3. Giới hạn (bounds) — áp trên toàn bộ kết quả

Mọi giới hạn (byte, token, số mục, thời gian) phải áp tại nơi **toàn bộ giá trị phát ra/lưu giữ đã biết**, tính cả phần bọc và metadata:

- Kết quả tool: chặn trên bản hoàn chỉnh sau pipeline (file 06, mục 4).
- Toàn bộ lần gọi model: system prompt, tool schemas, lịch sử và ngữ cảnh chạy cộng lại phải vừa ngân sách của step sau khi trừ phần dự trữ cho đầu ra (file 07, mục 6). Giới hạn này áp tại bộ ghép — nơi toàn bộ yêu cầu đã biết — không áp rời rạc từng mảnh.
- Stream từ model: stall timeout + trần output của model.
- Kết quả đọc file/truy vấn: cửa sổ có trang, không nạp vô hạn vào bộ nhớ/ngữ cảnh.
- Hàng đợi (inbox, job, sự kiện): có giới hạn khai báo; tràn thì xử lý theo chính sách đã khai (từ chối/tràn ra đĩa) chứ không phình vô hạn.

Test giới hạn phải gồm: vừa dưới hạn, đúng hạn, vượt hạn bởi một đơn vị, và đầu vào một-mảnh-khổng-lồ (một chunk duy nhất vượt hạn) cùng giới hạn tính theo byte với ký tự đa byte.

---

## 4. Lỗi: có kiểu, có cấu trúc, có chủ

1. **Lỗi là dữ liệu có cấu trúc** (mã máy-đọc-được + thông điệp cho người + ngữ cảnh), không phải chuỗi tự do để nơi khác đoán. Consumer rẽ theo mã, không theo câu chữ (file 05, mục 1.3).
2. **Mỗi lỗi có một chủ xử lý rõ ràng** và một cam kết: việc phụ thuộc dừng lại, trạng thái không tin cậy được báo ra mặt, kết cục không rõ thì nói là không rõ — không nuốt lỗi thành "chắc là ổn".
3. **Không xuất bản trạng thái trước điểm chốt.** Chỉ phát thông báo/cập nhật trạng thái suy ra *sau khi* thao tác thành công; cache, prompt, UI echo, replay đều suy ra từ cùng một nguồn sự thật sau chốt.
4. **Một thao tác bất đồng bộ = một bộ điều khiển vòng đời.** Readiness, huỷ, dispose, đặt chỗ… nếu không có chủ/điểm lắng riêng thật sự cần thiết thì gấp vào một transaction duy nhất, giữ tính rollback và lắng hoàn toàn khi teardown.
5. **Catch rỗng phải gọi tên được lỗi và lý do bỏ qua**, và `try` của nó chỉ chứa đúng một câu lệnh. Mọi catch khác phải hoặc xử lý thật, hoặc bọc lại kèm ngữ cảnh rồi ném tiếp.
6. Lỗi cấu hình/khởi động: fail loud (file 01, mục 3.6). Lỗi lúc chạy trong một turn: trở thành kết quả có cấu trúc của turn đó (file 03), không giết tiến trình — trừ lỗi phá vỡ bất biến của log, thứ phải dừng việc phụ thuộc và báo rõ trạng thái không tin cậy.

---

## 5. Quan sát (observability)

- **Telemetry là một seam**: các kênh báo cáo (sự kiện sử dụng, dấu vết session) đăng ký như provider; gắn module telemetry không tự nó thu thập gì — chỉ sự kiện được gửi tường minh mới đi ra.
- Mọi dữ liệu rời tiến trình đều qua một điểm thu và **loại bỏ bí mật/nội dung nhạy cảm** trước khi gửi; session log đầy đủ chỉ rời máy theo một chính sách tải lên tường minh mà người dùng kiểm soát.
- **Chẩn đoán tại chỗ**: mọi thất bại của adapter/plugin/persistence phải lần được về (plugin nào, session nào, seq nào, request id của provider) từ log chẩn đoán cục bộ mà không cần gửi gì ra ngoài.
- Đo hiệu năng (thời gian tới token đầu, độ dài lịch sử…) suy ra từ chính log/stream đã ghi — không dựng hệ đo song song lệch nguồn.
- Đo ngữ cảnh cũng vậy: token theo từng loại của mỗi step tính lại được từ log bằng bộ đo (file 07, mục 6.1; bản chụp sẵn trong `request/context` khi vượt ngưỡng), số lần gọi tool và tỉ lệ lỗi tool, kích thước kết quả tool, và các lần nén (đoạn bị che, tỉ lệ rút gọn) đều suy ra từ `tool/result` và sự kiện nén đã ghi. Đây là dữ kiện đầu vào bắt buộc khi chỉnh prompt/tool (file 06, mục 2.4; file 14, mục 2) — một thay đổi ngữ cảnh không kèm số đo trước/sau là thay đổi chưa kiểm chứng.

---

## 6. Kiểm thử các bất biến an toàn

Mỗi thay đổi chạm tới ranh giới tin cậy phải kèm test phủ định (test chứng minh bị chặn), ví dụ:

- Tool bị deny chạy được bằng đường gọi vòng → test phải đỏ.
- Approval `never` bị một người trả lời đăng ký muộn qua mặt → test phải đỏ.
- Kết quả tool lớn hơn giới hạn đúng 1 byte → bị cắt/tràn đúng chính sách.
- Log bị sửa tay (nhảy seq) → reader từ chối, không resume.

---

## 7. Tiêu chí nghiệm thu

- [ ] Rà soát mọi đường thực thi của model: mỗi đường đều qua sandbox đã phân giải và approval tại điểm thực thi; liệt kê được từng đường và chốt của nó.
- [ ] Không tìm thấy giá trị bí mật nào trong một session log mẫu, một bản dump cấu hình, và một payload telemetry mẫu.
- [ ] Tắt (kill -9) ở các điểm ngẫu nhiên trong một turn: hệ thống hoặc hoàn tất đúng, hoặc khôi phục đúng theo file 11 — không có trạng thái "nửa vời im lặng" nào không được báo cáo.
