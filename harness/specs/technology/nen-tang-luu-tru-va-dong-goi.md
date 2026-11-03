# Technology — Nền tảng, lưu trữ và đóng gói

> Các quyết định "chạy trên máy nào, dữ kiện nằm ở đâu trên đĩa, giam thực thi bằng gì, phát hành ra sao". Seam tương ứng trong core: persistence ([core/11](../core/11-luu-tru-va-khoi-phuc.md)), sandbox ([core/08, mục 2.5](../core/08-capability-seams.md)), composition ([core/12](../core/12-composition-va-cau-hinh.md)).

---

## 1. Hệ điều hành và thế giới thực thi

- Liệt kê OS đích (ví dụ: macOS trước, Windows/Linux sau) và với mỗi OS: shell mặc định cho seam `shell`, cách quản cây tiến trình con, và đường dẫn thư mục harness (home của ứng dụng: nơi chứa profile, session, bí mật cấp người dùng).
- **Thế giới thực thi phải nhất quán**: seam filesystem, subprocess và sandbox của một session phải cùng trỏ về một nơi (cùng máy, cùng vùng chứa, cùng máy từ xa). Trộn "file ở máy A, lệnh chạy ở máy B" cho cùng một session là lỗi cấu hình phải bị phát hiện sớm, không phải hành vi được hỗ trợ.
- Kết nối từ xa (SSH hoặc tương đương) là một *bộ ba provider đi cùng nhau* cho fs + subprocess + sandbox, đăng ký như một thế giới thực thi duy nhất.

## 2. Định dạng lưu session trên đĩa

Chọn backend đầu tiên cho seam persistence. Yêu cầu của core/11: mỗi session là một đơn vị độc lập, đọc được tiền tố hợp lệ, `stat/list` chỉ cần header, và có đường migration theo phiên bản.

| Lựa chọn | Mạnh | Yếu |
|---|---|---|
| **Một file log nối thêm mỗi session** (JSONL, có tuỳ chọn nén) *(tham chiếu)* | Đơn giản, đọc/ghi kiểu stream, dễ kiểm tra bằng mắt, hợp với append-only | Tìm kiếm toàn văn phải dựng chỉ mục riêng |
| SQLite (mỗi session một DB hoặc một DB chung có schema phiên bản đơn điệu) | Truy vấn/tìm kiếm tại chỗ, giao dịch tốt | Log append-only phải mô phỏng trên bảng; xuất/đọc thô kém trực quan hơn |
| Kho đối tượng/đám mây | Session dùng chung nhiều máy | Độ trễ và ngoại tuyến trở thành vấn đề của core phía trên; chỉ nên là provider thứ hai |

Quyết định đi kèm, ghi rõ:

- **Chính sách ghi**: cửa sổ gom batch khi ghi nền và điểm `flush` tường minh ở đâu (mặc định tham chiếu: gom theo cửa sổ thời gian cố định, flush tại biên turn và khi đóng session).
- **Nén**: có/không, và định dạng thế hệ file có mang phiên bản trong tên để migration ghi bản mới bên cạnh bản cũ (core/11, mục 4).
- **Chỉ mục phụ**: tìm kiếm session (toàn văn, theo thời gian) là một chỉ mục *suy ra được* từ log — xoá đi dựng lại được, không bao giờ là nguồn sự thật.
- **Vị trí trên đĩa**: thư mục theo dự án hay theo người dùng; quy ước đặt tên; cách một session được định vị khi xuất.

## 3. Sandbox theo hệ điều hành

Điền cho mỗi OS đích (hợp đồng chế độ và mức `full/partial` ở core/08, mục 2.5):

| OS | Cơ chế | Mức thực thi kỳ vọng | Ghi chú |
|---|---|---|---|
| macOS | Seatbelt qua `sandbox-exec` | | Xem thêm [ung-dung-macos-native.md](ung-dung-macos-native.md) — khác với App Sandbox của Apple |
| Linux | bubblewrap và/hoặc Landlock | Landlock trên kernel cũ chỉ là `partial` | |
| Windows | Token hạn chế + ACL | Thường `partial` | Khai rõ phần không giam được |
| (Không giam) | `danger-full-access` | — | Chỉ khi người dùng chọn tường minh ở tầng composition |

Nếu một OS không có cơ chế đạt mức yêu cầu tối thiểu của bạn: ghi rõ harness trên OS đó chạy ở chế độ nào và hiển thị sự thật đó cho người dùng thế nào — không lặng lẽ chạy không giam.

## 4. Đóng gói và phát hành

| Quyết định | Lựa chọn điển hình |
|---|---|
| Dạng phân phối lõi | Gói thư viện theo hệ sinh thái ngôn ngữ *(tham chiếu: gói npm theo workspace)* / một binary duy nhất / gói vào tài nguyên của app desktop đã ký |
| Kênh cài plugin | Registry công cộng của hệ sinh thái / thư mục plugin do người dùng quản / chỉ plugin đóng sẵn |
| Cập nhật lõi | Theo trình quản lý gói / tự cập nhật kèm app / không tự cập nhật |
| Tương thích ngược dữ liệu | Đọc được mọi định dạng đã phát hành qua chuỗi migration (core/11, mục 4); không hỗ trợ hạ cấp (ghi đời mới rồi mở bằng đời cũ thì đời cũ từ chối) |

## 5. Quyết định của lần này

- OS đích và thứ tự ưu tiên: macOS ≥ 14 (arm64) là đích duy nhất của v1; OS khác đỗ lại.
- Thư mục harness trên mỗi OS: `~/Library/Application Support/XuViHarness/` (profile, sessions, dữ liệu Waldorf, đính kèm). Workspace làm việc của agent là thư mục dự án người dùng chọn, tách khỏi thư mục dữ liệu này.
- Backend lưu session: mỗi session một file log JSONL append-only (đúng core/11: đọc được tiền tố hợp lệ, header đọc riêng được). Dữ liệu domain Waldorf: JSON Codable ghi nguyên tử (ghi file tạm → thay thế) kèm backup trước khi ghi; đây là cài đặt đầu tiên của seam persistence — chuyển SQLite qua đúng seam khi quy mô/truy vấn đòi, không sửa consumer.
- Chính sách ghi/flush/nén: append sự kiện theo lô nhỏ, flush tại biên turn/step và khi đóng session/thoát app; không nén ở v1 (định dạng mang phiên bản trong header để migration ghi bản mới bên cạnh).
- Sandbox theo OS (bảng mục 3 đã điền): macOS — Seatbelt `sandbox-exec` cho shell tool, mức kỳ vọng `partial`→`full` tuỳ hồ sơ giam, báo thành thật; các OS khác chưa hỗ trợ ở v1 nên không khai.
- Dạng phân phối + kênh plugin: SwiftPM package → script lắp `.app` ký ad-hoc; plugin chỉ là module đóng sẵn trong package (Waldorf), chưa có kênh cài plugin ngoài ở v1.
- Lý do một dòng: lưu trữ đơn giản nhất vẫn giữ bất biến log-là-sự-thật và khôi phục được; mọi nâng cấp đi qua seam, không qua sửa consumer.

## 6. Ảnh hưởng lên core

- Không đổi core. Backend lưu trữ cài seam ở core/11; cơ chế giam cài seam ở core/08.
- Nếu một OS thiếu khả năng mà một bất biến core cần (ví dụ không huỷ được cây tiến trình con): đó là giới hạn phải khai trong mục "giới hạn đã biết" của provider và hiển thị cho người dùng — không sửa bất biến core cho vừa một nền yếu.
