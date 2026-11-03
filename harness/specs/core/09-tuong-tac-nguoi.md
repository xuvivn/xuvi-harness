# 09 — Tương tác với người: approval, câu hỏi, lệnh, kế hoạch

> Model không phải chủ thể duy nhất ra quyết định. File này định nghĩa mặt phẳng con người: khi nào phải hỏi, hỏi không được thì sao, và những kênh nào người dùng điều khiển harness **mà không qua model**.

Đọc trước: [03](03-agent-loop.md), [06](06-tools-va-thuc-thi.md).

---

## 1. Approval: "việc này được làm không?"

Một dịch vụ duy nhất trả lời câu hỏi đó cho một hành động cụ thể.

### 1.1. Kết quả đóng, fail-closed

```
ApprovalOutcome = 'allowed-once' | 'rejected' | 'cancelled' | 'unavailable'
```

- Chỉ `allowed-once` mới mở cổng, và chỉ cho đúng hành động đã hỏi.
- `rejected` (người từ chối / chính sách cấm), `cancelled` (yêu cầu bị rút), `unavailable` (không có ai trả lời được, người trả lời lỗi hoặc trả sai dạng) — cả ba đều là **không**.
- Không có kết quả "cho phép luôn từ nay về sau" ở tầng này. Muốn nhớ quyết định thì một module chính sách phía trên tự ghi nhớ và tự trả lời các lần sau.

### 1.2. Chính sách theo session

Mỗi session có chính sách hiệu lực, ghi trong log (sự kiện đổi chính sách — replay dựng lại được):

| Chính sách | Hành vi |
|---|---|
| `ask` (mặc định) | Hỏi chuỗi người trả lời đã đăng ký; không ai trả lời → `unavailable` → bị từ chối |
| `never` | Mọi câu hỏi đều `rejected` ngay, **không phát tới bất kỳ người trả lời nào** — kể cả người trả lời đăng ký sau với ưu tiên cao nhất. Đây là lập trường cho chạy không người giám sát (CI, chạy tự động). |

Chính sách được thực thi **bên trong** dịch vụ approval, trước khi dispatch — không dựa vào thứ tự listener.

### 1.3. Luồng một lần hỏi

```
request(agent, toolName, callId?, reason?, signal?)
  → yêu cầu phải nằm trong một turn đang mở (để cặp kiểm toán có chỗ đứng trong log)
  → ghi approval/asked (log-only, kèm id duy nhất của lần hỏi)
  → áp chính sách: never → rejected
                  ask → chuỗi người trả lời (waterfall):
                          ai sở hữu yêu cầu thì trả kết quả,
                          không thì gọi next(); người đầu tiên trả chiếm quyết định
  → ghi approval/decided
  → trả outcome
```

- Yêu cầu **không mang tham số tool** — người trả lời gắn câu hỏi vào thẻ tool call đã hiển thị qua `callId`. Tránh hai bản sao của cùng dữ kiện lệch nhau.
- Tín hiệu huỷ kích hoạt → kết quả `cancelled` ngay; câu trả lời đến muộn bị bỏ.
- Cặp sự kiện `asked/decided` là log-only: không vào lịch sử model. Điều model thấy là kết quả tool suy ra từ outcome, cộng với bản chụp runtime context phản ánh chính sách hiện tại (file 07).

### 1.4. Ai là "người trả lời"

Bất kỳ mặt tiếp xúc nào cũng đăng ký được người trả lời: UI chat (hỏi người thật), một cầu nối tự động hoá (quyết định bằng máy cho agent của nó), hoặc một module chính sách (tự động cho phép một số mẫu đã biết). Approval không biết và không cần biết đó là người hay máy.

---

## 2. Hỏi người dùng một câu hỏi thật

Khác approval (hỏi có/không cho một hành động), đôi khi model cần *hỏi và nhận câu trả lời tự do/cấu trúc* từ người dùng (qua một tool chuyên dụng).

- Một seam hỏi/đáp: người hỏi đăng một câu hỏi thuộc về một agent; mặt tiếp xúc sở hữu agent đó trả lời; không ai sở hữu → câu hỏi không trả lời được và tool nhận kết quả lỗi rõ ràng, không treo vô hạn.
- Câu hỏi và câu trả lời đi vào lịch sử qua đường ghi log thông thường (đây là dữ kiện model thấy → quy tắc 3.2 ở file 01 áp dụng).

---

## 3. Lệnh của người (human commands)

Kênh điều khiển không qua model: người dùng gõ một lệnh (thường có tiền tố `/`) và harness thực thi trực tiếp.

- Một registry lệnh: `register(command) → disposer`. Lệnh nhận (agent/session hiện tại, tham số) và tác động lên miền của nó.
- Lệnh **không** trở thành tin nhắn model. Kết quả của lệnh là trạng thái UI, trừ khi handler tự ghi một thay đổi bền vững vào miền của nó (ví dụ lệnh đặt mục tiêu ghi sự kiện mục tiêu vào log — và *phần model thấy* của thay đổi đó vẫn phải qua log như thường).
- Mặt phẳng lệnh gồm: khám phá (gợi ý/autocomplete), phân tích cú pháp, dispatch, huỷ, và hiển thị kết quả — do mặt tiếp xúc và các plugin lệnh sở hữu.

Ví dụ họ lệnh điển hình: nén ngữ cảnh, đặt mục tiêu, đổi model, đổi chính sách approval, xem/kiểm soát việc nền.

---

## 4. Chế độ kế hoạch (plan mode)

Một trạng thái cộng tác theo session: agent chỉ được khảo sát và trình một **kế hoạch**; thực thi bị chặn tới khi người dùng duyệt kế hoạch qua một lối ra có kiểm soát (lệnh/tool duyệt).

- Trạng thái kế hoạch là dữ liệu bền vững theo session (dựng lại được từ log), không phải cờ trong bộ nhớ của một agent.
- Việc chặn thực thi phải được thực thi **tại thao tác thực hiện quyết định** (pipeline tool từ chối thao tác ghi khi đang ở chế độ kế hoạch), không chỉ giấu tool khỏi prompt — đường gọi vòng nào cũng không qua mặt được.

---

## 5. Phản hồi của người (feedback)

Người dùng có thể gắn phản hồi lên một tin nhắn của model hoặc cả session. Feedback là sự kiện log-only thuộc về session đó, kèm kênh xuất ra ngoài (telemetry/báo cáo) nếu được cấu hình. Nó không sửa lịch sử đã gửi.

---

## 6. Bất biến của file này

1. **Không có đường nào tới hành động nhạy cảm mà bỏ qua approval khi chính sách yêu cầu hỏi** — kể cả đường gọi trực tiếp không qua tool (xem quy tắc "thực thi quyết định tại thao tác ra quyết định", file 13).
2. **Hỏi mà không có ai trả lời được = từ chối**, ở mọi mặt tiếp xúc. Một harness headless lặng lẽ cho phép mọi thứ là một harness hỏng.
3. Mọi quyết định của người để lại **vết kiểm toán** trong log (hỏi gì, kết quả gì), tách khỏi nội dung model thấy.

---

## 7. Tiêu chí nghiệm thu

- [ ] Đặt chính sách `never`: một tool cần approval bị từ chối tức thì, không UI nào nhận được câu hỏi; đổi lại `ask` giữa phiên có hiệu lực từ tool call kế tiếp.
- [ ] Chạy không UI (headless) với chính sách `ask` và không người trả lời nào đăng ký: tool bị từ chối với lý do rõ ràng, turn vẫn kết thúc sạch.
- [ ] Một lệnh người dùng (ví dụ đổi model) thực thi mà không sinh tin nhắn model nào; hiệu quả của nó thấy được ở step kế tiếp.
- [ ] Ở chế độ kế hoạch, thử ghi file bằng cả đường tool thường và đường gọi trực tiếp: cả hai đều bị chặn.
