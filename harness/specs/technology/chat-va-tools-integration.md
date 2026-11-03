# Technology — Mặt chat và tích hợp tools

> File này quyết định **trải nghiệm hội thoại cụ thể**: mặt tiếp xúc nào, tool nào xuất hiện trong đó, tool ngoài nối ra sao, và chúng trình bày thế nào. Cơ chế bên dưới — tool registry, pipeline, approval, hiển thị từ log — nằm ở [`core/06`](../core/06-tools-va-thuc-thi.md), [`core/08`](../core/08-capability-seams.md), [`core/09`](../core/09-tuong-tac-nguoi.md) và không đổi theo file này.

---

## 1. Mặt tiếp xúc

| Lựa chọn | Mô tả | Khi nào chọn |
|---|---|---|
| Headless một-lần-chạy *(tham chiếu: profile `headless`)* | Nhận một việc, chạy, in kết quả, thoát | Tự động hoá, CI, kiểm thử; cũng là mặt nên làm đầu tiên khi tự xây (core/14, giai đoạn 4) |
| Web chat *(tham chiếu: profile `web`)* | Host phục vụ một ứng dụng chat trong trình duyệt: danh sách session, transcript, stream trực tiếp, thẻ tool, hộp hỏi approval | Sử dụng hằng ngày đa nền tảng |
| Ứng dụng desktop gói cùng host đó | Host + cửa sổ native bọc web client | Xem [ung-dung-macos-native.md](ung-dung-macos-native.md) nếu cần native thật |
| Nhúng qua SDK *(tham chiếu: profile `sdk`)* | App khác điều khiển harness qua giao thức máy-máy | Harness là một phần của sản phẩm khác |
| Kênh chat ngoài (bot trên nền tảng nhắn tin…) | Một adapter mặt tiếp xúc: tin nhắn nền tảng ↔ inbox của agent; approval bằng nút bấm của nền tảng | Khi người dùng sống trong nền tảng đó. Lưu ý: approval qua kênh ngoài vẫn fail-closed khi không trả lời được |

Bất biến cho mọi mặt: **mặt tiếp xúc không sở hữu session**. Nó hiển thị từ feed sự kiện của session + khung stream sống (core/03, mục 7), và gửi ý định qua agent (followup/steer/cancel/chọn model…). Tắt mặt tiếp xúc, agent vẫn chạy xong việc của nó.

## 2. Bộ tool bật trong chat

Khai báo bộ tool mặc định theo *nhóm khả năng* (mỗi nhóm là một seam ở core/08). Đánh dấu bật/tắt và ghi preset quyền đi kèm:

| Nhóm tool | Ví dụ tool | Mặc định lần này | Yêu cầu approval? |
|---|---|---|---|
| File | đọc / ghi / sửa / liệt kê / tìm | | ghi & sửa: theo chính sách |
| Shell | chạy lệnh, terminal bền | | theo preset |
| Web | tìm kiếm / lấy trang | | thường không |
| Lập trình | LSP, chạy test qua shell | | như shell |
| Việc & phối hợp | job nền, todo, subagent, workflow | | subagent: theo cấu hình |
| Hỏi người | hỏi người dùng, xin approval | | — |
| Skill | nạp skill | | không |
| Ngoài (MCP) | xem mục 3 | | như tool thường |

- **Preset quyền** (đóng gói sẵn tổ hợp chính sách cho người dùng chọn): ví dụ `hỏi-trước-khi-ghi` · `tự-do-trong-workspace` · `chỉ-đọc`. Preset chỉ là dữ liệu cấu hình trên các cơ chế của core/06 + core/09 — không phải chế độ riêng trong code.
- Tool **không** bật mặc định vẫn đăng ký được vào registry; chỉ là không nằm trong bản phối của profile/preset này (core/12).
- Bảng trên là một **quyết định ngân sách**, không chỉ là danh sách tính năng: mỗi nhóm bật thêm tốn token schema ở mọi step và thêm khả năng chọn sai tool. Bật theo nguyên tắc bộ tối thiểu khả thi của core/06, mục 2.1, và ghi trần token cho kết quả tool vào ô quyết định #15 của [quyet-dinh-hien-tai.md](quyet-dinh-hien-tai.md).

## 3. Tích hợp tool ngoài (MCP và tương đương)

Điền cho mỗi nguồn ngoài:

| Trường | Điền gì |
|---|---|
| Tên server | Tiền tố đặt cho tool của nó |
| Vận chuyển | Tiến trình con stdio / HTTP streaming |
| Lệnh/endpoint + môi trường | Không chứa bí mật dạng văn bản trần — bí mật qua tham chiếu (ai-model-runtime, mục 3) |
| Chính sách khởi động lỗi | Chặn bật tính năng / bật không có server đó và báo rõ (chọn một; mặc định tham chiếu: báo rõ, giữ các tool đã khám phá được của lần nối tốt gần nhất là **không** — kết nối hỏng thì làm mới thất bại nhưng giữ thế hệ tool trước trong phiên đang chạy) |
| Thời hạn mỗi yêu cầu | Số ms |

Nhắc lại bất biến (core/08, mục 2.9): tool ngoài đi qua đúng pipeline và approval như tool nhà; ngắt server thì tool của nó biến mất sạch.

## 4. Trình bày trong chat

- Mỗi tool tự khai presenter thuần (core/06, mục 1, quy tắc 5): thẻ đang-chờ từ args, thẻ hoàn thành từ args + kết quả đã lưu. File này chỉ quyết định **phong cách**: thẻ thu gọn/mở sẵn, hiển thị diff dạng nào, output dài thì cắt ở đâu trên UI (việc cắt hiển thị không thay dữ kiện trong log).
- Stream: hiển thị chunk sống nhưng trạng thái cuối cùng luôn lấy từ sự kiện đã chốt trong log; mở lại session phải vẽ lại y hệt (kiểm chứng ở core/14, giai đoạn 8).
- Sự kiện plugin vẽ thành một nút hội thoại riêng phải mang id nghiệp vụ ổn định (core/04, mục 7) — khai báo yêu cầu này cho mọi tính năng bạn thêm vào mặt chat.

## 5. Quyết định của lần này

- Mặt tiếp xúc chính (và phụ, nếu có): chính — app macOS native SwiftUI (chat/agent + màn hình Waldorf); phụ — headless CLI (một việc rồi thoát, cho test/dogfood).
- Bộ tool mặc định (bảng mục 2 đã điền): File (đọc/glob/grep tự do; ghi/sửa qua approval), Shell (qua approval + sandbox-exec), Việc & phối hợp (todo bật; **subagent bật chỉ cho orchestrator**, preset worker không có tool này nên độ sâu tối đa 1; job/workflow tắt ở v1), Hỏi người/approval bật, Skill tắt ở v1, Web tắt (sản phẩm chỉ gọi `api.meta.ai`).
- Preset quyền mặc định: `ask` cho ghi-file/chạy-lệnh trong app; preset worker kế thừa và chỉ được bớt quyền (worker research = chỉ đọc). Headless dogfood dùng `never` giới hạn trong bản sao workspace dùng-một-lần.
- Nguồn tool ngoài: không ở v1.
- Phong cách thẻ tool (một dòng): thẻ thu gọn theo presenter thuần (đang chờ từ args, hoàn thành từ args + kết quả đã lưu), cây uỷ quyền orchestrator → worker hiển thị model + trạng thái + bản chưng cất, mở lại session vẽ lại y hệt.
- Lý do một dòng: bộ tool tối thiểu đủ để agent tự lập trình và tự tiến hoá (gồm uỷ quyền có trần), không tool nào bật chỉ để cho đủ danh sách.

## 6. Ảnh hưởng lên core

- Không đổi core. Bật/tắt tool và preset là dữ liệu composition (core/12); mặt tiếp xúc là consumer của agent/session (core/03, 04).
- Nếu một mặt tiếp xúc cần dữ kiện hiển thị mà log không có: thêm **loại sự kiện** theo core/04, mục 7 — không tạo kênh dữ liệu riêng giữa host và client.
