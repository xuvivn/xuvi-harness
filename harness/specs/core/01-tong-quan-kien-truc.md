# 01 — Tổng quan kiến trúc

> File này trả lời: harness gồm những khối nào, chúng nói chuyện với nhau ra sao, và những quy tắc nào **không bao giờ được phá**. Mọi file core còn lại chỉ là chi tiết hoá của file này.

---

## 1. Bức tranh một câu

```
Người dùng ──► [Mặt tiếp xúc] ──► Agent ──► Agent Loop ──┬─► Model Gateway ──► Mô hình AI
                       ▲              │        │          ├─► Tool Registry ──► Tools ──► Thế giới ngoài
                       │              │        ▼          └─► Prompt Assembly
                       └──── sự kiện hiển thị ── Session Log (nhật ký sự kiện)
```

- **Mặt tiếp xúc** (mặt tiền): chat UI, CLI một-lần-chạy, API cho app khác nhúng, app desktop… Đây là lựa chọn công nghệ → `technology/`.
- **Agent + Agent Loop**: bộ máy điều khiển cuộc hội thoại (file 03).
- **Session Log**: cuốn sổ ghi mọi sự kiện, là nguồn sự thật (file 04).
- **Model Gateway**: cổng gọi mô hình, trung lập nhà cung cấp (file 05).
- **Tool Registry**: nơi đăng ký và thực thi công cụ có kiểm soát (file 06).
- **Prompt Assembly**: nơi ghép system prompt + danh sách tool cho mỗi lần gọi mô hình (file 07).

---

## 2. Các khối và trách nhiệm

| Khối | Sở hữu | Không được làm |
|---|---|---|
| Module system | Nạp/gỡ module, cấp "context" chứa các dịch vụ, định tuyến sự kiện | Không chứa logic nghiệp vụ |
| Agent registry | Danh sách agent đang sống, tạo/resume agent qua factory đã đăng ký | Không tự cài đặt vòng lặp |
| Agent loop | Vòng lặp turn/step, gọi model, gọi tool, ghi log | Không biết UI, không biết nhà cung cấp model cụ thể |
| Session log | Ghi sự kiện append-only, suy ra lịch sử cho model | Không làm I/O lưu trữ trực tiếp (đó là persistence, file 11) |
| Model gateway | Registry các adapter model, hợp đồng streaming | Không chứa logic hội thoại |
| Tool registry | Đăng ký tool, pipeline thực thi có guard/approval | Không tự quyết định chính sách quyền (đó là module chính sách) |
| Prompt assembly | Ghép các mảnh prompt + tool schema theo thứ tự; đo và báo ngân sách token của lần ghép (file 07, mục 6) | Không gọi model, không tự nạp dữ kiện ngoài ba đường ở file 07, mục 7 |
| Capability seams | Các khả năng thay thế: file, shell, lưu trữ, web… | Xem file 08 |
| Human interaction | Hỏi người dùng, lệnh của người, approval | Xem file 09 |
| Persistence | Làm cho log bền vững, khôi phục sau crash | Không hiểu nội dung hội thoại |

---

## 3. Tám quy tắc bất biến

Đây là "hiến pháp" của harness. Phá một quy tắc là phá kiến trúc, dù code vẫn chạy.

### 3.1. Không có nhân đặc quyền

Mọi khối — kể cả agent loop, model adapter, tool registry — đều là module đăng ký như nhau. Hệ quả:

- Muốn đổi vòng lặp → thay module vòng lặp, không sửa chỗ khác (mọi nơi khác chỉ phụ thuộc vào *giao diện* Agent, không phụ thuộc cài đặt vòng lặp).
- Muốn thêm tính năng → cắm module mới cạnh các module cũ.

### 3.2. Model-visible ⟺ logged

Bất cứ dữ kiện nào mô hình nhìn thấy trong một yêu cầu (tin nhắn, system prompt, kết quả tool, ngữ cảnh chèn thêm) đều phải tái dựng được 100% từ session log.

- Muốn cho mô hình thấy một dữ kiện mới → định nghĩa một **loại sự kiện mới**.
- Không được "lén" gắn thêm dữ kiện lúc gọi model mà không ghi log. Vi phạm quy tắc này khiến replay, fork và kiểm thử đều sai lệch không truy vết được.

### 3.3. Đăng ký là hiệu ứng có thể thu hồi (effect)

Mọi đóng góp vào hệ thống (tool, mảnh prompt, listener, provider) đều đăng ký qua một cơ chế trả về **hàm huỷ (disposer)**. Gỡ module → gọi các disposer → hệ thống trở về đúng trạng thái trước khi nạp. Không đăng ký "treo" kiểu biến toàn cục.

### 3.4. Giao tiếp chéo qua dịch vụ và sự kiện, không import trực tiếp cài đặt

Module tìm nhau qua **context** bằng khoá dịch vụ (ví dụ `tools`, `llm`, `sessions`), và chặn/chính sách hoá qua **sự kiện**. Không module nào import thẳng lớp cài đặt của module khác ở tầng trên nó. Xem file 02.

### 3.5. Một seam có đủ 3 vai

Một khả năng thay thế được (seam) chỉ hoàn chỉnh khi có: **Định nghĩa** giao diện + **Provider** cài đặt + **Consumer** sử dụng. Chỉ có một vai thì chưa phải seam. Đổi provider phải đổi được hành vi toàn hệ thống mà không sửa consumer. Xem file 08.

### 3.6. Lỗi cấu hình phải nổ sớm và rõ (fail loud)

Thiếu provider, tham chiếu sai, config không hợp lệ → báo lỗi ngay lúc nạp hoặc tại điểm sớm nhất giải quyết được. Không lặng lẽ bỏ qua, không âm thầm dùng giá trị mặc định bịa ra. Giá trị mặc định, nếu có, phải là một bước "chuẩn hoá yêu cầu" tường minh ở nơi sở hữu nó (file 12).

### 3.7. Ranh giới tin cậy phải được kiểm chứng

Dữ liệu từ model (tham số tool dạng JSON), từ file đĩa, từ mạng, từ tiến trình con, từ người dùng qua wire — đều là dữ liệu không tin cậy, phải validate tại đúng ranh giới đó. Ngược lại, dữ liệu cùng tiến trình đã được hệ thống kiểu đảm bảo thì không validate thừa. Xem file 13.

### 3.8. Ngữ cảnh là ngân sách hữu hạn

Cửa sổ ngữ cảnh không phải kho chứa không đáy: mỗi token thêm vào đều làm loãng khả năng chú ý của mô hình, và chất lượng nhớ/suy luận suy giảm dần theo độ dài (không có "vách cứng" — chỉ có dốc suy giảm). Hệ quả bắt buộc:

- Mỗi lần ghép phải hướng tới **tập token nhỏ nhất có tín hiệu cao nhất** đủ cho việc hiện tại; "có thể liên quan" chưa phải lý do để nạp.
- Mọi thành phần của lần gọi (system prompt, tool schema, lịch sử, ngữ cảnh chạy, kết quả tool) đều phải **đo được bằng token** và cộng được thành một ngân sách so với cửa sổ của tuyến hiện tại (file 07, mục 6).
- Dữ kiện lớn hoặc ít dùng đi qua **định danh nhẹ + truy hồi đúng lúc bằng tool**, không nạp toàn văn từ đầu (file 07, mục 5).
- Việc dài hơn cửa sổ được xử lý bằng nén, ghi chú ngoài ngữ cảnh hoặc agent con có ngữ cảnh sạch (file 10, mục 6 và mục 1) — không bằng cách phình prompt vô hạn.

Quy tắc này không mâu thuẫn 3.2: dữ kiện chỉ được *chọn* khi cần, nhưng một khi đã cho mô hình thấy thì vẫn phải qua log như thường.

---

## 4. Ba loại sự kiện — đừng nhầm

| Loại | Bền vững? | Dùng khi |
|---|---|---|
| **Session event** | Có — ghi vào log | Sự thật phải sống sót qua reload (tin nhắn, tool call, kết quả…) |
| **Agent event** (sống) | Không | Quan sát/chặn công việc đang bay (trước step, đang stream, sắp kết thúc turn…) |
| **Capability event** (sống) | Không | Gắn chính sách/adaper vào một seam mà không import vòng lặp (trước/sau khi tool chạy, trước khi ghi file…) |

Chọn sai loại là lỗi thiết kế hay gặp nhất: UI cần hiển thị lâu dài → session event; chính sách chặn tool → capability event; đồng bộ giao diện tức thời → agent event.

---

## 5. Luồng một turn (bản rút gọn)

```
turn/start
  nhận đầu vào từ inbox (tin nhắn người dùng / lệnh tiếp tục)
  chụp ảnh lựa chọn model của session; ghép prompt + tool schemas
  → sự kiện sống "trước step": listener được sửa/loại đầu vào
  step/start
    chuẩn bị lời gọi model (gắn adapter; huỷ lúc này thì chưa ghi gì)
    ghi system/message + user/message vào log
    suy ra lịch sử từ log → đóng băng yêu cầu → stream
      chunk* (sự kiện sống cho UI) → assistant/message (ghi log khi hoàn tất)
    tool/call* → pipeline tool (file 06) → tool/result*
  step/end
  còn tool nợ hoặc có đầu vào mới → step tiếp theo
  → sự kiện sống "turn sắp dừng": listener được chèn việc cuối
turn/end (kèm lý do: hoàn thành / bị huỷ / lỗi / chạm trần token…)
```

Định nghĩa chính xác từng bước, kể cả huỷ giữa chừng và thử lại: [03-agent-loop.md](03-agent-loop.md).

---

## 6. Cái gì thuộc core, cái gì không

| Thuộc core (file này và các file core) | Không thuộc core (vào `technology/`) |
|---|---|
| Vòng lặp turn/step, inbox, cơ chế huỷ | Ngôn ngữ lập trình, runtime, framework plugin cụ thể |
| Định dạng logic của session log và cách suy ra lịch sử | Lưu log bằng file JSONL, SQLite hay cloud |
| Hợp đồng adapter model và giao thức streaming logic | Danh sách provider/model cụ thể, cách xác thực, giá token |
| Pipeline thực thi tool và các điểm chặn | Tool cụ thể nào được bật trong chat, tích hợp MCP ra sao |
| Cơ chế approval hỏi/never, fail-closed | UI hỏi approval trông thế nào, phím tắt nào |
| Cơ chế profile/preset xếp lớp | App là web, CLI hay macOS native; đóng gói, ký số |

---

## 7. Tiêu chí nghiệm thu cho file này

Khi xây xong bản đầu tiên, bạn phải trả lời "có" cho cả bốn:

- [ ] Tôi có thể tắt một tool/provider bằng cấu hình mà không sửa code vòng lặp?
- [ ] Tôi có thể thay cài đặt vòng lặp bằng một cài đặt "phát lại bản ghi" (replay) để test mà UI và tool không nhận ra sự khác biệt?
- [ ] Cho tôi một session log bất kỳ, tôi tái dựng được đúng yêu cầu đã gửi cho model ở mọi step?
- [ ] Cho tôi một step bất kỳ, tôi nói được nó đã tiêu bao nhiêu token cho từng loại (prompt, tool schema, lịch sử, kết quả tool) và harness sẽ làm gì khi tổng số tiến gần cửa sổ của tuyến?

Nếu một câu trả lời là "không" → một quy tắc ở mục 3 đã bị phá. Quay lại sửa kiến trúc trước khi thêm tính năng.
