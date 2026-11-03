# 08 — Capability seams: các khả năng thay thế được

> "Seam" là cách harness giữ mọi khả năng có thể hoán đổi: hệ thống file, chạy lệnh, lưu trữ, tìm web… File này định nghĩa mẫu seam và liệt kê các seam chuẩn cùng hợp đồng logic của chúng. Cài đặt cụ thể từng seam (công nghệ nào, trên OS nào) thuộc `technology/`.

Đọc trước: [02](02-module-va-plugin.md).

---

## 1. Mẫu seam: luôn đủ 3 vai

```
┌─────────────────┐   đăng ký    ┌──────────────────┐
│ Service Provider │ ───────────► │ Service Definition│ ◄── dùng qua giao diện ── Consumer
│ (cài đặt cụ thể) │              │ (giao diện + từ   │                            (thường là
└─────────────────┘              │  vựng + registry) │                             một tool)
                                  └──────────────────┘
```

| Vai | Sở hữu | Ví dụ với seam "shell" |
|---|---|---|
| **Service Definition** | Giao diện, kiểu dữ liệu, khoá dịch vụ, sự kiện của seam | "Thực thi lệnh shell" gồm những thao tác nào |
| **Service Provider** | Một cài đặt cụ thể; có thể nhiều provider cùng lúc hoặc độc quyền tuỳ seam | Chạy trên máy local / trong sandbox / qua SSH |
| **Consumer** | Bên sử dụng qua giao diện, thường là tool cho model hoặc một dịch vụ khác | Tool `bash` |

Quy tắc:

1. **Thiếu một vai là chưa thành seam.** Chỉ có giao diện mà không provider thì là chữ chết; chỉ có cài đặt mà không giao diện thì không thay được.
2. Ba vai tách thành module riêng **khi chúng tiến hoá độc lập**; gộp lại khi chúng là một mối quan tâm duy nhất. Tách/gộp là quyết định đóng gói, không đổi hợp đồng.
3. **Consumer chỉ phụ thuộc Definition, không phụ thuộc Provider.** Đây là quy tắc cho phép "đổi một điểm, đổi cả sản phẩm": trỏ seam filesystem và subprocess sang một môi trường từ xa thì tool đọc file, shell và LSP đi theo sang đó luôn, không fork tool.
4. Definition **không** được thiết kế theo ý một consumer duy nhất. Nếu một phương thức công khai chỉ có đúng một nơi gọi nội bộ, hãy truyền một closure riêng thay vì phình giao diện.
5. Provider đăng ký qua effect và có disposer; gỡ provider thì consumer thấy seam vắng mặt và lỗi fail loud khi dùng, không âm thầm rơi về cài đặt khác.

### Chuẩn hoá yêu cầu (request/spec)

Tại ranh giới của một seam, tách hai bước:

```
Request (người gọi muốn gì, còn thiếu được)
   → resolve(request) -> Spec   ← bước tường minh của bên sở hữu seam:
                                   điền mặc định, validate, chuẩn hoá
   → run(spec)                  ← chỉ chạy spec đã hoàn chỉnh
```

Không giấu mặc định kiểu `?? giá-trị-mặc-định` rải rác trong `run()`. Người đọc phải thấy mọi mặc định ở đúng một chỗ.

---

## 2. Danh mục các seam chuẩn

Mỗi seam dưới đây ghi: khoá dịch vụ gợi ý, hợp đồng logic cốt lõi, và biến thể provider điển hình. Tên khoá chỉ là quy ước đặt tên — giữ ổn định trong cài đặt của bạn.

### 2.1. `fs` — Hệ thống file

- Thao tác: đọc/ghi/sửa file, liệt kê, tìm kiếm — mọi đường dẫn người dùng đưa vào phải được phân giải về một đích của backend trước, không giả định là đường dẫn local.
- Biến đổi file đi qua **sự kiện ý định** (waterfall: ý định ghi / ý định sửa) để một module chính sách gắn quy tắc "đọc trước khi sửa", giới hạn phạm vi… mà tool không cần biết chính sách tồn tại. Bỏ module chính sách thì seam vẫn nguyên vẹn, chỉ mất quy tắc đó.
- Provider: local · trong sandbox · qua kết nối từ xa.

### 2.2. `subprocess` — Tiến trình con

- Gốc của mọi thực thi: sinh tiến trình kèm argv, môi trường, thư mục chạy, stream stdout/stderr, huỷ theo cây tiến trình.
- Mọi seam chạy-thứ-gì-đó (shell, terminal, LSP, subagent ngoài…) đều xây trên seam này. Đổi provider subprocess là đổi "thế giới thực thi" cho tất cả.

### 2.3. `shell` — Thực thi lệnh shell

- Nhận một yêu cầu lệnh, chuẩn hoá thành spec (lệnh, cwd, env, timeout, chế độ chạy nền…), chạy qua `subprocess`.
- Provider: local trực tiếp · bọc qua sandbox · từ xa. Consumer: tool shell cho model, các hook cần chạy lệnh.
- Môi trường shell (biến môi trường được quản lý, hàm trợ giúp…) là một registry riêng đăng ký vào seam, không viết cứng trong provider.

### 2.4. `terminals` — Terminal bền vững

- Phiên terminal tương tác sống qua nhiều lần gọi (PTY hoặc tương đương), có chủ sở hữu theo session, có giới hạn khôi phục màn hình.
- Provider đăng ký backend; consumer là tool terminal cho model và terminal cho người dùng trên UI.

### 2.5. `sandbox` — Giam tiến trình

- Nhận argv của *cùng thế giới thực thi* và một chính sách hiệu ứng file, trả argv đã bọc (hoặc quyết định bỏ qua bọc).
- Chế độ: `read-only` (chỉ đọc + các đích ghi tối thiểu hệ thống cần) · `workspace-write` (ghi trong gốc workspace + các gốc phụ được cấp + vùng tạm của backend) · `danger-full-access` (không giam — consumer tự chạy argv gốc, không gọi seam).
- Mức thực thi là **sự thật phải báo cáo**: `full` (giữ đúng mọi cam kết của chế độ) hoặc `partial` (backend/kernel chỉ giữ được một phần). Consumer cần ranh giới tuyệt đối phải từ chối hoặc hiển thị rõ khi chỉ có `partial` — không được coi như `full`.
- Chính sách được phân giải **trọn gói cho từng lần gọi** (chế độ + gốc workspace + các gốc ghi phụ) trước khi quyết định có bọc hay không. Gốc ghi phụ không tồn tại/không phải thư mục/là symlink → từ chối lần gọi đó.
- Mạng và khả năng nhìn thấy tiến trình khác nằm ngoài từ vựng này, trừ khi cài đặt của bạn khai báo thêm.

### 2.6. `web` — Truy cập web

- Registry các provider tìm kiếm và lấy nội dung (fetch). Consumer: tool web cho model.
- Provider thay nhau theo cấu hình; kết quả về consumer theo một dạng chuẩn, lỗi provider này không kéo sập registry.

### 2.7. `lsp` — Máy chủ ngôn ngữ

- Điều hướng mã nguồn (định nghĩa, tham chiếu…) qua một provider giao tiếp với language server bằng tiến trình con.
- Consumer: tool `lsp` cho model.

### 2.8. `browserUse` / `computerUse` — Trình duyệt và máy tính

- Mỗi seam là một registry **độc quyền theo tên**: một provider đăng ký một tên, tự sở hữu luôn tool và tài nguyên trình duyệt/máy tính của mình cho từng session sống. Dịch vụ chung không định nghĩa API thao tác trình duyệt — vì mỗi provider có năng lực khác nhau.
- Quyền điều khiển máy tính (consent theo ứng dụng, huỷ…) là một seam đi kèm do provider gốc sở hữu.

### 2.9. `mcp` — Công cụ từ máy chủ ngoài (Model Context Protocol hoặc tương đương)

- Không có dịch vụ `mcp` chung. Mỗi kết nối tới một server ngoài là một plugin riêng: tự khám phá tool của server đó và đăng ký vào tool registry chung (tên tool mang tiền tố tên server để không đụng nhau), tự đóng kết nối khi bị gỡ.
- Một dịch vụ tài nguyên dùng chung (đọc resource từ các server đã nối) tồn tại riêng; vắng server nào thì mất resource của server đó, không mất dịch vụ.
- Tool từ MCP đi qua **đúng pipeline** ở file 06 — chính sách, approval, ghi log không có ngoại lệ cho "tool ngoài".

### 2.10. `storage` — Lưu trữ không-phải-session

- Hub lưu trữ key-value/đơn vị mờ cho dữ liệu phụ (cài đặt miền, trạng thái workspace…): nhiều backend đăng ký cạnh nhau theo tên; các "dạng dữ liệu" (domain form) dịch thao tác có kiểu xuống primitive của hub.
- Session log **không** đi qua seam này — nó có seam persistence riêng (file 11).

### 2.11. `credentials` — Bí mật

- Cấu hình chỉ mang **tham chiếu** tới bí mật (ví dụ tên biến môi trường), không mang giá trị. Provider sở hữu giá trị.
- Consumer phân giải tham chiếu **mỗi lần thao tác** (mỗi yêu cầu model một lần) — bí mật được xoay (rotate) là có hiệu lực ngay yêu cầu kế tiếp, không cần khởi động lại.
- Giá trị rỗng = không tồn tại, ở mọi provider.
- Mặt đọc cho UI chỉ trả lời "đã cấu hình chưa, từ lớp nào, ghi được không" — **không có khe nào cho giá trị lọt qua**.

### 2.12. `settings` — Cài đặt người dùng

- Chiếu các trường cấu hình "dễ bay hơi" của các plugin đang nạp thành biểu mẫu có phiên bản (revision); sửa qua một trình soạn cấu hình duy nhất có khoá chống ghi đè lẫn nhau (revision cũ → từ chối).
- UI không bao giờ nhận giá trị bí mật trong biểu mẫu (xem 2.11).

### 2.13. Các seam hỗ trợ khác (định nghĩa ở file chuyên biệt)

| Seam | Vai trò | File sở hữu logic |
|---|---|---|
| `compaction` | Nén ngữ cảnh | 10 |
| `subagents` | Uỷ quyền cho agent con / hệ ngoài | 10 |
| `jobs` | Việc chạy nền | 10 |
| `workflow` | Chạy kịch bản điều phối nhiều agent | 10 |
| `skills` | Registry gói chỉ thị nạp theo yêu cầu | 10 |
| `approval`, `userQuestions` | Hỏi người dùng | 09 |
| `sessionPersistence` | Lưu session log bền vững | 11 |
| `attachments` | Lưu nhị phân bền vững, định danh theo nội dung | 11 |
| `spill` | Tràn kết quả lớn ra lưu trữ phụ | 11 |
| `webhook` | Kích hoạt session từ sự kiện ngoài | 12 |

---

## 3. Sự kiện của seam

Ngoài phương thức dịch vụ, seam có thể phát sự kiện mang tên miền của nó (ví dụ `fs/*`, `tools/*`, `telemetry/*`) để chính sách và adapter gắn vào mà không import vòng lặp. Đây là đường dành cho *chính sách và quan sát*; gọi khả năng trực tiếp thì dùng phương thức (file 02, mục 6).

---

## 4. Tiêu chí nghiệm thu

- [ ] Thay provider `shell` từ local sang bản bọc sandbox bằng cấu hình: tool shell, hook và terminal dùng nó đều đổi hành vi, không sửa dòng code consumer nào.
- [ ] Một seam chỉ có Definition mà cố tình không nạp Provider: nơi dùng báo lỗi rõ ràng khi gọi, hệ thống vẫn khởi động và chẩn đoán được thiếu gì (trừ khi seam đó được khai là bắt buộc lúc nạp — xem file 12).
- [ ] Xoay một bí mật (đổi giá trị ở provider): yêu cầu model kế tiếp dùng giá trị mới, không restart.
- [ ] Nối một MCP server thử nghiệm: tool của nó xuất hiện với tên có tiền tố, đi qua approval như tool thường; ngắt server, tool biến mất sạch.
