# 12 — Composition, profile và cấu hình

> Harness không chạy như một khối duy nhất: nó được **phối** từ các lớp module. File này định nghĩa cách phối đó — nhờ nó, cùng một lõi chạy được nhiều kiểu ứng dụng và mỗi người dùng tự tuỳ biến được mà không fork code.

Đọc trước: [02](02-module-va-plugin.md).

---

## 1. Ba khái niệm phối ghép

| Khái niệm | Là gì | Ví dụ |
|---|---|---|
| **Bundle** | Một gói phân phối: một lớp vá cấu hình (danh sách hàng module + config của chúng) và code mà nó gắn | "gói cơ sở dùng chung", "gói mặt web" |
| **Profile** | Một bản phối có tên, lưu trong thư mục harness của người dùng: xếp chồng các bundle + plugin cài thêm + lớp vá riêng của người dùng | `web`, `headless`, `sdk`, hồ sơ cá nhân của bạn |
| **Preset** | Bản phối *riêng cho một session/agent* bên trong một profile đang chạy | một agent con với bộ tool/persona khác hẳn |

### Xếp lớp

Một profile chạy được tạo bằng cách áp các lớp lên một danh sách rỗng, theo thứ tự:

```
danh sách rỗng
  → các bundle của profile (đúng thứ tự khai báo)
  → lớp vá của profile (file vá riêng trong thư mục profile)
  → lớp vá cấp home (áp cho mọi profile)
  → các overlay --patch trên dòng lệnh lần chạy này
```

- Một lớp vá **địa chỉ một hàng module theo id** và thay **toàn bộ** config của hàng đó (không merge sâu), hoặc chèn hàng mới. "Lớp trên thắng" theo từng hàng.
- Hàng bị vô hiệu hoá (`disabled`) không nạp code của nó chút nào.
- Dãy lớp phải *xem trước được*: có một lệnh dump toàn bộ bản phối hiệu lực (mọi hàng, mọi config đã phân giải) mà không cần boot thật. Mọi hàng trong bản dump đều thay được bằng một lớp vá của bạn.
- **Bundle hỏng thì bỏ qua bundle đó**, báo cáo một lần lúc khởi động, các bundle còn lại giữ nguyên thứ tự. Nhưng lỗi ở *lớp vá của profile/người dùng* thì dừng khởi động — đó là ý định trực tiếp của chủ máy, không được lặng lẽ bỏ.

Một profile nên có một **bundle cơ sở dùng chung** (model adapter, tool cơ bản, persistence, sandbox + chính sách approval, settings, credentials) rồi mỗi kiểu ứng dụng cộng thêm bundle của riêng nó — thay vì mỗi profile lặp lại toàn bộ danh sách.

---

## 2. Các kiểu profile chuẩn (hình dạng, không phải công nghệ)

| Profile | Hình dạng | Ghi chú |
|---|---|---|
| `web` | Server + giao diện chat đầy đủ | Có nạp nóng cấu hình (mục 5) |
| `headless` | Chạy một việc rồi thoát, không server | Chính sách approval thường là `never`; không nạp nóng |
| `sdk` | Server giao thức cho app khác gọi vào | Đường duy nhất để nhúng harness |
| `sdk-minimal` | Một bundle độc lập sở hữu toàn bộ cây module của nó, cố ý **không** dùng bundle cơ sở chung | Ngoại lệ có chủ đích: bề mặt nhỏ nhất có thể |
| `acp`/tự động hoá | Server cho hệ agent ngoài điều khiển | Approval do phía điều khiển trả lời bằng máy |

Nguyên tắc khởi chạy: **chỉ profile có tên mới khởi chạy ứng dụng**. Không có "đường tắt" kiểu mỗi package tự phơi bày một executable riêng để chạy ứng dụng — mọi đường chạy đều qua cùng một launcher chọn profile. (Executable phục vụ build/test/demo nội bộ thì phân loại riêng, không phải đường chạy ứng dụng.)

---

## 3. Cấu hình của plugin

1. Mỗi plugin khai báo một **schema cấu hình có kiểu**; config sai → lỗi lúc nạp với thông báo chỉ rõ trường nào sai. Không có "đọc config bằng niềm tin".
2. **Không hằng số chỉnh được viết cứng trong plugin.** Mọi lựa chọn thay đổi theo triển khai (timeout, giới hạn, đường dẫn, model mặc định…) là trường Config sửa được từ file phối. Hằng số giao thức, đặc tả ngoài và bất biến an ninh thì giữ cố định. Một hằng `DEFAULT_*` hay hook test không tính là "cấu hình được".
3. Mặc định hoá là bước `resolve(request) → spec` tường minh ở nơi sở hữu (file 08, mục 1), không rải `?? default` trong code chạy.
4. Bí mật không nằm trong config: config chỉ mang *tham chiếu* tới bí mật (file 08, mục 2.11).
5. Trong file phối, một số trường cho phép biểu thức động được tính lúc nạp (điều kiện theo môi trường) — nhưng chỉ tại các vị trí được khai báo rõ (config của plugin, cờ disabled); các metadata còn lại giữ nguyên văn. Điều kiện theo môi trường nên dùng overlay/lớp vá, không nhồi logic vào mọi hàng.

**Một catalog cấu hình sinh tự động** (mọi plugin, mọi trường, kiểu và mặc định) phải tồn tại và kiểm chứng được với code — người dùng tra cứu ở đó thay vì đọc source.

---

## 4. Kiểm soát tương thích phiên bản

Trước khi một plugin được import, bản phối kiểm tra yêu cầu phiên bản harness của nó với phiên bản đang chạy:

- Không đạt → hàng đó bị vô hiệu hoá trong *bản sao của launcher* (không sửa file của người dùng), và báo cáo nêu rõ package, phiên bản, yêu cầu không đạt.
- Miễn trừ (exemption) chỉ theo đúng cặp `package@version` ↔ phiên bản runtime, lưu ở file riêng của profile — nâng cấp một trong hai phía là miễn trừ hết hiệu lực. File miễn trừ hỏng thì coi như không có miễn trừ và báo rõ, không chặn khởi động.
- Kiểm tra này là rào tương thích, **không phải sandbox** chống code độc.

---

## 5. Nạp nóng (HMR) và sửa cấu hình lúc chạy

- Một module điều phối nạp nóng (tuỳ chọn theo profile) theo dõi: manifest profile, các file vá của người dùng — và áp lại bản phối khi chúng đổi. Sửa *code* của module (module roots) là opt-in riêng.
- Mọi ghi cấu hình lúc chạy (từ UI settings, từ quản lý plugin) đi qua **một hàng đợi duy nhất** cùng với nạp nóng và một khoá file cấp ứng dụng — không có hai đường ghi đua nhau.
- Nạp lại thất bại: giữ bản đang chạy tốt, báo lỗi rõ ràng; không để hệ thống ở trạng thái nửa mới nửa cũ.
- Profile không bật nạp nóng thì đổi cấu hình = khởi động lại. Đó là lựa chọn hợp lệ cho headless/sdk.

---

## 6. Quản lý plugin lúc chạy

Một dịch vụ quản lý plugin (dùng chung cho CLI và UI): cài/gỡ/nâng cấp gói plugin vào profile, bật/tắt hàng, và báo cáo trạng thái (đã lưu vs đang chạy). Thao tác gói chạy ngoài hàng đợi nạp nóng; sau khi xong, một bản phối kế tiếp được công bố nguyên tử cho các lần nạp lại sau.

Preset theo session (file 01) cũng đi qua cùng cơ chế lớp vá, áp vào bên trong khai báo của preset với cùng thứ tự ưu tiên; một dịch vụ nằm trong preset cần một "realm cô lập" riêng cho mỗi agent nếu nó không được phép dùng chung với bản global.

---

## 7. Webhook và kích hoạt từ bên ngoài

Một dịch vụ runtime webhook giữ các **rule tin cậy** đã đăng ký: sự kiện từ ngoài (qua adapter của từng nguồn) khớp rule → tạo một session mới hoặc đánh thức session chỉ định. Rule không tin cậy (từ dữ liệu người dùng chưa kiểm chứng) không bao giờ được đăng ký.

---

## 8. Tiêu chí nghiệm thu

- [ ] Dump bản phối hiệu lực của một profile: mọi hàng đều thấy được; vá một hàng bằng file của mình → dump phản ánh đúng và lần chạy dùng đúng config đó.
- [ ] Tạo một profile mới từ một template có sẵn, đổi đúng một bundle → chạy được mà profile gốc không đổi.
- [ ] Đưa một config sai kiểu cho một plugin → khởi động báo lỗi nêu đúng plugin và trường, trước khi bất kỳ yêu cầu model nào chạy.
- [ ] (Profile có HMR) Sửa file vá → bản phối mới áp dụng không cần restart; sửa thành bản phối lỗi → bản cũ vẫn chạy và lỗi được báo.
- [ ] Cài một plugin khai báo yêu cầu phiên bản không đạt → nó bị từ chối kèm báo cáo, phần còn lại của profile vẫn lên.
