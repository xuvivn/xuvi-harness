# Technology — Ứng dụng macOS native

> File này chỉ áp dụng khi mặt tiếp xúc là **ứng dụng macOS native**. Nó quyết định công nghệ cửa sổ, kênh nói chuyện giữa phần native và lõi harness, và các nghĩa vụ với hệ điều hành (sandbox, ký số, quyền riêng tư). Lõi harness không biết nó đang chạy trong app native — nó chỉ chạy như một profile như mọi profile khác ([core/12](../core/12-composition-va-cau-hinh.md)).

Nếu mặt tiếp xúc của bạn là web/headless: bỏ qua file này.

---

## 1. Chia trách nhiệm: native sở hữu gì, harness sở hữu gì

| Phía native (shell của app) | Phía harness (lõi) |
|---|---|
| Cửa sổ, menu, khay trạng thái, thông báo hệ thống, phím tắt toàn cục | Agent, session, model, tool (toàn bộ core/) |
| Vòng đời app: mở, ẩn, thoát, mở lại cửa sổ | Vòng đời agent — độc lập với cửa sổ: đóng cửa sổ không giết việc đang chạy |
| Hộp thoại hệ thống: chọn thư mục/file, xin quyền truy cập | Quyết định *khi nào cần* hỏi (qua các seam approval/questions ở core/09) |
| Cập nhật ứng dụng, mở file bằng app, deep link | Không biết các thứ này tồn tại; nhận ý định đã chuẩn hoá qua một kênh vào |

Nguyên tắc: phần native là một **mặt tiếp xúc** theo đúng nghĩa ở [chat-va-tools-integration.md](chat-va-tools-integration.md), mục 1 — cộng thêm nghĩa vụ với OS. Nó không giữ nguồn sự thật hội thoại thứ hai.

## 2. Các lựa chọn công nghệ

### 2.1. Công nghệ cửa sổ/UI

| Lựa chọn | Mạnh | Yếu |
|---|---|---|
| SwiftUI/AppKit thuần native | Trải nghiệm macOS chuẩn nhất, tích hợp OS sâu nhất, dung lượng nhỏ | UI hội thoại phức tạp (stream, markdown, thẻ tool) phải tự dựng; tách hẳn khỏi web client nếu sau này cần web |
| WKWebView bọc lại web client của harness | Dùng lại toàn bộ UI chat đã xây cho web | Trải nghiệm "native" ở mức trung bình; cầu nối JS↔native là một bề mặt phải bảo trì |
| Framework desktop đa nền (Electron/Tauri kiểu) *(tham chiếu: DeepSeek Harness dùng Electron, đóng gói lõi vào resources đã ký, với một profile riêng do app sở hữu)* | Một codebase cho nhiều OS; trưởng thành | Dấu ấn native thấp hơn; nghĩa vụ ký/notarize và cập nhật vẫn như app native thật |

### 2.2. Kênh native ↔ lõi

| Lựa chọn | Ghi chú |
|---|---|
| Lõi chạy **trong cùng tiến trình** app (thư viện/engine nhúng) | Đơn giản nhất khi cùng ngôn ngữ; crash của lõi là crash của app — phải cân nhắc |
| Lõi chạy **tiến trình con** do app sinh, nói chuyện qua IPC/socket cục bộ có xác thực *(tham chiếu)* | Lõi độc lập vòng đời và ngôn ngữ; app khởi động lõi kèm một profile riêng của app, chờ tín hiệu sẵn-sàng rồi mới nối UI. Thêm nghĩa vụ: quản lý tiến trình con, cổng cục bộ, token xác thực kênh |
| Lõi là server cục bộ độc lập, app chỉ là client | Hợp khi muốn nhiều client (web + native) dùng chung một lõi đang chạy |

Yêu cầu cho kênh, dù chọn cách nào: có bắt tay sẵn-sàng (UI không hiển thị trạng thái bịa trước khi lõi lên), có đường báo lỗi chí mạng tách khỏi đường dữ liệu, và thoát app thì lõi được tắt có trật tự (drain việc đang chạy theo core/02) hoặc được giữ sống theo chủ đích đã khai — chọn một và ghi vào đây.

### 2.3. Nghĩa vụ với macOS

| Chủ đề | Quyết định cần ghi |
|---|---|
| App Sandbox của Apple | Bật hay không. **Chú ý**: App Sandbox của OS và sandbox thực thi của model ([core/08, mục 2.5](../core/08-capability-seams.md)) là hai lớp khác nhau — ghi rõ cả hai, đừng gộp. Nếu bật App Sandbox: entitlement truy cập file do người dùng chọn là bắt buộc để harness làm việc với dự án thật |
| Cơ chế giam thực thi trên macOS | Seatbelt (`sandbox-exec`) là lựa chọn tham chiếu cho backend macOS của seam sandbox; mức thực thi phải báo `full/partial` thành thật (core/13) |
| Ký số & notarize | Chứng chỉ Developer ID, hardened runtime, quy trình notarize khi phát hành ngoài App Store |
| Quyền hệ thống | Điều khiển máy tính (computer-use) cần quyền Accessibility/Screen Recording: app phải xin và giải thích; từ chối thì tính năng tắt sạch, không lỗi dây chuyền |
| Deep link / mở-từ-Finder | Scheme URL và loại file app nhận; ý định được chuẩn hoá thành lệnh cho agent, không đi đường tắt |
| Cập nhật | Cơ chế cập nhật và kênh (ổn định/thử nghiệm); cập nhật không được chạm thư mục dữ liệu harness |
| Thư mục dữ liệu | App dùng một thư mục harness riêng của nó (profile riêng do app sở hữu) nhưng dùng chung định dạng dữ liệu với bản CLI nếu có — nâng cấp một phía không phá phía kia |

## 3. Quyết định của lần này

- Công nghệ cửa sổ:
- Kênh native ↔ lõi:
- App Sandbox (Apple) bật/tắt + entitlement cần:
- Backend giam thực thi macOS:
- Ký số/notarize:
- Việc lõi khi đóng cửa sổ / thoát app:
- Lý do một dòng:

## 4. Ảnh hưởng lên core

- Không đổi core. App native chọn một profile (core/12) và một mặt tiếp xúc (chat-va-tools-integration).
- Thứ duy nhất core cần thấy từ phía native là các *ý định đã chuẩn hoá*: mở workspace này, gửi tin nhắn này, trả lời approval kia — qua đúng các giao diện agent/approval/questions của core/03 và core/09.
